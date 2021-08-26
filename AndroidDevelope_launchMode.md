# Android Develope 3 - Activity Launch Mode - 學習J筆記
今天想來寫點基礎的東西，同時也給我一點動力回去看書，加深印象
大家在創建 Activity 的時候，其實有些設定是可以影響到 Activity 的實作機制的，今天要來說的就是 **Activity Launch Mode**，也就是*啟動模式*。通常來說 Activity 在建立時可以被分為四種啟動模式：**Stand、SingleTop、SingleTask、SingleInstance**
而不同的啟動模式會影響我們的 Activity 在後台實作影響堆疊(stack)的方式

## 範例
在 AndroidManifest.xml 裡面通常可以看到如下的宣告，非常簡單
```xml=
<activity android:name=".LoginActivity">
</activity>
```
那在宣告 Activity 給 Application 的時候，我們就可以添加 Launch Mode 給 Activity (當然只會設定一種，我這邊就把其他的註解起來)
```xml=
<activity android:name=".LoginActivity"
    android:launchMode="standard">
    <!--android:launchMode="singleTop"-->
    <!--android:launchMode="singleTask"-->
    <!--android:launchMode="singleInstance"-->
</activity>
```

## 介紹
接下來就來分別介紹四種啟動模式

### standard
顧名思義就是標準模式，一般來說我們不設定的話，這就會是 Activity 預設的啟動模式。今天有 Activity A,B,C，A 設定為 standard 模式，每呼叫一次 A 的時候，就會新增一個 A 實例，也就是說我如果連續生成五個 A，那 BackStack 內就會有五個 A 疊再一起
適用場景：不會重複呼叫同一個 Activity 的時候(可以完善的利用 BackStack 來記錄足跡)

### singleTop
singleTop 模式的特色是，如果生成 A 實例之前，BackStack 頂部剛好是 A 的話，那就會沿用已存在的 Activity，但是如果是一個交錯動作，A->B-A 的話，那麼 BackStack 一樣會堆疊為 A->B->A
適用場景：單一頁面完全刷新
### singleTask
singleTask 的模式就有點偏激，如果今天我在生成 A 的時候，BackStack 裡面存在 A 的話，那就會先清空整個 BackStack 然後將 A 生成。這種做法主要是在特化今天由 A 為進入點的流程，如果重新呼叫 A 那必然後面生成的所有邏輯都要清除，重新從 A 開始整套流程。故名曰  singleTask
適用場景：遊戲關卡流程，從關卡 A - C，無法跳關的設計。

### singleInstance
singleInstance，他擁有 singleTask 的所有特性，但是極為不同的是，單一 Activity 會綁定一個單獨的 backStack。什麼意思呢?在實際操作下的結果就是，我們從 A 切換到 B 之後，就無法返回 A 了，因為它本身就被視為是相異且唯一的 Task
再用更多的 Activity 來講解，同樣的三個 Actvity A,B,C，B 為 singleInstance，我們照順序啟動 A -> B -> C，但當我們從C 觸發返回的時候，會直接回到 A，因為 B 和 A,C 使用的 backStack 是不同的
適用場景：只有一個 Activity 的流程，且用完即丟棄。
可能會有人有疑問，這聽起來相當不科學呀!因為如果我需要獨立運行的流程，我大可用 Service，在前端顯示出來且單一運行的流程，何不使用 finish() 呢?這個原因就是可重複呼叫節省資源，下一次在建立 B 實例的時候，Appliaction 可以直接使用 backStack-B 裡面的 B，除非我們主動刪除這個獨立的 backStack，不然在 Application 運行期間他是會一直存在的，所以在某些特殊需求的布局當中，singleInstance 還是有它的價值存在的


## 小結
其實原本是打算寫 FragmentManager 的哈哈，後來發現不對阿，上一層的 Activity 都還沒有寫過，不能再這樣亂亂來了，但是我自己的確是屬於做中學的人，常常先做好了某個樣板才回去好好看資料，我覺得這樣的模式有時候也是有好處，按部就班有的時候不會遇到問題，那就沒辦法真正的全面了解這項技術，因為開發就是在踩坑中進步嘛(

2021/08/25

###### tags: `Android` `Launch Mode` `Activity`