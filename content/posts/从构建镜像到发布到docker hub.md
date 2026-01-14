---
title: "从构建镜像到发布到docker Hub"
date: 2020-02-15T11:57:35+08:00
categories: [Docker]
tags: [Docker, "Docker hub"]
slug: build-dockerfile-and-publish-to-docker-hub
---

## 前言

以前自己弄过 `docker` 镜像，不过一直都是自己在用。最近公司有了一些新的需求，正好自己可以完整的跑一边从构建镜像到发布。然后在阿里云那边弄 `k8s` 和任务啥的。阿里云那边下篇再说，我们这边就先弄镜像相关。

## 编写 DockerFile

这次我们需要弄一个 `ffmpeg` 的镜像。因为需求就是 `ffmpeg` 所以就直接用这个了，根镜像我们选用 `alpine` 因为这个更新的快，依赖处理的也好，并且容量非常小，因为我们不是从源码构建。所以选用这个是非常方便的。由于没有多余操作，所以 `DockerFile` 也非常的简单，直接看代码

```sh
FROM alpine:3.11
MAINTAINER  M1racle Hao <crazyhl@163.com>

RUN     apk  add --no-cache --update ffmpeg

CMD         ["--help"]
ENTRYPOINT  ["ffmpeg"]
```

直接一个 `apk` 的安装就都搞定了。`ENTRYPOINT` 是入口根命令, `CMD` 可以理解为一个默认的命令参数，如果我们输入自己的以后，就会覆盖这个。具体可以看 [https://yeasy.gitbooks.io/docker_practice/content/image/dockerfile/](https://yeasy.gitbooks.io/docker_practice/content/image/dockerfile/) 和 [https://www.jianshu.com/p/f0a0f6a43907](https://www.jianshu.com/p/f0a0f6a43907)。前面的文章是完整说明，后面的文章更容易懂。

最后，给大家看下目录结构，我这边的计划是根据系统版本和 `ffmpeg` 的版本来区分
![https://raw.githubusercontent.com/M1racle-Hao/blog-image/master/2020/02/15/QQ20200215-120951%402x.png](https://raw.githubusercontent.com/M1racle-Hao/blog-image/master/2020/02/15/QQ20200215-120951%402x.png)

## 构建镜像

`DockeFile` 编辑好以后，我们就可以开始构建镜像了，使用命令

```shell
docker build -t ffmpeg:apline3.11-4.21-r3 .
```

耐心等一会儿，这个网络是你懂得。所以可能得多试几次，或者自己想办法。等到输出下面这段，就说明成功了

```shell
Successfully built 766e2e917edb
Successfully tagged ffmpeg:apline3.11-4.21-r3
```

## 上传到 Docker Hub

镜像编译好以后，我们就可以准备上传了。`docker login` 以后 我们可以输入 `docker push` 想要上传的镜像，这时候是不能上传的，我们应该先给这个镜像设置一个 `tag` 指向到我们的库

```shell
docker tag ffmpeg:apline3.11-4.21-r3 crazyhl/ffmpeg:apline3.11-4.21-r3
```

然后这时候我们在 `push`，执行

```shell
docker push crazyhl/ffmpeg:apline3.11-4.21-r3
```

网络依旧要自己搞定哦。等待上传好我们就可以去自己的首页看到了。

## 绑定 github 
这部分我们就可以配置好，自己的 `github` 然后当我们修改文件以后，去docker hub 就可以自己配置编译了。这部分可以参考 [https://blog.csdn.net/chang_li/article/details/81288724](https://blog.csdn.net/chang_li/article/details/81288724) ，这个文章有些过时，但是可以参照着弄。