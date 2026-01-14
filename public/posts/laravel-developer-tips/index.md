# 2022年11月21日 Laravel 开发中的几个小注意点


## 添加语言
[https://publisher.laravel-lang.com/](https://publisher.laravel-lang.com/) 完全按照这边的文档来即可。
文档有两个 https://laravel-lang.com/ 这里面就没有之前的写的详细

## 使用 sail 开发的时候 npm run dev 文件无法加载
当我们在使用 sail 开发的时候 `npm run dev` 后，本地却没法找到 css 文件。此时修改 `vite.config.js` 文件，添加 
```js
server: {
    hmr: {
        host: 'localhost',
    }
},
```
这段即可正常使用了。具体可以看下图
![](https://www.cimple.ink/static/images/202211211432688.png)

## 关于共性问题
最近在弄的东西很多的使用方式都是在参照 [https://filamentphp.com/](https://filamentphp.com/)。
但是代码都是按照我的想象去写了，也因此走了很多弯路，大概写了一周多，才有了这么一个卖出第一步版本。很难，但很快乐。
就在今天上午解决一个样式按需加载探测路径问题，让我十分开心的时候，我刚才看到了 filament 也是同样的解决方案，看来，方案都一样，关键是得有想法。有了想法，去实现就好了，最怕大脑一切空白。

---

> 作者: [M1racleHao](https://github.com/crazyhl)  
> URL: http://localhost:1313/posts/laravel-developer-tips/  

