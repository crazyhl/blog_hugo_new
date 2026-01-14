# Laravel Passport 用docker测试时候遇到的坑


最近在学习 laravel `psssport` ，可是在本地使用 docker 测试的时候却遇到了一个报错。

```php
cURL error 7: Failed to connect to movielaravel.test port 80: Connection refused (see http://curl.haxx.se/libcurl/c/libcurl-errors.html)
```

想了很久，一开始以为是内循环了，但是使用 postman 却可以得到正确的结果，后来想到，是不是因为 docker 容器内部无法解析到 我的测试域名呢，搜索了一波，在 docker-compose 文件中配置了 `extra_hosts` 问题的到解决，难受，活生生被阻挡了2个小时。

在需要内部调用的地方，增加配置，示例如下
```php
extra_hosts:
      - "movielaravel.test:172.28.12.3"
```

这样，就在docker的内部，做好host的映射了，也就能够正常访问了


---

> 作者:   
> URL: http://localhost:1313/posts/laravel-passport-pits-encountered-when-testing-with-docker/  

