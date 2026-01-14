# 升级 Homestead 小记录


![](https://laravel.com/assets/img/components/logo-homestead.svg)


又是一个周末，自觉状态调整的不错了，于是就像弄点东西玩，然后，就想到了升级相关版本，vagrant 和 virtual box 都升级完了，然后我又升级了 homestea 的脚本，并且切换到了最新的 release tag 上，最后执行 vagrant box update，会提示 `==> homestead-7: Box 'laravel/homestead' not installed, can't check for updates.` 这个么个东西，在网上简单的看了一下也没发现什么

最后灵机一动，切换会原来的 tag，然后再升级 box 再换到最新的 tag 就 ok 了。

哈哈，太机智了，记录一下顺序吧，以后就不会出问题了，先升级 box，然后升级 homestead 脚本，最后切换到脚本最新的 release 的 tag 就大功告成了，继续写代码

对了，升级以后还得重新配置一下 composer 中国镜像


---

> 作者: [M1racleHao](https://github.com/crazyhl)  
> URL: http://localhost:1313/posts/%E5%8D%87%E7%BA%A7-homestead-%E5%B0%8F%E8%AE%B0%E5%BD%95/  

