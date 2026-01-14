# Mysql 批量更新多行笔记


今天在公司写代码的时候使用到了 mysql 的批量更新，但是呢，自己之前对其了解的果然不够多啊。所以，回家以后赶紧查查，做好笔记，让自己有更深的记忆。以及更多的了解。


首先我们构造好了一个测试表，如下图

![图片alt](https://www.cimple.ink/images/2019/03/06/387ea4a3dff34a73889d73dcda85d1d3.png)


<!--more-->

简单说一下结构，`name` 姓名， `sex` 分组，`group` 分组。

我们先执行一下代码，在来解释一下 sql 语句的作用，代码如下
```php
update test set test.group = case name
	when '小白' then 9
	when '小红' then 10
	end,
where name in ('小白', '小红')
```

代码执行后，结果如下图

![图片alt](https://www.cimple.ink/images/2019/03/06/68725ebf9f2349e9dab85c5dea833b69.png)

可以看到，小白和小红的 `group` 的值都改变了。

好了，我们来说说上面 `sql` 语句的作用，我们在这个例子里面需要修改的是 `group` 的值
常规的写法是 `update test set group=? where name=?` 这个是执行一次的时候的写法，那么我们为什么还要上面的写法呢，因为这样单条执行多次的开销比较大，所以应该用上面的写法，来更新多条语句。好吧，前面又说了一堆废话，让我们正式的来说一下上面的语句，首先，我们在给 group 赋值的时候，采用 `case when then` 的写法，这个写法，类似于我们 php 里面的 `switch` 语句，其实直接理解为 `switch` 就好了，所以那条语句用文字来说的话就是我们要给 `group` 更改值，当 `name` 为小白的时候把值更改为9，当 `name` 为小红的时候，把值更新为10。好了，`case when then` 的部分我们就简述完毕了，但是 `when` 里面的值是从哪里来的呢，答案就在 `where` 里面，在 `where` 里面我们吧 `name` 的条件供给上面的 `when` 来使用。大家可以试试把条件减少，看看有什么效果。好吧，我直接公布答案就好了，`when` 捕获不到的部分不会被更改。代码如下
```php
update test set test.group = case name
	when '小白' then 9
	when '小红' then 10
	end
where name in ('小白')
```
结果如下，

![图片alt](https://www.cimple.ink/images/2019/03/06/a06ffd726572058da1cfa046a1a7c365.png)

我们可以看到小红并没有被更改到。

我们在做一个实验，其实我们还有一个 `else` 条件可用，这个类似 `switch` 里面的 `default` 。代码如下

```php
update test set test.group = case name
	when '小白' then 9
	when '小红' then 10
	else 11
	end
where name in ('小白','大白')
```

结果如下

![图片alt](https://www.cimple.ink/images/2019/03/06/8ed7b5f3834f7623e7d8f34893e41faa.png)

看到了么，`大白` 没有被任何 `when` 所捕获到，所以，执行了 `else` 部分，把值更改为了11。

好了，其实可用的就是这么多了，但是呢，今天我脑残想了另一个问题，多条件的问题，其实这个问题也不是很难，只不过是我想复杂了而已。我们在上一个例子把，我们要把 `group` 为 `1`，的 `小红` 和 `小白` 的 `sex` 值更改为 `2`。那么这个语句该怎么写呢，不要想的太复杂，按照常规的写法写就可以了，直接上代码

```php
update test set test.sex = case name
	when '小白' then 2
	when '小红' then 2
	else 11
	end
where name in ('小白','小红') and test.group = 1
```

结果如下

![图片alt](https://www.cimple.ink/images/2019/03/06/77a84df28580eaab1d64cc1aaf9c2b23.png)

看到了吧，多条件的写法，也如此的简单。

好了，今天的笔记就到这里，持续学习，持续记录。加油

---

> 作者:   
> URL: http://localhost:1313/posts/mysql-batch-update-multiline-notes/  

