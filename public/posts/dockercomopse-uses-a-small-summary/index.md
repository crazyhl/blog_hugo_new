# Docker-Comopse 使用小总结


说句题外话，东西还得多用才能越来越明白啊。话说，今天休息在家用windows 的台式机弄docker，用来配置在台式机的开发环境，这次在台式机的配置目的就是在本地主机尽量不装任何开发用的相关软件，比如nginx、mysql、php这种。对了，这还得说句题外话，如果啥都不装，我连博客都写不了，不过为了台式机的干净，我依然选择在打开笔记本写博客。

本地学习到的知识，主要原因是我我加了个node镜像用来编译vue写的代码。原本我都是使用`dokcer-compose up --build` 这种方式编译加启动容器的，以前看 `laradock` 文档的时候里面就没用过`docker-compose --build` 编译加启动的方案，而是用 `docker-compose build xxx` 然后在`docker-compose up xxx` 这种方式，我以前始终不解，今天终于明白了，如果用 `dokcer-compose up --build` 这种方式操作，会由于 `node` 镜像不会持久化启动而反复报错，如果先 `build` 在根据需要 `up` 就不会造成这种问题了。当然如果配置了`restart: "no"`之后也就启动一次然后退出，也并没有什么关系，不过为我还是选择遵从`laradock`的方案，毕竟那么多人的经验还是值得参考的。

当我们需要使用`node` 的时候，就是用 `docker-compose run --rm node bash` 这种就可以进入到容器内部了，加`--rm`的原因是启动以后我们还得手工销毁，加了以后当我们退出的时候就自动销毁`container`了，而且我们在`compose` 的文件中配置了 `node` 的 `restart: "no"` 这样当我们停止以后也不会自己在跑起来。

最后还得吐槽一下windows 的 crlf会让docker启动失败啊，很难受，把sublime配置为lf之后才解决。

---

> 作者: [M1racleHao](https://github.com/crazyhl)  
> URL: http://localhost:1313/posts/dockercomopse-uses-a-small-summary/  

