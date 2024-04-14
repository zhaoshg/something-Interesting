---
title: Docker Compose安装、使用和示例
date: 2017-04-19 15:30:52
url: docker_compose_install_use_exp
categories: docker
tags: 
	- docker
	- docker-compose
---

Compose是用于定义和运行复杂Docker应用的工具。你可以在一个文件中定义一个多容器的应用，然后使用一条命令来启动你的应用，然后所有相关的操作都会被自动完成。

# 1. 安装Docker和Compose
   Compose是用于定义和运行复杂Docker应用的工具。你可以在一个文件中定义一个多容器的应用，然后使用一条命令来启动你的应用，然后所有相关的操作都会被自动完成。
   
## 安装
```ps
$curl -L "https://github.com/docker/compose/releases/download/1.11.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
    
$chmod +x /usr/local/bin/docker-compose
```


<!--more-->


## 测试安装结果

```ps
$docker-compose --version
docker-compose version: 1.11.2
```

# 2. 使用Compose
使用Compose只需要简单的三个步骤：

**首先**，使用Dockerfile来定义你的应用环境：
```yml
FROM python:2.7
ADD ./code
WORKDIR /code
RUN pip install -r requirements.txt
```
其中，requirements.txt中的内容包括：
```
flask
redis
```
再用Python写一个简单的app.py
```python
from flask importFlaskfrom redis importRedisimport os
app =Flask(__name__)
redis =Redis(host='redis', port=6379)@app.route('/')def hello():
    redis.incr('hits')return'Hello World! I have been seen %s times.'% redis.get('hits')if __name__ `"__main__":
    app.run(host="0.0.0.0", debug=True)
```
**第二步**，用一个compose.yaml来定义你的应用服务，他们可以把不同的服务生成不同的容器中组成你的应用。
```yml
web:
  build:.
  command: python app.py
  ports:
         - "5000:5000"
  volumes:
         - .:/code
  links:
         - redis
redis:
  image: redis
```
**第三步**，执行docker-compose up来启动你的应用，它会根据compose.yaml的设置来pull/run这俩个容器，然后再启动。

```
Creating myapp_redis_1...
Creating myapp_web_1...
Building web...
Step 0 : FROM python:2.7
2.7: Pulling from python
...
Status: Downloaded newer image for python:2.7
 ---> d833e0b23482
Step 1 : ADD . /code
 ---> 1c04b1b15808
Removing intermediate container 9dab91b4410d
Step 2 : WORKDIR /code
 ---> Running in f495a62feac9
 ---> ffea89a7b090
Attaching to myapp_redis_1, myapp_web_1
......
redis_1 | [1] 17 May 10:42:38.147 * The server is now ready to accept connections on port 6379
web_1   |  * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
web_1   |  * Restarting with stat
```

# 3. Yaml文件参考
在上面的yaml文件中，我们可以看到compose文件的基本结构。首先是定义一个服务名，下面是yaml服务中的一些选项条目：

`image`:镜像的ID  
`build`:直接从pwd的Dockerfile来build，而非通过image选项来pull  
`links`：连接到那些容器。每个占一行，格式为SERVICE[:ALIAS],例如 – db[:database]  
`external_links`：连接到该compose.yaml文件之外的容器中，比如是提供共享或者通用服务的容器服务。格式同links  
`command`：替换默认的command命令  
`ports`: 导出端口。格式可以是：
```yml
ports:-"3000"-"8000:8000"-"127.0.0.1:8001:8001"
```
`expose`:导出端口，但不映射到宿主机的端口上。它仅对links的容器开放。格式直接指定端口号即可。  
`volumes`：加载路径作为卷，可以指定只读模式：
```yml
volumes:-/var/lib/mysql
 - cache/:/tmp/cache
 -~/configs:/etc/configs/:ro
```
`volumes_from`：加载其他容器或者服务的所有卷
```yml
environment:- RACK_ENV=development
  - SESSION_SECRET
