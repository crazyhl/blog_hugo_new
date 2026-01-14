---
title: 手把手使用 electron 自己动手开发 upyun 文件上传客户端
tags: [electron, 客户端, 又拍]
categories: [前端]
slug: handson-use-electron-to-develop-your-own-upyun-file-upload-client
date: 2019-07-05 16:23:25
toc: false
---
更换了域名，需要弄很多东西，累死了，没错这篇文章唯一的图片就是通过这篇文章编译的工具上传成功的

#### 项目地址
[https://github.com/crazyhl/upyun-electron](https://github.com/crazyhl/upyun-electron)

#### 初始准备

##### 参照文章 
[https://electronjs.org/docs/tutorial/first-app#installing-electron](https://electronjs.org/docs/tutorial/first-app#installing-electron)

1. 创建项目文件夹 `upyun-electron`
2. 初始化项目 `npm init`
![](https://raw.githubusercontent.com/M1racle-Hao/blog-image/master/2019/07/05/a3f0ef04-6203-5e66-af79-2d8e0d4fd18e.jpeg)

<!--more-->

3. 创建刚才设定的入口文件 `main.js`，安装 `electron` （采用 yarn 或者 npm 都行，我个人用的 yarn），在 `package.json` 的 `scripts` 添加命令 `"start": "electron ."` 用来启动我们的项目

4. `main.js` 中写入如下代码 具体注释

```js
const { app, BrowserWindow } = require('electron')

// 保持对window对象的全局引用，如果不这么做的话，当JavaScript对象被
// 垃圾回收的时候，window对象将会自动的关闭
let win

function createWindow () {
  // 创建浏览器窗口。
  win = new BrowserWindow({
    width: 800,
    height: 600,
    webPreferences: {
      nodeIntegration: true
    }
  })

  // 加载index.html文件
  win.loadFile('index.html')

  // 打开开发者工具
  win.webContents.openDevTools()

  // 当 window 被关闭，这个事件会被触发。
  win.on('closed', () => {
    // 取消引用 window 对象，如果你的应用支持多窗口的话，
    // 通常会把多个 window 对象存放在一个数组里面，
    // 与此同时，你应该删除相应的元素。
    win = null
  })
}

// Electron 会在初始化后并准备
// 创建浏览器窗口时，调用这个函数。
// 部分 API 在 ready 事件触发后才能使用。
app.on('ready', createWindow)

// 当全部窗口关闭时退出。
app.on('window-all-closed', () => {
  // 在 macOS 上，除非用户用 Cmd + Q 确定地退出，
  // 否则绝大部分应用及其菜单栏会保持激活。
  if (process.platform !== 'darwin') {
    app.quit()
  }
})

app.on('activate', () => {
  // 在macOS上，当单击dock图标并且没有其他窗口打开时，
  // 通常在应用程序中重新创建一个窗口。
  if (win === null) {
    createWindow()
  }
})

// 在这个文件中，你可以续写应用剩下主进程代码。
// 也可以拆分成几个文件，然后用 require 导入。
```

5. 添加 `main.js` 中用到的 `index.html` 文件

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <title>Hello World!</title>
  </head>
  <body>
    <h1>Hello World!</h1>
    We are using node <script>document.write(process.versions.node)</script>,
    Chrome <script>document.write(process.versions.chrome)</script>,
    and Electron <script>document.write(process.versions.electron)</script>.
  </body>
</html>Copy

```

6. 控制台执行 `npm start` 可以看到我们的项目跑起来了，上面的记录几乎复制了 `electron` 中的教程，不过不怕麻烦，主要是还是为了更详尽的了解相关的构成，毕竟不是专业的前端，很多东西弄起来还得理解才行，第五步的代码要好好看看。后面的都是在这个的基础上搞定的。

#### 处理拖拽

##### 参考文章 
[http://www.w3school.com.cn/html5/html_5_draganddrop.asp](http://www.w3school.com.cn/html5/html_5_draganddrop.asp) 
[https://steemit.com/utopian-io/@pckurdu/file-drag-and-drop-module-in-electron-with-text-editing-example](https://steemit.com/utopian-io/@pckurdu/file-drag-and-drop-module-in-electron-with-text-editing-example)

1. 了解 `html 拖拽` 和 `electron 拖拽`, 看上面的参考文章
2. 在 html 添加测试代码

```html
<div style="width: 100%; height: 100px; border: 1px solid #c5c5c5;text-align: center;">
        <p id="drag-file">Drag File Here.</p>
    </div>

    <textarea id="txtarea" style="width:100%; height:350px;margin-top: 16px; border: 1px solid #c5c5c5;"></textarea>
    <script>
        var dragFile = document.getElementById("drag-file");
        // 监听这个事件的原因是 默认地，无法将数据/元素放置到其他元素中。如果需要设置允许放置，我们必须阻止对元素的默认处理方式。
        dragFile.addEventListener('dragover', function (e) {
            e.preventDefault();
        });
        dragFile.addEventListener('drop', function (e) {
            e.preventDefault();
            e.stopPropagation();

            for (let f of e.dataTransfer.files) {
                console.log('The file(s) you dragged: ', f)
            }
        });
    </script>
```
注意这里面我们跟上面的参考文章中多监听了 `dragover` 事件，而里面只做了一件事，就是阻止默认时间的运行，具体原因参考[http://www.w3school.com.cn/html5/html_5_draganddrop.asp]()，后续如果我有更好的方案，我会更改这部分的代码以及说明。到这我们已经可以拖拽文件了，接下来，让我们把文件显示到div里面，显示文件列表

3. 引入 vue 

```html
    <div id="app">
        <h1>Hello World!</h1>
        We are using node {{nodeVersion}},
        Chrome {{chromeVersion}},
        and Electron {{electronVerson}}.
    </div>

    <div id="drag-file" style="width: 100%; height: 100px; border: 1px solid #c5c5c5;text-align: center;">
        <p>Drag File Here.</p>
    </div>

    <textarea id="txtarea" style="width:100%; height:350px;margin-top: 16px; border: 1px solid #c5c5c5;"></textarea>

    <script src="./js/vue.js"></script>
    <script>
        var dragFile = document.getElementById("drag-file");
        // 监听这个事件的原因是 默认地，无法将数据/元素放置到其他元素中。如果需要设置允许放置，我们必须阻止对元素的默认处理方式。
        dragFile.addEventListener('dragover', function (e) {
            e.preventDefault();
        });
        dragFile.addEventListener('drop', function (e) {
            e.preventDefault();
            e.stopPropagation();

            for (let f of e.dataTransfer.files) {
                console.log('The file(s) you dragged: ', f)

            }
        });

        var app = new Vue({
            el: '#app',
            data: {
                nodeVersion: process.versions.node,
                chromeVersion: process.versions.chrome,
                electronVerson: process.versions.electron,
            }
        })
    </script>
```

修改 `index.html` 引入 `vue` 并将原来现实版本的代码修改为 `vue` 的形式。再次 `npm start` 发现跟以前现实的内容是一直的，至此说明引入 `vue` 是 ok 的。

4. 将拖拽部分改造为 `vue` 控制，并能够将文件信息显示为一个列表

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="UTF-8">
    <title>又拍云上传工具 Electron 版</title>

    <link rel="stylesheet" href="./css/bulma.css">
    <link rel="stylesheet" href="./css/index.css">
</head>

<body>
    <div id="app">
        <div id="drag-file" v-on:dragover.prevent v-on:drop="fileDrop($event)">
                <ul id="upload-file-list" v-if="uploadFileList.length">
                    <li v-for="file in uploadFileList">
                        <div class="level">
                            <div v-if="file.fileInfo.type.startsWith('image')" class="column is-one-fifth">
                                <img class="preview" :src="file.fileInfo.path" />
                            </div>
                            <div class="column">{{ file.fileInfo.path }}</div>
                            <div v-if="file.status === 0" class="column is-one-fifth">准备上传</div>
                        </div>
                    </li>
                </ul>
                <p id="drag-tips" v-else>Drag your upload file!</p>
        </div>
        <div class="weui-cell">
            <div class="weui-cell__bd">
                <textarea  id="txtarea" class="weui-textarea" rows="10"></textarea>
            </div>
        </div>
    </div>

    <script src="./js/vue.js"></script>
    <script>
        var app = new Vue({
            el: '#app',
            data: {
                nodeVersion: process.versions.node,
                chromeVersion: process.versions.chrome,
                electronVerson: process.versions.electron,
                uploadFileList: [],
                uploadFileNameList: [],
            },
            methods: {
                fileDrop: function (eveent) {
                    for (let f of eveent.dataTransfer.files) {
                        console.log('The file(s) you dragged: ', f)
                        if (this.uploadFileNameList.indexOf(f.path) >= 0) {
                            continue;
                        }
                        this.uploadFileList.push({
                            'fileInfo': f,
                            'status': 0,
                            'class': '',
                        });
                        this.uploadFileNameList.push(f.path);
                    }
                },
            },
        })

        function test() {
            alert('abc');
            app._data.nodeVersion = 123;
        }
    </script>
</body>

</html>
```

到这步，前端基本上算是时间完了，还需要完成的就是等待文件上传完成后更新主页面的样式了，那个等我们处理完文件上传之后再去弄。PS: 可以看到我引入了一些样式文件，这部分可以在 `git` 里面找到我就不贴上来占用篇幅了

#### 处理文件上传

##### 参考文章 
[https://electronjs.org/docs/tutorial/application-architecture#main-and-renderer-processes](https://electronjs.org/docs/tutorial/application-architecture#main-and-renderer-processes)
[https://electronjs.org/docs/api/ipc-main](https://electronjs.org/docs/api/ipc-main)
[https://electronjs.org/docs/api/web-contents#contentssendchannel-arg1-arg2-](https://electronjs.org/docs/api/web-contents#contentssendchannel-arg1-arg2-)
[https://electronjs.org/docs/api/ipc-renderer](https://electronjs.org/docs/api/ipc-renderer)
[https://steemit.com/utopian-io/@pckurdu/file-drag-and-drop-module-in-electron-with-text-editing-example](https://steemit.com/utopian-io/@pckurdu/file-drag-and-drop-module-in-electron-with-text-editing-example)

从这部分开始我就会只复制改动部分的代码了，要不然占用篇幅会越来越大,代码中的 `...`部分代表以前的代码，如果在代码中部插入代码，我会粘贴进来插入代码的前后一行作为标记，这样就不会有差错了。

1. `Render 进程` 通知 `Main 进程` 有文件被添加了

`index.html` 添加 `ipcRenderer` 
```html 
<script>
        const {ipcRenderer}=require('electron')

        var app = new Vue({
```
`index.html` 通知主进程
```html

fileDrop: function (eveent) {
    for (let f of eveent.dataTransfer.files) {
        ...
        // 通知主进程 有文件被添加了
        ipcRenderer.send('fileAdd', f.path)
    }
},
```

2. `Main 进程` 接受消息，并传递给对应 `Render 进程` 消息

`main.js` 头部引入 `ipcMain` 
```js
const { app, BrowserWindow, ipcMain } = require('electron')
```
`main.js` 文件最后添加代码处理收到的消息，并通知 'Render 进程' 消息
```js
ipcMain.on('fileAdd', (event, filePath) => {
  console.log(filePath)
  event.reply('receiveNotice', filePath, 'lalala');
})
```
这里面用 `event.reply` 或者 `event.sender.send` 都是可以的，我个人理解的就是原页面的就用 `reply` 跨页面的就用 `sender.send` 也不知道理解的是不是正确的

`index.html` 添加处理代码 `Main 进程` 发送过来的消息,在 `</script>` 结束标签上方添加代码
```html
        ipcRenderer.on('receiveNotice', function (event, message, status) {
            console.log(message);
            console.log(status);
        });
</script>
```
到这里我们的通信过程就打通了，就可以进行后续的操作了

3. `Main 进程` 进行文件检查，如果是文件则通过，如果是文件夹则进行错误提示

修改 `fileAdd` 的通知，别忘了在`main.js` 头部引入 `const fs = require('fs');`
```js
// 状态约定  0 准备上传 1 上传中 2 上传成功 -1 不是文件 -2 上传失败

ipcMain.on('fileAdd', (event, filePath) => {
  console.log(filePath)
  fileStates = fs.lstatSync(filePath);
  isFile = fileStates.isFile();
  if (isFile) {
      // 准备上传
  } else {
    event.reply('receiveNotice', filePath, -1, 'has-text-danger');
  }
})
```

在html代码部分，在 `receiveNotice` 添加代码
```html
ipcRenderer.on('receiveNotice', function (event, filePath, status, className) {
    let index = app.uploadFileNameList.indexOf(filePath);
    if (index >= 0) {
        app.uploadFileList[index].status = status;
        let statusText = '准备上传';
        switch(status) {
            case 1:
                statusText = '上传中';
                break;
            case 2:
                statusText = '上传成功';
                break;
            case -2:
                statusText = '上传失败';
                break;
            case -1:
                statusText = '这不是个文件';
                break;
            case 0:
            default:
                statusText = '准备上传';
                break;
        }
        app.uploadFileList[index].statusText = statusText;
        app.uploadFileList[index].class = className;
    }
});
```
`index.html` 的 `fileDrop` 部分更新了`uploadFileList` 的结构
```html
fileDrop: function (eveent) {
    for (let f of eveent.dataTransfer.files) {
        ...
        this.uploadFileList.push({
            'fileInfo': f,
            'status': 0,
            'statusText': '准备上传',
            'class': '',
        });
        ...
    }
},
```
现在状态更改线程间通信也都做好了，下面就开始上传吧

#### 上传文件

##### 参考文章

[https://github.com/upyun/node-sdk](https://github.com/upyun/node-sdk)

1. 添加 `upyun sdk`

```shell
yarn add upyun --production
```
`production` 参数的作用是 安装 `package.json` 中 `dependencies` 里面的包，不会安装 `devDependencies` 里面的

2. 在 `main.js` 添加 相关代码

引入需要的东西 upyun 是 又拍云上传用的，path是获取文件相关信息用的，uuidv5是用来生成uuid的这个别忘记 npm 或者 yarn 安装一下 `yarn add uuid`
```js
const upyun = require('upyun')
const path=require('path');
const uuidv5 = require('uuid/v5');
```

```js
const cdnUrl = 'yourCdnBaseUrl';
ipcMain.on('fileAdd', (event, filePath) => {
  console.log(filePath)
  fileStates = fs.lstatSync(filePath);
  isFile = fileStates.isFile();
  if (isFile) {
      // 准备上传
      const service = new upyun.Service('your service name', 'your operate name', 'your operate password')
      const client = new upyun.Client(service);
      const fileExtName = path.extname(filePath);
      console.log('fileExtName:' + fileExtName);
      const date = new Date();
      const dateFormat = '/' + date.getFullYear() + '/' + ("0" + (date.getMonth() + 1)).slice(-2) + '/' + ("0" + date.getDate()).slice(-2);
      const remotePath = dateFormat + '/' + uuidv5(filePath, uuidv5.URL) + fileExtName;
      console.log('remotePath:' + remotePath);
      client.initMultipartUpload(remotePath, filePath).then(function ({fileSize, partCount, uuid}) {
        console.log(fileSize)
        console.log(partCount)
        console.log(uuid)
        event.sender.send('receiveNotice', filePath, 1, 'has-text-info')
        Promise.all(Array.apply(null, {length: partCount}).map((_, partId) => {
          return client.multipartUpload(remotePath, filePath, uuid, partId)
        })).then(function () {
          console.log('finish:' + uuid);
          client.completeMultipartUpload(remotePath, uuid)
          event.sender.send('receiveNotice', filePath, 2, 'has-text-success', cdnUrl + remotePath)
        });
      });
  } else {
    event.reply('receiveNotice', filePath, -1, 'has-text-danger');
  }
})
```

至此一个简单的 upyun electron版本的客户端工具就做好了，自己用的时候别玩了个替换 `cdnurl` `service name` `operate name` `operacate password` 哦

当然目前这个版本还有很多问题，比如多文件会卡顿，大文件主进程卡死等等问题，不过第一版就可以完美的这样结束了，后续的改动包括界面美化，上传线程池，配置文件等等不少东西。


