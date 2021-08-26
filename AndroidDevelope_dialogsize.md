# Android Develope 4 - Dialog 自訂&自適應大小 - 學習J筆記
相信大家初學 Android 的時候，常常在使用彈出視窗的時候，如果要進行文字排版、按鈕排版會感覺特別麻煩，許多初學的文章會推薦去使用 AlertDialog，因為配置簡單，還有三個按鈕跟布局好的標題欄、訊息欄，但是其實 AlertDialog 之所以曰其名，就是因為它最好只拿來做 Alert，如果需要更進階的互動、佈局，還是乖乖的寫個 layout 然後去配置給 Dialog 類吧。
但既然用了 Dialog，我是不是應該要把它的大小配置的更自由呢? 因為我打算在 Fragment 實作一套邏輯，其中會使用到彈出視窗，但是 Fragment 的螢幕佈局其實不像是 Activity 那樣可以比較完美的自適應，所以我們就需要透過一些手段來「手動自適應」

## 範例
既然想要自適應配置大小，那當然就要取得螢幕的大小，再來調整長寬，那要怎麼取得螢幕大小呢? 就得靠 WindowManager 了

大致的流程如下：
1. 建立一個 WindowManager 實例
2. 從 WindowManager 實例取得現在的顯示器資料
3. 取得長寬
4. 建立 Dialog
5. 建立一個 WindowManager.LayoutParams 實例去接 Dialog 的 default 參數
6. 覆寫參數之後，再讓 Dialog 重新以這個參數去設定大小

那接下來就上程式碼吧

### Activity demo
這個是在 Activity 的寫法，可以將 Dialog 的大小定義成 0.5 倍的螢幕長寬
```java=
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    
    WindowManager windowManager = getWindowManager();
    Display display = windowManager.getDefaultDisplay();
    int width = display.getWidth();
    int heigth = display.getHeigth();

    Dialog dialog = new Dialog(this);
    dialog.setContentView(R.layout.dialog_layout);
    dialog.show();

    WindowManager.LayoutParams layoutParams = dialog.getWindow().getAttributes();
    layoutParams.width = (int)(width * 0.5);
    layoutParams.heigth = (int)(heigth * 0.5);
    dialog.getWindow().setAttributes(layoutParams);
}
```


需要注意的是，WindowManager 是 Activity 類的方法，要在 Fragment 使用，必須要先 requireActivity 取得當下的 Activity 實例
此外，Display 的 getWidth()、getHeigth() 方法已經在 API-level13 被 ~~DISS~~ 了，Google 提出了取代的方案是，建立一個 Point 實例，用來存取 display 的長寬

### Fragment demo
以下是在 Fragment 運行的寫法，並且做了方法的修改
```java=
@Override
public void onViewCreated(@NonNull View view, @Nullable Bundle savedInstanceState) {
    super.onViewCreated(view, savedInstanceState);
    
    WindowManager windowManager = requireActivity().getWindowManager();
    Display display = windowManager.getDefaultDisplay();
    Point point = new Point();
    display.getSize(point);

    Dialog dialog = new Dialog(this.getContext());
    dialog.setContentView(R.layout.dialog_layout);
    dialog.show();

    WindowManager.LayoutParams layoutParams = dialog.getWindow().getAttributes();
    layoutParams.width = (int)(point.x * 0.5);
    layoutParams.heigth = (int)(point.y * 0.5);
    dialog.getWindow().setAttributes(layoutParams);
}
```

## 小結
個人覺得在越後期，Dialog 的自定義排版就顯得越重要，因為專案工程師每個案子出來都會希望有點新意，不然也沒辦法時刻督促自己學習。而且Dialog 可以搭配很多 View 去做變化、互動，生命週期的部屬也會是很好的學習課題。

2021/08/26

###### tags: `Android` `JAVA` `Dialog` `WindowManger`