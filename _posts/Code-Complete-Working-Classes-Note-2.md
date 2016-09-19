---
title: 'Code Complete: Working Classes Note - 2'
date: 2016-09-12 11:27:24
tags: [Code Complete]
categories: 閱讀筆記
---

這份筆記是在閱讀 Code Complete 時，閱讀時認為重要的重點節錄，
文中範例皆來自 Code Complete 書中。  

<!-- more -->

如果您在閱讀時發現有錯誤的地方，請來信告知我，謝謝！

## Good Abstraction

簡單來說，抽象化就是以簡單的方式來做到複雜的操作，因此一個類別的介面也該把實作的細節以抽象化的方式藏起來。  

以書中的例子來說明抽象化應該怎麼做吧！  

以下例子是一個 `Employee` 類別，包含了姓名、地址、電話 ... 等員工資訊：

```c
class Employee {
public:
// public constructors and destructors
	Employee();
	Employee(
		FullName name,
		String address,
		String workPhone,
		String homePhone,
		TaxId taxIdNumber,
		JobClassification jobClass
	);
	virtual ~Employee();
	// public routines
	FullName GetName() const;
	String GetAddress() const;
	String GetWorkPhone() const;
	String GetHomePhone() const;
	TaxId GetTaxIdNumber() const;
	JobClassification GetJobClassification() const;
	...
private:
	...
};
```

所以說，有了 `GetName`, `GetAddress` 等 routine，使用者根本不需要知道他其中的細節，僅需要使用這個 routine 即可。  
至於不好的抽象化實作就不在此列出來，書中範例所展示的 poor abstraction 是所有的 routine 非常繁雜混亂，分不清楚用途及目的。

### Present a consistent level of abstraction in the class interface

有一個需要注意的重點是抽象化要保持在同一個層級，  
這樣子說還是有些抽象不好懂，舉個例子來說明的話便是：

在 `AddEmployee` 這個名字看來很明顯的是在 `Employee` 層，  
但 `NextItemInList` 卻是在以 `Employee` 為元素的陣列這一層。  
```c
public:
	void AddEmployee(Employee employee);
	...
	Employee NextItemInList();
	Employee FirstItem();
	...
```

### Provide service in pairs with their opposites

大部分的 Function 都是雙向的，舉個例子來說明：  
如果現在有一個 `AddEmployee` 的 routine，那相對的也就需要一個 `DelEmployee`，  
如果現在有個 `TurnLightOn`，那也就會有另一個叫做 `TurnLightOff`，  
在實作時，務必確定你的 service 是雙雙成對的！

### Move unrelated information to another class

有時你在實作時會發現這個 class 需要處理兩塊來源完全不同的資料，  
那也就意味著你必須要把他們拆分開成兩個 class 囉。

### Make interfaces programmatic rather than semantic when possible

