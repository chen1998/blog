---
title: Setting subdomain with apache
date: 2016-06-08 23:18:43
tags: [apache, linux]
categories: Linux相關
---

### 在設定 subdomain 的步驟

1. 到域名商設定 A record

<!-- more -->

2. 設定 apache 的 virtual host
3. 如果網站有 https，可能需要重新申請憑證


##### 前情提要

我的主機是 `aweimeow.tw`，然後我要再申請 `blog.aweimeow.tw` 來做為部落格，
所以才有以下的故事。

##### 設定 A record

我總共在我的 domain 供應商設定了兩筆 record

|主機名稱         |紀錄類型|IP            |
|-----------------|--------|--------------|
|aweimeow.tw      |A       |139.162.31.138|
|blog.aweimeow.tw |A       |139.162.31.138|


##### 設定 apache

這邊會需要把 VirtualHost 複製一份作為 `blog.aweimeow.tw` 的設定檔使用
複製完後，把設定檔改成 subdomain 的就可以了

以下有幾個需要修改的地方：
- ServerName: 改成 `blog.aweimeow.tw`
- DocumentRoot: 改成 subdomain 的根目錄
- SSL ... 等: 放置憑證的位置

```apache
<VirtualHost *:443>
        # The ServerName directive sets the request scheme, hostname and port that
        # the server uses to identify itself. This is used when creating
        # redirection URLs. In the context of virtual hosts, the ServerName
        # specifies what hostname must appear in the request's Host: header to
        # match this virtual host. For the default virtual host (this file) this
        # value is not decisive as it is used as a last resort host regardless.
        # However, you must set it for any further virtual host explicitly.
        ServerName aweimeow.tw

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www

SSLCertificateFile path/to/file/cert.pem
SSLCertificateKeyFile path/to/file/privkey.pem
Include path/to/file/options-ssl-apache.conf
SSLCertificateChainFile path/to/file/chain.pem
</VirtualHost>
```

##### 重新申請憑證

我的憑證是在 [Let's Encrypt](https://letsencrypt.org/) 的服務，
但是原本的憑證是給 `aweimeow.tw` 用的，所以要重新申請憑證，

在 letsencrypt 重新申請憑證的方式是：

```bash
./certbot-auto certonly --standalone -d blog.aweimeow.tw -d aweimeow.tw
```

這樣子就會重新發一個憑證給 `blog.aweimeow.tw` 跟 `aweimeow.tw` 用，
到這邊就大功告成了。

