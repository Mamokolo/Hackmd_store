# 用 WinApi 進行視窗控制 2 - 學習J筆記

接續上一篇，今天沒什麼事情就打算先來完成這篇了
上一篇先介紹了 **WinApi** 裡面部分 func 的用法跟實作面的細節
這一篇就會直接介紹我最後做出的兩套 Solution，以 **Google** 還有 **Facebook** 作為開啟網頁的範例，並且使用兩個螢幕，解析度分別為 **1366x768**、**1280x1024**

## Solution & Rundown (1)

最後這次一共寫了 4 支程式，然後各自編譯出四個 .exe執行檔
因為在一支程式內使用操控 UI 的 WinApi func 會造成多線程搶來搶去的問題，所以導致我無法按照我想要的順序去進行我想要的動作
EX: 開設視窗的同時他可能就已經 call 完 FindWindow()、ShowWindow() 等函式
所以我便將整套流程規劃為四個步驟然後使用批次執行檔去跑四個 .exe執行檔

### Step one - findWindow.cpp

首先第一個執行檔，先透過網頁名稱找我們想要的視窗，如果 **FindWindow()** 返回 **NULL**，則開啟指定瀏覽器並且導向指定網址
```C++=
#include <Windows.h>
#include <stdio.h>
#include <string.h>
#include <iostream>
using namespace std;

int main(int argc, char* argv[])
{
    if(FindWindow(NULL ,"Google - Google Chrome") == NULL){
        ShellExecute(0,0,"chrome.exe","www.google.com  --new-window",0,SW_SHOW);
    }
	 
    if(FindWindow(NULL,"Facebook - Google Chrome") == NULL){
        ShellExecute(0,0,"chrome.exe","www.facebook.com  --new-window",0,SW_SHOW);
    }	
}
```
### Step two - restoreWindow.cpp
第二個執行檔，透過 **FindWindow()** 找到剛才開啟的或是已經存在的視窗之後，用 **ShowWindow()** 將視窗恢復成「還原」狀態
之前說過，必須恢復成還原狀態是因為視窗在「最大化」的狀態無法正常的移動到跨螢幕的部分
```C++=
#include <Windows.h>
#include <stdio.h>
#include <string.h>
#include <iostream>
using namespace std;

int main(int argc, char* argv[])
{
    HWND hq=FindWindow(NULL ,"Google - Google Chrome");    
    HWND hq2=FindWindow(NULL ,"Facebook - Google Chrome");    

    ShowWindow(hq,SW_RESTORE);
    ShowWindow(hq2,SW_RESTORE);
}
```
### Step three - moveWindow.cpp
第三個執行檔，用 **FindWindow()** 找到我們想要的視窗後，透過 **MoveWindow()** 將視窗移動到指定的位置
這邊我想要 **Google - Google Chrome** 視窗留在螢幕1，所以指定他的 **X** 座標在 **0**，然後我想要 **Facebook - Google Chrome** 視窗移動到螢幕2，所以我將 **X** 座標設在 **1380**，確保視窗一定會在第二個螢幕內
```C++=
#include <Windows.h>
#include <stdio.h>
#include <string.h>
#include <iostream>
using namespace std;

int main(int argc, char* argv[])
{
    HWND hq=FindWindow(NULL ,"Google - Google Chrome");    
    HWND hq2=FindWindow(NULL ,"Facebook - Google Chrome"); 

    MoveWindow(hq,0,0,800,500,false);
    MoveWindow(hq2,1380,0,800,500,false);
}
```
### Step four - showWindow.cpp
第四個執行檔，基本上就是第二個執行檔的反向操作
透過 **FindWindow()** 找到剛才開啟的或是已經存在的視窗之後，用 **ShowWindow()** 將視窗轉成「最大化」狀態
```C++=
#include <Windows.h>
#include <stdio.h>
#include <string.h>
#include <iostream>
using namespace std;

int main(int argc, char* argv[])
{
    HWND hq=FindWindow(NULL ,"Google - Google Chrome");    
    HWND hq2=FindWindow(NULL ,"Facebook - Google Chrome"); 
    
    ShowWindow(hq,3);
    ShowWindow(hq2,3);
    /** 這樣寫也可以
    ShowWindow(hq,SW_MAXIMIZE); 
    ShowWindow(hq2,SW_MAXIMIZE);
    **/
}
```
### Step five - rundown.bat
但是連續執行四個執行檔依舊會因為 WinApi 視窗的操作使用多線程而造成無法依照順序執行的問題，為了解決他，我們便需要使用批次執行檔，並且在各執行黨之間保留一定的 timeout，完成 .bat 檔之後就可以嘗試執行，成功後就可以將 .bat 檔排入工作排程，然後讓電腦在啟動、登入或是指定的時間點自動進行視窗排序囉~

