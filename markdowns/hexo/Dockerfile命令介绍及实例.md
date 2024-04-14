---
title: Dockerfile命令介绍及实例
date: 2017-04-20 14:34:18
url: into_dockerfile_example
categories: docker
tags:
	- docker
	- dockerfile
---
# Dockerfile命令介绍及实例
基础镜像可以用于创建Docker容器。镜像可以非常基础，仅仅包含操作系统；也可以非常丰富，包含灵巧的应用栈，随时可以发布。当你在使用Docker构建镜像的时候，每一个命令都会在前一个命令的基础上形成一个新层。这些基础镜像可以用于创建新的容器。本篇文章将手把手教您如何从基础镜像，一步一步，一层一层的从Dockerfile构建容器的过程。

<!--more-->

## Docker简介
Docker项目提供了构建在Linux内核功能之上，协同在一起的的高级工具。其目标是帮助开发和运维人员更容易地跨系统跨主机交付应用程序和他们的依赖。Docker通过Docker容器，一个安全的，基于轻量级容器的环境，来实现这个目标。这些容器由镜像创建，而镜像可以通过命令行手工创建或 者通过Dockerfile自动创建。

## Dockerfile
Dockerfile是由一系列命令和参数构成的脚本，这些命令应用于基础镜像并最终创建一个新的镜像。它们简化了从头到尾的流程并极大的简化了部署工作。Dockerfile从FROM命令开始，紧接着跟随者各种方法，命令和参数。其产出为一个新的可以用于创建容器的镜像。

# Dockerfile 语法
在我们深入讨论Dockerfile之前，让我们快速过一下Dockerfile的语法和它们的意义。
## 什么是语法？
非常简单，在编程中，语法意味着一个调用命令，输入参数去让应用执行程序的文法结构。这些语法被规则或明或暗的约束。程序员遵循语法规范以和计算机 交互。如果一段程序语法不正确，计算机将无法识别。Dockerfile使用简单的，清楚的和干净的语法结构，极为易于使用。这些语法可以自我释义，支持注释。

## Dockerfile 语法示例
Dockerfile语法由两部分构成，注释和命令+参数

## Dockerfile 命令
Dockerfile有十几条命令可用于构建镜像，下文将简略介绍这些命令。

### ADD
ADD命令有两个参数，源和目标。它的基本作用是从源系统的文件系统上复制文件到目标容器的文件系统。如果源是一个URL，那该URL的内容将被下载并复制到容器中。
```yml
# Usage: ADD [source directory or URL] [destination directory]
ADD /my_app_folder /my_app_folder 
```

### CMD
和RUN命令相似，CMD可以用于执行特定的命令。和RUN不同的是，这些命令不是在镜像构建的过程中执行的，而是在用镜像构建容器后被调用。
```yml
# Usage 1: CMD application "argument", "argument", ..
CMD "echo" "Hello docker!"
```
### ENTRYPOINT
ENTRYPOINT 帮助你配置一个容器使之可执行化，如果你结合CMD命令和ENTRYPOINT命令，你可以从CMD命令中移除“application”而仅仅保留参数，参数将传递给ENTRYPOINT命令。
```yml
# Usage: ENTRYPOINT application "argument", "argument", ..
# Remember: arguments are optional. They can be provided by CMD
# or during the creation of a container.
ENTRYPOINT echo
# Usage example with CMD:
# Arguments set with CMD can be overridden during *run*
CMD "Hello docker!"
ENTRYPOINT echo
```

### ENV 
ENV命令用于设置环境变量。这些变量以”key=value”的形式存在，并可以在容器内被脚本或者程序调用。这个机制给在容器中运行应用带来了极大的便利。
```yml
# Usage: ENV key value
ENV SERVER_WORKS 4
```
### EXPOSE
EXPOSE用来指定端口，使容器内的应用可以通过端口和外界交互。
```yml
# Usage: EXPOSE [port]
EXPOSE 8080
```
### FROM
FROM命令可能是最重要的Dockerfile命令。改命令定义了使用哪个基础镜像启动构建流程。基础镜像可以为任意镜 像。如果基础镜像没有被发现，Docker将试图从Docker image index来查找该镜像。FROM命令必须是Dockerfile的首个命令。
```yml
# Usage: FROM [image name]
FROM ubuntu 
```

