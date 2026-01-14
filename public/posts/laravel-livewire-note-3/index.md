# Laravel Livewire 使用笔记（3）


![](https://www.cimple.ink/static/images/laravel-livewire.png)

## 前言
上一篇我们完成了文章列表的显示，不过还没有分页。今天我们就来搞定分页这部分就好了。

## 开始

分页还是比较简单的，只需要一步即可（注意，这里我们没有考虑很复杂的场景，仅仅是分页。复杂的东西后面如果有场景会考虑弄一下。）
```php
class Pagination extends Component
{
    use WithPagination;

    public function render()
    {
        return view('livewire.post.pagination', [
            'posts' => Post::paginate(10),
        ]);
    }
}
```
其实就是把上一篇的 `all` 方法，换成了 `paginate` 方法，并且引入了 `WithPagination Trait` 即可。
前端加上
```php
{{ $posts->links() }}
```
这样一个简单的分页就完成了。

## 还不够
这边分页的样式我们并不是很满意，可能需要更丰富的定义需求。
```php
./vendor/bin/sail artisan livewire:publish --pagination
```
把模板文件发布出来。这样我们改动本地模板文件就可以达到自定义样式的目的了。

现在样式可以自定义了，那么语言还是英文，那么还得调整一下语言。这里就很简单了。
首先安装包
```php
./vendor/bin/sail composer require laravel-lang/publisher laravel-lang/lang laravel-lang/attributes --dev
```
添加语言
```php
./vendor/bin/sail php artisan lang:add zh-CN  
```
最后修改 `app.config` 中的 `lang` 字段，改为 `zh_CN` 就ok了。

## 最后
别看这篇很短，但是上周末我可以看了很多文档的。所以要自己多看文档，我这个仅仅算是个引导，光看引导是没用的。要自己多看才行。

下一篇想弄表单验证。以及把 event 也能用上了。那就下一篇再说。

demo 项目地址：[livewire-demo](https://github.com/crazyhl/livewire-demo)

---

> 作者:   
> URL: http://localhost:1313/posts/laravel-livewire-note-3/  