我自己最後是嘗試在各個操作之間間隔三秒是這個專案能夠穩定運行的安全間隔，但是會因為開瀏覽器的時間、網路速度而產生差異
如果視窗開得很慢那麼瀏覽器視窗的設定就會因而變慢，導致無法成功使用 FindWindow() 找到控制碼而操作失敗，務必注意

以下是批次執行檔內容 & 說明
```C++=
//如果 .bat 檔沒有跟四支程式放在一起的話，那就必須要添加絕對路徑在執行檔名稱前，或是先用 cd ~ 指令進入執行檔所在資料夾

findWindow.exe

timeout /t 3 //cmd 的 timeout 指令，數字代表要暫停的時間
restoreWindow.exe

timeout /t 3
moveWindow.exe

timeout /t 3
showWindow.exe

```


## Solution & Rundown (2)

寫給客戶的東西，就是比較呆的那種，有做過廠務的案子大概都知道，不能有太多繁瑣的操作，像這次的輔助程式就是全自動的設計
但是自己難免會想弄點東西來玩，所以我就合併了執行檔們，添加了一些自定義的設計，讓使用者可以決定自己想開的視窗是導向哪裡的，並且可以自定義擺放的視窗位置，都是一些很簡單的邏輯，但是在這裡提供出來

簡單來說在最外層包了一個 while 迴圈，用來判斷是否還需要做下一次的全體設定，中間為了解決多線程問題，就利用 **cin>>** 作為斷點，必須輸入過指令才能進到下一步，不能單純以 **enter** 鍵動作
其中必須注意的是，因為將我們輸入的「視窗名稱」以及「網址」跟參數做一個合併，必須要將其存為 LPCSTR 型別的參數字串
**cin** 輸入的內容先存入 **string** 陣列，再將其合併參數內容並存入 **LPCSTR** 陣列，以下是 **string** 轉 **LPCSTR** 在 C++ 中實作的方法:
```C++=
string = "ABC";
LPCSTR lpcstr = s.c_str();
```

