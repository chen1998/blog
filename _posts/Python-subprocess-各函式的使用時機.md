---
title: Python subprocess 各函式的使用時機
date: 2016-09-09 23:12:42
tags: [python, subprocess]
categories: Coding札記
---

有感於每次使用 subprocess 都要看好多資料，
因此我將 python doc 重點整理及各種情況下如何使用 subprocess 詳細記錄並解說。  

<!-- more -->

以下例子均在 **Ubuntu 14.04** & **python2** 操作。

## subprocess module introduction
* subprocess.call
* subprocess.check_call
* subprocess.check_output
* subprocess.Popen

### subprocess.call
> Run the command described by args. Wait for command to complete, then return the returncode attribute.

也就是說，我們的 code 會等到指令執行結束才回傳結果，用這個例子來試試看：

```python
subprocess.call(['sleep', '1'])
```

結果會在 1 秒後回傳 `0`，這個 `0` 有著特殊意義，他代表著 [return code](https://en.wikipedia.org/wiki/Exit_status)

這邊需要注意的是，`arg` 代入的參數是 `list` 型態，  
如果需要帶入字串的話，可以加上一個參數，像以下例子：
```python
subprocess.call('sleep 1', shell=True)
```
`shell=True` 表示會呼叫一個 `/bin/sh` 來執行這條指令，  
不過因為有 [Command Injection](https://www.owasp.org/index.php/Command_Injection) 的問題，所以並不是推薦的方法。

當然，你也可以使用這種方式來執行指令：
```python
# 利用 str 的 split 把 str 以空白做區隔切成一個 List
subprocess.call('sleep 1'.split())
```

### subprocess.check_call
> Run command with arguments. Wait for command to complete. If the return code was zero then return, otherwise raise [CalledProcessError](https://docs.python.org/2/library/subprocess.html#subprocess.CalledProcessError).

也就是說，如果 `return code` 為 `0` 才回傳 `return code`，否則拋出例外。  
通常 `return code` 為 `0` 表示正常執行，因為 `return code` 又稱為 `Exit Status`，  
如果不為零，則代表執行程式的錯誤代碼。  
這個 function 在不需執行結果，**只需要執行狀態的時候適用。**  

舉個例子：
```python
if subprocess.check_call(['ls']) == 0:
    print('Command Success Execute')
else:
    print('Oops, something error')
```

### subprocess.check_output
> Run command with arguments and return its output as a byte string.

這個則是很實用的 function 了，他會把輸出直接回傳給你，  
不過值得注意的一點是，與上一個 function 相同，如果 `return code` 不是 0 也會拋出例外。

舉個例子，先試著讓他拋出例外：
```python
try:
    output = subprocess.check_output('exit 1', shell=True)
except subprocess.CalledProcessError:
    print('Exception handled')
```

然後則是在 `return code` 是 0 的狀態：
```python
try:
    output = subprocess.check_output(['ls'])
except subprocess.CalledProcessError:
    print('Exception handled')
# 然後試著印出 output 吧
```

### subprocess.Popen
> Execute a child program in a new process. On Unix, the class uses os.execvp()-like behavior to execute the child program.

這是我最常使用的 function，因為有時候我會需要接收來自 `stderr` 的訊息。


舉個例子，先寫段 code 來達成一個簡易的功能，  
如果輸入過短則把錯誤訊息輸出到 `stderr`，  
否則便把 `'Hello name'` 輸出到 `stdout`
```python
#! /usr/bin/python
import sys

name = raw_input()
if len(name) < 3:
    print >> sys.stderr, "Name is too short"
else:
    print "Hello %s" % name

```

接下來試著使用 `subprocess` 調用這個程式吧！

首先是 input 長度正常的結果
```python
from subprocess import Popen, PIPE
subprocess.Popen(['python', 'test.py'], stdout=PIPE, stderr=PIPE, stdin=PIPE)
stdout, stderr = p.communicate(input='aweimeow\n')
# stdout = 'Hello aweimeow\n'
```

再來是 input 長度過短的結果
```python
from subprocess import Popen, PIPE
subprocess.Popen(['python', 'test.py'], stdout=PIPE, stderr=PIPE, stdin=PIPE)
stdout, stderr = p.communicate(input='aweimeow\n')
# stderr = 'Name is too short\n'
```

如此一來便可以輕鬆的接到 `stdout` 及 `stderr` 了！
