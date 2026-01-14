# Webpack4 Typescript 配置笔记

最新需要写一些 `js` 的东西，正好就想试用一下 `typescript` 试用以后感觉舒服，那么需求来了，我们需要编译出来的文件，在外部可以调用，所以就有了下面的笔记

参考文章[https://www.cnblogs.com/mahidol/p/8874300.html](https://www.cnblogs.com/mahidol/p/8874300.html)

npm 安装依赖
```shell
npm install --save-dev typescript ts-loader webpack webpack-cli
```

在项目根目录创建两个文件夹 `src` 和 `dist` 以及一个测试用的入口文件 `index.html`

`src` 用来存放我们的源码
`dist` 用来保存生成后的源码

`index.html` 的内容
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>

</body>
<script type="text/javascript" src="./dist/app.js" ></script>
</html>
```

入口文件仅仅是用来测试的，所以仅仅引入了输出的js 我们在 `chrome` 控制台测试

根目录创建 `typescript` 配置文件 `tsconfig.json`

```json
{
  "compilerOptions": {
    "module": "commonjs",
    "target": "es5",
    "sourceMap": true
  },
  "exclude": [
    "node_modules"
  ]
}
```

创建 `webpack` 配置文件，采用[https://www.webpackjs.com/guides/production/](https://www.webpackjs.com/guides/production/)
中的方案，只不过有些小修改

安装 `webpack-merge` 包
```shell
npm install --save-dev webpack-merge
```

`webpack.common.js` 公用的配置文件

```javascript
const path = require('path');

module.exports = {


    entry: './src/index.ts',
    output: {
        filename: 'app.js',
        path: path.resolve(__dirname, 'dist')
    },

    module: {
        rules: [{
            test: /\.ts$/,
            use: "ts-loader"
        }]
    },
    resolve: {
        extensions: [
            '.ts'
        ]
    }
};
```

`webpack.dev.js` 开发中的配置文件

```javascript
const merge = require('webpack-merge');
const common = require('./webpack.common.js');
    module.exports = merge(common, {
        mode: 'development',
});

```

`webpack.prod.js` 输出的的配置文件

```javascript
const merge = require('webpack-merge');
const common = require('./webpack.common.js');
module.exports = merge(common, {
    mode: 'production',
    output: {
        filename: 'app.min.js',
        path: path.resolve(__dirname, 'dist')
    },
});

```

在 `package.json` 中增加两条命令

```json
"start": "webpack --config webpack.dev.js",
"build": "webpack --config webpack.prod.js"
```

这里跟文档不一样，毕竟文档的开发部分配置文件用的测试服务器，我们这个单纯的就是自己测试，所以依然还是用 `webpack` 命令编译一下。

至此，我们在普通场景下已经可以使用了。但是由于我们的需求是，这个文件不仅仅在 `html` 中调用，还需要在外部调用编译出来的代码，但是上面的代码是在外面无法调用的，所以我们继续搞。


在 `webpack.common` 配置文件中的 `output` 属性增加两个配置
```json
library: "MediaParser",
libraryTarget: 'var',
```

简单点说就是导出为名称为 `MediaParser` 的变量，这样我们就可以在外部 `scrpit` 中调用了

具体的这两个参数可以看 `webpack` 中的文档[https://www.webpackjs.com/configuration/output/#output-librarytarget](https://www.webpackjs.com/configuration/output/#output-librarytarget) [https://www.webpackjs.com/guides/author-libraries/](https://www.webpackjs.com/guides/author-libraries/)

收工

---

> 作者:   
> URL: http://localhost:1313/posts/webpack4-typescript-configuration-notes/  

