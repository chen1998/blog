---
title: ssh using private key instead of enter pasword
date: 2016-08-04 20:50:08
tags: [ssh]
categories: Linux相關
---

在 ssh 的時候，屢次輸入帳號密碼實屬浪費生命之事，

如果可以把這件事情簡化的話一定很棒吧！

<!-- more -->

今天學到了這些 `config ssh` 的技巧，

如果能早些學會就能夠省掉好多時間了！


#### 在這篇文章大致會提及到
1. 生成公私鑰
2. 把公鑰放到另一台主機上
3. 設定 `/etc/ssh/ssh_config` 節省登入時間


#### 生成公私鑰

```bash
$ ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): /root/.ssh/id_rsa
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
80:77:9d:ea:e6:4c:72:07:f6:92:6f:43:75:33:87:83 root@ubuntu
The key's randomart image is:
+--[ RSA 2048]----+
|                 |
|     .   . .     |
|    . o . o  . . |
|     . o .  E * .|
|        S  . . = |
|       o +.      |
|      . B.o      |
|       B +o      |
|        o...     |
+-----------------+
```

至於在 `/root/.ssh/id_rsa` 處可以改成任意名稱，
在 `/root/.ssh` 之下會產生兩個檔案：`id_rsa`, `id_rsa.pub`

這邊的 `id_rsa.pub` 是公鑰，`id_rsa` 則是私鑰，
請務必保管好自己的 private key。


#### 把公鑰放到另一台主機上

先把公鑰複製到該台主機當中
```bash
$ scp ~/.ssh/id_rsa.pub root@remote_ip:/tmp/.
```

並對該台主機執行
```bash
$ cat /tmp/id_rsa.pub >> ~/.ssh/authorized_keys
```

這樣子就成功了！


#### 設定本機的 `ssh_config`

如果沒有設定的話，也可以使用以下指令登入
```bash
ssh -i ~/.ssh/id_rsa root@remote_ip
```

不過設定過後將會更簡單更方便：
開啟 `/etc/ssh/ssh_config`，設定說明如下
```
Host myhost                              # 幫他取個名字
    Hostname remote_ip                   # 機器的 IP
    PreferredAuthentications publickey   # 偏好用 publickey 驗證
    IdentityFile ~/.ssh/id_rsa           # privatekey 的路徑
    Port 22                              # port number
    User root                            # 使用者名稱
```

這樣子就可以用 `ssh myhost` 來登入了！

不過如果先前有設定 `passphrase` 的話，在登入的時候會需要輸入哦。

