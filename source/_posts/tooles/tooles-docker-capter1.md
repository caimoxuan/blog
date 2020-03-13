---
title: 用docker来简化本地环境
date: 2019-10-27 15:14:53
thumbnail: /gallery/thumbnail/docker.jpg
categories:
    - 工具介绍
tags:
    - 推荐
    - tools
---

&emsp;&emsp;随着应用系统的开发越来越复杂，所需要使用的中间件越来越多，可能在有内网环境的时候可以使用公用的开发资源，但是在没有内网环境的时候，想要调试程序，要么屏蔽中间件在代码中的依赖，要么就在本地安装并启动中间件。
&emsp;&emsp;如果程序只用到了一个mysql，那么安装了倒也不妨，万一应用程序还需要zookeeper、mongodb、redis等等中间件，你的电脑上就需要装一大堆的中间件，然后在调试程序的时候一一启动，最后启动程序开始调试。个人觉得这样的方式有点麻烦，而如果使用docker的话，只要安装一个docker，剩下的中间件就是用docker镜像的方式来启动，再也不用一个个的去下载中间件了。

<!-- more -->

## docker的安装
> docker在目前的unix内核的系统中是直接支持的（linux、mac），而向我这样的开发者毕竟经费有限，用的windows，就需要费点周折。
> 首先，在写这篇文章的时候，docker只对window10做了支持，因为docker也是一种虚拟化技术，window10才有hyper-v功能的支持，于是如果window10以下的系统，暂时是不能太舒服的用docker。
windows方式安装docker desktop就像安装一般应用一样，安装完成的时候要注意开启hyper-v服务（自行百度）,然后启动dcoker服务（它会在任务栏右侧或是隐藏图标中出现一个小鲸鱼）的本地磁盘映射，勾选要映射的磁盘（不需要持久化数据的可以不用）。
linux下docker的安装有很多方式，yum安装比较简单的，使用命令(他也是用的yum))
``` bash
wget -qO- https://get.docker.com | sh
```
启动docker：
``` bash
service docker start
```
查看docker信息
``` bash 
docker info
```
输出docker版本号安装完毕，接下来开始配置自己的开发环境中间件。

## 配置一个docker的redis容器

1. 拉取相关中间件的镜像，一般来说都是使用latest版本，如果没有上面特别的版本要求（elasticSearch的版本要注意，各个版本连接差异比较不同），这里以redis为例(要有网络):
``` bash
docker pull redis
```
2. 查看当前docker中所有的镜像，不出意外的话刚刚的redis镜像也在其中了：
``` bash 
docker images
```
3. 启动镜像：
``` bash
docker run -p 6379:6379 -v D:/data/redis:/data -d redis:latest 
```
这里的几个参数：
> -p 绑定端口 -p 本机端口:docker端口
> -v 绑定存储卷 -v 本机目录:docker目录
> -d 以后台方式运行 返回一个容器id
> 其他相关参数可以参考`菜鸟教程`
4. 查看启动的镜像:
``` bash
docker ps
```
这时应该可以看到刚刚启动的redis，之后想要的操作都需要使用显示的id来操作（也可以在启动的时候-name指定名称操作）;
5. 关闭启动的镜像（关机也会关闭）
``` bash
docker stop container_id(docker ps 显示的container_id)
```
6. 下次重新启动，镜像被关闭了或者电脑重启了之后：
``` bash
docker ps -a //查看镜像（无论运行与否） 找对应的container_id

docker start $container_id //启动容器，这个容器和上次配置的命令是一样的
```

## 总结
&emsp;&emsp;这样就完成了一个reids的docker本地docker服务了，该容器启动的时候就可以像使用安装的redis一样的时候reids。如果想要其他的中间件，步骤也是一样的，如果想要本地配置文件启动，也是可行的。之后开发的时候，只要看一眼之前配置的容器，启动就可以了，个人觉得这样相对来说方便了许多。