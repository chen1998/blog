---
title: config https on nginx use letsencrypt
date: 2016-08-17 09:27:32
tags: [linux, nginx, https, ssl]
categories: Linux相關
thumbnailImage: https://imgur.com/OdEK1UL.jpg
thumbnailImagePosition: right
---

最近和 g0v 的夥伴一起把 [itaigi](https://github.com/g0v/itaigi) 的後端移到新機器上，

雖然有設定過 https，但是因為不熟 nginx 讓我弄了好一段時間 XD

謹以此文來記錄這一次學習到的心得

<!-- more -->

#### 這篇文章會提及的有 ...
1. 申請 letsencrypt SSL 憑證
2. 設定 nginx
3. 其他紀錄：nginx 設定 proxy 方式 
 

備註：本篇的環境：nginx on Ubuntu 14.04.5 LTS

#### 申請 letsencrypt SSL 憑證

根據 [let's encrypt](https://letsencrypt.org/) 的網站導引會來到 [certbot](https://certbot.eff.org/)

![certbot](https://imgur.com/OdEK1UL.jpg)

選擇好自己的 服務/作業系統 後就可以得到一份很簡單的 Guide，

*   安裝
    ```
    wget https://dl.eff.org/certbot-auto
    chmod a+x certbot-auto
    ```

*   執行
    ```
    $ ./certbot-auto
    ```
    ```
    ./path/to/certbot-auto certonly
    ```
    ```
    ./path/to/certbot-auto certonly --webroot -w /root-path -d example.com -d www.example.com
    ```

不過因為我只是要憑證而已，所以就只有用 `certonly` 這個 Flag，
需要注意的是：憑證都是跟著 domain 的，所以你必須要先有一個 domain，
執行的過程中 `certbot` 會在機器上開一個 web server，
所以也必須先把機器本身的 web server 服務關掉
```
sudo service nginx stop
```

這個步驟都很簡單，只需要跟隨 `certbot` 的引導即可完成，
完成後會產生四個檔案在 `/etc/letsencrypt/live/<your domain>/.` 之下
會有這四個檔案：

|File Name     |Description                 |
|--------------|----------------------------|
|cert.pem      |server cert only            |
|chain.pem     |intermediates               |
|fullchain.pem |server cert + intermediates |
|privkey.pem   |private key                 |

而 nginx 需要的則是 `fullchain.pem` 與 `privkey.pem`

#### 設定 nginx

接下來我們要把資訊寫到 nginx 的設定檔裡面，

首先我們可以在 `/etc/nginx` 開一個叫做 `ssl` 的資料夾，用以存放憑證

```
sudo mkdir /etc/nginx/ssl
cp /etc/letsencrypt/live/<your domain>/fullchain.pem /etc/nginx/ssl/.
cp /etc/letsencrypt/live/<your domain>/privkey.pem /etc/nginx/ssl/.
```

並且於 `/etc/nginx/sites-available/default` 這個設定檔中設定 https

```
server {
        # 服務監聽在 443 port (https default port)
        listen 443 default_server;
        listen [::]:443 default_server ipv6only=on;

        # 填入 domain name
        server_name your_domain_here;

        # 這三行用以啟用 ssl 並填入憑證路徑
        ssl on;
        ssl_certificate /etc/nginx/ssl/fullchain.pem;
        ssl_certificate_key /etc/nginx/ssl/privkey.pem;

        location / {
            # your config here
        }
}
```

設定完後再重啟服務就完成了！

#### 其他紀錄：nginx 設定 proxy 方式

在處理這件事情的時候，和一位夥伴學到了一些有趣的東西，
因為我們要搬移機器，在 DNS record 尚未 update 的時候，
後端的 IP 是指向舊的機器，但是服務總不能因為這樣而停歇吧！ 

因此使用了 `upstream`：
```
upstream real_server {
    server <real server ip>:<port> fail_timeout=0;
}

...
server {
    ...
    location / {
        ...
        proxy_pass http://real_server;
    }
    ...
}
```

這樣就能把流量導到正確的 IP 上了！

在 DNS record 還沒更新的時候真的很方便 XD
