# 设计模式系列-监听模式


## 先说点什么

设计模式以前也看过，但大都走马观花。最近也是巧，发现了一本书 《人人都懂设计模式》，简单翻了几页，觉得我能开心的读进去，并且我也觉得我到了可以思考设计模式的时候了。在以前，经验比较少，一些场景没接触过，也没法感同身受的思考为什么要这么设计？有时候明明把代码弄得层次多了更复杂了，读起来也麻烦了。为什么还要搞这么个东西呢？随着经验的增多，更多的理解了折中取舍，为了系统的稳定，为了将来开发的更舒爽，一些初期的痛也是要承受的。所以，开始了 `设计模式`。Let's do it。

## 什么是监听模式

> Define a one-to-many dependency between objects so that when one objectchanges state,all its dependents are notified and updated automatically.
>
> 在对象间定义一种一对多的依赖关系，当这个对象状态发生改变时，所有依赖它的对象都会被通知并自动更新。

## 我的理解

其实我个人的理解啊这个模式优点依赖倒置的意思。以前我们在写代码的时候都是在需要通知的时候主动调起被通知的一方。

这样写代码耦合性就会强，每次改动都要去通知的一方去修改代码。比如下面这样。

```go
package main
import (
	"fmt"
)

type Student struct {
	Name string
}

func (stu Student) RecordHomework()  {
	fmt.Println(stu.Name + " 记录作业")
}

type Teacher struct {
	Students []Student
}

func (teacher Teacher) NotifySutdentRecordHomework()  {
	for _, stu := range teacher.Students {
		stu.RecordHomework()
	}
}

func main()  {
	stu1 := Student{
		Name: "小明",
	}
	stu2 := Student{
		Name: "小红",
	}
	stu3 := Student{
		Name: "小白",
	}

	teacher := Teacher{
		Students: []Student{stu1, stu2, stu3},
	}

	teacher.NotifySutdentRecordHomework()
}
```

用一个老师通知学生记录作业来举例子，说一下需要修改的地方

* 添加学生的时候
* 通知改变的时候

比如下面这样

```go
package main
import (
	"fmt"
)

type Student struct {
	Name string
}

func (stu Student) RecordHomework()  {
	fmt.Println(stu.Name + " 记录作业")
}

func (stu Student) CallMomHasHomework()  {
	fmt.Println("给 " + stu.Name + " 妈妈打电话，告诉妈妈留作业了")
}

type Teacher struct {
	Students []Student
}

func (teacher Teacher) Notify()  {
	for _, stu := range teacher.Students {
		stu.RecordHomework()
		stu.CallMomHasHomework()
	}
}

func main()  {
	stu1 := Student{
		Name: "小明",
	}
	stu2 := Student{
		Name: "小红",
	}
	stu3 := Student{
		Name: "小白",
	}
	stu4 := Student{
		Name: "小绿",
	}

	teacher := Teacher{
		Students: []Student{stu1, stu2, stu3, stu4},
	}

	teacher.Notify()
}
```

跟上面相比，我们增加了一个学生 `stu4`，并且通知记录作业变成了两件事，一个是记录作业，另一个是告诉小朋友妈妈留作业了，以免小朋友不记得做作业。

这时候可以看出，我们要修改通知方法。修改东西，就意味着风险增加。真实的项目可没有这么短的代码，如果改动错误，就麻烦的很。

## 监听模式解决了什么问题？

通过上面分析如果要改动有两个方面

1. 增加被通知的对象
2. 修改通知的方法

监听模式并没有优化增加被通知的对象这方面的东西，他优化的是通知方式有改变的时候，通过上面的代码，可以看到我们修改了通知的方法。

那么如何优化呢？看下面。



```go
package main

import (
	"fmt"
)

// 观察者
type Observer interface {
	ReceiveNotify()
}

// 可被观察的
type Observable struct {
	ObserverList map[string]Observer
}

func (o *Observable) AddObserver(name string, observer Observer) {
	o.ObserverList[name] = observer
}

func (o *Observable) RemoveObserver(name string) {
	delete(o.ObserverList, name)
}

func (o Observable) Notify() {
	if len(o.ObserverList) == 0 {
		return
	}
	for _, observer := range o.ObserverList {
		observer.ReceiveNotify()
	}
}

type Student struct {
	Name string
}

func (stu Student) ReceiveNotify()  {
	stu.RecordHomework()
	stu.CallMomHasHomework()
}

func (stu Student) RecordHomework() {
	fmt.Println(stu.Name + " 记录作业")
}

func (stu Student) CallMomHasHomework() {
	fmt.Println("给 " + stu.Name + " 妈妈打电话，告诉妈妈留作业了")
}

type Teacher struct {
	Observable
}



func main() {
	stu1 := Student{
		Name: "小明",
	}
	stu2 := Student{
		Name: "小红",
	}
	stu3 := Student{
		Name: "小白",
	}
	stu4 := Student{
		Name: "小绿",
	}

	teacher := Teacher{
		Observable{ObserverList: make(map[string]Observer)},
	}

	teacher.AddObserver("stu1", stu1)
	teacher.AddObserver("stu2", stu2)
	teacher.AddObserver("stu3", stu3)
	teacher.AddObserver("stu4", stu4)

	teacher.Notify()
}

```



可以看到，增加了 `Observable` ，让 `Teacher` 来继承，代表 `Teacher` 是可被观察的，然后 `Observable`  增加了3个方法分别是增加观察者、移除观察者和进行通知。紧接着 我们增加了 `Observer` 接口，接口中有 `ReceiveNotify` 方法，代表了收到通知后需要处理的方法，`Student` 类实现了这接口。

看到这里，正如我最上面所说的代码麻烦起来了，但是好处也来了，我们吧通知的方法解耦了，让观察者自行实现收到通知后的操作，我们以后改动就只需要改动观察者就好了。也许你会说还是要改代码啊。还是那句话现实中的要远比示例代码复杂的多，如果都在被观察者里面改动万一不小心吧别的类型的观察者改坏了怎么办。采用监听模式之后我们只需要改动对应的观察者类型就好，别的类型不用动。这样风险就远远降低了。

还有一点，如果不采用这种方式，我们在进行各种通知的时候可能也得判断观察者是什么类型的，如果不想判断还是要抽象出来一个接口。可以看到抽象出来的接口就跟我们的观察者接口一样，既然已经如此了，莫不如继续在弄一个被观察者类型了。一步到位，省心省力。一次的复杂，让我们以后的风险都降低了。

## 再说点更多的

昨天在书里看到说观察者模式，发布订阅模式其实都可以理解为监听模式。昨天在知乎还看到有人分析发布订阅模式其实还有个 `broker` ，从我使用不太丰富的理解 `broker `实际上解决了两个问题，进一步解耦以及进行持久化，今天简单搜索了一下，貌似 `redis` 的 pub/sub 就没有 `broker`。这个东西等我以后在研究研究再说，先给自己留个坑吧。



## 最后要说的

上面都是自己理解的，不一定靠谱，有啥不正确的地方，一定要跟我说啊。



---



## 如果看爽了不如来这关注我一波

![关注](https://www.cimple.ink/static/images/1d7cc52e159eea7bf63a0ee39bddc45b.png)

---

> 作者:   
> URL: http://localhost:1313/posts/design-pattern-oberver-pattern/  