```
`env_file`：从一个文件中导入环境变量，文件的格式为RACK_ENV=development  
`extends`:扩展另一个服务，可以覆盖其中的一些选项。一个sample如下：
```yml
webapp:
  build:./webapp
  environment:- DEBUG=false- SEND_EMAILS=false
development.yml
web:extends:
    file: common.yml
    service: webapp
  ports:-"8000:8000"
  links:- db
  environment:- DEBUG=true
db:
  image: postgres
```
`net`：容器的网络模式，可以为”bridge”, “none”, “container:[name or id]”, “host”中的一个。  
`dns`：可以设置一个或多个自定义的DNS地址。  
`dns_search`:可以设置一个或多个DNS的扫描域。  
其他的`working_dir`, `entrypoint`, `user`, `hostname`, `domainname`, `mem_limit`, `privileged`, `restart`, `stdin_open`, `tty`, `cpu_shares`，和`docker run`命令是一样的，这些命令都是单行的命令。例如：
```yml
cpu_shares:73
working_dir:/code
entrypoint: /code/entrypoint.sh
user: postgresql
hostname: foo
domainname: foo.com
mem_limit:1000000000
privileged:true
restart: always
stdin_open:true
tty:true
```

# 4. docker-compose常用命令

在第二节中的`docker-compose up`，这两个容器都是在前台运行的。我们可以指定-d命令以daemon的方式启动容器。除此之外，docker-compose还支持下面参数：  

`--verbose`：输出详细信息  
`-f` 制定一个非docker-compose.yml命名的yaml文件  
`-p` 设置一个项目名称（默认是directory名）  

`docker-compose`的动作包括：

&emsp; `build`：构建服务  
&emsp; `kill -s SIGINT`：给服务发送特定的信号。  
&emsp; `logs`：输出日志  
&emsp; `port`：输出绑定的端口  
&emsp; `ps`：输出运行的容器  
&emsp; `pull`：pull服务的image  
&emsp; `rm`：删除停止的容器  
&emsp; `run`: 运行某个服务，例如docker-compose run web python manage.py ps  
&emsp; `start`：运行某个服务中存在的容器。  
&emsp; `stop`:停止某个服务中存在的容器。  
&emsp; `up`：create + run + attach容器到服务。  
&emsp; `scale`：设置服务运行的容器数量。例如：docker-compose scale web=2 worker=3



# 5. 个人示例

## 示例一
```ps
$vim ~/workspace/docker-compose-env.yml
```
### 代码：
```yml
mysql:
  image: mysql:5.6.35
  container_name: mysql_db
  expose:
    - "3306"
  volumes:
        - /var/lib/mysql/:/var/lib/mysql/
  environment:
        - MYSQL_ROOT_PASSWORD=yourpassword
  net: mynet

rbac:
  image: hub.c.163.com/zhaoshg/tomcat:7-jre8-alpine
  container_name: rbac
  expose:
        - "8080"
  volumes:
        - /home/zhaoshg/workspace/projects/rbac/:/usr/local/tomcat/webapps/rbac
        - /home/zhaoshg/logs/rbac/tomcat/:/usr/local/tomcat/logs/
        - /home/zhaoshg/logs/rbac/:/home/logs/rbac/
  net:  mynet
  links:
        - mysql:mysql_db
```

- mynet是创建的overlay网络， 详见[基于consul的Docker-overlay跨多宿主机容器网络](http://note.youdao.com/group/#/16052066/(folder/137189937//full:md/137649015)?full=true)

### 运行
```ps
$docker-compose -f ~/workspace/docker-compose-env.yml up -d
```
### 查看启动情况
```ps
$docker ps

CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
936d6a048068        mysql:5.6.35        "docker-entrypoint..."   2 hours ago         Up 2 hours          3306/tcp                 workspace_mysql_1
f1827893e1b4        43b76557edc3        "catalina.sh run"        2 hours ago         Up 2 hours          8080/tcp                 workspace_rbac_1
```

## 示例二

```ps
$vim ~/workspace/docker-compose-projects.yml
```

### 代码：
```yml
discenter:
  image: hub.c.163.com/zhaoshg/tomcat:7-jre8-alpine
  container_name: discenter
  ports:
        - "8888:8080"
  volumes:
        - /home/zhaoshg/workspace/projects/discenter/:/usr/local/tomcat/webapps/ROOT
        - /home/zhaoshg/logs/discenter/:/usr/local/tomcat/logs/
  net: mynet

cms:
  image: hub.c.163.com/zhaoshg/tomcat:7-jre8-alpine
  container_name: cms
  ports:
        - "8887:8080"
  volumes:
        - /home/zhaoshg/workspace/projects/cms/:/usr/local/tomcat/webapps/ROOT
        - /home/zhaoshg/logs/cms/tomcat/:/usr/local/tomcat/logs/
        - /home/zhaoshg/logs/cms/:/home/logs/cms/
  net: mynet
```

### 运行
```ps
$docker-compose -f ~/workspace/docker-compose-projects.yml up -d
```
### 查看启动情况
```ps
$docker ps 

CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
44c331824b2f        43b76557edc3        "catalina.sh run"        2 hours ago         Up 2 hours          0.0.0.0:8888->8080/tcp   workspace_discenter_1
b3b5ec77f75a        43b76557edc3        "catalina.sh run"        2 hours ago         Up 2 hours          0.0.0.0:8887->8080/tcp   workspace_cms_1
f1827893e1b4        43b76557edc3        "catalina.sh run"        2 hours ago         Up 2 hours          8080/tcp                 workspace_rbac_1
936d6a048068        mysql:5.6.35        "docker-entrypoint..."   2 hours ago         Up 2 hours          3306/tcp                 workspace_mysql_1
```

## 示例三
```yml
version: '2.1'

services:
  mysql:
        image: hub.c.163.com/zhaoshg/mysql:5.6.35
        container_name: mysql
        volumes:
          - /home/zhaoshg/workspace/docker-data/mysql/:/var/lib/mysql/
        expose:
          - "3306"
        environment:
          - MYSQL_ROOT_PASSWORD=yourpassword
        healthcheck:
          test: ["CMD", "curl", "-f", "http://localhost:3306 || exit 1"]
          interval: 10s
          timeout: 5s
          retries: 3
#       mem_limit: 256m


  memcached:
        image: memcached:1.4.36-alpine
        container_name: memcached
        ports:
                - "11211:11211"
        mem_limit: 56m

  fdfssingle:
        image: hub.c.163.com/zhaoshg/fastdfs
        container_name: fdfssingle
        ports:
                - "22122:22122"
        volumes:
                - /home/zhaoshg/fastdfs:/fastdfs
        mem_limit: 200m


  myweb:
        image: hub.c.163.com/zhaoshg/tomcat
        container_name: myweb
        environment:
                - JAVA_OPTS=-Djava.rmi.server.hostname=192.168.2.139 -Dcom.sun.management.jmxremote=true -Dcom.sun.management.jmxremote.local.only=false -Dcom.sun.management.jmxremote.port=18900 -Dcom.sun.management.jmxremote.rmi.port=18901 -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.authenticate=false -Xms512m -Xmx512m
        links:
                - mysql
                - memcached
        depends_on:
          mysql:
                condition: service_healthy
          memcached:
                condition: service_started
        ports:
                - "8100:8080"
                - "18900:18900"
                - "18901:18901"
        volumes:
                - /home/zhaoshg/workspace/projects/discenter/:/usr/local/tomcat/webapps/ROOT
                - /home/zhaoshg/logs/discenter/tomcat/:/usr/local/tomcat/logs/
                - /home/zhaoshg/logs/discenter/web/:/home/logs/discenter/
        healthcheck:
          test: curl -f http://localhost:8080/favicon.ico || exit 1
          interval: 30s
          timeout: 5s
          retries: 3
```

参考:  
[Compose Document](http://docs.docker.com/compose/)