以下程式碼就直接放出來給大家參考，還可以在上面做很多變化
大致流程就是:
1. 輸入「視窗名稱」
2. 如果還要繼續輸入就 y (那就重複 1&2
    如果沒有的話就輸入 n 
3. 找尋使用者輸入的視窗是否存在
    是 -> 沒事
    否 -> 請求使用者輸入 URL 並開啟視窗導向指定網址
4. 斷點 請求使用者輸入 5 -> enter
5. 將視窗強勢設為「還原」狀態
6. 使用者需要依據視窗名稱配置視窗位置
    這裡使用數字來代表螢幕編號，內部預設的螢幕解析度要再調整程你使用的螢幕解析度，才能定位正確
7. 將視窗分配到指定螢幕畫面中，設定其大小為 800x500
8. 斷點 請求使用者輸入 5 -> enter
9. 將視窗強制設為「最大化」狀態
10. 詢問使用者是否需要再進行一次操作(可以進行第二次操作並且移動

以下是完整的程式碼
```C++=
#include <Windows.h>
#include <stdio.h>
#include <string.h>
#include <iostream>
using namespace std;

int main(int argc, char* argv[])
{
    string firstCommand = "y";
    while(firstCommand != "n"){
		
        string website[5] = {"","","","",""};
        string tempURL = "";
        string submitFlag;
        int inputCount=0;
		
        while(true){
            //取得網頁名稱 + 設定LPCSTR字串內容 
            cout<<"Enter Website name of first window: ";
            cin>>website[inputCount];
            website[inputCount] = website[inputCount] + " - Google Chrome";

            if(inputCount==4){// 設定最多只能輸入五次
                inputCount +=1; 
                break;
            }

            cout<<"Keep submit?(y/n) ";
            cin>>submitFlag;
            if(submitFlag=="n"){
                inputCount += 1;
                break;
            }
            else if(submitFlag=="y"){
                inputCount += 1;
                continue;
            }
            else{
                //輸入錯誤要跳出來重新輸入 ，只認 y/n 
            }
        }
		
        LPCSTR lpcstrWebsite[5];
        for(int i=0;i<inputCount;i++){
            lpcstrWebsite[i]  = website[i].c_str();
            if(FindWindow(NULL ,lpcstrWebsite[i]) == NULL){
                cout << "指定視窗 "<< website[i] <<" 不存在，請輸入網址: ";
                cin >> tempURL;
                tempURL = tempURL + " --new-window";
                ShellExecute(0,0,"chrome.exe",tempURL.c_str(),0,SW_SHOW);
            }
        }
		
        int command;
        cout<<"Start restet windows...please pressed 5 & enter"<<endl;
        cin>>command;

        HWND hq[5] = {0,0,0,0,0};
        //HWND hq1,hq2,hq3,hq4,hq5;

        for(int i=0;i<inputCount;i++){
            hq[i] = FindWindow(NULL,lpcstrWebsite[i]);
        }
        for(int i=0;i<inputCount;i++){
            ShowWindow(hq[i],SW_RESTORE);
        }

        int place[5] = {0,0,0,0,0};
        cout<<"請輸入螢幕排序:"<<endl;

        for(int i=0;i<inputCount;i++){
            cout<<"視窗 "<<website[i]<<" 配置到 螢幕:";
            cin>> place[i];
        }

        for(int i=0;i<inputCount;i++){
            MoveWindow(hq[i],1921*(place[i]-1),0,800,500,false);
        }

        cout<<"確認放大視窗 請輸入 5 並按 Enter"<<endl;
        cin>>command; 

        for(int i=0;i<inputCount;i++){
            ShowWindow(hq[i],3);
        }

        cout<<"Start Setting? (y/n)  ";
        cin>>firstCommand;
    }
}
```

當然其中還有很多邏輯上的缺陷，像是因為是全部一起進行移動、放大等操作，因此預設的 **HWND[]**、**int[]** 都是固定的大小，我是以五個螢幕去設計這套流程的，所以使用者至多只能輸入五個視窗名稱，針對五個視窗進行操作。

也有人會想為什麼不把移動式窗的指令做成輸入一次就移動，因為移動視窗的步驟會搶走 cmd 的 focus，這樣還要切回來設定，所以就一次設定完一次跑比較好，只要切回來一次

但這都是可以修改的，只是C++做動態規劃比較麻煩，這邊就不繼續往下寫，但實做的部分大概就是這樣去做，在這邊提供方法給大家運用
## 後記

這次的 Solution 原先我也不懂，感謝FB社團的版友給的很多建議。同時也是因為做專案吧，才會需要去學習這麼多零碎的東西，像是上周在寫 java Spring boot 的 API 專案，Golang API Server 也有嘗試，剛來公司的時候還寫了完全沒碰過的網頁三本柱，現在基本操作也是得心應手 反倒是 Android App 才寫了一支半...
出社會就是這樣吧，短短三個月的累積我認為就超越大學四年的實作累積了，但是當然沒有大學的理論基礎，還有對程式的想像、現實的建構，我現在也不可能學這麼多這麼快，只希望能持續進步，共勉之囉~

2021/07/15

###### tags: `C++` `WinApi` `MIS``上工日誌`