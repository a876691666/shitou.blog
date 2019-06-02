title: react fiber 学习记录 【一】
tags:
  - react
  - 学习记录
categories:
  - front-end
date: 2019-05-27 22:31:00
---
### 一、为什么要了解React Fiber
起因是我们项目的UI库进行了x位版本的升级，也就是`0.x => 1.x`的一次升级，这个UI库的1.x版本依赖使用`react 16`，而0.x版本和我们的项目是依赖使用的`react 15`，在查阅了官方和别人的升级过程后，发现
- `componentWillMount`
- `componentWillUpdate`
- `componentWillRecieveProps`

三个生命周期被官方标记为不安全，在`react 16`中可以增加`UNSAFE_`前缀来使用
- `UNSAFE_componentWillMount`
- `UNSAFE_componentWillUpdate`
- `UNSAFE_componentWillRecieveProps`

并增加了两个新的生命周期
- `static getDerivedStateFromProps`
- `getSnapshotBeforeUpdate`

在`react 17`进行正式取代。

本着求知的精神继续查阅发现react官方将推行`异步渲染`这一功能。
而被标记为不安全的生命周期在异步渲染的过程中会被`重复触发`，故被标记为不安全并被取代。

---
### 二、什么是异步渲染，为什么要使用它
在react 16以前，对我们的react组件的更新和渲染是以`深度优先`、`堆栈`和`同步`的方式进行的，它会一直执行到栈空为止。

大量的同步计算任务阻塞了浏览器的 UI 渲染。默认情况下，JS 运算、页面布局和页面绘制都是运行在浏览器的主线程当中，他们之间是互斥的关系。如果 JS 运算持续占用主线程，页面就没法得到及时的更新。当我们调用setState更新页面的时候，React 会遍历应用的所有节点，计算出差异，然后再更新 UI。整个过程是一气呵成，不能被打断的。如果页面元素很多，整个过程占用的时机就可能超过 16 毫秒，就容易出现掉帧的现象。

就是当一次更新或者一次加载开始以后，diff virtual dom并且渲染的过程是一口气完成的。如果组件层级比较深，相应的堆栈也会很深，长时间占用浏览器主线程，一些类似`用户输入`、`鼠标滚动`等操作得不到响应。借Lin的两张图，视频 [A Cartoon Intro to Fiber - React Conf 2017](https://www.youtube.com/watch?v=ZCuYPiUIONs)
![react sync](/images/20180428113702655.jpg)
在页面阻塞的这段时间中甚至连在浏览器`控制页面关闭`都无法操作（因为浏览器也需要页面的代码执行完）

而异步渲染的方式可以解决上面的问题，就是把一整个渲染任务拆分成一个一个小任务，并根据系统的空闲时间进行渲染：
![upload successful](/images/pasted-0.png)

如果用户在某一个小任务执行时进行了操作，渲染进度将会停下来去执行`更高优先级（用户操作）`的事情。

他解决了我们什么痛点：
1. 同步渲染时用户操作无法响应
2. 更好的性能

例子：[react-fiber-vs-stack-demo](https://claudiopro.github.io/react-fiber-vs-stack-demo/)

---
### 三、试用异步渲染
#### 官方文档：

“fiber” reconciler 是一个新尝试，致力于解决 stack reconciler 中固有的问题，同时解决一些历史遗留问题。Fiber 从 React 16 开始变成了默认的 reconciler。

你可以在[`这里`](https://github.com/acdlite/react-fiber-architecture)和[`这里`](https://medium.com/react-in-depth/inside-fiber-in-depth-overview-of-the-new-reconciliation-algorithm-in-react-e1c04700ef6e)，深入了解 React Fiber 架构。虽然这已经在 React 16 中启用了，但是 async 特性还没有默认开启。

#### 官方更新日志：

##### 16.6.0 (October 23, 2018)
---
- Add React.memo() as an alternative to PureComponent for functions. (@acdlite in #13748)
- Add React.lazy() for code splitting components. (@acdlite in #13885)
- React.StrictMode now warns about legacy context API. (@bvaughn in #13760)
- React.StrictMode now warns about findDOMNode. (@sebmarkbage in #13841)
- ***Rename unstable_AsyncMode to unstable_ConcurrentMode. (@trueadm in #13732)***
- Rename unstable_Placeholder to Suspense, and delayMs to maxDuration. (@gaearon in #13799 and @sebmarkbage in #13922)

---
所以如果你是`react 16.6以前`的版本请使用`React.unstable_AsyncMode`，`react 16.6以后`的版本请使用`React.unstable_ConcurrentMode`来体验异步渲染模式。

```jsx
import React from 'react';
import ReactDOM from 'react-dom';
import App from 'components/App';

const AsyncMode = React.unstable_AsyncMode || React.unstable_ConcurrentMode;
const createApp = (store) => (
      <AsyncMode>
        <App store={store} />
      </AsyncMode>
);

export default createApp
```