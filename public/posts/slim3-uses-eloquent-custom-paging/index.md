# Slim3使用Eloquent自定义分页

最近一年私下里写东西一直都是在使用 slim 框架，其实说是一直在写东西，可是到现在真的一点输出都没有，不过小技巧倒是学会了不少，自己写个小验证器啊什么的，并且对于框架的一些理解也加深了，最好的是工作中很多都把这些小技巧用到了。

好了，吐槽了这么多，我又要说句废话了，写了一年的东西才写到分页你敢信？不过这是真的，写了一年了，代码终于累计300行了，可以进入分页的过程了。

以前用 laravel 的时候觉得 Eloquent 麻烦的很，麻烦不是指使用，而是指看代码麻烦，不过用起来还是很爽的。所以当我在使用 slim 需要连接数据库的时候毫不犹豫就继续用了 Eloquent。

当我写到分页的时候，发现调用方法失败，那么就继续引入 illuminate/pagination 好了。这下查询有问题不能正确获取页码，还有一些其他的参数也都是不对的，而且到前端的时候由于使用的模板是 twig，不能像 blade 模板那样在代码用调用 model 的相关方法，想要调用就得扩展 twig，那又是一桩麻烦事，既然如此，我就在代码中调用好了，然后在输出到模板中去，但是当我想偷懒使用 links 输出分页部分的时候报错了，在看报错，发现又得引入 blade 等一系列的东西，那引入了这么多，还不如直接使用 laravel 算了。所以，在使用 slim 不变的情况下，我们就自己扩展一个分页方法吧。

最初我是想继承 model 然后在里面写方法的，后来发现不行，不行的原因是，model代用的大部分方法其实都是在 new 一个 builder 之后链式调用的，这里我又得多说一句了奥，laravel 中同一个类名，会在很多个包下都有的，自己查看的时候要小心别看错了。（其实，集成 model 也是可以的，就是太麻烦了，而且没法链式调用）

所以在继承 model 行不通之后，我们就从 builder 入手，通过 google 和看代码可以找到这个方法
```php
/**
  * Create a new Eloquent query builder for the model.
  *
  * @param  \Illuminate\Database\Query\Builder  $query
  * @return \Illuminate\Database\Eloquent\Builder|static
  */
public function newEloquentBuilder($query)
{
    return new Builder($query);
}
```
这个方法就是 model 里面产生新 builder 用的，那我们就在我们的 model 中 覆盖这个方法就好了。
```php
<?php


namespace App\Model;

use App\Model\Builder\CustomBuilder;
use Illuminate\Database\Eloquent\Model;

class Base extends Model
{
    /**
     * Create a new Eloquent query builder for the model.
     *
     * @param \Illuminate\Database\Query\Builder $query
     * @return \Illuminate\Database\Eloquent\Builder|static
     */
    public function newEloquentBuilder($query)
    {
        return new CustomBuilder($query);
    }
}
```
注意代码里面的 builder 引用，别引用错了哦。

好了，CustomBuilder 就是我们自定义的 builder 了，
```php
<?php

namespace App\Model\Builder;

use App\Utils\Paginate;
use Illuminate\Database\Eloquent\Builder;

class CustomBuilder extends Builder
{

    public function myPaginate($perPage = 15, $columns = ['*'])
    {
        // 获取页码
        $page = Paginate::getPageNum();
        // 获取 总数 和 条目
        $results = ($total = $this->toBase()->getCountForPagination())
            ? $this->forPage($page, $perPage)->get($columns)
            : $this->model->newCollection();
        // 计算总页数
        $totalPage = $total ? ceil($total / $page) : 0;
        // 输出
        return (object)[
            'currentPage' => $page,
            'perPage' => $perPage,
            'totalCount' => $total,
            'totalPage' => $totalPage,
            'data' => $results,
        ];
    }
}
```

可以看到我们定义了一个 `myPaginate` 方法实现我们的分页，其实这么做的目的只有一个覆盖自带分页在输出时候的一些操作，输出我们想要的各种数据。接下来我们就可以在代码和模板的任何地方使用我们输出的数据，哪怕想实现 laravel 中 `links` 的方法，只要我们扩展一下 twig 就ok了，具体怎么做呢，可以等着看下一篇文章了。

---

> 作者: [M1racleHao](https://github.com/crazyhl)  
> URL: http://localhost:1313/posts/slim3-uses-eloquent-custom-paging/  

