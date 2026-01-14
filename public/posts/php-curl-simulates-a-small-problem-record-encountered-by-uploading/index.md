# Php Curl 模拟上传遇到的小问题记录


今天在公司写东西的时候遇到了一个奇怪的小问题，就是在用 curl 模拟文件上传的时候发现上传失败，最初始采用的代码如下

```php
[
    'file' => '@' . $fileName
]
```

通过查看文档发现了下面的一段说明

![图片alt](https://www.cimple.ink/images/2018/02/28/fzbbyyirm4vqvp3ywkl7h2m15mpr4w5b.png)

从这可以得知，`CURLOPT_SAFE_UPLOAD ` 这个参数可以控制，由于 `php 5.5` 以前的默认值是 `false` ，所以我可能以前并没有关注，但是在 `php 5.6` 的时候这个默认值是 `true` 了，所以当我们想要采用上面那种方式就需要设置 `CURLOPT_SAFE_UPLOAD ` 这个值为 `false` 才可以使用。

但是需要注意的是在 `php 7` 以后这个属性被删除了，所以当我们想要上传的时候，必须采用 `CURLFile` 才可以上传。具体可以参见 [http://php.net/manual/zh/class.curlfile.php](http://php.net/manual/zh/class.curlfile.php)

所以现在就不要考虑 `CURLOPT_SAFE_UPLOAD ` 这个参数了，以后在模拟上传的时候直接采用 `CURLFile` 就可以了


---

> 作者:   
> URL: http://localhost:1313/posts/php-curl-simulates-a-small-problem-record-encountered-by-uploading/  

