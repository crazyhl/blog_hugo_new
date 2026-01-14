# Composer 源码小记录


话说在住院的时候，想起来了以前看过的一个帖子有说在加载 psr4 的时候记录的 namespace 的长度是用来替换路径用的。

但是在今天中午去看加载源码的时候发现已经没有这个替换了。于是赶紧翻看 github 上面的改动记录，发现已经不使用那种方式了，而是采用截取的方式去获取文件名什么的。然后路径就直接使用现存的了。

代码如下图

![图片alt](https://www.cimple.ink/images/2018/03/20/cdd27610e187292157e7744e6ac0d034.png)

估计等在过一段那个 namespace 的长度也可以不用了，但是看改动记录，还是存在，静待观察


---

> 作者:   
> URL: http://localhost:1313/posts/composer-source-code-record/  

