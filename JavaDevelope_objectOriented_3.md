# 物件導向 3 - 學習J筆記

這次的篇章將要開始以Java這個程式語言去探討先前談過的四大概念&在類別當中極度重要的建構子的概念

## Java中的類別(class)
先前已經有對類別做過一些簡單的概念描述
>類別可以將不同資料型態的變數集合起來形成一個結構
這些變數將成為類別的成員
不僅如此，我們還可以為類別定義屬於該結構的函式。

同時
>類別是物件導向程式最基本的單元，是產生同一類物件的基礎
類別可以由使用者自行定義
### 語法
以下就是在Java中的定義類別語法
```java=
[封裝等級] [修飾字] class 類別名稱 [extends 父類別] [implements介面]{
    [封裝等級] [修飾字] 資料型態 field名稱1;
    [封裝等級] [修飾字] 資料型態 field名稱2;
                    :
                    :
    [封裝等級] [修飾字] 回傳值資料型態 method名稱1(參數串宣告){
                method的內容
    }
    [封裝等級] [修飾字] 回傳值資料型態 method名稱2(參數串宣告){
                method的內容
    }
                    :
                    :    
                    :
}

```

在語法當中所有被[ ]的內容都是可以省略的，不一定要賦予定義
而class的[封裝等級]則是除了 public 就是沒有，我們不能夠宣告一個 class 為 private 或是 protected
而每當 class 被宣告為 public 時，這個class就是我們的主類別，必須和.java檔名相同；同時也只有主類別可以被其他程式 import


### 封裝等級
而對於 filed 和 method，在 Java 裡面就可以依照自己的使用方法隨意宣告其封裝等級

在 Java 中，如果省略了成員的封裝等級宣告，則該成員就只能夠被"相同 package "的類別存取，這個就是 Java 的"package封裝等級"

因此我們可以歸納 Java 中的四種封裝等級:
* public    所有類別都可存取
* private   只有在同class內才可存取
* protected 同class或是衍生class及可存取
* package   相同package內的class才可存取(限定範圍內的public)

>在類別中，『不同的存取等級』代表『不同的資料保護方式』，資料之所以需要保護，主要是為了讓各類別的資料獨立開來，不易被其他不相干的函式修改，達到物件導向程式設計中，「資料封裝」及「資料隱藏」的功能。

### example

接下來可以看一下定義class的舉例
```java=
public [修飾字] void class racecar extend car implement race{
    private [修飾字] int engine;
    public [修飾字] int steering_wheel;
    protected [修飾字] int window;
                    :
                    :
    private [修飾字] int power(int engine){
                    :
    }
    public [修飾字] String turn(int steering_wheel){
                    :
    }
}
```
在舉例中，racecar 是一個主類別，繼承了父類別 car，實作 race 這個介面。
racecar 包含了三個 field 和兩個 method ，其中 engine 是 private，身為精密組件的引擎當然只有該類別可以調用；而方向盤是會讓使用者操控的，因此宣告為 public；窗戶 window 的設定因車種會有不同的調整，但是賽車的窗戶也不是能夠隨意被更改的，因此宣告為 protected 讓衍生類別和同類別可以進行操作
調整馬力的方法設定為 private，因為這不屬於使用者介面的考慮範圍。而轉向操作就交由駕駛來進行，因此是 public method

## Java中的物件(Object)

>當我們想要使用類別時，必須透過宣告變數並產生實體後才能夠使用（除了僅使用被宣告為 static 的變數與方法外）
而由類別宣告的變數，稱之為物件參考或物件變數。而產生的實體則稱為物件實體(instance)

###  語法

我們可以延伸我們上面寫過的 class racecar，然後看看下面的例子來了解其中的概念
```java=
racecar ford;
```
上面這裡宣告了一個參考(品牌)為 ford (福特)的賽車，但是至宣告參考的時候，他其實並沒有產生出一個實際上的物件，就像是賽車比賽的名單中登錄了 ford 的車子，但是實際上來說這台車子並沒有出現在賽車場上
>而在 Java 邏輯中，被宣告但是沒有實體的變數則會預設其值為 null
>單純的宣告語法，可以出現在任何 class 或是 method 當中
```java=
ford = new racecar();
```
因此下一步在這裡做的就是，藉由 new 來創建一個實例，使這台 ford 可以實際上的出現在賽車場，讓駕駛員操控
>在 Java 當中，我們產生實體的時候如果需要帶入建構子的話，就必須在()內輸入相應的參數，不然就會是預設的建構參數
>單獨產生實體的語法，只能夠出現在 method 當中。因為在 class 層級當中，如果單獨出現產生實體的語法就表示他和宣告與法是拆分的，那勢必會打亂程式邏輯

當然，我們可以在宣告物件的同時產生實體
```java=
racecar ford = new racecar();
```
這個語法當然就不受限出現在 class 或是 method

