---
title: Docker常见问题及解决方案
date: 2017-04-19 15:29:43
url: docker_q_and_a
categories: docker
tags: docker
---
# Docker常见问题及解决方案

#### 1. container中启动tomcat8.5，访问/manager时直接报403错误，不提示输入用户名密码
#### 解决方案：
在$CATALINA_BASE/conf/Catalina/localhost下新增manager.xml文件：
```xml
<Context privileged="true" antiResourceLocking="false" docBase="${catalina.home}/webapps/manager">
        <Valve className="org.apache.catalina.valves.RemoteAddrValve" allow="^.*$" />
</Context>
```
下面是tomcat8.5的问题,下面是官方文档的解释:

>A default Tomcat installation includes the Manager. To add an instance of the Manager web application Context to a new host install the manager.xml context configuration file in the $CATALINA_BASE/conf/[enginename]/[hostname] folder


<!--more-->

#### 2. 去除sudo
#### 解决方案：
- 如果还没有 docker group 就添加一个：
    
```ps
$sudo groupadd docker
```
- 将用户加入该 group 内。然后退出并重新登录就生效啦。
```ps
$sudo gpasswd -a ${USER} docker
```
- 重启 docker 服务
```ps
$sudo service docker restart
```
- 切换当前会话到新 group 或者重启 X 会话
```ps
$newgrp - docker
OR
$pkill X
```
注意，最后一步是必须的，否则因为 groups 命令获取到的是缓存的组信息，刚添加的组信息未能生效，所以 docker images 执行时同样有错。
- 原因分析
  因为 /var/run/docker.sock 所属 docker 组具有 setuid 权限
```ps
$sudo ls -l /var/run/docker.sock
srw-rw---- 1 root docker 0 May  1 21:35 /var/run/docker.sock
```
#### 3. 镜像启动后自动退出
#### 解决方案：
- 加 `-d -it` 参数


#### 4. centos7 中docker info报错docker bridge-nf-call-iptables is disabled  
#### 解决方案：
```ps
$sudo vim /etc/sysctl.conf
#增加
  net.bridge.bridge-nf-call-ip6tables = 1
  net.bridge.bridge-nf-call-iptables = 1
  net.bridge.bridge-nf-call-arptables = 1
$sudo reboot
```

#### 5. docker 端口映射错误
```ps
docker: Error response from daemon: driver failed programming external connectivity on endpoint mysql (93720099240d3fed6a1316d6f1de7001331f8e4c587163653ffbf8b49375b8c1):  (iptables failed: iptables --wait -t nat -A DOCKER -p tcp -d 0/0 --dport 3306 -j DNAT --to-destination 172.17.0.2:3306 ! -i docker0: iptables: No chain/target/match by that name.
 (exit status 1)).
```
#### 解决方案：
```ps
$sudo pkill docker
$sudo iptables -t nat -F
$sudo ifconfig docker0 down
$sudo brctl delbr docker0
$sudo systemctl restart docker.service
```
