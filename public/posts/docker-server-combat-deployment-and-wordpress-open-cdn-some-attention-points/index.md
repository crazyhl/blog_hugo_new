# Docker服务器实战部署以及wordpress开启cdn的一些注意点



虽然本地docker已经弄完了好几个月了，在最近也在持续的优化中，包括把能配置的部分转移到 `env`文件中。
上周把自己的博客系统废弃了，因为当时写这个系统也仅仅是为了 `laravel`的学习而已，用了将近2年，虽然很稳定，但是也有很多不满意的地方，虽然现在换到了 `wordpress`也有一些不满意的地方，但是还好插件丰富，周边也很多，可以慢慢改造成自己满意的样子，下面主要就是记录一下这次服务器实战部署的踏坑情况。

* `mysql`如果使用 `8`及以上的版本的时候，需要进入 `msyql`容器内部，修改一下密码方案，否则数据库连不上</li>
* 注意代码文件夹写入权限问题，目前 `docker`设置的用户是 `www`，所有如果不是 `www`用户的话，可能会导致无法写入文件的</li>
* `wordpress`开启又拍 `cdn`以后样式无法加载，在又拍后台设置 `缓存控制-&gt;参数跟随-&gt;全程跟随后`之后就ok了。</li>
* 目前写代码还是比较习惯 `markdown`，所以目前日志都会在印象笔记写好，然后在转换，由于代码高亮方便跟 `wordpress`还有点不一致，所以自己写了个简单的转换器。</li>

现在还有的小问题就是图片上传了，慢慢在弄一个图片上传工具就ok了。


---

> 作者: [M1racleHao](https://github.com/crazyhl)  
> URL: http://localhost:1313/posts/docker-server-combat-deployment-and-wordpress-open-cdn-some-attention-points/  

