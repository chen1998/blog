---
title: 'Code Complete: Defensive Programming Note-1'
date: 2016-09-17 20:44:03
tags: [Code Complete]
categories: 閱讀筆記
---

Defensive Programming (防禦性程式設計) 就如同 defensive driving 一樣，對於其他駕駛抱持著懷疑的心態，小心翼翼的防範他們做出對自己有危險的舉動。此節將從過濾不合法的輸入開始，並帶到如何以 assert (斷言) 來撰寫程式。

<!-- more -->

## 前言

Chapter: 8.1 Protecting Your Program from Invalid Inputs
Chapter: 8.2 Assertions

這份筆記是在閱讀 Code Complete 時，閱讀時認為重要的重點節錄，以及我的心得筆記，文中範例皆來自 Code Complete 書中。  

如果您在閱讀時發現有錯誤的地方，請來信告知我，謝謝！

## 關於那些不合法的輸入

你一定有聽過 "Garbage in, garbage out" 吧，其實就是一種 買者自負[1] 的心態，讓使用者承擔使用時的風險。  

很顯然這是不夠好的，在我們設計時，比較好的方法應該是 `garbage in, nothing out`, `garbage in, error message out`, `no garbage allowed in`。

通常有以下三種方法來處理所謂的 Garbage in:

### Check the values of all data from external source

只要資料是來自於: 檔案、使用者、網路、其他外部介面 ... 等，  
就必須做適當的檢查，如：檢查數字在許可範圍內、字串的長度不會過長導致無法處理、檔案不會大到無法解析。  
當然還有其他必要的檢查，如：[Buffer Overflow](https://en.wikipedia.org/wiki/Buffer_overflow) (緩衝區溢位)、[SQL Injection](https://en.wikipedia.org/wiki/SQL_injection) (資料庫隱碼攻擊)、[Cross Site Scripting](https://en.wikipedia.org/wiki/Cross-site_scripting) (XSS) ... 等。  

＊以上連結皆取自 Wikipedia，亦可參閱 OWASP 的說明

### Check the values of all routine input parameters

檢查傳入 routine 的參數也是很重要的一件事情，這就與上一點檢查外部來源的資料一樣重要。

### Decide how to handle bad inputs

一旦你發現有非法的參數傳入，你會怎麼做、你該怎麼做？根據不同的情形，你有一堆處理方式可以選擇，這個在 `8.3 Error-Handling Techniques` 會進一步說明。

## 斷言 (Assertion)

assert 是開發時十分經常使用到的技術，例如用來確定使用者提供的檔案不超過 50,000 行、確定敘述式的布林值回傳結果是 True ... 等。  

在 Java 當中可參酌以下範例使用：

```java
assert denominator != 0
// 確定 denominator 變數不會超出預期恰巧等於 0
```

有很多時候都可以用上 assert，例如想要確認 ... ：
* 輸入的值有落在預期的範圍之內
* 某一個 file 或 stream 是開啟或關閉的（以防讀寫資料發生 IOError）
* file 或 stream 的讀取位置在檔案的開始
* file 或 stream 的開啟模式，如 `read-only`, `write-only`, `both read and write` ... 等
* 僅限輸入的變數不在 routine 的執行過程中遭到變更
* pointer (指標) 為非 NULL
* 傳入陣列之類容器的內容數量有達到至少某一個數量
* 確認容器內已經有初始化資料
* 當 routine 執行的開始（或結尾），某一個容器是空的

不過這些都只是一些常見的用法，通常 assertion 所拋出的錯誤訊息不會在發行版的程式碼中見到，assert 通常都是給開發維護時使用的。

## 使用 assert 的 guideline (指南)

### Use error-handling code for conditions you expect to occur; use assertions for conditions that should never occur

斷言是被用來檢查 `應該不可能會發生的情況`，而在你預期這段程式碼會發生一些特例狀況時，你該使用的是 error-handling 而不是 assert。更直白的說法是，Error Handling 用來檢查使用者錯誤的輸入資料，而斷言用來檢查程式執行時會發生的 Bug。  

Error-Handling Code 是用來處理異常的情況，而且它也會以優雅的方式來做出對應的回應，但如果這時由 assert 介入，讓 assert 去把這些異常的狀況都處理掉，那Error-Handling 就沒辦法處理這些問題了，事到如今，也僅能夠修改原始碼重新編譯過程式。  

以另一個角度來思考，讓我們把斷言當作是可執行的說明文件 (executable documentation)，你不能奢望程式碼靠著它們運作，但是它們用更具體的方式將程式的 input 條件來展示給你看，這甚至比註解來得更有效、更易理解。

### Avoid putting executable code into assertions

切記不要把程式碼放在 assert 裡，因為在 compile 程式的時候，如果把 assert 關掉，這行程式碼就會被 pass 掉了。

```python
# Assume that DoSomething will return 0 when everything is okay
assert DoSomething(), 0
```

像以上這個例子，如果在 `python -O` 時，就會無視掉 `assert`，如此一來程式碼就不能被執行到了。  

正確的做法應該是：

```python
result = DoSomething()
assert result, 0
```

### Use assertions to document and verify preconditions and postconditions

用 assert 作為文件並驗證在程式碼執行前後的變數、資料狀態

```python
age = int(raw_input("Tell Me Your Age: "))
pre_age = age
assert age > 0 and age < 100
AfterOneYear()
assert age == pre_age + 1
```

### For highly robust code, assert and then handle the error anyway

在任何 error condition (錯誤的狀況中)，routine 只會使用 assertion 或 error-handling 其中之一，決不會同時使用。  

不過在大型專案當中卻不太一樣，程式設計師的時間被切碎去維護各個版本的程式碼，因此在大型專案的開發團隊，程式設計師會將他們的 Code 仔細透徹的 Review 過一遍，甚至寫 Unit Test 來防止出錯。  

因此在這種情況，很可能同個錯誤情形會有 Assertion 和 Error-Handling 同時處理它。


## 註記

[1] 買者自負 買家購入的資產（通常指物業）不論好壞，均要自己負責。
