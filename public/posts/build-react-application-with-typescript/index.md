# 使用 React Template 构建前端应用


## 前言
原本我一直都是用 vue 的，但是呢，我主力还是后端，所以前端这方面的东西也是属于浅尝辄止的。最近想弄一些前端的东西，观察了一圈，react 貌似还是要比 vue 的生态好一些。既然如此，那就开始用 react 好了。另外为什么要用 typescript 呢，当我在用 vue + typescript 的时候我名没有太明显的感受到跟 js 的区别。但是当我在用 ts 实现公司的一个真实的东西的时候，我体会到了 ts 的爽。所以从那以后我都是尽可能的用 ts 了。好了，正式开始吧，不知道关于 react 的东西会写几篇文章呢。目标是超过 tdd 那个的 5 篇。

## 正文开始

创建项目，[https://zh-hans.reactjs.org/docs/static-type-checking.html#typescript](https://zh-hans.reactjs.org/docs/static-type-checking.html#typescript) 参照的是这篇文章。

```shell
npx create-react-app my-app --template typescript
```

通过这一步呢，项目就创建好了，这块我有一个问题，index 和 App 两个文件都引入了相同的css，难道在编译的时候，只编译了自己用到的部分，否则不是浪费了么？还是会在编译的时候做优化?等以后研究研究。

项目创建好了，接下来准备引入路由，关于基础相关的东西，我也是用到什么就去查什么，不过官方文档核心概念部分，我也都是看完了的，大家如果没有看完，还是看完的好。路由部分用的是 [https://reactrouter.com/web/guides/quick-start](https://reactrouter.com/web/guides/quick-start)
安装也是一步操作

```shell
npm install react-router-dom
```

```shell
npm i -D @types/react-router-dom
```

然后就可以按照文档的说明在做一些实践了。注意一点就是使用 ts 的时候需要执行第二条命令，否则会有报错。 代码如下，
```typescript
import React from 'react';
import { BrowserRouter as Router, Link, Switch, Route } from 'react-router-dom';
import './App.css';

function App() {
  return (
    <Router>
      <div>
        <nav>
          <ul>
            <li>
              <Link to="/">Home</Link>
            </li>
            <li>
              <Link to="/about">About</Link>
            </li>
            <li>
              <Link to="/users">Users</Link>
            </li>
          </ul>
        </nav>
        <Switch>
          <Route path="/about">
            <About />
          </Route>
          <Route path="/users">
            <Users />
          </Route>
          <Route path="/">
            <Home />
          </Route>
        </Switch>
      </div>
    </Router>
  );
}

function Home() {
  return <h2>HOme</h2>
}

function About() {
  return <h2>Aboutv</h2>
}
function Users() {
  return <h2>Users</h2>
}
export default App;

```
如果要使用 路由相关的组件，最外层也要有 Router，这里用的是 `BrowserRouter`。导航用的是 `Link` 组件。页面文件最外层使用 `Switch` 嵌套。每个页面文件用 `Route` 包裹。 `Route` 里面有个 path 属性，第一个匹配到的就会被展示出来。

官方第二个示例是嵌套路由，
```typescript
import React from 'react';
import { BrowserRouter as Router, Link, Switch, Route, useRouteMatch, useParams } from 'react-router-dom';
import './App.css';

function App() {
  return (
    <Router>
      <div>
        <nav>
          <ul>
            <li>
              <Link to="/">Home</Link>
            </li>
            <li>
              <Link to="/about">About</Link>
            </li>
            <li>
              <Link to="/topics">Topics</Link>
            </li>
          </ul>
        </nav>
        <Switch>
          <Route path="/about">
            <About />
          </Route>
          <Route path="/topics">
            <Topics />
          </Route>
          <Route path="/">
            <Home />
          </Route>
        </Switch>
      </div>
    </Router>
  );
}

function Home() {
  return <h2>Home</h2>
}

function About() {
  return <h2>Aboutv</h2>
}
function Topics() {
  let match = useRouteMatch();
  return (
    <div>
      <h2>Topics</h2>
      <ul>
        <li>
          <Link to={`${match.url}/components`}>Components</Link>
        </li>
        <li>
          <Link to={`${match.url}/props-v-state`}>Props v. State</Link>
        </li>
      </ul>

      <Switch>
        <Route path={`${match.path}/:topicId`}>
          <Topic />
        </Route>
        <Route path={match.path}>
          <h3>Please select a topic {match.path}</h3>
        </Route>
      </Switch>
    </div>
  )
}

type TopicParams = {
  topicId: string;
};

function Topic() {
  let { topicId } = useParams<TopicParams>();
  return <h3>Requested topic ID: {topicId}</h3>
}

export default App;

```

这里主要注意， useParams 需要弄一个类型声明，这也是 ts 好的地方，另外这里的嵌套路由，我觉得用起来比 vue 感受要好一些，说不好，在慢慢体会吧，另外关于 match 部分看文档 [https://reactrouter.com/web/api/match](https://reactrouter.com/web/api/match)


## 总结
第一天要弄的东西不多，就先这样，然后所有的代码都在这里 [https://github.com/crazyhl/react-practise](https://github.com/crazyhl/react-practise) 最后，这个东西会形成我最终要实现的东西的开源版本。

---

> 作者:   
> URL: http://localhost:1313/posts/build-react-application-with-typescript/  

