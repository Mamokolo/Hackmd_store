# Android Develope 5 - okhttp3 ft. AsyncTask
相信寫 Android app 的很多人應該都會用到 okhttp 這個第三方工具包，因為在我們進行聯網的時候用 java jdk 的 httpClient 是會出很多錯誤的。
我前面已經有用兩章篇幅來介紹 AsyncTask 還有 okhttp 了，這邊我就會講解一下如果我們想要在 UI 上面跟聯網做一些互動，然後不想自己寫 Hanlder/thread 的時候，可以怎麼搭配 okhttp 還有 AsyncTask

## 架構
簡單來說，我會使用 AsyncTask 直接包覆 okhttp 的聯網，再依靠回傳的數據來進行 UI 更新。
以下可以直接上程式碼(我這邊就不去介紹 API server 怎麼寫，單純實作 Client 端；實作的功能為登入功能

```java=
private class Login extends AsyncTask<String,String,String>{
    private static final String TASK_PROGRESS = "Task progress" ;

    @Override
    protected void onPreExecute() {
        callLoginDialog();
        Log.i(TAG,"Login...");
        publishProgress("Login Prepare");
    }
    @Override
    protected String doInBackground(String... params) {
        String username = params[0];
        String password = params[1];
        final String[] responseResult = {""};
        Thread thread = Thread.currentThread();
        publishProgress("Login..");


        /* 連線設定 */
        ConnectionSpec spec = new ConnectionSpec.Builder(ConnectionSpec.COMPATIBLE_TLS)
                .tlsVersions(TlsVersion.TLS_1_2, TlsVersion.TLS_1_1, TlsVersion.TLS_1_0)
                .allEnabledCipherSuites()
                .build();//解决在Android5.0版本以下https无法访问
        ConnectionSpec spec1 = new ConnectionSpec.Builder(ConnectionSpec.CLEARTEXT).build();//兼容http接口
        OkHttpClient client = new OkHttpClient().newBuilder().connectionSpecs(Arrays.asList(spec, spec1))
                .addInterceptor(new HttpLoggingInterceptor().setLevel(HttpLoggingInterceptor.Level.BASIC))
                .build();
                
        /*建立 String 格式的 RsquestBody*/
        MediaType JSON = MediaType.parse("application/json");
        JSONObject json = new JSONObject();
        try{
            json.put("userId",username);
            json.put("password",password);
        }catch (Exception e){
            Log.i(TAG,"json: "+e.getMessage());
        }
        final RequestBody requestBody = RequestBody.create(JSON, String.valueOf(json));
        
        /*Send Request*/
        Request request = new Request.Builder()
                .url("http://[IP_ADDRESS]/[API_PATH]")//要送去的網址ser
                .post(requestBody)
                .addHeader("Content-Type","application/json")
                .build();
        Call call = client.newCall(request);
        call.enqueue(new Callback() {
            @Override
            public void onFailure(@NotNull Call call, @NotNull IOException e) {//這個 Func 主要是如果連線失敗的時候進入
                Log.i(TAG,"HttpRequestError " + e.getMessage());
                responseResult[0] = "error";
            }
            @Override
            public void onResponse(@NotNull Call call, @NotNull Response response) throws IOException {//連線只要成功就會進這，所以 API 回傳的 invalid 也會在這

                String responseString = response.body().string();
                if(responseString.contains("error")){
                    responseResult[0] = "failed";
                }
                else if(responseString.contains("access_token")){
                    responseResult[0] = "success";
                    String string = responseString
                            .replace("{","")
                            .replace("}","")
                            .replace("\"","");
                    String[] res = string.split(",");
                    for (String re : res) {
                        String[] temp = re.split(":");
                        loginData.put(temp[0], temp[1]);
                    }
                }else{
                    responseResult[0] = "unexpected";
                }
                Log.i(TAG,"Login Done");
            }
        });
        /* 這裡很重要 */
        int timerCount = 0;
        while(responseResult[0].equals("")){
            if(timerCount>=20){
                responseResult[0]="timeout";
            }
            else{
                timerCount++;
            }
            try {
                Thread.sleep(500);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
         /* 這裡很重要 */
        return responseResult[0];

    }

    @Override
    protected void onProgressUpdate(String... values) {
        Log.i(TASK_PROGRESS,values[0]);
        final TextView textView = LoginActivity.this.loginDialog.findViewById(R.id.textView_loginProgress);
        textView.setText(values[0]);
    }

    @Override
    protected void onPostExecute(String result) {
        Log.i(TAG,"Login prepare done");

        if(!result.isEmpty()){
            //TODO: Deal with every return msg, include wrong msg
            Log.i(TAG,"Login Result: "+ result);
            if(result.equals("success")){
                Log.i(TAG,"登入成功");
                endLogin();
                //TODO: 登入成功後要做的事情...
            }
            else if(result.equals("fail")){
                Log.i(TAG,"登入失敗");
                //TODO: 登入失敗後要做的事情...
                endLogin();
            }
            else if(result.equals("unexpected")){
                Log.i(TAG,"非預期異常");
                //TODO: 登入異常後要做的事情...
                endLogin();
            }
            else if(result.equals("timeout")){
                Log.i(TAG,"連線超時");
                //TODO: 登入失敗後要做的事情...
                //因為我自己是有設計 reLogin 的功能，所以這邊我只關閉 AsyncTask 線程，但是其他事情不做
            }
            else {
                Log.i(TAG,"連線失敗");
                callWarningDialog("連線失敗");
                endLogin();
            }
        }
        else{
            Log.i(TAG_ERROR,"Nothing Return");
            callWarningDialog("登入動作\n無效");
            endLogin();
        }
    }

    private void endLogin(){
        loginTask.cancel(true);
        //可能有設計一些 UI 要關閉
    }

    @Override
    protected void onCancelled() {
        Log.i(TAG,"Login Dismissed");
    }
}

```

## 源碼解說
有兩個比較重要的點，首先看第一個是在程式碼 54 行的地方，也就是 onResponse 的方法
因為我這支程式沒有引入解析 Json 的第三方工具包，所以我將 Response.body() 直接解析為 String
請看我的用法:
```java=
 String responseString = response.body().string();
```
在這裡要注意的是 
    
1. string() 這個方法是將 response.body() 內容轉成 String 的型別，如果使用 response.body().toString，則只會將表面的 byteValue show出來，並不會轉碼成 String 型別的物件\
2. string() 這個方法會切斷 I/O-Stream，只能呼叫一次其設計原理如下:

> 在實際開發中，響應主體 RessponseBody 持有的資源可能會很大，所以 OkHttp 並不會將其直接儲存到記憶體中，只是持有資料流連線。只有當我們需要時，才會從伺服器獲取資料並返回。同時，考慮到應用重複讀取資料的可能性很小，所以將其設計為一次性流(one-shot)，讀取後即 "關閉並釋放資源"。

第二個點則是在程式碼 78-91 行的地方
```java=
int timerCount = 0;
while(responseResult[0].equals("")){
    if(timerCount>=20){
        responseResult[0]="timeout";
    }
    else{
        timerCount++;
    }
    try {
        Thread.sleep(500);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}
```
這裡為甚麼要這樣寫呢，是因為 okhttp POST 方法是透過 callback 去完成的，也就是說我們在 AsyncTask 這個異步線程當中再開了一個異步線程，如果這裡沒有讓 AsyncTask 的線程 sleep() 的話，那只要沒有再瞬間完成回傳 result 的話，每次傳出去的 result 就都會是預設值(現在是 null)

因此我們才要設計一個，如果當下紀錄的 result 為 "" 的話，AsyncTask 線程就睡 0.5 秒，當然這些都是可以自己設定的數字，重點是要保證邏輯的完整性，讓回傳結果之前和之後的判斷能有明確的分水嶺

## 結語
以上就是 okhttp3 跟 AsyncTask 在 Android 當中實作的小範例，本次篇幅不多，大多都是程式碼，主要是這一塊踩到的坑不多，希望能夠幫助看到這篇文章的朋友

2021/10/26

###### tags: `Android` `學習計畫` `AsyncTask` `okhttp3` `httpClient`