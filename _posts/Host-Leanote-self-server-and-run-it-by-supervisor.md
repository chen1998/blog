---
title: Host Leanote self-server and run it by supervisor
date: 2016-09-21 12:05:34
tags: [linux, apache, supervisor]
categories: Linux相關
---

此篇僅紀錄安裝時 Leanote 所碰到的問題與安裝流程及搭配 `supervisor` 及 `apache proxy` 心得，
<!-- more -->
更詳細之教學可參考 [官方文件](https://github.com/leanote/leanote/wiki/Leanote-源码版详细安装教程----Mac-and-Linux)

Environment: 
* Ubuntu 14.04.1 LTS
* go version go1.6 linux/amd64
* supervisor version 3.3.1

## Install Dependency

### Get go First

```bash
wget http://www.golangtc.com/static/go/1.6/go1.6.linux-amd64.tar.gz
tar zxvf go1.6.linux-amd64.tar.gz
# 要存 go 的位置，請根據情況自行修改，在此會這樣設定是因為我平常不使用 go，是因為 leanote 才需要
mkdir /home/user/leanote/go
mkdir /home/user/leanote/gopackage
# 創建資料夾後請把剛剛解包的內容放到 `go` 目錄之下
```

在官方文件所提的配置環境變數，因為我使用 zsh ，所以我是直接於 `~/.zshrc` 內新增。
```bash
export GOROOT=/home/user/leanote/go
export GOPATH=/home/user/leanote/gopackage
export PATH=$PATH:$GOROOT/bin:$GOPATH/bin
```

### Get revel and leanote

```bash
wget https://github.com/leanote/leanote-all/archive/master.zip
unzip master.zip
# 並將解壓後資料夾中的 src 複製到 gopackage 內
go install github.com/revel/cmd/revel
# 上述這一行會使用到前面環境變數的路徑，所以如果出錯請檢查是不是放錯位置了
```

### Install mongodb

這個部分與官方教學相同，請參閱官方文件操作。

### Install supervisor

除了以下這個方法，我也嘗試過用 apt-get 來安裝，只是在 apt-get 安裝的版本號是 3.0b2，這個版本的 supervisor 在環境變數設定時會有些問題，會沒辦法把現在系統的環境變數帶到 supervisor 裡。

```bash
pip install supervisor
```

## Build up

這個時候應該資料庫也匯入好了，不過請務必記得修改 `app.secret` 這個密鑰哦！  

另外還需要修改一下資料庫管理員的帳號、密碼、信箱，我是直接從資料庫下指令改的，網頁端好像可以修改，請從網頁試看看或是參考以下資料庫指令。  

不過我沒使用過 mongodb，以下方式也是找資料的 XD 可能做法沒這麼漂亮

```sql
# 選擇操作的資料庫
> use leanote
# 更新 admin 的密碼，填入 md5-hash 值
> db.users.update({Username : 'admin'}, {$set : {Pwd : 'md5-hash'}})
```

### supervisor configuration

我們需要啟動服務有兩條指令：
```bash
mongod --dbpath /home/user/leanote/data # 啟動資料庫服務
revel run github.com/leanote/leanote
```

所以接下來要做的事情便是使用 supervisor 來自動啟動服務，首先先在 `/etc/supervisor/conf.d` 新增一個檔案，名稱隨意，我使用的是 `leanote.conf`，並加入以下內容：
```
[program:mongod]
command=/home/user/leanote/mongodb/bin/mongod --dbpath /root/aweimeow.tw/note/data
user=root
autorestart=true

[program:leanote]
command=/home/user/leanote/gopackage/bin/revel run github.com/leanote/leanote
user=root
autorestart=true
```
接下來是環境變數的設定，開啟 `/etc/supervisor/supervisord.conf` 修改內容，只需修改 supervisord 這塊：
```
[supervisord]
environment=GOPATH="/home/user/leanote/gopackage", GOROOT="/home/user/leanote/go"
logfile=/var/log/supervisor/supervisord.log ; (main log file;default $CWD/supervisord.log)
pidfile=/var/run/supervisord.pid ; (supervisord pidfile;default supervisord.pid)
childlogdir=/var/log/supervisor            ; ('AUTO' child log dir, default $TEMP)
```

### Apache Proxy Pass & Proxy

```
<VirtualHost *:443>
    ServerName note.your.domain
    ServerAdmin webmaster@localhost

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined

    SSLCertificateFile /cert/file/locate
    SSLCertificateKeyFile /cert/file/locate
    SSLCertificateChainFile /cert/file/locate

    LoadModule proxy_connect_module /usr/lib/apache2/modules/mod_proxy_connect.so
    LoadModule proxy_ftp_module /usr/lib/apache2/modules/mod_proxy_ftp.so
    LoadModule proxy_http_module /usr/lib/apache2/modules/mod_proxy_http.so

    SSLEngine on
    SSLProxyEngine On
    SSLProtocol all -SSLv3
    ProxyRequests Off
    ProxyPreserveHost On
    ProxyPass / http://localhost:9000/
    ProxyPassReverse / http://localhost:9000/
</VirtualHost>
```

設定之後就能夠把 `note.your.domain` 重導向至開在 `localhost:9000` 上的 leanote 了！

