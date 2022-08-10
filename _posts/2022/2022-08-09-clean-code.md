---
title: "Clean Code 閱讀筆記"
subtitle: ""
excerpt: "Clean Code"
layout: post
author: "blueskyson"
header-style: text
tags:
  - note
---

![](https://raw.githubusercontent.com/blueskyson/image-host/master/2022/cleancode.jpg)


註: 以下規則只代表這個學派多年所歸納下來的經驗守則，並不代表以下所有規則都是正確、不可違背的，請根據實際情況調整。

## Chapter 2: 有意義的命名

- 變數、函式、類別的名稱應該要告訴你它為什麼要在這裡出現、它的用途、該如何使用它。如果一個名稱需要註解來輔助，這個名稱無法展現它的意圖，例如用 `elapsedTimeInDays` 取代 `d`。
- 避免留下喪失原意的錯誤線索，例如不要使用 `accountList` 來表示一群帳戶，除非它真的是 `List` 型態，用 `accountGroup`、`accounts` 都比較好。即便這個變數的型態真的是 `List`，也盡可能不要把型態加到變數名稱當中，例如程式中如果並存 `Name`、`NameString`、`NameObject`，如果沒有特別約定，後面維護的人無從得知使用哪一個才是正確的。
- 盡量使得命名容易看出差異，`ControllerForHandlingString` 和 `ControllerForStroageString` 就很難直接看出差異。
- 不要為了能通過編譯而故意拼錯字、加數數字序列、加入無意義的字詞，他們都屬於無效資訊，無法使人知道程式作者的意圖，例如不要用 `a1`、`a2` 或 `knew`、`kclass`、`mystruct`。
- 使用可以被唸出來的名稱。例如不要因為懶惰用 `gentstmp`，用 `generateTimestamp`，當你與其他開發者溝通時後者才能明確表達意思。
- 除非變數跨越的 scope 很小，不然盡量使用長一點的名字，方便搜尋程式碼時快速找到名稱，並且不會搜尋到無關的程式碼。比如將很常用的變數命名為 `e`，搜尋程式碼的時候整個文件的 `e` 都會被反白，不利於查找、重構。
- 變數、函式避免匈牙利命名法。此外，作者個人不喜好使用 `I` 命名介面，而是偏好將實作命名為 `Imp`。
- 類別、物件使用名詞命名或名詞片語命名，如 `AddressParser`。方法使用動詞或動詞片語命名，如 `deletePage`。
- 每個概念都固定使用一種詞彙表達，例如不要同時有 `fetchNumber`、`getNumber`，不要同時有 `controller` 和 `driver`。
- 添加有意義的上下文資訊，例如使用 `addrState` 代替 `state`，這樣讀者會知道這是代表地址的州別。不要添加無意義的上下文資訊，例如你的程式叫 `Gas Station Delux`，然後你將所有類別、物件都命名為 `GSD...`，這樣 `GSD` 不僅沒有實質意義，在 IDE 自動補齊時還會列出一堆不相干的 `GSD...`。

## Chapter 3: 函式

- 函式的首要準則： 簡短！究竟要多短沒有一個準則，本書作者曾閱讀朋友寫的程式，其中幾乎每個函式都不超過 4 行。
- if、else、while 等敘述應該只有一行，並且那一行程式通常是函式呼叫的敘述。這也意味函式不該包含巢狀結構。
- 函式應該只做一件事，它們應該把這件事做好，而且它們只該做這件事。透過「TO (要) 段落」，例如 TO `RenderPageWithSetupsAndTeardowns` 描述了納入設定與拆解並將葉面轉化為 HTML。
- 降層準則：閱讀程式碼時，能夠像閱讀一連串 TO 段落，每個段落敘述目前所處的抽象層次。
- 避免大量使用 switch 敘述，如果無法避免使用 switch，最好的作法是結合工廠模式，在抽象工廠透過 switch 產生衍生介面的實例，如此一來就能最少化的使用 switch。當然很多時候的情況不是那麼單純，所以作者偶而會違反這個準則。
- 不要害怕取較長的函式名稱，一個較長且具描述性質的名稱比較短但難以理解的名稱好。
- 函式的參數盡量不要超過兩個，過多參數會強迫讀者去了解細節。
- 不要用布林值的 flag 當作參數，此舉等同於讓這個函式做兩件事，例如 `render(true)` 會讓讀者感到困惑，進而閱讀函式的宣告 `render(bool isSuite)`，應該分裂成兩個函式 `renderForSuite()` 以及 `renderForSingleTest()`。
- 如果有一些機制可以將兩個參數換成一個參數時就要好利用，例如 `writeField(outStream, name)` 改成 `outStream.writeField(name)`。
- 利用建立物件的方式也可以減少參數數量，例如 `makeCircle(double x, double y, double radius)` 可以改成 `makeCircle(Point center, double radius)`。
- 避免 Side Effect，除非函式名稱有提及他會改變某些外部狀態，函式不該改變毫不相關的物件的狀態，也不該改變全域變數的狀態，例如：
  ```non
  bool checkPassword(string username, string password) {
      // ...
      Session.initialize();
      return true;
  }
  ```
  函式名稱只提到他會檢查密碼，但內部卻呼叫 `Session.initialize()`，造成 Side Effect，也違反了函式只做一件事的準則。解決方法為把 `Session.initialize()` 移去更外層的函式；或是將此函式改名為 `checkPasswordAndInitializeSession`。
- 指令和查詢的分離，以下這個函式會設定某屬性的值，當設定成功會回傳 `true` 該屬性不存在回傳 `false`，導致詭異的敘述 `if (set("username", "unclebob"))...`。將指令與查詢分開後，改善為
  ```non
  if (attributeExists("username")) {
      setAttribute("unclebob");
  }
  ```
- 用例外處理取代回傳錯誤代碼。定義錯誤代碼代表在鼓勵開發者使用 switch 判別錯誤狀況，用例外處理就能將邏輯抽離出來。 

第一個版本的程式碼一定是又長又複雜的，有很多巢狀邏輯、很長的參數串、隨意的命名、重複的程式碼，在寫完通過測試的函式後，再開始細細琢磨和改善程式碼，使其符合上面的準則，作者自己也無法第一次就寫出漂亮的程式碼。程式設計大師寫的程式就像是在訴說故事，透過良好的命名與漂亮的結構來描述系統的故事。

## Chapter 4: 註解

程式碼會被修改和演化，程式區塊會被搬來搬去，但註解無法每次都跟著一起移動，時間久了註解逐漸變得不準確，甚至會誤導開發者。寫註解的另一個動機是因為程式碼寫得太糟糕，必須用註解去解釋這個區塊做了什麼。

**真正有益的註解是想辦法不寫註解**，請盡可能透過良好的命名來表達程式碼的意圖，在大部分情況下你想寫下的註解都能融入到函式名稱中。

### 有意義的註解

- 法律型註解：`// Copyright (C) 2003 ... All rights reserved` 這樣的註解內如不該包含契約與法律條款，最好讓其參考標準的許可或外部文件。
- 資訊型註解：下面這個例子說明了一個抽象方法的回傳值，這樣的資訊有時非常有用:
  ```non
  // Return an instance of the Responder being tested.
  protected abstract Responder responderInstance();
  ```
  這個註解其實可以融入函式命名中：
  ```non
  protected abstract Responder responderBeingTested();
  ```
- 對意圖的解釋: 你可以了解程式設計師想做甚麼：
  ```non
  // This is our best attempt to get a race condition
  // by creating large number of threads.
  for (int i = 0; i < 2500; i++)
      // ...
  ```
- 闡明：有時候函式屬於標準函式庫不能改程式碼時，加上闡明的註解就有用處：
  ```non
  assertTrue(a.compareTo(a) == 0)    // a == a
  ```
  但是闡明註解有風險，你必須保證他的正確性。
- 對於後果的告誡：
  ```non
  // Don't run unless you have some time to kill
  public void _testWithReallyBigFile() {
      // ...
  }
- TODO：請定期審視、移除不必要的代辦事項，不要讓程式碼充斥一堆 TODO。

### 糟糕的註解

- 喃喃自語：如果某一段註解只有作者知道意義，其餘人看了還必須去翻其他部分的程式才能知道到底發生了什麼，乾脆就不要留下這條註解。
- 多餘的註解：如果程式碼表達的信息、意圖已經足夠明確，就不需要再用註解去解釋，說不定讀者理解註解的時間比理解程式碼的時間還要長。
- 誤導型註解：註解不夠精確，導致其他程式設計師誤用。
- 規定型註解：規定每個每個變數都要有註解。
- 日誌型註解：請用原始碼管控系統維護日誌。
- 干擾型註解：陳述很明顯的事實，又不提供任何新的資訊。
- 可怕的干擾：javadoc 等文檔生成器可能要求不必要的註解如：
  ```non
  /** The name. */
  private String name;
  ```
- 位置的標誌物：不要過度使用橫幅來區隔程式碼區塊如：
  ```non
  // Section 1 //////////////////////
  ```
- 右大括號後面的註解：有時候巢的層數太深，程式設計師會在右大括號後面寫：
  ```non
  try {
      while (1) {
      
      } // while
  } // try
  catch (Exception e) {
  
  } // catch
  ```
  請先嘗試簡短函式來取代這種註解。
- 出處及署名：請用原始碼管控系統代替 `/* Add by Rick */` 這種註解。
- 被註解的程式碼：原始碼管控系統會幫我們記住被刪除的程式碼，所以如果目前不需要該區塊就放心的刪除吧。
- HTML 形式的註解：HTML 標籤應該是文件工具的責任，不該出現在註解中
- 非區域性的資訊：請確保註解只會用來描述附近的程式碼。
- 過多的資訊：不要把討論的細節或演算法的細節放在註解中。

## Chapter 5: 編排
