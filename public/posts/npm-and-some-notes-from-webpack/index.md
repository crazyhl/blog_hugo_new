# Npm 以及 Webpack 的一些笔记


先说点没用的，工作了3年了，相关东西也都接触一些，但是前端呢，还是按照以前的套路，做页面模版，写 js，写 css，但是现代相关的东西却没有深入的了解过。这不好啊，趁着现在有些空闲时间就想学习一下vue，但是 vue-cli 是使用 webpack 打包的，所以第一步就应该学习一下 webpack 趁机了解一下现代前端的开发思想。

本篇笔记总结自:  [http://www.jianshu.com/p/42e11515c10f](http://www.jianshu.com/p/42e11515c10f)，并针对笔记当日的最新版本对配置项进行响应的配置

####安装 webpack

```shell
//全局安装
npm install -g webpack
//安装到你的项目目录
npm install --save-dev webpack
```

####编译项目

```shell
webpack {entry file/入口文件} {destination for bundled file/存放bundle.js的地方}
```

####不适用繁杂的命令，使用配置文件编译项目

```shell
module.exports = {
  entry:  __dirname + "/app/main.js",//已多次提及的唯一入口文件
  output: {
    path: __dirname + "/public",//打包后的文件存放的地方
    filename: "bundle.js"//打包后输出文件的文件名
  }
}
```

####使用 node 的 package.json 设置 script 来执行编译

```shell
// 在package.json 的 scripts 里面添加下面的命令
"build": "webpack"
// 前面的是指在利用 npm run 后面执行的命令，问号后面的是真正执行的命令
```

####使用webpack构建本地服务器

```shell
// 在 webpck.config.js 加入下面的配置文件
devServer: {
    contentBase: "./public",//本地服务器所加载的页面所在的目录
    // colors: true,//终端中输出结果为彩色 新版本貌似不需要这个了
    historyApiFallback: true,//不跳转
    inline: true//实时刷新
  } 
```

大概就这么多吧，Loader 和 Bable 没有总结，毕竟这两个扩展性比较强，需要用的时候，在查或者看文档就 ok 了，基本上 webpack 也就了解了一下了。


嘿呀，好久没有写东西了，很爽啊，正在逐步找回节奏，慢慢笔记会更好的。加油


---

> 作者: [M1racleHao](https://github.com/crazyhl)  
> URL: http://localhost:1313/posts/npm-and-some-notes-from-webpack/  

