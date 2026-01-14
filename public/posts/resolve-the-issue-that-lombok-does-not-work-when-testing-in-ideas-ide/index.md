# 解决 Ideas Ide 内点击测试按钮 Lombok 不生效的问题


最近转战 java 了，于是使用 springboot 快速的创建项目，在项目中使用 lombok 注解来自动生成 getter/setter 和 construct 相关方法。

此时，我发现当我在 idea 运行 test 的时候 target 中没有根据注解生成相关的方法，导致测试失败。

![Pasted image 20250110144403](https://raw.githubusercontent.com/M1racle-Hao/blog-image/master/20250115103450243.png)

但是当我在 mevan 中运行 test 的时候，却没有任何问题。
![Pasted image 20250110144441](https://raw.githubusercontent.com/M1racle-Hao/blog-image/master/20250115103450244.png)

通过 gpt 以及百度、google 了很多方案，却没有解决，因为这个其实是配置造成的，我对于 ideas 的配置极度自信，让我浪费了很多时间，所以我决定记录一下这个问题的解决方案。

![Pasted image 20250110144743](https://raw.githubusercontent.com/M1racle-Hao/blog-image/master/20250115103450245.png)

这个图是我创建测试项目的时候的配置，没有配置 `processor path`。而是通过项目的路径获取的，但是通过 spring-starter 创建的项目却不是这样的
![](https://raw.githubusercontent.com/M1racle-Hao/blog-image/master/20250115104056291.png)

这个是我已经修复了的配置，可以看到是使用了 `processor path` 配置的。

但是配置的路径 `Users/xxx/.m2/repository/org/projectlombok/lombok/unknown/lombok-unknown.jar` 
不知道为什么版本 `unknown` 了。所以，在 ide 内运行 test 就失效了。

那么这里我们可以使用两个方法，一个是手动修改正确 `processor path` 的路径，另一个是使用 `Obtain processors from project classpath` 这个配置就完事了。

---

> 作者:   
> URL: http://localhost:1313/posts/resolve-the-issue-that-lombok-does-not-work-when-testing-in-ideas-ide/  

