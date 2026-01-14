# Slim3 自定义twig模板扩展方法

前几天我们自定义了模型的分页，但是在输出到模板的时候会发现生成模板也是个重复的操作，既然如此，我们就扩展个twig模板的方法，这样以后当我们要生成分页的时候就只需要调用这个方法就可以了，毕竟人懒，麻烦一次以后都舒服。上个图先

![](//up-cdn.cimple.ink/2019/07/13/46a40014-e281-514c-81cd-6be3a511d425.png)
![](//up-cdn.cimple.ink/2019/07/13/94cc0d83-4c51-5a80-b897-228d899270da.png)

这里我们一共做了两个样式，一个是比较传统的样式，一个是 laravel 默认的样式，都挺好看，想用哪个用哪个。

参考文档 [https://twig.symfony.com/doc/2.x/advanced.html](https://twig.symfony.com/doc/2.x/advanced.html)

源码如下
```php
<?php


namespace App\Twig\Extension;

use Twig\Environment;
use Twig\Extension\AbstractExtension;

/**
 * twig 模板分页扩展
 * Class Paginate
 * @package App\Twig\Extension
 */
class Paginate extends AbstractExtension
{
    public function getFunctions()
    {
        return [
            new \Twig\TwigFunction('links', [$this, 'links'], ['needs_environment' => true]),
            new \Twig\TwigFunction('links2', [$this, 'links2'], ['needs_environment' => true]),
        ];
    }

    public function links(Environment $env, $data, $showTotal = true, $extraClass='', $eachSideCount = 2, $pageName = 'page', $template = 'paginate/links.html')
    {
        $links = $this->generateLinkItem($data['currentPage'], $data['totalPage'], $data['path'], $eachSideCount, $pageName);
        $env->load($template)->display(compact('data', 'links', 'showTotal', 'extraClass'));
    }

    public function links2(Environment $env, $data, $showTotal = true, $extraClass='', $eachSideCount = 3, $pageName = 'page', $template = 'paginate/links.html')
    {
        $links = $this->generateLinkItem2($data['currentPage'], $data['totalPage'], $data['path'], $eachSideCount, $pageName);
        $env->load($template)->display(compact('data', 'links', 'showTotal', 'extraClass'));
    }

    private function generateLinkItem($currentPage, $totalPage, $path, $eachSideCount = 2, $pageName = 'page')
    {
        $totalCount = $eachSideCount * 2 + 1;
        $start = $currentPage - $eachSideCount;
        $end = $currentPage + $eachSideCount;
        if ($totalPage <= $totalCount) {
            $start = 1;
            $end = $totalPage;
        } else {
            if ($currentPage <= $eachSideCount) {
                $start = $currentPage - abs($eachSideCount - $currentPage - 1);
                $end = $currentPage + $eachSideCount + ($eachSideCount - $currentPage + 1);
            }

            if ($currentPage + $eachSideCount > $totalPage) {
                $start = $currentPage - $eachSideCount - ($currentPage + $eachSideCount - $totalPage);
                $end = $currentPage + $eachSideCount - ($currentPage + $eachSideCount - $totalPage);
            }
        }
        $links = [];

        if ($totalPage > $totalCount) {
            if ($currentPage > $eachSideCount + 1) {
                $links[] = $this->generateLinkData('首页', $path, 1, $currentPage, false, null, $pageName);
            }

            $links[] = $this->generateLinkData("«", $path, ($currentPage == 1 ? 1 : ($currentPage - 1)), $currentPage, ($currentPage == 1 ? true : false), false, $pageName);
        }


        for ($i = $start; $i <= $end; $i++) {
            $links[] = $this->generateLinkData($i, $path, $i, $currentPage, false, null, $pageName);
        }

        if ($totalPage > $totalCount) {
            $links[] = $this->generateLinkData("»", $path, ($currentPage == $totalPage ? $totalPage : ($currentPage + 1)), $currentPage, ($currentPage == $totalPage ? true : false), false, $pageName);

            if ($totalPage - $currentPage > 2) {
                $links[] = $this->generateLinkData('末页', $path, $totalPage, $currentPage, false, null, $pageName);
            }
        }

        return $links;
    }


    private function generateLinkItem2($currentPage, $totalPage, $path, $eachSideCount = 3, $pageName = 'page')
    {
        $totalCount = $eachSideCount * 2 + 6;
        $start = ($currentPage - $eachSideCount < 1) ? 1 : $currentPage - $eachSideCount;
        $end = ($currentPage + $eachSideCount > $totalPage) ? $totalPage : $currentPage + $eachSideCount;

        $links = [];
        $links[] = $this->generateLinkData("«", $path, ($currentPage == 1 ? 1 : ($currentPage - 1)), $currentPage, ($currentPage == 1 ? true : false), false, $pageName);

        if ($totalPage <= $totalCount) {
            $start = 1;
            $end = $totalPage;
            for ($i = $start; $i <= $end; $i++) {
                $links[] = $this->generateLinkData($i, $path, $i, $currentPage, false, null, $pageName);
            }
        } else {

            if ($currentPage < $eachSideCount * 2 + 2) {
                for ($i = 1; $i <= $eachSideCount * 2 + 2; $i++) {
                    $links[] = $this->generateLinkData($i, $path, $i, $currentPage, false, null, $pageName);
                }
                $links[] = $this->generateLinkData('...', false, '', $currentPage, true, null, $pageName);
                for ($i = $totalPage - 1; $i <= $totalPage; $i++) {
                    $links[] = $this->generateLinkData($i, $path, $i, $currentPage, false, null, $pageName);
                }
            } else if ($currentPage > $totalPage - ($eachSideCount * 2 + 2)) {
                for ($i = 1; $i <= 2; $i++) {
                    $links[] = $this->generateLinkData($i, $path, $i, $currentPage, false, null, $pageName);
                }
                $links[] = $this->generateLinkData('...', false, '', $currentPage, true, null, $pageName);
                for ($i = $totalPage - ($eachSideCount * 2 + 2); $i <= $totalPage; $i++) {
                    $links[] = $this->generateLinkData($i, $path, $i, $currentPage, false, null, $pageName);
                }
            } else {
                for ($i = 1; $i <= 2; $i++) {
                    $links[] = $this->generateLinkData($i, $path, $i, $currentPage, false, null, $pageName);
                }
                $links[] = $this->generateLinkData('...', false, '', $currentPage, true, null, $pageName);
                for ($i = $start; $i <= $end; $i++) {
                    $links[] = $this->generateLinkData($i, $path, $i, $currentPage, false, null, $pageName);
                }
                $links[] = $this->generateLinkData('...', false, '', $currentPage, true, null, $pageName);
                for ($i = $totalPage - 1; $i <= $totalPage; $i++) {
                    $links[] = $this->generateLinkData($i, $path, $i, $currentPage, false, null, $pageName);
                }
            }

        }

        $links[] = $this->generateLinkData("»", $path, ($currentPage == $totalPage ? $totalPage : ($currentPage + 1)), $currentPage, ($currentPage == $totalPage ? true : false), false, $pageName);

        return $links;
    }

    private function generateLinkData($name, $path, $index, $currentPage, $isDisable = false, $isActive = null, $pageName = 'page')
    {
        return [
            'name' => $name,
            'url' => $path . (stripos('?', $path) ? '&' : '?') . $pageName . '=' . $index,
            'isDisabled' => $isDisable,
            'isActive' => $isActive === null ? (($index == $currentPage) ? true : false) : $isActive,
            'isLink' => $path === false ? false : true,
        ];
    }
}
```

好了，说一下重要的部分 `['needs_environment' => true]` 我们需要加上这个参数，原因是我们在代码中使用twig实例生成模板。剩下的也没啥难度了，就是单纯的生成链接节点了。

好了，我们在贴一下模板部分
```html
<div>
    <ul class="pagination pagination-sm m-0 {{extraClass}}">
        {% if showTotal %}
        <li class="page-item disabled"><a class="page-link" href="#">共 {{data['totalCount']}} 条 {{data['totalPage']}} 页</a></li>
        {% endif %}
        {% for link in links %}
        <li class="page-item{% if link['isDisabled'] %} disabled{% endif %}{% if link['isActive'] %} active{% endif %}">
            {% if link['isLink'] %}
            <a class="page-link" href="{{link['url']}}">{{link['name']}}</a>
            {% else  %}
            <span class="page-link">{{link['name']}}</span>
            {% endif %}
        </li>
        {% endfor %}
    </ul>
</div>
```

看，模板就这么少，麻烦的就是生成节点的部分了，需要好好考虑。这里面很多代码都是参考了 laravel 的 `pagination` 部分。

对了，分页部分我们也加了一些参数，我也一起贴上来好了
```php
class CustomBuilder extends Builder
{

    public function myPaginate($perPage = 15, $columns = ['*'])
    {
        // 获取页码
        $page = Paginate::getPageNum();
        // 调用 builder 自带的方法获取总条数
        $results = ($total = $this->toBase()->getCountForPagination())
            ? $this->forPage($page, $perPage)->get($columns)
            : $this->model->newCollection();

        $totalPage = $total ? ceil($total / $page) : 0;

        $path = $basePath = Paginate::getBasePath();
        $queryParams = Paginate::getQueryParams();
        unset($queryParams['page']);

        $queryStr = http_build_query($queryParams);
        if ($queryStr) {
            $path .= '?' . $queryStr;
        }

        return [
            'currentPage' => $page,
            'perPage' => $perPage,
            'totalCount' => $total,
            'totalPage' => $totalPage,
            'data' => $results,
            'basePath' => $basePath,
            'queryStr' => $queryStr,
            'path' => $path,
        ];
    }
}
```

增加的部分仅仅是提供生成模板的时候用的。

好了，分页至此通过两篇文章就都搞定了。代码还得优化，自己看的都难受，等以后优化后会继续写的。

---

> 作者: [M1racleHao](https://github.com/crazyhl)  
> URL: http://localhost:1313/posts/slim3-custom-twig-template-extension-method/  

