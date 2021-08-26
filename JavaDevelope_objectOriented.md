# 物件導向 2 - 學習J筆記
上次看了四大概念，這次來看三大特性
滿常在實習、junior engineer 的面試題目看到有關三大特性的題目的
每次做口頭回答的時候都感覺自己大學念了快四年還是什麼都不懂，懷抱虛心的態度持續學習囉:)

## 物件導向的三大特性
物件導向含有三大特性
* 封裝
* 繼承
* 多型

使得物件導向更適合用來開發、管理大型的程式、專案
下面則會根據三大特性來做介紹

---


### 封裝性(encapsulation)
封裝性可以幫助我們區分可被外部使用的特性以及受到保護的內部特性
除非外部程式允許讀取的資料，否則資料不能夠被外部程式改變，也就是說，封裝即為一種保護特性。
以此為根基，物件導向將物件分為三種等級( Java 當中還有package 等級)
* public(公用)
public 等級的物件開放給所有的程式碼取用
* private(私用)
private 等級的物件只開放給"相同類別"內的程式碼取用
* protected(保護)
protected 等級的物件則是開放給"相同"或是"衍生"類別的程式碼取用

#### Example

以車子為例，今天一台汽車 object，在假設我們買下他，是這台汽車的持有人的狀況下，我們坐在車內，可能就會出現下列的情況
* public property：方向盤、油門、煞車、儀表板、排檔桿、座椅、車門、油箱蓋、雨刷、廣播器、杯架...等
* private property：引擎、煞車帶、衛星天線...等
* public method：發動、加速、減速、換檔、開雨刷、轉彎...等
* private property：燃料供給、懸吊系統調整、齒輪比調整、油量電量測量反應...等

**今天如果站在兩個角度去思考：駕駛人、汽車製造商**

對於駕駛人來說可以操控的屬性像是使用方向盤、油門、煞車，這些使用者可以決定他要如何操作、取用的屬性，就是 public property。

駕駛人可以操作，且可以透過自己喜好改變的操作(像是駕駛人可以透過方向盤轉彎)，在程式當中就通常會設定為 public method

而對於一般來說駕駛人沒辦法掌握的，換句話說只有汽車製造商可以取用、調整這些零件(一般的駕駛人沒辦法更改汽車的某些內裝)，這些零組件就是 private property。

或是一般的駕駛人沒辦法觸及到的技術問題，比如大部分的駕駛人只會他可以透過方向盤轉彎，但是只有汽車製造商可以決定方向盤往左轉汽車就要跟著往左轉，這就是 private method
>被封裝的屬性則是不可存取的，除非透過外部方法間接存取

以下是一個簡單的封裝實作
將轉彎這個功能封裝到懸吊系統這個類別(物件)上


```java=
public class drive{
    public static void main(String[] args) {
        //產生物件
    	steering_system steering_system = new steering_system();
        //方向盤左打
        String steering_wheel = new String("left");
        //然後轉彎這件事情就交給轉向系統steering_system去搞定
        steering_system.turn(steering_wheel);
    }
}
class steering_system {
    //在轉彎這件事情上，輪胎不是可直接被操作的物件
    private String tires = new String("strength");
    
    //駕駛員無法決定如何轉彎，因此不讓駕駛員去觸碰到轉彎細節
    private void turn_left(){
        //左轉的細節
    	tires = "left";
    	System.out.println(tires);
    }
    private void turn_right(){
        //右轉的細節
    	tires = "left";
    	System.out.println(tires);
    }
    //駕駛員可以透過互動來轉彎，因此他可以呼叫轉彎這個方法
    public void turn(String steering_wheel){
        if(steering_wheel=="left"){
            turn_left();    
        }
        else if(steering_wheel=="right"){
            turn_right();
        }
    }
}

 
```
#### Conclusion
在上述例子當中，駕駛員可以決定要左轉還是右轉，但是不能決定如何左轉或是如何右轉

這樣的區分主要是在表達，工程師當然不會對自己的操作有任何的阻擋行為，但是工程師必須致力在於不讓他人、外部程式破壞、更動自己的程式碼邏輯，不只是做到保護，同時也是在讓程式碼彼此之間互相獨立，顯得更加乾淨。

---

