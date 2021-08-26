# 我的第一支專案程式 RFID/Barcode/QRcode R/W for NordicID - 上工日誌

## 前言
從4月中進到這間公司，然後到現在再等畢業證書，也快三個月了，過去三個月實際參與開發了兩個案子，分別是一隻 Android 程式和系統的網頁前端。網頁的部分我覺得沒啥好分享的，就是從頭學寫 *.html .js .css* 那些的，以前沒碰過，所以像個小孩子一樣亂寫(X，如果以後有機會更進一步我再來寫點東西紀錄好了

但是今天主要是想把我自己寫 Android 的一點心得分享出來，還有這次專案開發使用到相關的技術做個紀錄。

## 簡介
這次做的東西是連接 Reader 去做 RFID/Barcode/QRcode 讀取/寫入的手持設備系統，Reader 是芬蘭的廠牌 NordicID HH85 (寫這篇主要的原因也是因為我當初在網路上幾乎找不到這個機器的中文教學，所以我想把基礎開發的心得分享出來)

原廠的 Github 其實有教學，但是很淺，真的很淺，SDK Document 也有種點到為止的感覺(是不是因為真的很少人在開發這類相關的阿...)，雖然你會覺得你好像可以推斷它的定義，但是還是缺乏確定性。很多我不懂的東西都是請主管寄信去原廠問的，所以希望這篇文章能幫助到其他有在開發這玩意兒的人。

礙於公司的條約，完整的內容並沒辦法整理成私人的 Github repo
在這邊就不分享，但是 Nordic 是有自己的 Github 和 DEMO APP 開源碼的，可以上這邊看，我也是從這份開源碼去研究的
[NordicID Github](https://github.com/NordicID/nur_sample_android)

## NordicID HH85
這支芬蘭的機器我自己也不是很懂，但基本上會來看這篇文章的人應該都大概了解這類型手持裝置的基本操作跟原理，我這裡就不太多講。
主要就是這支手持前端有一個 Reader 讀取 RFID/Barcode and Qrcode，然後下方有板機，可以按壓做 R/W 的指令，讀取 Tag 的時間長短、開始和結束都可以做很彈性的設定，後面會講解到。
## NurAPI SDK
專屬於 NordicID 原廠的 SDK，沒有他就沒辦法跟該廠的手持裝置互動，那其實原廠總共提供了 6 個 SDK 給使用者 import，分別是 **NiduLib.jar、TDTLib.jar、NurApiAndroid.arr、NurDeviceUpdate.arr、NurSmartPair.arr**、以及我們提到的 **NurApi.jar**

但是 import 的方式卻不是統一的，首先 NiduLib.jar、TDTLib.jar、NurApi.jar 直接放到 app\libs裡面，並且在 build.gradle 當中的 dependencies 裡面如下宣告:
```java=
dependencies{
                    :
    implementation files('libs/NurApi.jar')
    implementation files('libs/TDTLib.jar')
    implementation files('libs/NiduLib.jar')
                    :
}
```
而 NurApiAndroid.arr、NurDeviceUpdate.arr、NurSmartPair.arr 則是要通過 module的方式引入，如果你也是用 AS 寫 Android 的話那可以照下列步驟:
1. 在 AS 的 Project 列表對著 project資料夾右鍵，點擊 Open Modules Setting

3. 在彈出來的設定視窗找到 "+" 按下去
![](https://i.imgur.com/MXEn2JF.jpg)
3. 在彈出的視窗找到 Import .JAR/.AAR Package
![](https://i.imgur.com/3oLPXol.jpg)
4. 點選之後就照著步驟做，然後就會先把 module 拉近 Project 了 (這時候還沒結束，因為 import 進來的 modules 也要盡到 dependency 才算真的跟專案結合)
5. 可以在 Project Structure 裡面設定，但有時候會拉不到，所以比較保險的做法是在 build.gradle 的 dependencies 直接設定
```java=
dependencies {
                        :
    implementation project(':NurSmartPair')
    implementation project(':NurApiAndroid')
    implementation project(':NurDeviceUpdate')
                        :
}
```
6. 最後右上角 Sync 就可以了

整個開發環境的攝製到這邊基本上就100%了，剩下的就是調用 SDK 裡面的各種 func，基本上 DEMO APP 裡面演示的都滿清楚，我接下來就針對我自己用的功能也就是 Read/Write 去討論一些開發遇到的癥結點

## Related function

在介紹實際用到的 func 之前，先講一下 NurApi 的基本架構
一開始都需要先建構 **NurApi** 物件
```java=
    NurApi nurapi = new NurApi();
```
再來就是 **NurApi** 有他為 trigger 設計的 Lifecycle(?
如果了解 Android lifecycle 的話應該對這種設計不陌生，也就是 *NurApiListener()*
完整的類別可以在 [這裡](https://github.com/NordicID/nur_sample_android/blob/master/2_EventListener.md) 看到
以下我只介紹在讀取寫入 RFID/Barcode or QRcode 基本會使用到的
```java=
private NurApiListener mNurApiListener = new NurApiListener()
{
                        :
    @Override 
    public inventoryStreamEvent(NurEventInventory event){ 偵測 RFID 的連結和 stream 內容，Document 原文【Tag data from inventory stream.】 }
    @Override
    public void IOChangeEvent(NurEventIOChange event) { 偵測 I/O 的狀態是否切換，Document 原文【IO Change.】 }
    @Override
    public void disconnectedEvent() { 偵測 Reader 是否斷開連結，Document 原文【Transport disconnected.】 }
                        :
}
```
我也不知道為啥 DEMO APP 基本上只用了這三個部分，像是其中還有 **clientDisconnectedEvent()** 這種更客製化的 event 可以偵測，但你只要知道基本上讀寫功能以上三個已經夠用了

再來介紹重要 func 的部分
首先是 **HandleIOEnevtRFID()** or **HandleIOEventBarcode()**
命名可以自訂，在 DEMO APP 應該只會看到 **HandleIOEvent()**
主要是講手持的操控如何跟 Reader 互動的運行方式，這邊我用 RFID 的部份去做講解，其中 **event.source == xxx** 是以數字代表不同的手持操作，詳細可看 Github，開源上面很多註解，然後我加了一些

```java=
private void HandleIOEvent_RFID(NurEventIOChange event){
    try{
        if (event.source == 100) {
            //Trigger down.(這裡的 Trigger 就是指他手持上面的板機沒有問題
            if(mSingleTagDoTask) return;
            if(event.direction == 1)//1代表還在按 Trigger
                StartInventoryStream(); //這是開始 RFID data stream 的函式，DEMO APP 直接複製+微調就好
            else if (event.direction == 0)//0代表放開
                StopInventoryStream();//這是停止 RFID data stream 的函式，DEMO APP 直接複製+微調就好
        }
        else if(event.source == 101) {
            //Power button pressed or released.
            Log.i(TAG,"Button Pressed/Released");
        }
        else if(event.source == 102) {
            //Unpair button pressed or released.
            if(event.direction == 1) {
                //Unpair button pressed. if SingleScan already running so nothing to do.
                Log.i(TAG,"event.source == 102 & event.direction == 1");
            }
        }
        else if(event.source == ACC_SENSOR_SOURCE.ToFSensor.getNumVal()) {
            //ToF sensor of EXA21 has triggered GPIO event
            // Direction goes 0->1 when sensors reads less than Range Lo filter (unit: mm)
            // Direction goes 1->0 when sensors reads more than Range Hi filter (unit: mm)
            if(event.direction == 1) {
                //There are something front of EXA21 ToF sensor, let's start inventory
                StartInventoryStream();
            }
            else {
                //Nothing seen at front of ToF sensor. It's time to stop inventory.
                StopInventoryStream();
            }
        }
    }catch{
    
    }
}
```


而其中的流程大約是這樣
1. 必須先做 Reader 的連接
2. 按下手持的按鈕進行 RFID or Barcode/QRcode 讀取
3. 透過 **HandleIOEvent_XXX()** 來建立手持指令與 func 的對應關係
4. 資料接收(從 Demo App 找以下 code reuse):
    (1). RFID: 透過 **Start/StopInventoryStream()** 來控制資料接收
     ```java=
    private void StartInventoryStream(){...}
    private void StopInventoryStream(){...}
    ```
    (2). Barcode/QRcode: 透過 **AccBarcodeResultListener** 物件直接取得 Tag 資料
    ```java=
    private final AccBarcodeResultListener mBarcodeResult = new AccBarcodeResultListener(){...}
    ```
5. 資料處理(從 Demo App 找以下 code reuse)
    (1). RFID: 透過 **NurApiListenr()** 當中的 **inventoryStreamEvent()** 這個 trigger point，在當下的時機透過 **NurApi()** 物件的 **getStorage()** 方法取得儲存當下讀取到的所有 RFID tag 的資料結構
    ```java=
    @Override
    public void inventoryStreamEvent(NurEventInventory event) {
        NurTagStorage tagStorage = nurApiVerify.getStorage(); //Storage contains all tags found
        NurTag tag = tagStorage.get(0);//只取第一個tag(這裡可以開始做變化)
    }
    ```
    
取得 RFID or Barcode/QRcode 內含的資料之後，就可以直接開始進行字串或是其他儲存結構的處理，其中 NordicID 對於 RFID tag 的讀寫也有包含在他們的 SDK 當中，用法都是固定的，Demo App 也有很完整的 code 可以 reuse。

由於我只打算對於 SDK 內我當初踩過的坑做一些介紹，所以我就不去詳述 Demo App 內 function 的流程，基本上就是抄就對了。

唯一需要注意的就是，在進行 R/W 功能的撰寫之前，還是先去了解 RFID tag 資料讀寫的底層方法，以及各標籤之間對於儲存空間規畫的差異，在使用 **writeTag()** 等寫入的 func 時會比較得心應手。

## Android 介面設計
## 後記
其實滿好笑的，我進公司的寫的第一支 Android 程式不是這支程式，而是長得跟這隻程式87%像的一支 Demo APP，因為客戶換了他們想用的機台還有他們的流程規劃也更動了，所以這支 HH85 的程式就變成我的第一支專案程式了。

但是我原本的程式應該會變成通用的 Demo APP，畢竟這種程式寫好一支後，稍做更動就可以流傳在各個機台當中，我之前的努力倒也是沒有白費，之後的目標就是將程式在拆分得更像 Depends on func的模式，將物件導向的功力發會的更加透徹了。

最後我想說，我寫出來的東西原則上，在運行的部分，都不會跟 DEMO APP 差異太多，使用者體驗，也就是在設計流程上面，我想才是專案工程師發揮價值的地方。今天幫客戶做出一支程式，怎麼樣才能最符合他們的使用情景，讓他覺得:「你家的產品好好用喔!」「感覺你們做出來的東西質感就是不一樣~」才是最好的 feedback。

絕大部分的程式碼都是建立在 reuse 上面，但是能夠 reuse 人家的東西還能做出獨創性，做出最 unique 而且讓人喜歡的組合，是我想繼續精進的目標。

2021/07/05

###### tags: `Android` `RFID` `BarCode/QRcode` `NordicID` `Reuse Work` `上工日誌`

