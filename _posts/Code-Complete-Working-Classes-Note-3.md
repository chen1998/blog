---
title: 'Code Complete: Working Classes Note - 3'
date: 2016-09-14 16:09:10
tags: [Code Complete]
categories: 閱讀筆記
---

在開發高品質程式時，定義好的類別與介面非常重要，且 `internal class` 的設計與實作也是如此重要。  

此節重點探討關於 `containment`, `inheritance`, `member functions and data`, `clas coupling`, `constructors`, `value vs. reference objects` 等議題。

<!-- more -->

## 前言

Chapter: 6.3 Design and Implementation Issues  

這份筆記是在閱讀 Code Complete 時，閱讀時認為重要的重點節錄，
文中範例皆來自 Code Complete 書中。  

如果您在閱讀時發現有錯誤的地方，請來信告知我，謝謝！

## Containment ("has a" Relationships)

如同標題所言，所謂的 `Containment` 便是指這個類別所 `擁有 ("has a")` 的屬性。  

就像是：有一個員工類別，它 `"擁有"` 名字、`"擁有"` 電話號碼 ... 等屬性。  

不過在設計類別時，請注意類別不要擁有過多的屬性，而 7 個是建議的數量，在 7±2 範圍內是較好的，不僅容易被使用者記住，也容易出錯。因此，`如果發現設計時類別的屬性數量太多，考慮看看要不要把它切分為小的類別吧`！

## Inheritance ("is a" replationships)