### 繼承性(inheritance)
>繼承性是為了達成重覆使用目的所採取的一種策略

例如:一台普通房車，加上尾翼、空力套件之後可以增加汽車行駛的穩定性，變成一台賽車；一台普通的飛機，掛載格林機槍就可以變成戰鬥機。

因此，賽車繼承了房車的所有屬性和方法，戰鬥機也同樣繼承了飛機的所有屬性和方法，這兩者是以他們繼承的類別去加以擴充的。

>在物件導向的程式設計當中，允許使用者定義**基底類別(base class)** 和 **衍生類別(derived class)**

衍生類別允許繼承基底類別的所有屬性和方法，然後透過
* 擴充屬性及擴充方法
* 修改(override)部分屬性及方法

來達成需求
有了這項技術，便可以在已經完成的技術之上，再進行擴充

下面我們再看一個例子，首先是 car.java

```java=
public class car {
    
    private String steeringwheel,key;
    private String wheels,engine;
    protected String door;
    
	//類別car的建構式
    public car() {
    
    }
    public car(String wheels,String engine,String steeringwheel,String key) {
    	this.wheels = wheels;
    	this.engine = engine;
    	this.steeringwheel = steeringwheel;
    	this.key = key;
    }
    //car所含有的private method，表示在程式運行中有使用到此邏輯，但是不讓呼叫的人使用的函式
    private boolean power(){
        if(key=="yes"){
            return true;
        }
        return false;
    }
    //car所含有的public method,表示是可以讓大家使用的函式
    public void turn(String steeringwheel) {
        wheels = steeringwheel;
        System.out.println(wheels);
    }
    public void move(String key) {
        if(power()) {
            engine = "heat_up";
        }
    }
    public void stop(String key) {
        if(!power()) {
            engine = "cold_down";
        }
    }
    public void open_door(String key) {
    	if(key=="no") {
    		door="open";
    	}
    	else if(key=="yes") {
    		door="close";
    	}
    }
}
```
再來看到繼承了普通房車但加了尾翼的賽車 racecar.java

```java=
public class racecar extends car{
    private String tail;
    
    //以定義來說，沒有尾翼也可以是賽車
    public racecar() {
        super();
    }
    //擴充建構子，將尾翼新增到建構子當中
    public racecar(String wheels,String engine,String steeringwheel,String key,String tail) {
        super(wheels,engine, steeringwheel, key);	
        this.tail = tail;
    }
    //新增方法
    public void setTail(String color){
        tail = color;
    }
}
```

同時我們也可以在這裡實作 protected 的變數或是方法
比如說，我們希望能夠在 racecar.java 裡面實作一個新的功能-超級加速
但是超級加速會修改到引擎的狀態，而引擎在原先在 car.java 當中是屬於 private 的變數。
我們既不想讓他被所有人修改，但是房車又不需要 superspeed 功能。這個時候就可以將 car.java 的宣告修改成下面的樣子
```java=
public class car {
    
    private String steeringwheel,key;
    private String wheels;
    protected String engine
                :
                :
                :
}
```
然後就可以在 racecar.java 裡面成功新增 superspeed 函式，並且有權限修改 engine 的狀態了
```java=
public void superspeed() {
    engine = "superspeed";
    System.out.println("Speed up above the limit");
}
```

>**良葛格的話匣子** 在設計類別的過程中，並不是一開始就決定哪些資料成員或方法要設定為 "protected"，通常資料成員或非公開的方法都會先宣告為 "private"，當物件的職責擴大而要開始使用繼承時，再逐步考量哪些成員要設定為 "protected"。

---

### 多型性(Polymorphism)

多型是物件導向中最抽象的名詞，舉例來說，英國人、美國人、台灣人都會煮奶茶，但是三人煮奶茶的方法不同，先油先鹽都不一定，所以其實煮奶茶這個型為是一個很抽象事情。
但是煮奶茶這三個字卻可以同時包含了各式各樣煮奶茶的方法，這種特質就是所謂的"多型性"

#### 多型的實作方法

