---
title: 'Code Complete: Working Classes Note - 1'
date: 2016-09-08 23:47:16
tags: [Code Complete]
categories: 閱讀筆記
---

這份筆記是在閱讀 Code Complete 時，閱讀時認為重要的重點節錄，  
文中範例皆來自 Code Complete 書中。

<!-- more -->

如果您在閱讀時發現有錯誤的地方，請來信告知我，謝謝！

## 關於本節

章節名稱：6.1 Class Foundations: Abstract Data Types (ADTs)  

由 ADT（抽象化資料型態）開始了解 OOP(Object-Oriented Programing)

## 需要 ADTs 的時機

假設我們不使用 ADTs，那我們要把字型大小設定為 12
```python
currentFont.size = 16
```
這個看起來不太好閱讀，所以假設我們現在建立了 library routines 的集合之後，Code 會好讀一些
```python
currentFont.size = PointToPixels(12)
```
或者讓屬性名稱有點變化，增加它的閱讀性
```python
currentFont.sizeInPixels = PointToPixels(12)
```
需要注意到的是，我們不能同時有兩種屬性：`currentFont.sizeInPixels` 和 `currentFont.sizeInPoints`  
因為如此 `currentFont` 會不知道該以哪個為基準  

如果需要將字體設定為粗體的話：
```python
currentFont.attribute = currentFont.attribute or 0x02
```
在我們以別的方式來代替 `0x02` 之後
```python
currentFont.attribute = currentFont.attribute or BOLD
```
或者像是這樣子
```python
currentFont.bold = True
```

有沒有注意到呢？我們總是需要直接去修改屬性的值。  
接下來讓我們看看應該怎麼做比較好！

## 使用 ADTs 的好處
* 把實作的細節藏起來
  > 在前述的例子當中，不會因為需要把字體設定成粗體，而把程式碼重頭到尾改了遍，只需要使用 `routine` 就可以了。  
  
* 對於 Code 的更改不會影響到整個程式
  > 理由同上，你可以只改動程式碼的一個地方就完成修正。  

* 可以讓使用的介面更直覺（直觀易讀）
  > 以前面的例子來說：`currentFont.size = 16` 並未定義到底是 Pixels 還是 Points，所以容易使人造成誤解。  
  
* 要增進程式效率的話，使修改更簡單
  > 在想要更動演算法提升效率，只需要改動少數幾個地方，不需要牽動到原本的整個程式。  
  
* 更容易除錯，較容易看到錯在哪裡
  > `currentFont.attribute = currentFont.attribute or 0x02`
  > 如果被寫成 `0x20`，那就會更難找出問題的來源。  

* Code 即 Document
  > 以上一點的例子繼續說明，如果我們使用 ADTs
  > `currentFont.SetBoldOn()` 是不是一眼就知道意思了呢？  

* 在你的程式裡，不需要一直把 data 傳進去
  > 在 ADTs 中，Private Variable 也只能被 structure 的 routine 存取  

在這邊讓我們定義一個 abstract data type，  
來定義幾個 routines 控制字型吧！

```python
currentFont.SetSizeInPoints( sizeInPoints )
currentFont.SetSizeInPixels( sizeInPixels )
currentFont.SetBoldOn()
currentFont.SetBoldOff()
currentFont.SetItalicOn()
currentFont.SetItalicOff()
currentFont.SetTypeFace( faceName )
```

## 在非物件導向語言環境中以 ADTs 處理多重實例

延續上一小節所講的 ADTs 的 routines，  
在 Non-Object-Oriented 環境中會像是這樣：
```c
SetCurrentFontSize( sizeInPoints )
SetCurrentFontBoldOn()
SetCurrentFontBoldOff()
SetCurrentFontItalicOn()
SetCurrentFontItalicOff()
SetCurrentFontTypeFace( faceName )
```

這些 Function 不會被 attach 到 Class 之中，  
所以當我們不只有一個字體物件時，我們會需要額外的處理：
以下列例子來說，有個做為識別的 ID: `fontId`
```c
CreateFont( fontId )
DeleteFont( fontId )
SetCurrentFont( fontId )
```
也就是說，我們現在有幾個方法可以實現我們想完成的事情：
1. 在使用 ADTs service 時明確指定實例
  > 在我們每次要更改字型物件時，都需要把 fontId 傳入，而 fontId 也是 code 唯一可以維護物件的參數。  

2. 把物件作為參數傳入 ADTs service
  > 你需要創造一個 Font 資料型別，並在每次呼叫 ADTs service 時都要傳入，如此一來便不需要 fontId
  > 需要注意的是，儘管 Font 物件的屬性能直接修改，你仍應使用 ADT service 去存取他，keeping Font Structure "Closed"。  

3. 使用精心設計過的方式來使用實例
  > 設計一個新的 function: `SetCurrentFont(fontId)`，
  > 呼叫後所有其他函式被呼叫時都會修改此 Font 物件，
  > 不過值得注意的是，使用這個方法會讓複雜性激增，
  > 因為你在使用 Font Function 都須注意到是否在對正確的物件修改
