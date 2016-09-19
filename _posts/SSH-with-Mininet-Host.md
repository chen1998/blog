---
title: SSH with Mininet Host
tags: [mininet, ssh, python, sdn]
date: 2016-05-04 21:24:52
earReading: true
thumbnailImage: https://i.imgur.com/nQtRA8c.png
thumbnailImagePosition: right
autoThumbnailImage: yes
metaAlignment: center
categories: Coding札記
---

#### Introduction

*   本篇會講述 sshd.py 的原理
*   Using Mininet v3.29

<!-- more -->
#### Topology

![](https://i.imgur.com/nQtRA8c.png)

#### Code

參考 Mininet Github example 的 [sshd.py](https://github.com/mininet/mininet/blob/master/examples/sshd.py)



讓我們先來看看核心程式碼：


```python
def connectToRootNS( network, switch, ip, routes ):
    """Connect hosts to root namespace via switch. Starts network.
      network: Mininet() network object
      switch: switch to connect to root namespace
      ip: IP address for root namespace node
      routes: host networks to route to"""
    # Create a node in root namespace and link to switch 0
    root = Node( 'root', inNamespace=False )
    intf = network.addLink( root, switch ).intf1
    root.setIP( ip, intf=intf )
    # Start network that now includes link to root namespace
    network.start()
    # Add routes from root ns to hosts
    for route in routes:
        root.cmd( 'route add -net ' + route + ' dev ' + str( intf ) )
```


這邊做了幾件事情，

1.  新增一個 Mininet Node，名為 root，且把 `inNamespace` 參數設定為 `False`

2.  將 root 與 switch（這裡是取 s1）連接起來
3.  把 root 的 IP 設定為 `10.123.123.1/32`

4.  設定機器的 Routing Table，把封包目的為 Mininet Host 的封包都由 root-eth0 代為送出

###### root = Node( 'root', inNamespace=False )

在 Mininet 創建 Node 時，如果沒有指定，`inNamespace` 預設為 `True`，

`inNamespace` 為 `True` 時，Mininet 會 NEW 出一個 Namespace，並將介面 attach 到該 Namespace 中，

所以在 `inNamespace` 為 `False` 時，會把介面 attach 到 root Namespace。

###### intf = network.addLink( root, switch ).intf1

將 root 與 s1 相連起來

###### root.setIP( ip, intf=intf )

設定 root-eth0 的 IP

###### root.cmd( 'route add -net ' + route + ' dev ' + str( intf ) )

前面之所以會寫是 "設定機器的 Routing Table"，

是因為 root 是被 attach 在 root namespace，

這個 Namespace 不是 h1-namespace 這類 Namespace，

而是在執行 Mininet 前就已經存在的 root namespace，

由下圖可以觀測到，root-eth0 與 eth0 都存在於 root namespace 之中：

![](https://i.imgur.com/sAPwALd.png)

而且在 Routing Table 也可以看到：

<figure class="figure-code code"><div class="highlight"><pre>root@ubuntu:~/mininet/examples# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         10.1.6.1        0.0.0.0         UG    0      0        0 eth0
10.0.0.0        0.0.0.0         255.255.255.0   U     0      0        0 root-eth0
10.1.6.0        0.0.0.0         255.255.255.0   U     0      0        0 eth0
</pre></div>
</figure>

因此，當我要 ping h1(10.0.0.1) 時，會經由 root-eth0 送給 h1，

###### 為甚麼 h1 能夠 reply 呢？

因為在這個拓樸中，h1 = `10.0.0.1/8`，如果 h1 的 IP 是 `10.0.0.1/24` 的話，

那 Routing Table 還會再新增一筆 Entry。

###### SSH into Host

除此之外，範例程式碼亦執行了：

<figure class="figure-code code"><figcaption><span>
</span></figcaption><div class="highlight"><pre>for host in network.hosts:
    host.cmd( '/usr/sbin/sshd -D &amp;' )
</pre></div>
</figure>

在每個 Host 執行 ssh daemon，

所以就可以在機器下 `ssh 10.0.0.1` 來操作 h1 了！