* **改寫/覆寫(override)**

    
例如英國人、美國人、台灣人，這三個物件都必然繼承了"文明人"這樣的類別，所以我們可以稱這三種人都為"衍生類別"。而在"文明人"這個類別的基底就包含了煮奶茶這個 method，我們就必須在衍生類別當中改寫煮奶茶這個 method 的細節。
這樣一來，不同類別產生的物件，在煮奶茶的時候就會體現不同的細節。

我們沿用 car.java 來做說明，在 car.java 中有 open_door() 函式，
```java=
public void open_door(String key) {
    if(key=="no") {
        door="open";
    }
    else if(key=="yes") {
        door="close";
    }
}

```
但是我們假設鑰打開賽車的們除了插鑰匙之外，還會需要輸入密碼，避免有人隨意進入昂貴的賽車，那麼我們就可以在racecar.java當中進行下面的修改
```java=
public void open_door(String key, String password) {
    	if(key=="no" && password=="1234") {
    		door="open";
    	}
    	else if(key=="yes") {
    		door="close";
    	}
    }
```
這樣子在建構出racecar這個類別的物件的時候，使用的open_door()就會是需要密碼才能開門，關門則一樣不需要

有趣的是，如果今天我們繼承父類別的時候，在父類和子類擁有一致的操作介面的情況下，能夠以子類別實作父類別的物件

以簡單例子來說，有父類別A和子類別B
```java=
class A{
        :
    public void func(int i){
        if(i==1){
        //do something here
        }
    }
        :
}
```
```java=
class B extends A{
        :
    public void func(int i, String s){
        if(s=="1" && i==1){
        //do something here
        }
    }
        :
}
```
那麼我就能夠用這樣的宣告來實作
```java=
    A smallstuff = new B();
```
上述宣告代表【以  B實作的 A 型態物件】 - **smallstuff**
在"操作介面"上來說 B 與 A 無異，因此他被宣告為 A 型態，但是在操作 func 的時候，會以 B 的方式來進行實作，也就是說他除了匹配 **i** 之外還要匹配 **s**，一共兩個變數。

當然這樣類型的宣告不是無時無刻都通用，但是在多型繼承當中，包含了 override、overload 交叉使用的情況之下常常可以看到這樣的利用。但是這就代表需要更加精準的設計才不會導致自己看不懂，或是實作出現錯誤。

* **多載(overload)**

overload 即為命名相同、署名不同的函式
可以透過以下的簡單例子來了解，我們假設今天車子發動之後，根據不同的檔位可以有不同的操作。
如果我們直觀的將檔位放到 func 裡面，那可能會長這樣
```java=
public void gear(string gear){//檔位可能是 P、R、N、1234
    if(gear==P){
        parking();
    }
    else if(gear==R){
        reversing();
    }
    else if(gear==N){
        neutral();
    }
    else{
        driving(gear);//把前進檔位再帶進driving方法
    }
}
```
但是這樣的實作方式，就表示我們必須要限定所有檔位都是 string 表示，因此我們其實可以這樣實作:
```java=
public void gear(string gear){
    if(gear==P){
        parking();
    }
    else if(gear==R){
        reversing();
    }
    else if(gear==N){
        neutral();
    }
}
public void gear(int driveGear){
    switch(driveGear){
        case 1:
            ...
            break;
        
        case 2:
            ...
            break;
        
        case 3:
            ...
            break;
              :
              :
              :
    }
}
```
根據我們送給 gear() 這個方法的參數不同(也可以是參數組合)，就可以改變這個方法的實作方式，這就是所謂的 overload
使用 overload 的好處就在於，我們可以根據輸入的參數去做更好的控管分類，因為通常輸入參數改變，就會代表著不同的實作方法，將其區別開來就能夠更好的維護程式

由以上兩種多型的機制可知，多型是一種呼叫相同名稱函式，但卻可執行不同函式內容的機制，因此，又有人將『多型』稱之為『同名異式』

---

### 小結

透過了解物件導向的三大特性，就可以去以一些實際的例子來做觀察





###### reference
[JAVA SE6 技術本本](https://caterpillar.gitbooks.io/javase6tutorial/content/c8_1.html)
[【c++類別class的語法大全】(1) 物件導向概念; 封裝與存取權限; class基礎語法; 預設建構子與拷貝建構子](https://ithelp.ithome.com.tw/articles/10230401)

###### tags: `JAVA` `學習計畫` `物件導向` 

























