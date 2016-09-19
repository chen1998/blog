---
title: ROPgadget - fail to load the dynamic library
date: 2016-06-19 11:58:45
tags: [python, reversing]
categories: 學習心得
---

在使用 ROPgadget 的時候出現了 `fail to load the dynamic library`

<!-- more -->

```
$ ROPgadget --binary level2
Traceback (most recent call last):
  File "/usr/local/bin/ROPgadget", line 15, in <module>
    import ropgadget
  File "/usr/local/lib/python2.7/dist-packages/ropgadget/__init__.py", line 14, in <module>
    import ropgadget.binary
  File "/usr/local/lib/python2.7/dist-packages/ropgadget/binary.py", line 13, in <module>
    from ropgadget.loaders.elf       import *
  File "/usr/local/lib/python2.7/dist-packages/ropgadget/loaders/__init__.py", line 13, in <module>
    import ropgadget.loaders.elf
  File "/usr/local/lib/python2.7/dist-packages/ropgadget/loaders/elf.py", line 13, in <module>
    from capstone   import *
  File "/usr/local/lib/python2.7/dist-packages/capstone/__init__.py", line 1, in <module>
    from capstone import Cs, CsError, cs_disasm_quick, cs_disasm_lite, cs_version, cs_support, version_bind, debug, CS_API_MAJOR, CS_API_MINOR, C
S_ARCH_ARM, CS_ARCH_ARM64, CS_ARCH_MIPS, CS_ARCH_X86, CS_ARCH_PPC, CS_ARCH_ALL, CS_MODE_LITTLE_ENDIAN, CS_MODE_ARM, CS_MODE_THUMB, CS_OPT_SYNTAX,
 CS_OPT_SYNTAX_DEFAULT, CS_OPT_SYNTAX_INTEL, CS_OPT_SYNTAX_ATT, CS_OPT_SYNTAX_NOREGNAME, CS_OPT_DETAIL, CS_OPT_ON, CS_OPT_OFF, CS_MODE_16, CS_MOD
E_32, CS_MODE_64, CS_MODE_BIG_ENDIAN, CS_MODE_MICRO, CS_MODE_N64, CS_SUPPORT_DIET
  File "/usr/local/lib/python2.7/dist-packages/capstone/capstone.py", line 162, in <module>
    raise ImportError("ERROR: fail to load the dynamic library.")
ImportError: ERROR: fail to load the dynamic library.
```

原本查了資料以為是版本問題，所以我用了
```
pip install -U capstone
```
來更新 capstone 到 `capstone-3.0.4`，但是問題依然存在，

```
$ ROPgadget --binary level2
Traceback (most recent call last):
  File "/usr/local/bin/ROPgadget", line 15, in <module>
    import ropgadget
  File "/usr/local/lib/python2.7/dist-packages/ropgadget/__init__.py", line 14, in <module>
    import ropgadget.binary
  File "/usr/local/lib/python2.7/dist-packages/ropgadget/binary.py", line 13, in <module>
    from ropgadget.loaders.elf       import *
  File "/usr/local/lib/python2.7/dist-packages/ropgadget/loaders/__init__.py", line 13, in <module>
    import ropgadget.loaders.elf
  File "/usr/local/lib/python2.7/dist-packages/ropgadget/loaders/elf.py", line 13, in <module>
    from capstone   import *
  File "/usr/local/lib/python2.7/dist-packages/capstone/__init__.py", line 230, in <module>
    raise ImportError("ERROR: fail to load the dynamic library.")
ImportError: ERROR: fail to load the dynamic library.
```

後來找到有人在 capstone 的 Github 開了 [issue](https://github.com/aquynh/capstone/issues/413)

一看才發現 `libcapstone.so` 被放到的路徑和預期的不太一樣，
因此我就用了 
```
cp /usr/local/lib/python2.7/dist-packages/usr/lib/python2.7/dist-packages/capstone/libcapstone.so /usr/local/lib/python2.7/dist-packages/capsto
ne/.
```
做手動修復，這樣子就可以 work 了，

或者參考 [https://github.com/aquynh/capstone/pull/589](https://github.com/aquynh/capstone/pull/589)
從 source code 來 install capstone。




