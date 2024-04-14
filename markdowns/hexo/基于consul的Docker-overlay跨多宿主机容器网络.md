---
title: 基于consul的Docker-overlay跨多宿主机容器网络
date: 2017-04-20 10:00:59
url: docker_consul_overlay
categories: docker
tags: 
	- docker
	- overlay
	- consul
	- 多宿主机
---

# 环境限制

- 必须安装key-value存储服务，如consul
- 宿主机已经安装docker engine
- 宿主机的hostname必须不同
- 内核大于3.16


<!--more-->


# 环境准备及角色分配

- 两台ubuntu的server

主机名 | ip     | 内核 | 启动docker容器名称 | docker engine版本 |consul服务
---|---|---|---|---|---
test26 | 192.168.2.26 | 4.10.4-1.el7.elrepo.x86_64 | test26（centOS7） | 17.03.0-ce | server
test139 | 192.168.2.139 | 4.10.4-1.el7.elrepo.x86_64 | test139（centOS7） | 17.03.0-ce | client

- 实验目标：
    
    两个CentOS7容器test26,test139网络互通, **注意本文中的zhaoshg@test26和zhaoshg@test139**

## 防火墙开放端口
分别在两台机器上运行：
```ps
$sudo firewall-cmd --permanent --add-port={8500/tcp,8300/tcp,8301/tcp,3375/tcp,2375/tcp}
$sudo firewall-cmd --reload
$sudo firewall-cmd  --list-all
```

## 运行分布式发现服务协调软件：consul

- 拉取consul镜像
```ps
$docker pull consul
```
- 启动consul
```ps
[zhaoshg@test ~]$docker run -d -it --net=host --name=consul_server -v /home/zhaoshg/consul_data:/data -e 'CONSUL_LOCAL_CONFIG={"skip_leave_on_interrupt": true}' -p 8300:8300 -p 8301:8301 -p 8301:8301/udp -p 8302:8302 -p 8302:8302/udp -p 8400:8400 -p 8500:8500 -p 53:53/udp consul agent -server -bootstrap -bind=192.168.2.26
[nrj@test ~]$docker run -d -it --net=host --name=consul_client -v /home/zhaoshg/consul_data:/data -e 'CONSUL_LOCAL_CONFIG={"leave_on_terminate": true}' -p 8300:8300 -p 8301:8301 -p 8301:8301/udp -p 8302:8302 -p 8302:8302/udp -p 8400:8400 -p 8500:8500 -p 53:53/udp  consul agent -bind=192.168.2.139 -join=192.168.2.26
```
- 可以启动一个server和多个agent(此处是一个)，然后让agent，join到consul集群中
- 启动时可能会遇到55/udp被占用，多半是虚拟网卡的问题，卸载就好：
```ps
$sudo virsh net-list
$sudo virsh net-destroy default 
$sudo virsh net-undefine default
$sudo service libvirtd restart
```

## 配置Docker并重启
- 在每一台docker宿主机上做如下配置，并重启docker
```ps
$sudo vim /etc/docker/daemon.json
//添加
{
  "tls": false,
  "hosts": ["unix:///var/run/docker.sock","tcp://0.0.0.0:2375"],
  "cluster-store": "consul://localhost:8500",
  "cluster-advertise": "enp0s3:2375"
}
$sudo systemctl daemon-reload
$sudo systemctl restart docker
```
- enp0s3代表两台宿主机交互所对应的网卡地址  分别是  192.168.2.26和192.168.2.139
- 集群配置

   `–cluster-store=` 参数指向docker daemon所使用key value service的地址（本例中即consul的服务地址）`–cluster-advertise=` 参数决定了所使用网卡以及docker daemon端口信息

- 宿主机配置

    上面的-H 的参数分别指定了docker demon服务的地址和协议

# 跨主机网络创建

## 创建overlay网络
- 创建
```ps
[zhaoshg@test26 ~]: $sudo docker network create -d overlay  mynet
```

- 验证
在test26上创建的multihost网络，会通过consul服务同步到test139上面
```ps
[zhaoshg@test139 ~]:$docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
490c2dbc35e0        bridge              bridge              local
045f3acd3c28        docker_gwbridge     bridge              local
023021bcc407        host                host                local
5104bce70f86        mynet               overlay             global
3bbb2a39fad8        none                null                local  
```

## 创建容器
- 创建测试容器
```ps
$docker run -d -it --net=mynet --name=test1 centos
$docker run -d -it --net=mynet --name=test2 centos
```


## 验证网络互通
```ps
zhaoshg@test139:~$  PID=$(docker inspect --format {{.State.Pid}} test2)
zhaoshg@test139:~$  sudo nsenter --target $PID --mount --uts --ipc --net --pid

[root@2516560c337f /]# yum install net-tools
[root@2516560c337f /]# ifconfig

eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1450
        inet 10.0.0.2  netmask 255.255.255.0  broadcast 0.0.0.0
        inet6 fe80::42:aff:fe00:2  prefixlen 64  scopeid 0x20<link>
        ether 02:42:0a:00:00:02  txqueuelen 0  (Ethernet)
        RX packets 31  bytes 2398 (2.3 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 18  bytes 1412 (1.3 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

[root@2516560c337f /]# 

[root@2516560c337f /]# ping 192.168.2.26

PING 192.168.2.26 (192.168.2.26) 56(84) bytes of data.
64 bytes from 192.168.2.26: icmp_seq=1 ttl=63 time=0.657 ms
64 bytes from 192.168.2.26: icmp_seq=2 ttl=63 time=0.853 ms
64 bytes from 192.168.2.26: icmp_seq=8 ttl=63 time=0.463 ms
64 bytes from 192.168.2.26: icmp_seq=9 ttl=63 time=0.681 ms
^C
--- 192.168.2.26 ping statistics ---
32 packets transmitted, 32 received, 0% packet loss, time 31736ms
rtt min/avg/max/mdev = 0.363/0.671/1.486/0.183 ms

[root@2516560c337f /]# 

[root@2516560c337f /]# ping 10.0.0.2

PING 10.0.0.2 (10.0.0.2) 56(84) bytes of data.
64 bytes from 10.0.0.2: icmp_seq=1 ttl=64 time=0.034 ms
64 bytes from 10.0.0.2: icmp_seq=2 ttl=64 time=0.068 ms
64 bytes from 10.0.0.2: icmp_seq=3 ttl=64 time=0.061 ms
64 bytes from 10.0.0.2: icmp_seq=4 ttl=64 time=0.068 ms
^C
--- 10.0.0.2 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3089ms
rtt min/avg/max/mdev = 0.034/0.057/0.068/0.016 ms

[root@2516560c337f /]#
```

- 验证结论:
    
    test139容器host2的ip=10.0.0.3，可以ping通test26，可以ping通test26上的容器host1的ip=10.0.0.2

# 如何使用静态ip

- 以上的实验步骤。container的ip都是自动分配的，如果需要静态的固定ip，怎么办？
- 在创建网络的过程中有区别

```ps
$sudo docker network create -d overlay --ip-range=192.168.20.0/24 --gateway=192.168.20.1 --subnet=192.168.20.0/24 mynet
$docker run -d -it --name test1 --net=mynet --ip=192.168.20.2 centos
$docker run -d -it --name test2 --net=mynet --ip=192.168.20.3 centos
```

# 删除网络
```ps
$docker network rm mynet
```