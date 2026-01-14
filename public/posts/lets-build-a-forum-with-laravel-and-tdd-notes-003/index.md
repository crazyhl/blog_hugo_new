# Let's Build a Forum With Laravel and TDD 笔记 003

![图片alt](https://raw.githubusercontent.com/M1racle-Hao/blog-image/master/2018/05/30/89a4759f89f6e8680d9c06d0661d13db.png)

今天学习了第三集和第四集，下面依然是个人总结

1. 没错，第三集用到了单元测试也就是 `unit` 的部分，跟我之前看其他文章了了解的差不多， `feature` 主要用来测试面向用户的功能 `unit` 是用来测试我们内部功能的，也就是一个个的最小单元，或者其他的合并单元，然后由这些单元组合成为 `controller` 里面的代码最终呈现给用户，然后 `feature` 再来测试 `controller` 里面的代码。
2. 关于测试以前都是弄出一个数据然后我们自己调试页面然后观察数据，现在就可以用 `unit` 和 `feature` 来让机器帮我们判定了。我们期待什么值，就让他来断言一下就ok了，但是个人还是比较担心出问题，比如数据不全神马的，这个还得后期继续考虑一下
3. `factory` 中 `make` 和 `create` 的区别， `make` 仅仅是生成了数据，`create` 不仅仅是生成了数据，并且将数据保存到数据库中，所以 `create` 可以理解为首先 `make` 然后 `save` 了。
4. 在测试文件中 `be` 方法的作用，就是把当前我们生成的用户，设置为登录状态
5. 在测试的时候不仅要测试正常的情况，也要测试异常的情况
6. 功能职责要分清楚，比如发布 `Reply` 的时候就应该把方法写入到 `ReplyController` 中
7. 关联关系居然可以添加数据，详情参见这里吧 [https://laravel.com/docs/5.6/eloquent-relationships#the-create-method](https://laravel.com/docs/5.6/eloquent-relationships#the-create-method)


---

> 作者: [M1racleHao](https://github.com/crazyhl)  
> URL: http://localhost:1313/posts/lets-build-a-forum-with-laravel-and-tdd-notes-003/  

