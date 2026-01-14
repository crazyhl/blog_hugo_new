# Laravel Livewire 使用笔记（5）最终篇


## 前言
一不小心已经写了5篇关于 `livewire` 的东西了。今天这个也到了最后一篇了，我们把登录注册退出都完善好就可以结束这个系列了。后续我在研究点东西后，就准备开新坑了。

## 一鼓作气

### 文章列表
```php
<div>
    {{-- In work, do what you enjoy. --}}
    <div>
        <livewire:auth.logout />
    </div>
    <div class="p-5">
        @foreach($posts as $post)
            @livewire('post.show', ['post' => $post])
        @endforeach
        {{ $posts->links() }}
    </div>
</div>
```
注意这行 `@livewire('post.show', ['post' => $post])`  之前我们用的是 `@include`。虽然样式没问题，但是如果使用 `Action` 就有问题了，会提示找不到方法，所以用 `livewire` 提供的方法。
### 登录部分
```php
<?php

namespace App\Http\Livewire\Auth;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;
use Livewire\Component;

class Login extends Component
{
    public $email;
    public $password;
    public bool $rememberMe = true;

    protected $rules = [
        'email' => 'required|email',
        'password' => 'required',
    ];

    public function updated($propertyName)
    {
        $this->validateOnly($propertyName);
    }

    public function login(Request $request)
    {
//        dd($this->validate());
        if (Auth::attempt($this->validate(), $this->rememberMe)) {
            $request->session()->regenerate();

            return redirect()->intended('/post/show');
        }

        $this->addError('email', 'The provided credentials do not match our records.');
    }

    public function render()
    {
        return view('livewire.auth.login');
    }
}
```
注意看，这里需要验证的和不要验证的是分开的。也就是 `rememberMe` 那个属性。

## 总结

最后3篇，为什么写这么短呢？因为本身并不复杂，而且更多的是 `laravel` 的东西了，所以我不太好说，大家多看文档吧。这三篇更多像是胶水。把想要的东西合并在一起。最后这个东西真的不复杂，自己看看代码吧。

demo 项目地址：[livewire-demo](https://github.com/crazyhl/livewire-demo)

---

> 作者:   
> URL: http://localhost:1313/posts/laravel-livewire-note-5/  

