# Laravel 自定义 Artisan 命令


laravel 的命令，以前就是简单的用了，并没有很仔细的用。

今天在写一个 artisan 命令的时候就踩了很多坑，其实可能就是自己以前并没有注意过，所以这次要记录下来。

```php
/**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'user:create 
                            {password : the user password}
                            {--username=admin : the user username}
                            {--email=admin@email.com : the user email}';
```

冒号后面可以写说明文字，在用 --help 的时候可以看到

在 输出 table 的时候 还得注意穿进去的 row 得是个二维数组

---

> 作者:   
> URL: http://localhost:1313/posts/laravel-custom-artisan-command/  

