# 檔案讀寫 1 - Excel to go - 學習J筆記

最近開始決定要留存自己實作上的經驗合作集。

這次面試的過程中收到了一個不限時的題目，要讀寫 Excel、XML 檔案，然後跟 DB 做連結，不限定語言和第三方的lib

本文使用 Golang 作為主體，涉及到的技術就是 Excel 的讀寫，XML的讀取，SQLite 指令在 Golang 的應用

## Excel to Go
首先介紹到的就是如何使用 golang 讀寫 excel


### import
首先下載 **excelize**，然後 import 進專案內，ezez:)
p.s 網上有些 excelize 的資料偏舊，現在 excelize 的作者已經把他歸納到這個路徑了  -2020.12還是有效
```go=
import("github.com/360EntSecGroup-Skylar/excelize")
```

### 讀取檔案
其中的 **xlsx** 變數就是之後代表這個 excel 檔案的變數
```go=
xlsx, err := excelize.OpenFile("FilePath")//
```
### 讀取工作表的值
首先用 **rows** 接下 **SHEET_NAME** 這個工作表的值=>會直接接下一個二維陣列
```go=
rows := xlsx.GetRows("SHEET_NAME")
```
通常讀取rows我們可以用兩個 for，但是因為我是寫 struct 去接的，所以用一個 for 讀就可以了
個人認為因為 struct 接下來的資料比較好維護，不想用單純的二維陣列去接
```go=
var catch struct
var Catch []struct
for _, row := range rows {
    catch.elements1 = row[0]
    catch.elements2 = row[1]
    catch.elements3 = row[2]
    catch.elements4 = row[3]
    catch.elements5 = row[4]
    catch.elements6 = row[5]
    catch.elements7 = row[6]
    catch.elements8 = row[7]
    catch.elements9 = row[8]
    catch.elements10 = row[9]
    Catch = append(Catch, catch)
}
```
這邊補充一下，struct slice 的接法，通常是用 struct 接完資料之後，再 append 進 struct slice。
如果 struct slice 沒有 init 的值，又想直接寫進 struct slice，會因為 nullpointer 的問題指不到第一個而錯誤喔。
雖然 slice 是很方便的動態空間分布，但是也有他的限制所在。
### 新增一個工作表
新增了一個名為 **SHEET_NAME** 的工作表
**index** 的話就是代表這個工作表的變數
```go=
index := xlsx.NewSheet("SHEET_NAME")
```
### 將SHEET_NAME工作表設為檔案開啟後預設的那個工作表
我沒用到，只是覺得很好玩
```go=
xlsx.SetActiveSheet("SHEET_NAME")
```
### 將資料寫入Excel
**excelize** 是將資料寫進儲存格
在這邊我們將"Hello world." 這段文字寫進 **xlsx** 開啟的那個檔案的 **Sheet2** 的 **A2** 儲存格
```go=
xlsx.SetCellValue("Sheet2", "A2", "Hello world.")
```
那你一定會說，這樣每一次寫進去都要打這麼長一串，很麻煩ㄟ
是的，由於 *SetCellValue()* 這個函式規定你工作表、儲存格這兩個值一定要提供 string 型態的文字，因此這個部分的麻煩不可少
但是我們可以用 golang 的 *fmt.Sprintf()* 解決


```go=
var sheet_name string = "SHEET_NAME" //這個步驟可做可不做，做了也只是一個概念
for i := 0; i<len(something); i++ {
    xlsx.SetCellValue("2", fmt.Sprintf("A%d", i), dustrict)
    //運用Sprintf將欄位的英文字跟i結合，就可以順利地用迴圈寫資料囉
    xlsx.SetCellValue("2", fmt.Sprintf("B%d", i), numOfGasstation)
	}
```
### 存檔
就是將檔案存到指定的路徑，再一次 ezez~
```go=
err = xlsx.SaveAs(AnsPath)
	checkErr(err)
```
### 報錯
個人的習慣是把 err 的錯誤寫一個函式出來回報
這樣有時候想再某個節點做 fatal 的時候可以用有 fatal 的 check 函式，比較直觀
```go=
func checkErr(err error) {
	if err != nil {
		panic(err)
	}
}
```

2020/12/23

###### tags: `Golang` `excelize`