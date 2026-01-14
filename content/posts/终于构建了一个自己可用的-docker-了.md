---
title: 终于构建了一个自己可用的 docker 了
tags: [docker]
categories: [技巧]
slug: i-finally-built-a-docker-that-can-use-myself
date: 2017-07-01 12:15:00
updated: 2017-07-01 12:15:00
toc: false
---

话说，想学习 docker 很久了，一直以来总是没有时间去实践，其实也不是没有时间，而是一直没有一个好的时机去实践。 这周重做了系统了以后就忘记弄 vagrant 了。然后昨天就想用心的看一下 laravel 的源码，以加深自己的理解。但是发现本地没有 vagrant 环境，正好趁机搞一下 docker 了，于是就搞了起来。

主要参考了下面这两篇文章

[https://segmentfault.com/a/1190000008833012](https://segmentfault.com/a/1190000008833012) 
[https://segmentfault.com/a/1190000008822648#articleHeader44](https://segmentfault.com/a/1190000008822648#articleHeader44) 


另外再进行 docker 写东西的时候，发现数据库连接不上了，所以查了一下，链接数据库的不用填写 localhost 或者 ip，直接填写 docker 的名字就 ok 了。这样就可以连接上了。

还有想说了，不知道咋说了，先学习吧。就这样