這一段是我琢磨最久，卻又無法完全理解的地方，  
[語法和語義的差別可以在這邊找到](http://stackoverflow.com/questions/17930267/what-is-the-difference-between-syntax-and-semantics-of-programming-languages)，  
> Syntax is the concept that concerns itself only whether or not the sentence is valid for the grammar of the language . 
> Semantics is about whether or not the sentence has a valid meaning.

但是在文中的意思卻與這邊不太相同：  
> The programmatic part consists of
the data types and other attributes of the interface that can be enforced by the compiler.
The semantic part of the interface consists of the assumptions about how the interface
will be used, which cannot be enforced by the compiler.

作者舉了一個例子：  
假設 RoutineA 必須在 RoutineB 之前被呼叫，那麼 programmer 通常會寫在註解當中，  

但是在 subtitle 裡面可以看到，希望 interface 可以多點 programmatic 少點 semantic，  
具體來說應該要怎麼做呢？不如我們就不要寫註解了吧，  
讓我們使用 `Asserts` Statements 來確保 RoutineA 會 call before RoutineB！

### Beware of erosion of the interface's abstraction under modification

盡力避免在 `修改或維護` 之後，介面的抽象化變得越來越亂，
舉個例子：
```c
class Employee {
	public:
		...
		bool IsZipCodeValid(Address address);
		...
}
```
在這個例子當中，完全看不出來 `檢查 ZipCode` 和 `Employee Class` 之間的關聯，所以同上，把他切分開囉。

### Don't add public members that are inconsistent with the interface abstraction

不要把無關聯的 routine 加到 interface 裡面。

### Consider abstration and cohesion together

這個是什麼意思呢？讓我們看看[這篇](http://stackoverflow.com/questions/10830135/what-is-high-cohesion-and-how-to-use-it-make-it)  

> To create a high cohesive solution, you would have to create a class Window and a class Sum. The window will call Sum's method to get the result and display it. This way you will develop separately the logic and the GUI of your application.

在這個例子當中，如果開發出來的結果是：  
把兩個值相加並在同個 class 呼叫 Window 把結果顯示出來，  
這顯然是沒把 `使用者介面` 及 `運算邏輯` 分割開的，  
所以在設計時，應該同時把 `abstraction(抽象化)` 與 `cohesion(相關性)` 考慮進去，  
這也與先前 `Move unrelated information to another class` 互相呼應。

## Good Encapsulation

在講完如何寫出好的抽象化及所需注意的重點後，那麼接下來在乎的是怎麼去封裝。  

> two concepts are related because, without encapsulation, abstraction tends to break down. In my experience, either you have both abstraction and encapsulation or you have neither. There is no middle ground.

### Minimize accessibility of classes and members

這是在設計封裝時需要注意的大重點之一，如果不確定哪些 routine 要封裝的話，  
作者提供了一個準則：  
> If exposing the routine is consistent with the abstraction, it’s probably fine to expose it. 
If you’re not sure, hiding more is generally better than hiding less.

### Don't expose member data in public
舉個例子，現在在三維空間有個 `Point` Class，那我們 expose 點座標的值：
```c
float x;
float y;
float z;
```

更好的做法是把這些改成：
```c
float GetX();
float GetY();
float GetZ();
void SetX(float x);
void SetY(float y);
void SetZ(float z);
```

這樣子的做法有一個好處，你無法確定實作的細節，說不定 x, y, z 都是 double，只是回傳時轉換而已。

### Avoding putting private implementation details into a class's interface

在好的封裝當中，programmer 沒辦法再看到實作的細節，  
這些細節會被以 `容易理解的命名方式(figuratively and literally)` 藏起來，  

在書中舉了這一個例子，實作細節仍然會於 header file 被看到：
```c
class Employee {
	public:
		...
	private:
		String m_Name;
		String m_Address;
		int m_jobClass;
}
```

因此 Scott Meyer 在 Effective C++ 2d ed. 一書當中的 Item 34 提到解法：

```c
class Employee {
	public:
		...
	private:
		EmployeeImplementation *m_implementation;
}
```

如此一來就可以把實作細節藏在 `EmploueeImplementation class` 之中了。

#### Don't make assumption about the class's users

我們不應該去假設 interface 會不會被用到，比起假設，寫些註解在 interface 會比較好，  
例如這個例子當中，會讓使用者更明白怎麼去使用：
```
 initialize x, y and z to 1.0 because DerivedClass blows
 up if they're initialized to 0.0
```

#### Favor read-time convenience to write-time convenence

簡單來說，Code 被閱讀的時候遠遠多過於寫 Code 的時候，  
所以如果能不貪圖方便而寫出不易閱讀的 Code 會比較好。

#### Be very, very wary of semantic violations of encapsulation

要非常小心一些 semantic violations 的狀況：  
> 在 call RoutineB 之前，必須先 call RoutineA

這被我們稱為是 `semantic violation`。  

以下有一些例子，在使用者使用 class 時：  
1. 不要呼叫 Class A 的 `InitializeOperations()` 因為 `PerformFirstOperation()` 會自動呼叫它。
2. 不要在呼叫 `employee.Retrieve()` 之前先呼叫 `database.Connect()`，因為 `employee.Retrieve()` 在有 connection 時無法接收資料。
3. 不要呼叫 Class A 的 `Terminate()` 因為 Class A 的 `PerformFinalOperation()` 已經呼叫過它。
4. ...

在此我只列出書上的三點，這些問題的共通點都是沒有搞清楚要怎麼使用 Class。在你碰到這種狀況時，你應該直接去問 Class 的作者而不是去翻 Source Code，而作者也不應該直接回答你，而是把 interface 的文件修正，再請你看過是不是能夠理解應該怎麼使用了。  

這樣子做能夠避免下一個碰到這狀況的使用者再次重蹈覆轍，當你這麼做了之後，下一個使用者也許就不會碰到這種問題了！

