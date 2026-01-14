# 计算本周一的时间戳


前一段有一个计算本周一时间戳的需求，于是用了下面的代码
```php
strtotime('previous monday');
```

原本一直运行都很好，可是还是没经过深入的测试，在周一的时候，这个值会计算到上一个周一的时间，导致了我们的一个显示出现了问题,但是还不能使用下面的代码
```php
strtotime('monday');
```

因为这样会导致在非周一的时候计算到下一个周一，

所以我们要在计算之前先判定一下当前是周几，如果是周一就采用 `'monday'` 非周一的时候采用`'previous monday'`

---

> 作者:   
> URL: http://localhost:1313/posts/calculate-the-timestamp-of-this-monday/  

