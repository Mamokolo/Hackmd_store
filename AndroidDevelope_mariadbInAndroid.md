# Android Develope 2 - MariaDB in android - 學習J筆記
上一篇寫到說，我是為了解決 DB 連線造成的問題，所以使用了 AsyncTask 的方法，在子線程進行 DB 的連線動作，納在 Android 要怎麼跟 DB 連線呢?今天就寫一篇來談談這個問題
首先，我選用的 DB 是 MariaDB，關於 DB 的特性今天就先不談，畢竟我也不熟，就單純來講一下怎麼在 Android App 上與 MariaDB 進行連線，如何使用 SQL 指令
## 開發環境
我這邊是使用 Android Studio，JDK 版本是 1.8，所以其實在環境部屬、library 的部分，跟我們在 Java 連線 DB 是一樣的。
首先，要使用 MariaDB 就一定要有 Driver，那其實在 MariaDB 的官網 ([MariaDB Knowledge Base](https://mariadb.com/kb/en/java-connector-using-gradle/)) 其實是可以看到 gradle 的引用方式的，所以我們不用像在 Eclipse 開發一樣，得自己從外部引入 .jar
同樣使用 JDK 1.8 的朋友在 gradle dependencies 的部分直接上這一條，然後 Sync Project 就可以使用了

```java=
dependencies {
    implementation 'org.mariadb.jdbc:mariadb-java-client:2.1.2'
}
```
但其實到這裡還沒有結束，如果只單純布置 MariaDB Connector 的話，會在其中遇到缺乏 jna 的狀況，因為在 JDBC-Maria 連線的底層方法當中，是需要 Jna 這套驅動程式庫的
Jna 對於 Android 的話就必須要以第三方引入了，網上搜尋 **Jna.jar**，下載之後放入專案的 lib 資料夾，最後在 gradle dependencies 的地方寫入這段
```java=
dependencies {
    implementation files('libs/jna-5.9.0.jar')
    //我下載的是5.9.0的版本，各位記得要改成自己.jar的檔名
}
```

但是...
在 Android Studio 使用 Jna 可能會遇到缺少動態配置的問題
>Caused by: java.lang.UnsatisfiedLinkError: Native library (com/sun/jna/android-arm/libjnidispatch.so) not found in resource path (.)

**這個問題不知道為甚麼我沒有遇到**(所以只實作 **jdbc-mariadb-driver** 的朋友也許可以忽略這個問題，遇到再進行處理)，可能是因為我沒有直接去互動底層 C/C++ 的語法，但是這個問題在網上一樣找的到解答，只要去官方 [github 的 dist 文件夾](https://github.com/java-native-access/jna/tree/master/dist) 找到你想要的 Jna 版本，並且下載「android-armv7.jar」並且把他解開就能找到對應的 libjnidispatch.so檔，最後自己在專案裡配置資料夾路徑 **..\app\src\main\jniLibs\armeabi-v7a** ，然後放進去就可以了

逐步配置下來應該可以讓你在 **Android Studio/ JDK 1.8/ mariadb-java-client-2.1.2/ Jna 5.9.0** 的環境底下進行 DB 的連線，但是使用第三方程式褲就是可能會踩到各種坑，所以大家遇到問題還是乖乖看 console 吧!

## 實作
在配置環境都完成之後，就可以開始實際操作啦!
但是當然在測試之前你必須要有 MariaDB，網路上很多安裝教學，MariaDB 的安裝幾乎就是一鍵到底，所以不用太擔心，建立好一個測試用的 table 之後，就可以開始測試了!

### 建立連線
```java=
Connection connection;
try{
    connection = DriverManager.getConnection("jdbc:mariadb://locolhost:3306/table_name","username","password");
}catch(SQLException sqle){
    sqle.printStackTrace();       
}
```
### 初始化 Driver
```java=
try {
    Class.forName("org.mariadb.jdbc.Driver").newInstance();
}catch (IllegalAccessException | InstantiationException | ClassNotFoundException classE) {
    classE.printStackTrace();
}
```
### 建立 Statment
```java=
Statement statement;
try {
    statement = connection.createStatement();
}catch(SQLException sqle){
    sqle.printStackTrace();       
}            
```
### 取得回傳資料
我這邊示範的是 SQL SELECT 指令，示範如何將資料存在 APP 端
Android 的 SQL Method 如同 JAVA，**ResultSet** 可以使用 ***statement.executeQuery()*** 來做取資料的步驟，也可以使用 ***.execute()*** 之後再從 ***statment.getResultSet()*** 取資料做成兩個步驟
有不少人會使用 ***resultSet.next()*** 確認是否有值，但是我個人還是習慣用 ***resultSet.first()***，確認友直之後才使用 ***resultSet.next()***
```java=
try {
    String username = "Jeff"
    ResultSet resultSet = statement.executeQuery("SELECT * FROM rable_name WHERE Username='"+username +"'");
    if( resultSet.first()) {
        //TODO: If DB has data, then pass
    }
    else {
        //TODO: or deny
    }
} catch (SQLException sqle){
    sqle.printStackTrace();    
}
```
```java=
try {
    String username = "Jeff"
    statement.execute("SELECT * FROM rable_name WHERE Username='"+username +"'");
    ResultSet resultSet = statement.getResultSet();
    if( resultSet.first()) {
        //TODO: If DB has data, then pass
    }
    else {
        //TODO: or deny
    }
} catch (SQLException sqle){
    sqle.printStackTrace();    
}
```

## 範例架構
我這邊羅列兩種架構給大家參考：
**第一種**是寫起來比較繁複，但是階段性區分的比較明顯，在除錯的時候可以藉由 Log 或是存入不同字串頭來確定是哪一個區段出了問題
**第二種**就是寫起來比較簡潔的方法，但由於 Exception 是一起接的，除錯上會麻煩一點。我自己是比較不喜歡這種合起來的寫法

範例狀況為：確認 DB 是否有該帳號，Y 則登入授權，N 則拒絕授權
第一種：
```java=
private String login(){
    String connectResult
    Connection connection;
    Statement statement;
    try {
        connection = DriverManager.getConnection("jdbc:mariadb://locolhost:3306/","username","password");
        try {
            statement = connection.createStatement();
            try {            Class.forName("org.mariadb.jdbc.Driver").newInstance();
                try {
                    ResultSet resultSet = statement.executeQuery("SELECT * FROM table_name WHERE username='"+ username +"'");
                    if( resultSet.first()) {
                        //TODO: If DB has data, then login pass, or login deny
                        connectResult = "LoginPass";
                    }
                    else {
                        connectResult = "LoginFailed";
                    }
                } catch (SQLException sqlLevel3) {
                    connectResult = "SQL_SELECT_ERROR: "+sqlLevel3.getMessage();
                }
            } catch (IllegalAccessException | InstantiationException | ClassNotFoundException classE) {
                connectResult = "CLASS_ERROR: "+classE.getMessage();
            }
        } catch (SQLException sqlLevel2) {
            connectResult = "STATEMENT_CREATE_ERROR: "+sqlLevel2.getMessage();
        }
    }catch (SQLException sqlLevel1) {
        connectResult = "CONNECTION_ERROR: "+sqlLevel1.getMessage();
    }
    return connectResult;
}

```
第二種：
```java=
private String login(){
    String connectResult
    Connection connection;
    Statement statement;
    try (connection = DriverManager.getConnection("jdbc:mariadb://locolhost:3306/","username","password")){
        try(statement = connection.createStatement()){
            try(Class.forName("org.mariadb.jdbc.Driver").newInstance()){
                try(ResultSet resultSet = statement.executeQuery("SELECT * FROM table_name WHERE username='"+ username +"'")){
                    if( resultSet.first()) {
                        //TODO: If DB has data, then login pass, or login deny
                        connectResult = "LoginPass";
                    }
                    else {
                        connectResult = "LoginFailed";
                    }
                }
            }
        }
    }catch(SQLException | IllegalAccessException | InstantiationException | ClassNotFoundException exception){
        connectResult = exception.getMessage();
    }
    return connectResult;
}
```
## 小結
嚴格說起來 Android 連接 DB 比我想像中的容易很多，能夠直接而且簡單的與 DB 溝通，就省下另外去寫 server 的時間。但是對於環境部屬的細節，還有 Jna 除錯的那部分，實在沒有花時間去研究他。
Jna 5.9.0 版本是筆者寫下這篇文的昨天才 release，我也是昨天才發現原來我用的是最新的，但是有去看了一下 issue 並沒有關於 android-armv7 的修正，在發布的 .jar 包裡面也沒有 android 的相關資料夾，必須另外下載。但是我的本意並不是要使用 Jna 去做底層程式碼的撰寫，所以才推斷也許是因為沒有用到 C/C++ Library 才沒有缺少動態程式庫部屬檔案。如果有其他意見或是關於這方面的解答，歡迎大家多做指教

2021/08/24

###### tags: `Android` `JAVA` `Java Native Access` `SQL` `MariaDB` `JDBC` `JDBC-Driver` 