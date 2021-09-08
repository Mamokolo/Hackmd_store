# Java Develope - PLC Communication
之前做的系統案最近已經進入測試實體化階段，整個產線的走向是依靠 PLC 去做控制，但是我們在全自動化系統上，選擇了 Java web 與 PLC 連動，因此並沒有使用主流的 PLC 編成軟體，而是依靠 HslCommunication 去規劃整套系統流程
我在這次的案子裡面則是打雜仔，畢竟寫 Java Application 本來就不是我的專業嘛...(小聲
但是還是有接觸到 HslCommunication，因為要幫主管作流程優化，那這次主要遇到的問題是，PLC R/W 資料遺失，因為是軟硬體的聯通，因此一開始我就想到了是不是在硬體端的作業時間過長，導致指令出去了但是被搶掉資源，果不其然在 HslCommunication 的 Github 就看到了下列的一行警示
> 如果你的程序也使用了一个端口，那么你在读取数据时候， 刚好也在写入（异步操作可能发生这样的情况），那么写入会失败！

沒錯，也就是在異步操作上幾乎是確定同時操作就會造成寫入或是讀取的失敗(取決於哪個為後手操作)

## Solution
為了解決這樣的問題，我們必須在連通 PLC 的時候去設定多個接口。在 PLC to PC 連接上是採用 TCP/IP，因此只要我們規劃多個連接埠去溝通 PLC 基本上就可以完美的解決軟硬體連接的問題。
在 HslCommunication 裡面針對 MelsecNet() 類別有設計物件 OperateResult 去取得操作的狀態、結果，我們能夠以此來判斷操作成功，並且能夠以此物件的回傳動作來當作操作結束的斷點。

```java=
String URL = "192.168.0.1";
/* 設定寫入、讀取各兩個 Port */
int mainPortW = 6000, backupPortW = 6001;
int mainPortR = 6002, backupPortR = 6003;
MelsecNet melsecNetWrite1 = new MelsecNet(URL,mainPortW);
MelsecNet melsecNetWrite2 = new MelsecNet(URL,backupPortW);
MelsecNet melsecNetRead1 = new MelsecNet(URL,mainPortR);
MelsecNet melsecNetRead2 = new MelsecNet(URL,backupPortR);
```

這樣起始我們就有四個連接 PLC 的入口了，只要交替使用入口就能夠有效的確保讀寫品質。而要如何判斷何時使用主要入口何時使用備用入口，只要依靠 OperateResult 就可以了。

以寫入作業為例：
```java=
public class plcDemo{
    public HashMap<Integer,String> portStatus = new HashMap<Integer,String>();
    public int mainPortW = 6000, backupPortW = 6001;
    public int mainPortR = 6002, backupPortR = 6003;
    public static void main(String[] args){
        portStatus.put(mainPortW,true);
        portStatus.put(backupPortW,true);
        portStatus.put(mainPortR,true);
        portStatus.put(backupPortR,true);
        
        for(int i = 0 ; i < 100 ; i ++){
            int flag = i%2; 
            /* 只要 MainPort 被鎖住就使用 BackupPort */
            if(portStatus.get(mainPortW)){
                System.out.println("MainPort");
                WritePLC(mainPortW,"D1501",flag);
            }
            else{
                System.out.println("BackupPort");
                WritePLC(backupPortW,"D1501",flag);
            }
        }
    }
    public void WritePLC(int port, String location, int flag){
        /* 每次呼叫寫入就把 port 的狀態 lock 起來*/
        portStatus.replace(port,false);
        try{
            /* 建立 PLC 連線*/
            MelsecMcNet melsec_net = new MelsecMcNet(PLCURL, port);
            /* 這裡的 OperateResult 是回給我們連線到 PLC server 的結果 */
            OperateResult connectResult = melsec_net.ConnectServer();
            if (connectResult.IsSuccess) {
            /* 這裡的 OperateResult 是回給我們寫入的結果 */
                OperateResult operateResult = melsec_net.Write(location, (short)flag);
                /* 如果沒有成功，就持續寫入(也可以用 forloop 替代，限定多少次之後彈出，送警示，避免 deadlock) */
                while(!operateResult.isSuccess){
                    operateResult = melsec_net.Write(location, (short)flag);
                }
            } 
            else {
                System.out.println("Connection failed:" + connectResult.Message);
            }			
            melsec_net.ConnectClose();
        }catch(Exception e){
            e.printStackTrace();
        }
        /* 上面的邏輯確定無誤之後 只要運作完就可以將 port unlock */
        portPool.replace(port, true);
    }
}

```

但光是這樣還是不夠，因為雖然我們在操作 PLC 的時候，通常不會有過快的指令行為。可是如果遇到多線程操作，又必須保持操作排序，單用兩個 port 去應對可能數個、十多個獨立線程的指令是完全不夠的。
因此我們需要使用 queue 來幫我們達成統合指令，並且做到先進先出，除此之外，我這邊還另外使用 timerTask 去 trigger queue 吐出我的資料，來確保不會造成 PLC 的負擔
以下是我設計的 queue 物件，主要是藉由 HashMap 去建構序列
```java=
class queue{
    int pointer_in=0, pointer_out=0;
    HashMap<Integer, String> locationMap = new HashMap<Integer,String>();
    HashMap<Integer, Integer> flagMap = new HashMap<Integer,Integer>();
    public queue() {

    }
    public void put(String location, int flag) {
        locationMap.put(pointer_in,location);
        flagMap.put(pointer_in,flag);
        if(pointer_in==200) {
            pointer_in=0;
        }
        else {
            pointer_in+=1;
        }
    }
    public Object[] pop() {
        Object[] pop = new Object[3];
        pop[0] = locationMap.get(pointer_out);
        pop[1] = flagMap.get(pointer_out);
        locationMap.remove(pointer_out);
        flagMap.remove(pointer_out);

        if(pointer_out==200) {
            pointer_out=0;
        }
        else {
            pointer_out+=1;
        }
        return pop;
    }
    public boolean isEmpty() {
        if(pointer_in==pointer_out) {
            return true;
        }
        else {
            return false;
        }
    }
}
```
在 queue 物件當中，我單純的設計了 put()、pop()、isEmpty() 等常見的函式。在 put()、pop() 裡面對於 pointer_in/out，我這裡是設計了到達200的時候序列就回到0，所以這個環狀 queue 只能夠放 200 個指令，但是其實已操作來說 20 個應該就是最大了，累積 200 個指令基本是不會出現的。另外需要注意的是，pop() 確定呼叫之後，必須要對資料結構進行該筆資料的清除，才能夠確保 queue 有足夠空間，如果 pointer_in 倒追 pointer_out，則會發生資料遺失

以下是 timerTask 作為 queue trigger 的範例，順便帶上判斷狀態的程式碼
```java=
public static void startTime() {
    if(timer==null){
        timer = new Timer();
    }
    TimerTask timerTask = new TimerTask() {
        @Override
        public void run() {
            if(!queue.isEmpty()) {
                switch (CheckPortEmpty()) {
                    case 1:
                        Object[] objects = queue.pop();
                        WritePLCState((String)objects[0], mainPort, (int)objects[1]);
                        break;
                    case 2:
                        objects = queue.pop();
                        WritePLCState((String)objects[0], backupPort, (int)objects[1]);
                        break;
                    case 3:
                        break;

                }
            }
            timerCount+=1;
        }
    };
    timer.schedule(timerTask,0,500);
}
private void stopTime(){
    if(timer!=null){
        timer.cancel();
        timer=null;
        timerCount=0;
    }
}
public static int CheckPortEmpty() {
    if(MainPortEmpty()) {
        return 1;
    }
    else if(BackupPortEmpty()) {
        return 2;
    }
    else {
        return 3;
    }
}
public static boolean MainPortEmpty() {
    return portPool.get(mainPort);
}
public static boolean BackupPortEmpty() {
    return portPool.get(backupPort);
}
```

## 小結
這次的十座基本上不算太難，只是規劃 solution 真的是一件麻煩事，特別是在產線上，如果一套流程被打掉的話，那程式的邏輯跟布局也都會被打亂，因此要怎麼使用 queue 跟實作指令的傳達，也是需要經過設計與討論