# Docker的安装

- 卸载旧版本 docker 或者 docker-engine

  ```
  $sudo yum remove docker \
                    docker-common \
                    container-selinux \
                    docker-selinux \
                    docker-engine
  ```

- 安装Docker CE

  ```
  $ sudo yum install -y yum-utils
  $ sudo yum-config-manager --add-repo  https://download.docker.com/linux/centos/docker-ce.repo
  $ sudo yum-config-manager --enable docker-ce-edge
  $ sudo yum makecache fast
  $ sudo yum install docker-ce
  ```

# 启动Docker

```
$sudo systemctl start docker
```

详见[官方文档](https://docs.docker.com/engine/installation/)

# 去除sudo

如果还没有 docker group 就添加一个：

```
$sudo groupadd docker
```

将用户加入该 group 内。然后退出并重新登录就生效啦。

```
$sudo gpasswd -a ${USER} docker
```

重启 docker 服务

```
$sudo systemctl restart docker
```

切换当前会话到新 group 或者重启 X 会话

```
$newgrp - docker
```

OR

```
$pkill X
```

注意，最后一步是必须的，否则因为 groups 命令获取到的是缓存的组信息，刚添加的组信息未能生效，所以 docker images 执行时同样有错。

- 原因分析
  因为 /var/run/docker.sock 所属 docker 组具有 setuid 权限

```
$sudo ls -l /var/run/docker.sock
srw-rw---- 1 root docker 0 May  1 21:35 /var/run/docker.sock
```

# 检查、测试

```
$docker info
$docker run hello-world
```