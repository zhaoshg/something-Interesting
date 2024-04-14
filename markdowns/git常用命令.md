



### 克隆

```shell
 git clone git@github.com:zhaoshg/zhaoshg.github.io.git
```



### 查看分支
```shell
git branch  	#查看本地分支
git branch -r 	#查看远程分支
git branch -a	#查看所有分支(本地+远程)
```



### 分支切换

```shell
#将远程debug分支下载至本地debug分支
git checkout origin debug:debug
#将远程debug分支下载至本地debug分支
git checkout -b master origin/master
#切换到本地debug分支
git checkout debug
```



### Fetch && pull

- git fetch : 相当于是从远程获取最新版本到本地，不会自动merge
- git pull：相当于是从远程获取最新版本并merge到本地

：相当于是从远程获取最新版本到本地，不会自动merge

```shell
#从远程的origin的master主分支下载最新的版本到origin/master分支上
git fetch origin master
#比较本地的master分支和origin/master分支的差别
git log -p master..origin/master
#进行合并
git merge origin/master
```

等价于

```shell
git fetch origin master:tmp
git diff tmp 
git merge tmp
```



### 远程debug分支覆盖本地debug分支

```shell
#先切换到master分支
git checkout master
#删除debug分支
git branch -D debug
#再将远程debug分支拉到本地debug分支，如果本地不存在，则新建
git fetch origin debug:debug
```



