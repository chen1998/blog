---
title: Spanning Tree Protocol 重點整理筆記
tags: [network, stp]
date: 2016-05-31 08:48:37
categories: 閱讀筆記
---

### Spanning Tree Protocol Steps

##### 1\. 選舉出 Root Bridge

*   只有一個 Bridge 可以做 `Root Bridge`

<!-- more -->
*   Root Bridge 的 port 都會是 `Designated Port`

*   Designated Port 通常只會在 `Forwarding State`


    (Forwarding State 表示可以收發流量)

##### 2\. 在 Non-Root Bridge 的 Port 之中選出 Root Port

*   每個 Non-Root Bridge 上都有一個 Root Port
*   Root Port 通常會在 `Forwarding State`

*   Spanning Tree 會計算 Path Cost，計算方式是利用 bandwidth
*   Root Port 是 Non-Root Bridge 距離 Root Bridge 最短（Cost 最小）的 Port

##### 3\. 每一段 Link 都只能有一個 Designated Port

*   選擇的規則：Designated Port 會長在離 Root Bridge 有最低 cost 的 path 的 Switch 上面
*   Non-Designated Port 通常是 Block State，這個能夠避免迴圈的出現

### Port Role

##### Root Port

*   只會存在於 `Non-root Bridge` 上面，而且是距離 Root Bridge 有最小 cost 的 Port

*   每一個 Bridge 只會有一個 Root Port

##### Designated Port

*   這種 Port 存在於 Root Bridge 及 Non-root Bridge 中

        *   在 Root Bridge 當中，所有 Port 都是 Designated Port
    *   每一條 Link 只能有一個 Designated Port

##### Non Designated Port

不會轉送 data frames，也不會紀錄封包來源的 mac address 進 mac address table

##### Disabled Port

相當於把 Switch Port Shutdown

### 五種 Port Status

##### Blocking

NonDesignated Port 不參與 frame forwarding，Port 會收到 BPDU

`BPDU決定了每個 Port 的 Role`

##### Listening

STP 已經根據 BPDU 決定某些 Port 可以參與 Frame Forwarding，在這個時候，Port 不只能夠接受 BPDU，也會送出自己的 BPDU 給鄰近要參與 STP 的 Switch

##### Learning

Layer 2 的 Port 已經準備要參與 Frame Forwarding 且產生 Content-addressable memory(CAM) table

##### Forwarding

Layer 2 的 Port 已經是拓樸的一個部分，他除了轉送封包之外也會收發 BPDU

##### Disable

Layer 2 的 Port 不參與 STP 也不會轉送封包
