# Android Develope 1 - AsyncTask - 學習J筆記
最近開始寫的案子開始要跟 DB 做一些溝通，在進行 DB 連線的時候會有很多流程的控管，為了要能讓系統按部就班的進行 DB 的連線，不導致線程衝突，所以必須做非同步任務管理，這次使用的技術則是 **AsyncTask**。
**AsyncTask** 本身包含的架構非常匹配用來製作需要 Loading 的畫面，以下就針對 **AsyncTask** 的語法來做介紹。

## AsyncTask 架構
**AsyncTask** 由 *.execute()* 以及 *.cancel()* 來進行任務的進入點與取消點，而內部的流程控制架構如下:
```java=
protected void onPreExecute(){
    //流程被呼叫之後就直接執行
    //在 MainThread 執行，通常可以用來做一些 prepare 的 UI 畫面
}
protected String doInBackground(String... strings){
    //複雜的主要執行邏輯，會另開 thread 執行
}
protected void onProgressUpdate(String... strings){
    //在 doInBackground 的流程當中，可以隨時呼叫 publishProgress(value) 來進行與前端的溝通
    //在 MainThread 執行，能夠更新 Loading 畫面
}
protected void onPostExecute(String string){
    //doInBackground 結束後接取回傳的內容進行判斷，同時中止流程
    //在 MainThread 執行，流程結束後更新 UI 畫面
}
protected void onCancelled() {
    //主程式當中可以直接藉由呼叫 .cancel(true) 來中斷 AsyncTask
    //同時也就不會觸發 onPostExcute()
    //在 MainThread 執行，流程結束後更新 UI 畫面
}
```

## 注意事項
單一 **AsyncTask** 物件只能夠執行一次，並且不建議使用 looper，如果是需要重複進行的流程，只要每次都 new task_name() 就好
實作 **AsyncTask** 類別的 class 一定要覆寫的方法只有 ***doInBaskground()*** 方法而已，但是如果不傳入參數與回傳，基本上就失去了使用這套非同步任務流程的意義了，可以使用 **Thread/Handler**，更加直觀!

## 範例
下面就上各方法的範例，寫法以 Android project basic Java 來示範
```java=
public void onCreate(Bundle savedInstanceState){
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    Button btn_start, btn_cancel;
    btn_start = findViewById(R.id.btn_id);
    btn_cancel = findViewById(R.id.btn_id2);
    AsyncTask task;
    
    btn_start.setOnClickListener( v -> {
        task = new TaskName();
        task.excute(para1, para2, para3);
    });
    btn_cancel.setOnClickListener( v -> {
        if(task!=null){
            if(!task.isCanceled()){
                task.cancel(true);
            }
        }
    });
}    

private class TaskName extends AsyncTask<String,String,String>{
    @Override
    protected void onPreExecute(){
        //這裡可以直接溝通全域的 TextView 或是 ImageView 做UI
    }
    
    @Override
    protected String doInBackground(String... strings){
        //這裡會把 .execute() 接近來的參數歸納為一個 String[]，所以我們可以直接這樣使用
        String para1 = string[0];
        String para2 = string[1];
        String para3 = string[2];
        
        String tmp = "";
        tmp = para1;
        publishProgress("First step");//publishProgress()也是可以傳入多個參數的
        para1 = para2;
        publishProgress("Second step");
        prar2 = para3;
        publishProgress("Third step");
        para3 = tmp1;
        publishProgress("End resort");
        
        String resort = para1+","+para2+","+para3;
        
        return resort;//這裡 return 的內容就是 onPostExecute() 接到的參數
    
    }
    
    @Override
    protected void onProgressUpdate(String... strings){
        //每次呼叫 publishProgress() 就會 trigger 一次
        Log.i(TAG,string[0]);//進來的訊息依樣會歸納為 String[]
    }
    
    @Override
    protected void onPostExecute(String string){
        //完成後就可直接取用回傳值
        String[] strings = string.split(",");
        log.i(TAG,"NEW Para1: "+string[0]);
        log.i(TAG,"NEW Para2: "+string[1]);
        log.i(TAG,"NEW Para3: "+string[2]);
        //這邊應該要印出新的排序
    }
    
    @Override
    protected void onCancelled() {
        Log.i(TAG,"Task Dismiss");
    }
}
```

## 小結
因為最近又開始想寫東西，所以就直接把這次做的東西當作我的第一篇 Android 筆記了~
使用過一次 AsyncTask 之後真的會有種愛不釋手的感覺，特別是開始要做一些聯網的操作，手機上面沒有辦法已非同步流程控管去做待機畫面的話真的很不人性化。
但是現行的 AsyncTask 一次只能夠 handle 五條線程，如果超過的話就會進入排隊，並且還是不要硬操會比較好。
個人覺得 AsyncTask 有效的將我們想要完成的操作獨立出來進行，更有助於模組化的開發 APP，濫用的話當然就是效能大打折扣，同時也會脫離 clean code 的開發理念。

2021/08/23

###### tags: `Android`  `JAVA` `AsyncTask` 
