# 基于 Laravel Livewire 的表单构造器


## 前言

从 10 月末研究完以后 `livewire`，就开始着手弄这个表单构造器了，其实前两年我的重心都在 `go` 上面，只不过今年下半年发现自己有些迷茫，于是就继续回到 `php` 的怀抱，其实做这个表单构造器是很久很久的计划了，一直想用 `go` 来实现，但是如果用 `go` 始终没有发现合适的模板，要么就做到最终版 - 前后端分离。

但是那是不实际的，由于我是想投入到真实环境中使用的，如果直接做到最后，不仅脱离实际，也不好进行。

既然回到了 `php` 就可以大展拳脚了，用 `laravel` 最初我想像 `laravel-admin` 和 `dcat` 那样来弄的，后来朋友给我推荐了 `livewire` 不得不说这个东西惊艳到了我。于是就有了前面的学习和使用笔记。

最终，在相关东西都准备好以后再 2022年11月7号 提交了第一次代码。

## 我都做了什么

最开始的半个月都是在进行各种实验，后来逐步开始写一些组件。中途还经历了从 `tailwindcss` 切换到 `adminlte` 的惨痛经历（恕我直言，如果我前度能力强一些，tailwind 会比 adminlte 使用起来让我更舒服）。

那么到今天，我觉得我可以给自己一个相对满意的交代了。我实现了下面的基础组件

* TextInput (这个可以支持切换 input 的type，达到各种类型的支持)
* PasswordInput
* CheckList
* CheckBox 支持 toogle 开关样式
* Radio
* SelectMulti
* Select
* Textarea
* WangEditor
* FileUploader

几乎把常用的组件都覆盖了。

## 接下来要做什么

准备用这个 form 来实现一个我计划需求的小项目，来测试是否可行，死皮赖脸找朋友在生产环境试用这个东西。

继续打磨组件，目标就是看齐 `laravel-admin` 、 `dcat` 和 `filamentphp`。

其实这个项目最初就是参照 `filamentphp` 的结构在做的，目前是第一个产品 form，后续会做 table。最后就是一个完整的后端项目了。

## 祝福

最后祝福大家新年快乐，年后在继续搞了。

好多年没这么开心了，踏踏实实写代码，比停留在理论上要让我踏实的多。

规划项目也得到了长足的提升，notion真是个好东西。

---

> 作者:   
> URL: http://localhost:1313/posts/laravel-livewire-based-form-builder/  

