# Let's Build a Forum With Laravel and TDD 笔记 002



![图片alt](https://raw.githubusercontent.com/M1racle-Hao/blog-image/master/2018/05/30/89a4759f89f6e8680d9c06d0661d13db.png)

今天学习了第二节，原以为一节课只有不到10分钟，一天可以学习好几节，但是呢，自己就得去学习不足的知识，结果不到一个10分钟的课程，自己就扩展到了将近40分钟，好了，说说今天的心得


1. 在写功能之前，先做好测试用例，这样可以方便我们去实现功能，也可以提供思路
2. 在 laravel 的 test 文件夹中有两个字文件夹，分别是 `feature` 和 `unit` 这两个一个是功能测试一个是单元测试，就在这里我去查了一下东西，但是理解的也不是很好下面说说自己的理解
    - 功能测试是面向用户的，也就是说我们需要测试代码中的 `controller` 部分，为什么这么说呢，因为只有 `controller` 部分才是面向用户的，不论这个功能有多大，这里面一个方法都是对用户的。
    - 单元测试就可以理解为其他的部分了，因为 `controller` 里面的代码都是由剩余部分的代码组成的，所以这些都是一个个的子单元
    - 其他部分的代码也有可能由其他的部分一起组成，所以我个人觉得也应该写单元测试
    - 不知道上面说的对不对
3. 写代码的时候应该划分好各自的职责


---

> 作者: [M1racleHao](https://github.com/crazyhl)  
> URL: http://localhost:1313/posts/lets-build-a-forum-with-laravel-and-tdd-notes-002/  

