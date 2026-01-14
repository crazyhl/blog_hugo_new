# Laravel Livewire 使用笔记（4）


## 前言
上一篇我们搞定了分页，原本今天想搞定表单验证相关的。但是想想为什么不趁着表单验证把用户登录注册自己实现一次呢？就不用官方提供的组件了。

## 开整

### 注册先行
#### 创建组件
```php
./vendor/bin/sail artisan make:livewire Auth/Register
```

#### 定义路由

```php
Route::get('/register', \App\Http\Livewire\Auth\Register::class);
```
#### 编辑页面
```php
<div class="h-screen w-screen flex justify-center items-center" style="background-image: url('/static/img/scattered-forcefields.svg')">
    <div class="inline-block w-80">
        <div class="text-2xl text-center">注册</div>
        {{-- To attain knowledge, add things every day; To attain wisdom, subtract things every day. --}}
        <form wire:submit.prevent="register">
            <div class="form-control w-full max-w-xs">
                <label class="label">
                    <span class="label-text">邮箱</span>
                </label>
                <input wire:model="email" type="text" placeholder="Type here" class="input input-bordered w-full max-w-xs" />
                @error('email')
                <label class="label">
                    <span class="label-text-alt  text-error">{{ $message }}</span>
                </label>
                @enderror
            </div>
            <div class="form-control w-full max-w-xs">
                <label class="label">
                    <span class="label-text">用户名</span>
                </label>
                <input wire:model="name" type="text" placeholder="Type here" class="input input-bordered w-full max-w-xs" />
                @error('name')
                <label class="label">
                    <span class="label-text-alt  text-error">{{ $message }}</span>
                </label>
                @enderror
            </div>
            <div class="form-control w-full max-w-xs">
                <label class="label">
                    <span class="label-text">密码</span>
                </label>
                <input wire:model="password" type="password" placeholder="Type here" class="input input-bordered w-full max-w-xs" />
                @error('password')
                <label class="label">
                    <span class="label-text-alt  text-error">{{ $message }}</span>
                </label>
                @enderror
            </div>
            <div class="form-control w-full max-w-xs">
                <label class="label">
                    <span class="label-text">确认密码</span>
                </label>
                <input wire:model="password_confirmation" type="password" placeholder="Type here" class="input input-bordered w-full max-w-xs" />
                @error('password_confirmation')
                <label class="label">
                    <span class="label-text-alt  text-error">{{ $message }}</span>
                </label>
                @enderror
            </div>
            <div class="flex justify-end">
                <button class="btn">注册</button>
            </div>
        </form>
    </div>
</div>
```
页面这部分简单说两句，其实就是用了 tailwind 和 daisy 的样式，注意的点就是 `model` 绑定和 `error` 的处理。

#### 组件部分
```php
<?php

namespace App\Http\Livewire\Auth;

use App\Models\User;
use DragonCode\Support\Concerns\Makeable;
use Illuminate\Validation\Rules\Password;
use Livewire\Component;

class Register extends Component
{
    public $name;
    public $email;
    public $password;
    public $password_confirmation;

    // 为了验证密码字段，不能采用 rules 属性，而采用 rules 方法
    protected function rules()
    {
        return [
            'name' => 'required|min:6|unique:users',
            'email' => 'required|email|unique:users',
            'password' => ['required', 'confirmed', Password::min(8)->mixedCase()->numbers()->symbols()],
            'password_confirmation' => 'required',
        ];
    }

    public function updated($propertyName)
    {
        $this->validateOnly($propertyName);
        // 如果输入的是密码字段，需要跟验证字段同时验证
        if (in_array($propertyName, ['password', 'password_confirmation'])) {
            $this->validateOnly('password_confirmation');
            $this->validateOnly('password');
        }
    }

    public function register() {
        $this->validate();

        $res = User::create([
            'name' => $this->name,
            'email' => $this->email,
            'password' => \Hash::make($this->password),
        ]);
        dd($res);
    }

    public function render()
    {
        return view('livewire.auth.register');
    }
}
```
这里验证没有采用 `rules` 属性，而是采用方法，是为了密码的验证构造，如果采用属性，不支持这种构造方式。另几个就是调用了 `updated` 方法。这里的特殊之处就是如果密码字段更新了，则需要同时验证两个密码和确认密码两个字段。在执行注册前，兜底调用一下 `validate` 方法进行验证。如果验证通过则进行注册。
最后我们看下数据库就可以了。

## 总结
今天就先到这，明天搞登陆后跳转，登录。等相关操作吧。

demo 项目地址：[livewire-demo](https://github.com/crazyhl/livewire-demo)

---

> 作者:   
> URL: http://localhost:1313/posts/laravel-livewire-note-4/  

