---
title: Use docker and openvswitch create network environment
date: 2016-07-08 18:10:47
tags: [docker, ovs, network]
categories: Coding札記
---

### 這篇內容會提及到 ...
- 安裝 docker
- 安裝 openvswitch
- 以 ovs 作為 switch 連接兩個 docker container 並讓網路運作

<!-- more -->

### Install docker
```
root@ubuntu:~# apt-get install docker.io
```

### compile ovs [1]

先從 [ovs官網](http://openvswitch.org/releases/) 找到需要的 ovs 版本

##### Download and unpack
```
root@ubuntu:~# wget http://openvswitch.org/releases/openvswitch-2.5.0.tar.gz
root@ubuntu:~# tar zxvf openvswitch-2.5.0.tar.gz
```

##### Build and install
```
root@ubuntu:~# cd openvswitch-2.5.0/
root@ubuntu:~/openvswitch-2.5.0# ./configure --prefix=/usr --with-linux=/lib/modules/`uname -r`/build
root@ubuntu:~/openvswitch-2.5.0# make
root@ubuntu:~/openvswitch-2.5.0# make install
root@ubuntu:~/openvswitch-2.5.0# make modules_install
root@ubuntu:~/openvswitch-2.5.0# rmmod openvswitch
root@ubuntu:~/openvswitch-2.5.0# depmod -a
```

##### Test install
```
root@ubuntu:~# ovs-vsctl show
506b6d7a-87bc-44dd-9564-fa0b94ee5821
    ovs_version: "2.5.0"
```

### Create network with docker and ovs [2]

##### network topology
![Network Topology](http://i.imgur.com/CipybJC.jpg)

##### create switch
```
root@ubuntu:~# ovs-vsctl add-br sw1  # Create ovs-br sw1
```

##### create docker container
```
root@ubuntu:~# docker run --net='none' --name='h1' -privileged -itd ubuntu:14.04
root@ubuntu:~# docker run --net='none' --name='h2' -privileged -itd ubuntu:14.04
```
在此使用 `-privileged` 的原因是，
docker default 是不能更改 container 的網路配置，
所以使用這個 flag 之後就可以設定 container 的網路組態了。

```
CONTAINER ID     IMAGE             COMMAND          CREATED          STATUS           PORTS         NAMES
189eee094aa1     ubuntu:14.04      "/bin/bash"      3 hours ago      Up 2 seconds                   h2    
ec4764d12845     ubuntu:14.04      "/bin/bash"      3 hours ago      Up 4 seconds                   h1    
```

##### create veth
```
root@ubuntu:~# ip link add sw1-eth1 type veth peer name h1-eth0
root@ubuntu:~# ip link add sw1-eth2 type veth peer name h2-eth0
```

##### add veth intf to ovs switch
```
root@ubuntu:~# ovs-vsctl add-port sw1 sw1-eth1
root@ubuntu:~# ovs-vsctl add-port sw1 sw1-eth2
```

##### get container's PID
```
root@ubuntu:~# docker inspect -f '{{.State.Pid}}' h1
3827
root@ubuntu:~# docker inspect -f '{{.State.Pid}}' h2
3848
```

##### putting veth into container's namespace
```
root@ubuntu:~# mkdir -p /var/run/netns
root@ubuntu:~# ln -s /proc/<h1_PID>/ns/net /var/run/netns/<h1_PID>
root@ubuntu:~# ip link set h1-eth0 netns <h1_PID>
root@ubuntu:~# ln -s /proc/<h2_PID>/ns/net /var/run/netns/<h2_PID>
root@ubuntu:~# ip link set h2-eth0 netns <h2_PID>
```

##### IP address configuration
use following command to access container
```
root@ubuntu:~# docker attach h1
root@ubuntu:~# docker attach h2
```

In here, I set h1's IP to 10.0.0.1/24, h2's IP to 10.0.0.2/24

##### make switch interface up
```
root@ubuntu:~# ifconfig sw1-eth1 up
root@ubuntu:~# ifconfig sw1-eth2 up
```

##### result
```
root@ec4764d12845:/# ping 10.0.0.2
PING 10.0.0.2 (10.0.0.2) 56(84) bytes of data.
64 bytes from 10.0.0.2: icmp_seq=1 ttl=64 time=0.559 ms
64 bytes from 10.0.0.2: icmp_seq=2 ttl=64 time=0.054 ms
^C
--- 10.0.0.2 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 0.054/0.306/0.559/0.253 ms
```


[1] [Installing new version of Open vSwitch](https://github.com/mininet/mininet/wiki/Installing-new-version-of-Open-vSwitch)
[2] [How to use OpenVswitch with Docker](http://cloudgeekz.com/400/how-to-use-openvswitch-with-docker.html)