### 存取

成功宣告並且創建物件實例之後，當然就是對物件的存取了
我們可以對物件做的事情，當然就取決於物件的實作類別當中，開放了那些 field 跟 method 給我們使用，也就是 public 的 field 還有 method
存取的方式也很簡單，只要使用 . 就可以了
```java=
racecar ford = new racecar();
System.out.println(ford.steering_wheel);
ford.turn(0);
```
沿用我們在 racecar 所寫的設定，由於 steeting_wheel 是 public field，因此我們是可以查看其值的，我們也能夠使用 turn()
而這些動作都必須建立在已經創建了 ford 這個實例之後才有效

## Java 中的方法(method)

method 和 field 一樣，可以被宣告為不同的封裝等級

>private 等級的 field 或 method 無法被外部程式所取用與執行，因此能夠達到物件導向「封裝」的目的。
>而 public 等級的 field 或 method 可以被外部程式所取用與執行，是物件導向封裝時對外唯一的管道，同時 private 等級的 field 或 method 也可以被 public 等級的成員函式呼叫

可以透過以下例子理解
```java=
public class airplane {
    private int power = 0;
    private boolean engine = true;
    private void speedup(){
        power+=1;
    }
    private void slowdown(){
        power-=1;
    }
    
    public void operate(boolean engine){
        this.engine = engine;//同步引擎的狀態
        if(engine==true) {
        	speedup();
        }
        else {
        	slowdown();
        }
    }
}

```
在 main 函式中就可以這樣寫
```java=
public class fly {
    public static void main(String[] arg) {
        boolean engine = true;
        airplane airplane = new airplane();
        airplane.operate(engine);
    }
}
```
在上述的程式碼中，我們透過了 operate 這個 public 的 method 去引導 private method -- **speedup() & slowdown()**
同時也可以無阻礙的改變 private field -- **int power** 的數值

## Java 中的建構子(constructor)

建構子又稱作建構函式，它是在創建實例的時候會自動執行的函式，建構子比較特別，因此他的定義與法與一般函式不同
```java=
[封裝等級] 建構函式名稱(參數串) //建構函式名稱就是類別名稱
{
            ……建構函式內容……
}
```
### 語法說明
* 建構函式名稱一定要和 class 名稱相同。
* 不可加上回傳值型態，連 void 也不行。如果加上回傳值型態（含 void）則不是建構函式，如此就不會在物件實體產生時自動執行。
* 沒有任何修飾字。
* 建構函式只能被 new 運算子於產生物件實體時自動呼叫，無法讓物件的一般函式自由呼叫。
* 封裝等級一般宣告為 public 等級。如果宣告為 private 等級則只有其他建構函式可以呼叫它。無法被 new 自動呼叫。

>對於 Java 而言，如果今天沒有宣告建構函式的話，Java 本身就會幫我們 default 出一個無參數建構函式。而如果我們已經設計一個以上專門應對某些情況的建構函式，我們就必須主動宣告一個無參數建構函式，不然編譯將會失敗。
>
沿用 airplane 類別來實作宣告建構函式的例子
```java=
public class airplane {
    private int power = 0;
    private boolean engine = true;
    
    public airplane() {
    //因為有新增建構函式，所以需要補上無參數建構函式
	}
    //建構物件時如果有附上布林變數key(如果有插上鑰匙的話)
    //就可以在建構實體的時候直接透過引擎狀態操作飛機
    public airplane(boolean key,boolean engine){
        if(key==true){
            operate(engine);
        }
    }
    private void speedup(){
        power+=1;
    }
    private void slowdown(){
        power-=1;
    }
    
    public void operate(boolean engine){
        this.engine = engine;//同步引擎的狀態
        if(engine==true) {
        	speedup();
        }
        else {
        	slowdown();
        }
    }
}
```
在 main 函式中就是這樣子進行的
```java=
public class fly {
    public static void main(String[] arg) {
        boolean engine = true;
        airplane airplane = new airplane();
        airplane.operate(engine);
        //透過新增的建構函式創建的實例
        airplane airplane2 = new airplane(true,false);
    }
}
```
看著上面的例子有沒有覺得很熟悉，相同的建構函式名稱，不同的建構函式參數...

沒有錯，建構函式也是能夠進行 overload 的!
透過不同參數組合所組成的建構函式，就可以對應不同的預設行為!
想想都覺得興奮啊!

## 小結
在這個部分把四大概念中的三個概念--類別、物件、方法，還有建構子這個概念劃分在一起，主要是因為覺得這四個概念在我們進行類別的設計上是比較主要的組成元素。
屬性(field)/變數的話，基本就不多贅述，因為就是相對來說比較簡單的概念，主要還是根據繼承性去劃分不同的封裝等級。

2021/01/06

###### tags: `JAVA`  `學習計畫` `Class` `Object` `Method` `Constructor`