### MAINTAINER
我建议这个命令放在Dockerfile的起始部分，虽然理论上它可以放置于Dockerfile的任意位置。这个命令用于声明作者，并应该放在FROM的后面。
```yml
# Usage: MAINTAINER [name]
MAINTAINER authors_name 
```
### RUN
RUN命令是Dockerfile执行命令的核心部分。它接受命令作为参数并用于创建镜像。不像CMD命令，RUN命令用于创建镜像（在之前commit的层之上形成新的层）。
```yml
# Usage: RUN [command]
RUN aptitude install -y riak
```
### USER
USER命令用于设置运行容器的UID。
```yml
# Usage: USER [UID]
USER 751
```
### VOLUME
VOLUME命令用于让你的容器访问宿主机上的目录。
```yml
# Usage: VOLUME ["/dir_1", "/dir_2" ..]
VOLUME ["/my_files"]
```
### WORKDIR
WORKDIR命令用于设置CMD指明的命令的运行目录。
```yml
# Usage: WORKDIR /path
WORKDIR ~/
```



# 如何使用Dockerfiles
使用Dockerfiles和手工使用Docker Daemon运行命令一样简单。脚本运行后输出为新的镜像ID。
```ps
# Build an image using the Dockerfile at current location
# Example: sudo docker build -t [name] .
$sudo docker build -t my_mongodb . 
```
## Dockerfile 示例一：创建一个最小的带glibc的linux镜像
在这部分中，我们讲一步一步创建一个Dockfile，这个Dockerfile可用于构建一个最小的linux容器。
### 创建一个Dockerfile
使用nano文本编辑器，让我们创建Dockerfile。
```ps
$sudo vi Dockerfile
```
### 定义文件和它的目的
让阅读者明确Dockerfile的目的永远是必要的。为此，我们通常从注释开始写Dockerfile。
```yml
############################################################
# Dockerfile to build a min linux based on alpine and glibc
# Based on alpine
############################################################
#设置基础镜像 
# Set the base image to alpine
FROM alpine:3.5
#定义作者
# File Author
MAINTAINER zhaoshg

# Here we install GNU libc (aka glibc) and set C.UTF-8 locale as default.
RUN ALPINE_GLIBC_BASE_URL="https://github.com/sgerrand/alpine-pkg-glibc/releases/download" && \
    ALPINE_GLIBC_PACKAGE_VERSION="2.25-r0" && \
    ALPINE_GLIBC_BASE_PACKAGE_FILENAME="glibc-$ALPINE_GLIBC_PACKAGE_VERSION.apk" && \
    ALPINE_GLIBC_BIN_PACKAGE_FILENAME="glibc-bin-$ALPINE_GLIBC_PACKAGE_VERSION.apk" && \
    ALPINE_GLIBC_I18N_PACKAGE_FILENAME="glibc-i18n-$ALPINE_GLIBC_PACKAGE_VERSION.apk" && \
    apk add --no-cache --virtual=.build-dependencies wget ca-certificates && \
    wget \
        "https://raw.githubusercontent.com/andyshinn/alpine-pkg-glibc/master/sgerrand.rsa.pub" \
        -O "/etc/apk/keys/sgerrand.rsa.pub" && \
    wget \
        "$ALPINE_GLIBC_BASE_URL/$ALPINE_GLIBC_PACKAGE_VERSION/$ALPINE_GLIBC_BASE_PACKAGE_FILENAME" \
        "$ALPINE_GLIBC_BASE_URL/$ALPINE_GLIBC_PACKAGE_VERSION/$ALPINE_GLIBC_BIN_PACKAGE_FILENAME" \
        "$ALPINE_GLIBC_BASE_URL/$ALPINE_GLIBC_PACKAGE_VERSION/$ALPINE_GLIBC_I18N_PACKAGE_FILENAME" && \
    apk add --no-cache \
        "$ALPINE_GLIBC_BASE_PACKAGE_FILENAME" \
        "$ALPINE_GLIBC_BIN_PACKAGE_FILENAME" \
        "$ALPINE_GLIBC_I18N_PACKAGE_FILENAME" && \
    \
    rm "/etc/apk/keys/sgerrand.rsa.pub" && \
    /usr/glibc-compat/bin/localedef --force --inputfile POSIX --charmap UTF-8 C.UTF-8 || true && \
    echo "export LANG=C.UTF-8" > /etc/profile.d/locale.sh && \
    \
    apk del glibc-i18n && \
    \
    rm "/root/.wget-hsts" && \
    apk del .build-dependencies && \
    rm \
        "$ALPINE_GLIBC_BASE_PACKAGE_FILENAME" \
        "$ALPINE_GLIBC_BIN_PACKAGE_FILENAME" \
        "$ALPINE_GLIBC_I18N_PACKAGE_FILENAME"
ENV LANG=C.UTF-8

```
保存Dockerfile。
### 构建镜像
使用上述的Dockerfile，我们已经可以开始构建镜像
```ps
$sudo docker build -t my_alpine .
```
## Dockerfile 示例二：创建一个最小化的JDK镜像
### 简述
目前JDK官方并不提供docker镜像，只有OpenJDK镜像提供，但是遇到必须使用JDK的情况时，我们就只能自己构建了。
和上个例子不同，我们使用上个例子中构建好的镜像，运用FROM命令和MAINTAINER命令
```yml
############################################################
# Dockerfile to build min JDK
# Based on my_alpine
############################################################
# Set the base image to Ubuntu
FROM my_alpine
# File Author 
MAINTAINER zhaoshg@qq.com
ENV JAVA_VERSION=8 \
    JAVA_UPDATE=121 \
    JAVA_BUILD=13 \
    JAVA_PATH=e9e7ea248e2c4826b92b3f075a80e441 \
    JAVA_HOME="/usr/lib/jvm/default-jvm"
#安装一些工具并下载JDK进行安装
#在安装完毕后删除一些不必要的文件进行精简
#删除之前安装的工具
RUN apk add --no-cache --virtual=build-dependencies wget ca-certificates unzip && \
    cd "/tmp" && \
    wget --header "Cookie: oraclelicense=accept-securebackup-cookie;" \
        "http://download.oracle.com/otn-pub/java/jdk/${JAVA_VERSION}u${JAVA_UPDATE}-b${JAVA_BUILD}/${JAVA_PATH}/jdk-${JAVA_VERSION}u${JAVA_UPDATE}-linux-x64.tar.gz" && \
    tar -xzf "jdk-${JAVA_VERSION}u${JAVA_UPDATE}-linux-x64.tar.gz" && \
    mkdir -p "/usr/lib/jvm" && \
    mv "/tmp/jdk1.${JAVA_VERSION}.0_${JAVA_UPDATE}" "/usr/lib/jvm/java-${JAVA_VERSION}-oracle" && \
    ln -s "java-${JAVA_VERSION}-oracle" "$JAVA_HOME" && \
    ln -s "$JAVA_HOME/bin/"* "/usr/bin/" && \
    rm -rf "$JAVA_HOME/"*src.zip && \
    rm -rf "$JAVA_HOME/lib/missioncontrol" \
           "$JAVA_HOME/lib/visualvm" \
           "$JAVA_HOME/lib/"*javafx* \
           "$JAVA_HOME/jre/lib/plugin.jar" \
           "$JAVA_HOME/jre/lib/ext/jfxrt.jar" \
           "$JAVA_HOME/jre/bin/javaws" \
           "$JAVA_HOME/jre/lib/javaws.jar" \
           "$JAVA_HOME/jre/lib/desktop" \
           "$JAVA_HOME/jre/plugin" \
           "$JAVA_HOME/jre/lib/"deploy* \
           "$JAVA_HOME/jre/lib/"*javafx* \
           "$JAVA_HOME/jre/lib/"*jfx* \
           "$JAVA_HOME/jre/lib/amd64/libdecora_sse.so" \
           "$JAVA_HOME/jre/lib/amd64/"libprism_*.so \
           "$JAVA_HOME/jre/lib/amd64/libfxplugins.so" \
           "$JAVA_HOME/jre/lib/amd64/libglass.so" \
           "$JAVA_HOME/jre/lib/amd64/libgstreamer-lite.so" \
           "$JAVA_HOME/jre/lib/amd64/"libjavafx*.so \
           "$JAVA_HOME/jre/lib/amd64/"libjfx*.so && \
    rm -rf "$JAVA_HOME/jre/bin/jjs" \
           "$JAVA_HOME/jre/bin/keytool" \
           "$JAVA_HOME/jre/bin/orbd" \
           "$JAVA_HOME/jre/bin/pack200" \
           "$JAVA_HOME/jre/bin/policytool" \
           "$JAVA_HOME/jre/bin/rmid" \
           "$JAVA_HOME/jre/bin/rmiregistry" \
           "$JAVA_HOME/jre/bin/servertool" \
           "$JAVA_HOME/jre/bin/tnameserv" \
           "$JAVA_HOME/jre/bin/unpack200" \
           "$JAVA_HOME/jre/lib/ext/nashorn.jar" \
           "$JAVA_HOME/jre/lib/jfr.jar" \
           "$JAVA_HOME/jre/lib/jfr" \
           "$JAVA_HOME/jre/lib/oblique-fonts" && \
    wget --header "Cookie: oraclelicense=accept-securebackup-cookie;" \
        "http://download.oracle.com/otn-pub/java/jce/${JAVA_VERSION}/jce_policy-${JAVA_VERSION}.zip" && \
    unzip -jo -d "${JAVA_HOME}/jre/lib/security" "jce_policy-${JAVA_VERSION}.zip" && \
    rm "${JAVA_HOME}/jre/lib/security/README.txt" && \
    apk del build-dependencies && \
    rm "/tmp/"*
```
保存 dockfile。