[繼承 (Inheritance)](https://goo.gl/bZLxFa) 是一個物件導向的重要概念，目的是能夠創建一個較簡單的 `基礎類別 (base class)` 後，再創造出子類別繼承自此基礎類別，這個子類別他會擁有來自於父類別的屬性資料。  

當你要使用繼承的時候，你要考慮幾個重點：
* 對於所有 `routine` 而言，有哪些是會被繼承的 (public or private)
* 被繼承的 `routine` 會不會有 [`default implementation`](http://stackoverflow.com/questions/18286235/what-is-the-default-implementation-of-method-defined-in-an-interface)
* `default implementation` 會不會被覆寫 (override)
* 對於每一個 `data member (變數、常數、列舉 ... 等)` 會不會被繼承。

補充：[`列舉 (enumeration) 的 Wiki 說明`](https://en.wikipedia.org/wiki/Enumeration)

### Design and document for inheritance or prohibit it

{% blockquote Joshua Bloch %}
Design and document for inheritance, or prohibit it.
{% endblockquote %}

如果要使類別能夠被繼承的話，那就必須妥善設計並以文件註解，否則就禁止類別被繼承。在 C++ 是 `non-virtual`、Java 是 `final`、而 Visual Basic 是 `non-overridable`，如此一來類別就沒辦法被繼承了。

### Adhere to the Liskov Substitution Principle

在讀這個章節之前，請先看過這篇，[關於 LSP 的說明](http://teddy-chen-tw.blogspot.tw/2012/01/4.html)，  

在書中舉的例子是：  
如果現在有一個基礎類別 `Account`，且有三個子類別：`CheckingAccount`、`SavingsAccount`、`AutoLoanAccount`，在這些類別當中都有一個 routine: `InterestRate()`，  
在呼叫 `CheckingAccount` 及 `SavingsAccount` 的 `InterestRate()` 會回傳利率。但是在呼叫 `AutoLoanAccount` 的 `InterestRate()` 卻是回傳使用者應繳還款的利息。  

根據 `Liskov Substitution Principle`，`AutoLoanAccount` 不應繼承自 `Account` 這個基類別，因為 `InterestRate()` 這個 routine 的語意 (semantic) 和 `AutoLoanAccount` 不同了，所以在此不應使用 `Account` 作為 `AutoLoanAccount` 的基礎類別。  

### Be suspicious of base classes of which there is only one derived class

在出現某個基礎類別只有一個子類別時，那就要注意了，通常這種狀況會不會是設計者的設計太超前了呢？  

也就是說，他對於未來的可能發展沒有足夠的認知了解，因此對未來的可能性而做過多的保留，「可能有一天會被用到」的這種想法其實並不好，多了一層 base class 也是沒有必要的。  

正確的做法是讓現在看起來清楚，直擊中心且簡單會比較妥當，在非必要的情況下才這麼做，否則不須有太多的繼承會比較好。

### Be suspicious of classes that override a routine and do nothing inside the derived routine

要注意不要覆寫 (override) 一個 routine，結果子類別的此 routine 卻不做任何事情。  

在文中的例子是有個 `class Cat`，且它有一個 `routine Scratch()`，不過接下來有個子類別繼承自 `Cat`，這隻貓的爪子被拔掉了，因此也沒辦法做 `抓` 的動作。  

那麼子類別 `ScractchlessCat` 的 `Scratch()` 怎麼辦呢？難道要覆寫它並不做任何事情嗎？相信你也一定會覺得這個做法好像不太對，這個做法有一些問題：  
* 違反了抽象化的 `Class Cat` 中 `Scractch` 的原意
* 當你要在產生其他繼承自 `Class ScractchlessCat` 的子類別時，會漸漸地發現失控了：
    * 如果這隻貓沒有尾巴呢？
    * 或是這隻貓也抓不了老鼠呢？
    * 還是說這隻貓也不喝牛奶？

我們總不可能命名一個 Class 叫做 `ScratchlessTaillessMicelessMilklessCat` 吧？  

所以說應該如何去解決這個問題呢？  
問題的源頭在於貓的基礎類別 `Class Cat`，它假定了貓一定會抓，所以從這邊著手解決問題吧！

### Avoid deep inheritance trees

盡力去避免繼承樹長得太深，換句話說就是盡力避免去繼承繼承再繼承，這樣會使程式越來越複雜。根據書中所言，人們能夠接受的層數是 2 ~ 3 層，太多層會導致人腦無法接受，進而不容易去 debug 找到問題。

## Summary for when to use inheritance or containment

* 如果多個 class 共用資料而不共用行為 (routine)
    * 你需要一個 object 容納這些資料並供 class 使用
* 如果多個 class 共用行為而不共用資料
    * 建立一個基礎類別供 class 繼承並定義 routine 行為
* 如果多個 class 共用行為且共用資料
    * 建立一個已定義好行為和資料的基礎類別並供 class 繼承

{% blockquote %}
Inherit when you want the base class to control your interface; contain when you want to control your interface.
{% endblockquote %}

## Member Function and Data

以下是一些繼承 member function 和 member data 的守則

### Keep the number of routines in a class as small as possible

太多的 routine 也伴隨著高錯誤率

### Disallow implicitly generated member functions and operators you don't want

有時候你想禁止某些 function 被調用，那麼在這種狀況你可以用宣告修飾字 (decorator) `private` 的方式來禁止。

### Minimize the number of different routines called by a class

隨著 routine 被 class 的內部呼叫增多，錯誤率也會提高

### Minimize indirect routine calls to other classes

上一點所說的行為被稱為 `direct connections`，這就已經夠危險 (夠容易出錯了)，現在提到的是被其他 class 呼叫 routine 的行為 - `indirect connections`。  

就像 `account.ContactPerson().DaytimeContactInfo().PhoneNumber()`，  

再更清楚的例子是：如果 ObjectA 創建了 ObjectB 的實例，ObjectA 當然能夠直接呼叫 ObjectB 的任何 routine，但是這應該要被極力避免。`account.ContactPerson()` 是可以的，但 `account.ContactPerson().DaytimeContactInfo()` 就是不好的了。

## Constructors

接下來就是一些關於建構子（構造函數）的守則了


### Initialize all member data in all constructors, if possible

盡可能在建構子內將所有資料都初始化，這也是 [Defensive Programming](https://en.wikipedia.org/wiki/Defensive_programming) 的一種。

### Enforce the singleton property by using a private constructor

如果你希望你所寫的 class 只能產生出一個實例的話，你可以以 `private` 修飾字來修飾建構子，透過把建構子藏起來後，再提供一個 `static` 的 `GetInstance()` routine，像是以下範例：

```java
public class MaxId {
    // constructors and destructors
    private MaxId() {
        ...
    }
    ...

    // public routines
    public static MaxId GetInstance() {
        return m_instance;
    }
    ...
    
    // private members
    private static final MaxId m_instance = new MaxId();
    ...
}
```

