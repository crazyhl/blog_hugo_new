# Laravel5.1 自定义分页样式的探索

题外话：查东西真心不能用百度，得用google，如果提前查google的话，也许这篇文章就不是自己写的了，可能就是转载的了，不过自己搞出来的东西记忆比较深刻，也是不错的，哈哈。下面进入正文。


话说，自己在写这个博客的时候要弄到分页，按照文档 `rander()` 出来以后居然是bootstrap的样式，这是什么鬼。百度了一圈之后，很多文章都是说改源码，这一定不科学。对于框架来说一定是可以扩展的，所以，开始爬源码吧。当我们调用 `paginate()` 方法的时候返回的是一个 `\Illuminate\Contracts\Pagination\LengthAwarePaginator` 实例，不过这个东西是个接口，肯定会有实现类的，所以，继续查，发现了 `Illuminate\Pagination\LengthAwarePaginator` 这个类实现了上边的接口，好吧，就决定是你了，皮卡丘。

继续看 `render` 方法

```php
    /**
     * Render the paginator using the given presenter.
     *
     * @param  \Illuminate\Contracts\Pagination\Presenter|null  $presenter
     * @return string
     */
    public function render(Presenter $presenter = null)
    {
        if (is_null($presenter) && static::$presenterResolver) {
            $presenter = call_user_func(static::$presenterResolver, $this);
        }

        $presenter = $presenter ?: new BootstrapThreePresenter($this);

        return $presenter->render();
    }
```

上面的最终返回了一个BootstrapThreePresenter，进去看一下，发现就是渲染分页样式的，所以说，我们只要自己定义一个分页样式类，并且传入render方法，就可以了。

下面我们自定义一个 `AmazeuiThreePresenter` 类，继承自BootstrapThreePresenter，因为这样就可以节省好多方法了。
下面上最终代码

```php
<?php

namespace App;

/**
 * Created by PhpStorm.
 * User: Chris
 * Date: 2016/8/21
 * Time: 20:34.
 */
class AmazeuiThreePresenter extends \Illuminate\Pagination\BootstrapThreePresenter
{
    public function render()
    {
        if ($this->hasPages()) {
            return sprintf(
                '<ul class="am-pagination am-pagination-centered">%s %s %s</ul>',
                $this->getPreviousButton(),
                $this->getLinks(),
                $this->getNextButton()
            );
        }

        return '';
    }

    /**
     * Get HTML wrapper for disabled text.
     *
     * @param string $text
     *
     * @return string
     */
    protected function getDisabledTextWrapper($text)
    {
        return '<li class="am-disabled"><span>'.$text.'</span></li>';
    }

    /**
     * Get HTML wrapper for active text.
     *
     * @param string $text
     *
     * @return string
     */
    protected function getActivePageWrapper($text)
    {
        return '<li class="am-active"><span>'.$text.'</span></li>';
    }
}

```

这样就ok了，好多方法 `BootstrapThreePresenter` 都实现出来了，所以我们只要修改一些对于我们样式相关的就可以了。

最后，我们在模板render的时候传入我们自定义的就可以了。如下

```php
 {!! $articles->render(new \App\AmazeuiThreePresenter($articles)) !!}
```

最后的最后，再写这个文章的时候，发现了 `static::$presenterResolver` 这个东西，是不是还有更简单的方法呢，待我研究研究

---

> 作者: [M1racleHao](https://github.com/crazyhl)  
> URL: http://localhost:1313/posts/laravel51-exploration-of-custom-paging-style/  