使用Dockerfile构建JDK镜像

## Dockerfile 示例三：创建一个最小的带JDK的tomcat镜像
```yml
#使用之前的JDK镜像
FROM FROM registry.cn-hangzhou.aliyuncs.com/zhaoshg1984/jdk:min

# File Author / Maintainer
MAINTAINER zhaoshg@qq.com

#set envs
ENV JAVA_HOME="/usr/lib/jvm/default-jvm"
ENV CATALINA_HOME /usr/local/tomcat
ENV PATH $CATALINA_HOME/bin:$PATH
RUN mkdir -p "$CATALINA_HOME"
WORKDIR $CATALINA_HOME
ENV TOMCAT_NATIVE_LIBDIR $CATALINA_HOME/native-jni-lib
ENV LD_LIBRARY_PATH ${LD_LIBRARY_PATH:+$LD_LIBRARY_PATH:}$TOMCAT_NATIVE_LIBDIR

RUN apk add --no-cache gnupg curl

ENV GPG_KEYS 05AB33110949707C93A279E3D3EFE6B686867BA6 07E48665A34DCAFAE522E5E6266191C37C037D42 47309207D818FFD8DCD3F83F1931D684307A10A5 541FBE7D8F78B25E055DDEE13C370389288584E7 61B832AC2F1C5A90F0F9B00A1C506407564C17A3 713DA88BE50911535FE716F5208B0AB1D63011C7 79F7026C690BAA50B92CD8B66A3AD3F4F22C4FED 9BA44C2621385CB966EBA586F72C284D731FABEE A27677289986DB50844682F8ACB77FC2E86E29AC A9C5DF4D22E99998D9875A5110C01C5A2F6059E7 DCFD35E0BF8CA7344752DE8B6FB21E8933C60243 F3A04C595DB5B6A5F1ECA43E3B7BBB100D811BBE F7DA48BB64BCB84ECBA7EE6935CD23C10D498E23
RUN set -ex; \
    for key in $GPG_KEYS; do \
        gpg --keyserver keyserver.ubuntu.com --recv-keys "$key"; \
    done

ENV TOMCAT_MAJOR 8
ENV TOMCAT_VERSION 8.0.43

# https://issues.apache.org/jira/browse/INFRA-8753?focusedCommentId=14735394#comment-14735394
ENV TOMCAT_TGZ_URL https://www.apache.org/dyn/closer.cgi?action=download&filename=tomcat/tomcat-$TOMCAT_MAJOR/v$TOMCAT_VERSION/bin/apache-tomcat-$TOMCAT_VERSION.tar.gz
# not all the mirrors actually carry the .asc files :'(
ENV TOMCAT_ASC_URL https://www.apache.org/dist/tomcat/tomcat-$TOMCAT_MAJOR/v$TOMCAT_VERSION/bin/apache-tomcat-$TOMCAT_VERSION.tar.gz.asc

RUN set -x \
    \
    && apk add --no-cache --virtual .fetch-deps \
        ca-certificates \
        tar \
        wget \
        openssl \
    && wget -O tomcat.tar.gz "$TOMCAT_TGZ_URL" \
    && wget -O tomcat.tar.gz.asc "$TOMCAT_ASC_URL" \
    && gpg --batch --verify tomcat.tar.gz.asc tomcat.tar.gz \
    && tar -xvf tomcat.tar.gz --strip-components=1 \
    && rm bin/*.bat \
    && rm tomcat.tar.gz* \
    \
    && nativeBuildDir="$(mktemp -d)" \
    && tar -xvf bin/tomcat-native.tar.gz -C "$nativeBuildDir" --strip-components=1 \
    && apk add --no-cache --virtual .native-build-deps \
        apr-dev \
        gcc \
        libc-dev \
        make \
#       "openjdk${JAVA_VERSION%%[-~bu]*}"="$JAVA_ALPINE_VERSION" \
        openssl-dev \
    && ( \
        export CATALINA_HOME="$PWD" \
        && cd "$nativeBuildDir/native" \
        && ./configure \
            --libdir="$TOMCAT_NATIVE_LIBDIR" \
            --prefix="$CATALINA_HOME" \
            --with-apr="$(which apr-1-config)" \
            --with-java-home="$JAVA_HOME" \
            --with-ssl=yes \
        && make -j$(getconf _NPROCESSORS_ONLN) \
        && make install \
    ) \
    && runDeps="$( \
        scanelf --needed --nobanner --recursive "$TOMCAT_NATIVE_LIBDIR" \
            | awk '{ gsub(/,/, "\nso:", $2); print "so:" $2 }' \
            | sort -u \
            | xargs -r apk info --installed \
            | sort -u \
    )" \
    && apk add --virtual .tomcat-native-rundeps $runDeps \
    && apk del .fetch-deps .native-build-deps \
    && rm -rf "$nativeBuildDir" \
    && rm bin/tomcat-native.tar.gz

# verify Tomcat Native is working properly


RUN sed '447 a        tail -f /dev/null' -i bin/catalina.sh \
    && set -e \
    && nativeLines="$(catalina.sh configtest 2>&1)" \
    && nativeLines="$(echo "$nativeLines" | grep 'Apache Tomcat Native')" \
    && nativeLines="$(echo "$nativeLines" | sort -u)" \
    && if ! echo "$nativeLines" | grep 'INFO: Loaded APR based Apache Tomcat Native library' >&2; then \
        echo >&2 "$nativeLines"; \
        exit 1; \
    fi

EXPOSE 8080
CMD ["catalina.sh", "start"]#
```

这种在构建过程中需要下载的情况，最好是托管到阿里云或者网易云，用他们的海外服务器来进行构建，否则会有些内容无法下载，例如上面的keyserver.ubuntu.com

具体的可以去我在[阿里云上的镜像仓库](https://dev.aliyun.com/list.html?namePrefix=zhaoshg1984)看看