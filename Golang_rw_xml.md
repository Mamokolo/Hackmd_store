# 檔案讀寫 2 - XML to Go - 學習J筆記
再上一篇完成了 Excel 讀寫之後，這次要做的就是 XML 的讀取，沒錯，沒有寫，畢竟是要將 XML 解析下來之後做分析，所以就不去鑽研寫的部分了(能不能寫其實我也不知道 lar)

## 事前準備
由於 Go 是 google 開發的，因此在 XML、json 這種格式讀取上面都有一個很固定的寫法，那就是用 struct 去完美接殺!
這個時候大家就會想，但是寫那個 struct 很麻煩啊!
沒錯，所以就在這邊推薦兩個好用的線上工具
### XML editor
將檔案匯入 editor 可以直接看到 XML 經過解析後的結構，再做資料處理的時候以此對照，可以確保自己對資料的掌握是否正確
[ONLINE XML EDITOR](https://www.tutorialspoint.com/online_xml_editor.htm)
### XML to GO struct converter
完整匯入XML的文字之後可以直接幫忙轉成對應的golang struct
[XML to Go struct](https://www.onlinetool.io/xmltogo/)

## XML to GO
那接下來就開始正題吧
一樣介紹幾個常用的方法，其實基本上就長這樣，也沒辦法再複雜了030
### import
一樣，下載，import，ezez~
這邊多import一個 **io/ioutil**，是做為讀取XML所需要用到的掛載
```go=
import ("encoding/xml"
        "io/ioutil")
```
### 開啟檔案
將指定路徑的檔案開啟，然後用 xmlFile 變數做為代表
```go=
xmlFile, err := os.Open("FilePath")
```
### 關閉檔案
```go=
defer xmlFile.Close()
```
### 讀取檔案
讀取剛剛被我們打開的資料 xmlFile
```go=
readFromXML, _ := ioutil.ReadAll(xmlFile)
```
### 解析檔案
透過 **Unmarshal** 解析 XML 檔案，然後再放入 Catch 這個資料結構裡面
```go=
var Catch struct_catch // 這個struct就是我們轉換之後的那個struct，要記得在外面先定義好
xml.Unmarshal(readFromXML, &Catch)
```
完成上述步驟之後，就成功地把XML檔接出來囉!
之後要做什麼~~壞壞的~~事情就是看大家的需求了~

2020/12/23

###### tags: `Golang` `XML`

