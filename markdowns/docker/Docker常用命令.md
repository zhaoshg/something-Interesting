# 常用命令

------

## 最常用几个命令

```
#显示现有镜像
$docker images

#查看正在运行的容器
$docker ps

#查看所有容器
$docker ps -a

#进入容器
$ docker exec -it <containerName/Id> "bin/bash"

#停止所有容器
$docker stop $(docker ps -a -q)

#删除所有已经停止的容器
$docker rm $(docker ps -a -q)

#删除所有的容器,包括正在运行的
$docker rm -f $(docker ps -a -q)
```

------

## **镜像操作**

### 检查Docker的安装是否正确：

```
$docker info
```

### 查询仓库的镜像

```
$docker search <name>
```

### 下载镜像

```
$docker pull <name>
```

### 删除镜像

```
$docker rmi <imageName>
```

### 删除所有镜像

```
$docker rmi $(docker images -q -a)
```

### 查看镜像列表：

```
$docker images
```

### 查看镜像的历史版本

```
$docker history <imageId>
```

### 将镜像推送到registry

```
$docker push <repo_name>
```

### 运行镜像

```
$docker run <imageId/imageName>
```

------

## **Docker 运行时常用参数**

```
  Usage: docker run [OPTIONS] IMAGE [COMMAND] [ARG...]    

-d, --detach=false         指定容器运行于前台还是后台，默认为false     
-i, --interactive=false    打开STDIN，用于控制台交互    
-t, --tty=false            分配tty设备，该可以支持终端登录，默认为false    
-u, --user=""              指定容器的用户    
-a, --attach=[]            登录容器（必须是以docker run -d启动的容器）  
-w, --workdir=""           指定容器的工作目录   
-c, --cpu-shares=0         设置容器CPU权重，在CPU共享场景使用    
-e, --env=[]               指定环境变量，容器中可以使用该环境变量    
-m, --memory=""            指定容器的内存上限    
-P, --publish-all=false    指定容器暴露的端口    
-p, --publish=[]           指定容器暴露的端口   
-h, --hostname=""          指定容器的主机名    
-v, --volume=[]            给容器挂载存储卷，挂载到容器的某个目录    
--volumes-from=[]          给容器挂载其他容器上的卷，挂载到容器的某个目录  
--cap-add=[]               添加权限，权限清单详见：http://linux.die.net/man/7/capabilities    
--cap-drop=[]              删除权限，权限清单详见：http://linux.die.net/man/7/capabilities    
--cidfile=""               运行容器后，在指定文件中写入容器PID值，一种典型的监控系统用法    
--cpuset=""                设置容器可以使用哪些CPU，此参数可以用来容器独占CPU    
--device=[]                添加主机设备给容器，相当于设备直通    
--dns=[]                   指定容器的dns服务器    
--dns-search=[]            指定容器的dns搜索域名，写入到容器的/etc/resolv.conf文件    
--entrypoint=""            覆盖image的入口点    
--env-file=[]              指定环境变量文件，文件格式为每行一个环境变量    
--expose=[]                指定容器暴露的端口，即修改镜像的暴露端口    
--link=[]                  指定容器间的关联，使用其他容器的IP、env等信息    
--lxc-conf=[]              指定容器的配置文件，只有在指定--exec-driver=lxc时使用    
--name=""                  指定容器名字，后续可以通过名字进行容器管理，links特性需要使用名字    
--net="bridge"             容器网络设置:  
                              bridge 使用docker daemon指定的网桥       
                              host    //容器使用主机的网络    
                              container:NAME_or_ID  >//使用其他容器的网路，共享IP和PORT等网络资源    
                              none 容器使用自己的网络（类似--net=bridge），但是不进行配置   
--privileged=false         指定容器是否为特权容器，特权容器拥有所有的capabilities    
--restart="no"             指定容器停止后的重启策略:  
                              no：容器退出时不重启    
                              on-failure：容器故障退出（返回值非零）时重启   
                              always：容器退出时总是重启    
--rm=false                 指定容器停止后自动删除容器(不支持以docker run -d启动的容器)    
--sig-proxy=true           设置由代理接受并处理信号，但是SIGCHLD、SIGSTOP和SIGKILL不能被代理
```

------

## **容器操作**

### 查看正在运行的容器：

```
$docker ps
```

### 启动时设置root用户密码

```
$docker run -d -e ROOT_PASS=<mypass> <imageID>
```

### 查看所有容器（包括已经关闭的）：

```
$docker ps -a
```

### 列出最近一次启动的container

```
$docker ps -l
```

### 用一行列出所有正在运行的container（容器多的时候非常清晰）

```
$docker ps | less -S
```

### 查看镜像/容器信息

```
$docker inspect <imageId/containerId>
```

### 显示一个运行的容器里面的进程信息

```
$docker top <containerName/containerID>
```

### 停止容器

```
$docker stop <containerId>
```

### 停止所有容器

```
$docker stop $(docker ps -q);
```

### 杀死容器

```
$docker kill <containerName/containerID>
```

### 杀死所有容器

```
$docker kill $(docker ps -q);
```

### 删除所有容器

```
$docker rm $(docker ps -a -q)
```

### 启动容器

```
$docker start <containerId>
```

### 重启容器

```
$docker restart <containerId>
```

### 从容器里面拷贝文件/目录到本地一个路径

```
$docker cp <containerName:/container_path> to_path  
$docker cp <containerID:/container_path> to_path
```

### 删除所有容器

```
$docker rm `docker ps -a -q`
```

------

## 如何连接运行中的容器

### 方法一（推荐）：

```
#后面的bin/bash由容器版本决定，可能是bin/bash（ubuntu\debian）,也可能是bin/sh（alpine）
$docker exec -it <containerId/containername> "bin/bash"
```

### 方法二：

```
$docker attach <containerId>
```

### 方法三：

#### 安装nsenter

```
$docker run --rm -v /usr/local/bin:/target jpetazzo/nsenter
```

#### 查看所连接容器的PID

```
#获取PID
$PID=$(docker inspect --format {{.State.Pid}} <container_name_or_ID>)
```

#### 连接容器

```
$sudo nsenter --target $PID --mount --uts --ipc --net --pid
```