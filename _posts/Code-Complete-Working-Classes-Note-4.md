---
title: 'Code Complete: Working Classes Note - 4'
date: 2016-09-16 10:51:22
tags: [Code Complete]
categories: 閱讀筆記
---

此章節重點於描述為什麼我們需要創建 Class，以及提供一些撰寫時的建議、需要避免的事項。

<!-- more -->

## 前言

Chapter: 6.4 Reason to Create a Class
Chapter: 6.5 Language-Specific Issues

這份筆記是在閱讀 Code Complete 時，閱讀時認為重要的重點節錄，
文中範例皆來自 Code Complete 書中。  

如果您在閱讀時發現有錯誤的地方，請來信告知我，謝謝！

# 6.4 Reasons to Create a Class

## 創建類別的好理由

### Model real-world objects

把真實世界中每一個東西做成一個類別雖然不是唯一的理由，不過仍然是個好理由，  
將現實世界的每一個資料數據建立成物件，並創造對應的 routine。

### Model abstract objects

關於創建類別的另外一個好理由是做出抽象的物件，舉例來說 `Shape` 是 `Circle` 和 `Square` 的抽象類別，不過把這些抽象概念從真實物件當中抽離並不是那麼令人肯定的（也就是說，並沒辦法好好地確定有哪些屬性會被抽離出來，每一個設計者做出來的結果都不盡相同），所以在物件導向設計當中把物件的抽象化概念實作出來仍然是很大的挑戰。

### Reduce complexity

讓我們看看最簡單且最重要的原因，我們要減少程式的複雜度，創建類別能夠把複雜的實作細節藏起來，讓我們在使用時不需要了解其中的原理，僅僅需要使用就可以了。

### Isolate complexity

讓我們看看各種複雜的東西：複雜的演算法、龐大的資料結構、錯綜複雜的通訊協定 ... 等，  
這些全部都是容易導致錯誤的原因，當我們創建不同的類別把這些複雜的事情隔離開時，那就變得比較容易找到問題的來源了。  
況且，如果你發現演算法不夠優秀，想要以新研發的演算法替代的話，這種設計模式也讓你可以輕鬆替換演算法。

### Hide implementation details

把實作的細節藏起來的優點就不在此重提了，可以參考前述三個章節所言！

### Limit effects of changes

還有一個優點便是可以把修改程式碼所造成的影響限制或降低，而且在這種設計之下也會讓修改變得容易。

### Hide global data

如果需要使用全域變數，你也能把他的實作細節以類別物件包裝起來，這樣子做有幾個好處：
* 修改資料架構而不改動使用它的主程式
* 監控有誰存取了這些全域變數

### Make central points of control

把所有要執行的事項全部以一個類別包起來會是一個不錯的點子，  
例如檔案、資料庫存取、印表機 ... 等，用一個 class 來讀寫資料庫也是 `centralized control` 的一種  
如果現在不是存取資料庫，而是一個檔案 (flat file)、記憶體中的資料 (in-memory data)，那也只需要改變這個地方。

### Facilitate reusable code

程式碼如果被寫成一個好的 class 也能夠容易像模組一樣被其他專案重用，相反的，如果我們的程式碼被嵌在一個較大的 Class 當中，那就沒有那麼容易被重複使用。

## 應該避免撰寫諸如此類的類別

### Avoid creating god classes

不要寫出一個 Class 能去 `Get()` 或 `Set()` 任何其他 Class 的資料。

### Eliminate irrelevant classes

假如一個 Class 只有資料而沒有行為 (behavior / routine)，那你需要思考是不是真的需要這個 Class，或是將他拆解成多筆資料結構，存放於各個使用到他們的 Class 中即可。

### Avoid classes named after verbs

如果一個 Class 只有行為而沒有資料，那思考看看要不要將他轉換成 `DatabaseInitialization()` 之類的 routine 放進其他 Class 中囉！

# 6.5 Language-Specific Issues

在不同的語言當中，對於子類別能否覆寫 (override) 父類別的 routine 有不同的定義。在 Java 除非把 routine 以 `final` 修飾字宣告才會禁止 override，否則預設可以 override。在 C++ 又不同了，預設是不能夠 override 的，除非以 `virtual` 修飾。諸如此類的問題都是需要被注意到的，以下的列表列出在不同語言當中可能會有不同行為的項目：

* 在繼承時建構子與解構子被覆寫的行為
* 在例外處理時建構子和解構子的行為
* Importance of default constructors (這句我沒辦法理解，因此保留原文)
* 呼叫解構子 (destructor) 和 finalizer 的時機
    * 延伸閱讀：[In C# what is the difference between a destructor and a Finalize method in a class?](http://stackoverflow.com/questions/1076965/in-c-sharp-what-is-the-difference-between-a-destructor-and-a-finalize-method-in)
* 在處理物件時，記憶體的行為

