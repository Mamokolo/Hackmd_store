# 檔案讀寫 3 - SQLite to Go - 學習J筆記

最後講一下這次建立 DB 有用到的 SQLite
因為我自己也沒有接觸過 SQL，完全、完全沒有碰過
所以其實關於 database 的知識也是再開始寫這個專案才慢慢摸出來的

那在 go 上面的文獻其實不多，我自己主要是參照這兩個網站去學
[GO-web編程實戰-go with sqlite的篇章](https://learnku.com/docs/build-web-application-with-golang/053-uses-the-sqlite-database/3183)
[Go實戰--go語言操作sqlite資料庫(The way to go)](https://www.mdeditor.tw/pl/2Lu6/zh-tw)
看別人寫的基礎架構，然後再搭配 SQL 的基本指令去嘗試說，每一次進去出來的資料長什麼樣子，我該怎麼接資料比較好
[TWCODE01.COM - SQLite教學](https://www.twcode01.com/sqlite/sqlite-commands.html)


## SQLite to GO
那接下來說一下基本的內容
### import
我使用的 **mattn/go-sqlite3** 是 go 有支援的三種 sqlite 驅動中唯一有支援 database/sql 的，所以大部分的人都選用他，我這個小白當然也用他:)
```go=
import(_ "github.com/mattn/go-sqlite3")
```
### 開啟DB
一樣的，我們必須要先開啟 Database 才能夠做事
那開啟 db 這個部分呢，都是預設如果路徑上沒有，就會自己開一個
```go=
db, err := sql.Open("sqlite3", DBpath)
```
### 關閉DB
那在這邊使用 **defer**，讓他在我們結束程式運行後關掉 Database
```go=
defer db.Close()
```
### go-sqlite3原理
在golang裡面呢，是利用 **go-sqlite3** 的函式，將 SQLite 指令用雙引號或反引號框起來，然後將這個字串丟入函式，讓 **go-sqlite3** 的 driver 幫我們跑 SQLite 指令
下面看例子應該能更加理解
### 建立table
下方程式碼當中，creatTable := 為一個用反引號框住的 string型態
這一長串 string 就是SQLite指令
那 SQLite 指令的內涵我就不多講
> P.S 這邊用反引號框住是因為在裡面有換行，如果用雙引號的話SQLite指令會吃到換行符號，那就可能會有 *syntax error*。
> 不喜歡反引號的人可以用雙引號然後打一長條不換行

再來就是下面的 *db.Exec()*
這行就是指db這個物件去執行這個SQLite指令

```go=
creatTable := `
    CREATE TABLE IF NOT EXISTS tebleOne(
    "id" INTEGER PRIMARY KEY AUTOINCREMENT,//整數，且設定為自動增加的主key
    "PARAONE"   VARCHAR(64) , //最多可以放64位元的文字
    "PARATWO"   VARCHAR(255) ,//最多可以放255位元的文字
    "PARATHREE" DOUBLE,       //浮點數
    );`
db.Exec(creatTable)
//欄位名稱一定要用雙引號框起來
//後面接的是欄位內的型別，一般來說有文字、數字、浮點數，根據使用的SQL不同需要注意不同型別怎麼選用
```
所以這個寫法也是可以的
```go=
db.Exec(`CREATE TABLE IF NOT EXISTS tebleOne("id" INTEGER PRIMARY KEY AUTOINCREMENT,"PARAONE" VARCHAR(64) ,"PARATWO" VARCHAR(255) ,"PARATHREE" DOUBLE,);`)
```
table 就相當於 excel 檔案的 sheet 工作表
就是存放資料的一大張表格
不同的是，DB 會要求使用者在建立 table 的時候就先規定好每個欄位要叫做甚麼名字，然後資料查詢的主 key 是哪一個欄位，一張 table 的主 key 也不一定只有一個，有興趣的朋友可以去搜尋"資料庫管理"、"資料正規化分析"這類型的文獻，相信會更有幫助

### 增加數據
表格建立好了之後，就可以把資料丟進DB了
以一個有三個資料欄位的tableOne為例
先 **perpare()** 一段 **INSERT** 的 SQLite 指令，指定他要將資料傳進 tableOne 的 *num,dustrict,population* 這三個欄位
然後再 *stmt.Exec* 把我們已經整理好的資料結構 Pop 裡面的資料丟進去
在這邊有用 for 迴圈一層一層丟，因為每一次 **INSERT** 指令下去都是接下來空的欄位加入數據的
```go=
for i := 1; i < len(Pop); i++ {
    stmt, err := db.Prepare("INSERT INTO tableOne(PARAONE, PARATWO, PARATHREE) values(?,?,?)")
    checkErr(err)
    _, err = stmt.Exec(Pop[i].Index, Pop[i].Dustrit, Pop[i].Population)
    checkErr(err)
}
```
在這邊要注意，SQLite 指令中的順序就是資料進入的順序
完成之後，就可以開啟 DB 檔案確認資料有沒有成功寫入了
接下來的用法在結構上都是依樣畫葫蘆，就是修改 SQLite 指令還有交付的參數
### 刪除數據
``` go=
stmt, err = db.Prepare("`DELETE` FROM tableOne WHERE id=?")
    checkErr(err)

    res, err = stmt.Exec(id)
    checkErr(err)

    affect, err = res.RowsAffected()
    checkErr(err)

    fmt.Println(affect)

```

### 更新數據
```go=
stmt, err = db.Prepare(`UPDATE tableOne SET PARAONE=? where id=?`)
    checkErr(err)

    res, err = stmt.Exec("CHANGED", id)//將指定id欄位的PARAONE換成文字CHANGED
    checkErr(err)
    
    affect, err := res.RowsAffected()
    checkErr(err)

    fmt.Println(affect)//印出被影響的那一欄位
```
### 查詢數據
查詢的部分，是使用 scan 去掃描傳出來的每一個欄位
用對應的型別去接資料
```go=
rows, err := db.Query("SELECT * FROM tableOne")
    checkErr(err)

    for rows.Next() {
        var id int
        var paraone string
        var paratwo string
        var parathree float64
        err = rows.Scan(&id, &paraone, &paratwo, &parathree)
        checkErr(err)
        fmt.Println(id)
        fmt.Println(paraone)
        fmt.Println(paratwo)
        fmt.Println(parathree)
    }
```


## 結語

在最後讀取的部分還有使用到 **JOIN、DISTINCT、GROUP、COUNT** 等 SQLite 的語法
如果之後我有更認真的學習 SQL 指令，再記錄一下我學習的成果。
才完成了 table 的 merge 還有指定資料的查找、計數等等
最後再將查找的資料透過設定好的 struct 接下來，在導入 excel 檔案，完成這次的題目
從接到題目到完成大約花了一周的時間，但是同時還有其他的事情在進行，學習加上程式的撰寫修改，前前後後大約花了30個小時有吧。第一次完整的紀錄我的學習歷程與成果，還是很高興的，給未來的我，希望你能夠繼續努力下去:)

2021/12/23

###### tags: `Golang` `SQLite` `Database`