# 需求

有时候，你需要制定容器的启动顺序，比如你的web依赖你的sql，当sql没有启动时，你的web是无法正常启动的。
所以，你就会需要控制你容器的启动顺序。

# 如何实现

很遗憾，目前docker自己的命令都无法真正实现这个功能。

# 那怎么办

docker官方文档告诉我们，我们可以这样做：

> Use a tool such as [wait-for-it](https://github.com/vishnubob/wait-for-it) or [dockerize](https://github.com/jwilder/dockerize). These are small wrapper scripts which you can include in your application’s image and will poll a given host and port until it’s accepting TCP connections.

# wait-for-it

它很简单，就是一段ps脚本（放到文章最下面的代码一）

## 怎么用

更简单,看下面

```
wait-for-it.sh host:port [-s] [-t timeout] [-- command args]
-h HOST | --host=HOST       要测试的IP或者域名
-p PORT | --port=PORT       要测试的TCP端口
                            一般情况下，你可以写成这种形式host:port
-s | --strict               只有测试通过，才执行后面的命令
-q | --quiet                静默模式，不输出任何状态信息
-t TIMEOUT | --timeout=TIMEOUT
                            超时时间设置，超过就执行后面的命令, 0代表永不超时，如果不带这个参数，就是默认的15秒
-- COMMAND ARGS             测试通过后要执行的命令和参数
```

例一：不带超时时间

```
[nrj@test139 workspace]$ ./wait-for-it.sh  www.baidu.com:81 -- echo "baidu is up"          
wait-for-it.sh: waiting 15 seconds for www.baidu.com:81
wait-for-it.sh: timeout occurred after waiting 15 seconds for www.baidu.com:81
baidu is up
```

例二：10秒超时

```
[nrj@test139 workspace]$ ./wait-for-it.sh  www.baidu.com:81 -t 10 -- echo "baidu is up"  
wait-for-it.sh: waiting 10 seconds for www.baidu.com:81
wait-for-it.sh: timeout occurred after waiting 10 seconds for www.baidu.com:81
baidu is up
```

例三：永不超时

```
[nrj@test139 workspace]$ ./wait-for-it.sh  www.baidu.com:81 -t 0 -- echo "baidu is up"     
wait-for-it.sh: waiting for www.baidu.com:81 without a timeout
```

例四：只有测试通过，才执行后面命令

```
[nrj@test139 workspace]$ ./wait-for-it.sh  www.baidu.com:81 -t 10 -s  -- echo "baidu is up"  
wait-for-it.sh: waiting 10 seconds for www.baidu.com:81
wait-for-it.sh: timeout occurred after waiting 10 seconds for www.baidu.com:81
wait-for-it.sh: strict mode, refusing to execute subprocess


[nrj@test139 workspace]$ ./wait-for-it.sh  www.baidu.com:80 -t 10 -s  -- echo "baidu is up"  
wait-for-it.sh: waiting 10 seconds for www.baidu.com:80
wait-for-it.sh: www.baidu.com:80 is available after 0 seconds
baidu is up
```

例五：静默

```
[nrj@test139 workspace]$ ./wait-for-it.sh  www.baidu.com:81 -t 5 -s -q  -- echo "baidu is up"
[nrj@test139 workspace]$ ./wait-for-it.sh  www.baidu.com:80 -t 5 -s -q  -- echo "baidu is up"  
baidu is up
```

# Docker里面怎么用

这个应该更简单吧。

1. 把这个脚本build进你的镜像里面，比如放到tomcat镜像的/bin下面去。
2. 比如你的tomcat镜像的名称是db，开放3306端口。
3. 那么你tomcat镜像的启动脚本就要这么写：`$docker run [OPTIONS] IMAGE[:TAG] wait-for-it.sh db:3306 -t 0 -q -s  /usr/local/tomcat/bin/catalina.sh run`

# Docker-Compose的depends_on+（healthcheck）

这个怎么用呢？前提条件是用docker-compose，还要使用v2.1的语法

例子：

```powershell
version: '2.1'

services:
  mysql:
        image: registry.cn-hangzhou.aliyuncs.com/zhaoshg1984/env:mysql-5.6.35
        container_name: mysql
        volumes:
          - /home/nrj/workspace/docker-data/mysql/:/var/lib/mysql/
        expose:
          - "3306"
        environment:
          - MYSQL_ROOT_PASSWORD=5Yg6f4x1%bDiX%Q*
        healthcheck:
          test: ["CMD", "curl", "-f", "http://localhost:3306 || exit 1"]
          interval: 10s
          timeout: 5s
          retries: 3


  memcached:
        image: memcached:1.4.36-alpine
        container_name: memcached
        ports:
                - "11211:11211"


  rbac:
        image: registry.cn-hangzhou.aliyuncs.com/zhaoshg1984/env:tomcat8-jre-8u121
        container_name: rbac
        links:
                - mysql
                - memcached
        depends_on:
          mysql:
                condition: service_healthy
          memcached:
                condition: service_started
        expose:
                - "8080"
        volumes:
                - /home/nrj/workspace/projects/rbac/:/usr/local/tomcat/webapps/rbac
                - /home/nrj/logs/rbac/tomcat/:/usr/local/tomcat/logs/
                - /home/nrj/logs/rbac/web/:/home/logs/rbac/
        healthcheck:
          test: curl -f http://localhost:8080/rbac/resource/images/favicon.ico || exit 1
          interval: 30s
          timeout: 5s
          retries: 3
```

上面的例子中，compose会等到redis启动之后、db服务“健康”之后再启动web。

# 代码一：wait-for-it 脚本内容

```powershell
#!/usr/bin/env bash
#Use this script to test if a given TCP host/port are available

cmdname=$(basename $0)

echoerr() { if [[ $QUIET -ne 1 ]]; then echo "$@" 1>&2; fi }

usage()
{
    cat << USAGE >&2
Usage:
    $cmdname host:port [-s] [-t timeout] [-- command args]
    -h HOST | --host=HOST       Host or IP under test
    -p PORT | --port=PORT       TCP port under test
                                Alternatively, you specify the host and port as host:port
    -s | --strict               Only execute subcommand if the test succeeds
    -q | --quiet                Don't output any status messages
    -t TIMEOUT | --timeout=TIMEOUT
                                Timeout in seconds, zero for no timeout
    -- COMMAND ARGS             Execute command with args after the test finishes
USAGE
    exit 1
}
wait_for()
{
    if [[ $TIMEOUT -gt 0 ]]; then
        echoerr "$cmdname: waiting $TIMEOUT seconds for $HOST:$PORT"
    else
        echoerr "$cmdname: waiting for $HOST:$PORT without a timeout"
    fi
    start_ts=$(date +%s)
    while :
    do
        (echo > /dev/tcp/$HOST/$PORT) >/dev/null 2>&1
        result=$?
        if [[ $result -eq 0 ]]; then
            end_ts=$(date +%s)
            echoerr "$cmdname: $HOST:$PORT is available after $((end_ts - start_ts)) seconds"
            break
        fi
        sleep 1
    done
    return $result
}
wait_for_wrapper()
{
    # In order to support SIGINT during timeout: http://unix.stackexchange.com/a/57692
    if [[ $QUIET -eq 1 ]]; then
        timeout $TIMEOUT $0 --quiet --child --host=$HOST --port=$PORT --timeout=$TIMEOUT &
    else
        timeout $TIMEOUT $0 --child --host=$HOST --port=$PORT --timeout=$TIMEOUT &
    fi
    PID=$!
    trap "kill -INT -$PID" INT
    wait $PID
    RESULT=$?
    if [[ $RESULT -ne 0 ]]; then
        echoerr "$cmdname: timeout occurred after waiting $TIMEOUT seconds for $HOST:$PORT"
    fi
    return $RESULT
}
#process arguments
while [[ $# -gt 0 ]]
do
    case "$1" in
        *:* )
        hostport=(${1//:/ })
        HOST=${hostport[0]}
        PORT=${hostport[1]}
        shift 1
        ;;
        --child)
        CHILD=1
        shift 1
        ;;
        -q | --quiet)
        QUIET=1
        shift 1
        ;;
        -s | --strict)
        STRICT=1
        shift 1
        ;;
        -h)
        HOST="$2"
        if [[ $HOST == "" ]]; then break; fi
        shift 2
        ;;
        --host=*)
        HOST="${1#*=}"
        shift 1
        ;;
        -p)
        PORT="$2"
        if [[ $PORT == "" ]]; then break; fi
        shift 2
        ;;
        --port=*)
        PORT="${1#*=}"
        shift 1
        ;;
        -t)
        TIMEOUT="$2"
        if [[ $TIMEOUT == "" ]]; then break; fi
        shift 2
        ;;
        --timeout=*)
        TIMEOUT="${1#*=}"
        shift 1
        ;;
        --)
        shift
        CLI="$@"
        break
        ;;
        --help)
        usage
        ;;
        *)
        echoerr "Unknown argument: $1"
        usage
        ;;
    esac
done
if [[ "$HOST" == "" || "$PORT" == "" ]]; then
    echoerr "Error: you need to provide a host and port to test."
    usage
fi
TIMEOUT=${TIMEOUT:-15}
STRICT=${STRICT:-0}
CHILD=${CHILD:-0}
QUIET=${QUIET:-0}
if [[ $CHILD -gt 0 ]]; then
    wait_for
    RESULT=$?
    exit $RESULT
else
    if [[ $TIMEOUT -gt 0 ]]; then
        wait_for_wrapper
        RESULT=$?
    else
        wait_for
        RESULT=$?
    fi
fi
if [[ $CLI != "" ]]; then
    if [[ $RESULT -ne 0 && $STRICT -eq 1 ]]; then
        echoerr "$cmdname: strict mode, refusing to execute subprocess"
        exit $RESULT
    fi
    exec $CLI
else
    exit $RESULT
fi
```