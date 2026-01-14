---
title: "利用Jenkins+Github自动部署hugo博客"
date: 2020-04-11T19:54:07+08:00
slug: jenkins-and-github-auto-deploy-hugo-blog
---

复工了，刚上班身体在休息了40天以后还有点受不了。人就更加懒了。貌似今年总结的东西会比去年更好一些，不是思想的东西就是工作中准备用的。挺好，感觉比去年写的更有目的。这次要写的东西是准备在公司部署的，利用 Jenkins 编译公司的一些东西。利用周末的时间抓紧自己研究一下，但是从今天的总结来看，应用到公司上面可参考不多，毕竟平台不同，需要准备的也不一样。不过整体流程跑通了。起码会在以后节省一些时间。

今天要弄的时候利用 Jenkins + Github 自动部署我们的 hugo 博客。服务器快到期了，需要迁移服务器，以前的博客都是自己敲命令用 rsync 同步的。迁移就需要准备很多东西了，在让我以后同步博客，肯定是不会很开心的，既然如此，就让自动化来搞定，给自己节省更多的时间。

安装我们就不说了，可以参考官方文档，都是中文的，我是用 docker 部署到我自己的服务器上面的。插件我除了推荐的部分，我还额外安装了 golang、Publish Over SSH。Publish Over SSH 这个是我们本地要用到的，golang是准备以后用的。可以不用安装。注意，第一次安装好以后会汉化不完全，看看插件 Localization: Chinese (Simplified) 是否安装了，没有就安装一下，重启以后就汉化完全了。


点击菜单进入`系统管理` -> `系统配置`，找到 `Publish over SSH` 的部分，设置按照如图，配置好自己的参数
![图片alt](https://raw.githubusercontent.com/M1racle-Hao/blog-image/master/2020/04/11/WX20200411-201435@2x.png)
没有 `SSH Server` 的部分就点击新增就可以出现了。没有红框密码的部分，点击一下高级就有了，勾选使用密码，在输入密码。 `Name` 自己输入自己定义的名字 `Host` 和 `Username` 就是服务器的 ip 和用户名，`Remote Directory` 可以理解为操作的根目录，最后设置好以后，点击 `Test` 没问题的话就会有 `Success` 的提示。测试成功后点击保存就可以了。

接下来，选择菜单 `新建任务`，名称还是自己填写自己的任务名，接下来选择我们这里选择 `构建一个自由风格的软件项目` 大家可以按照自己的需求选择，然后点击`确定`进入下一步进入配置。
![图片alt](https://raw.githubusercontent.com/M1racle-Hao/blog-image/master/2020/04/11/WX20200411-202240@2x.png)
在源码管理，我们这边是用的 `git`，可以根据自己的需求自己选择。别忘了账号密码的配置。否则拉代码会有问题哦，因为我们是私有的项目。
![图片alt](https://raw.githubusercontent.com/M1racle-Hao/blog-image/master/2020/04/11/WX20200411-202427@2x.png)
接下来我们在 `构建触发器` 选择 `GitHub hook trigger for GITScm polling`
![图片alt](https://raw.githubusercontent.com/M1racle-Hao/blog-image/master/2020/04/11/WX20200411-202610@2x.png)
是用方法可以看文档的说明，我们使用的方法，在文档的下图位置
![图片alt](https://raw.githubusercontent.com/M1racle-Hao/blog-image/master/2020/04/11/WX20200411-202655@2x.png)
这里说的就是在`系统管理` -> `系统配置`中可以看到 hook 的 url，我们把他填写到 github 项目设置里面的 webhook 就可以了，如下两张图。

![图片alt](https://raw.githubusercontent.com/M1racle-Hao/blog-image/master/2020/04/11/WX20200411-203510@2x.png)
如果看不到，点击高级就可以了。
![图片alt](https://raw.githubusercontent.com/M1racle-Hao/blog-image/master/2020/04/11/WX20200411-203758@2x.png)
这里注意配置什么情况下载通知，我们这里只要配置在有 `push` 的时候通知就足够了。

继续回到 Jenkins 的项目配置


![图片alt](https://raw.githubusercontent.com/M1racle-Hao/blog-image/master/2020/04/11/WX20200411-202808@2x.png)

执行脚本就按照自己的配置就可以了，我这边就是下载了 `hugo` 的可执行文件，然后编译了一下。
在`构建后操作`我们有两个操作，一个是服务器那边拉取这边编译好以后的博客到服务器，用 `rsync` 拉取。
![图片alt](https://raw.githubusercontent.com/M1racle-Hao/blog-image/master/2020/04/11/WX20200411-202839@2x.png)
第二个操作是 `Delete workspace when build is done` 都弄好后清除，每次编译都全新拉取，我觉得会更安全一些，至于速度慢一些可以接受。

最后点击保存，点击`立即构建`测试一波，不出意外应该都可以部署了。

最后，如果一切顺利，这边文章，将是自动部署上去的。
最近混进了一个大神群，他们都很努力，我也得加油了，更努力的追赶大神们。
