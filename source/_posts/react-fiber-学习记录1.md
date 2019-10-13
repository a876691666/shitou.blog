title: react fiber 学习记录
tags:
  - react
  - 学习记录
  - react性能优化
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
<!-- more -->
#### 官方文档：

##### Stack reconciler
> “stack” reconciler 是 React 15 及更早的解决方案。虽然我们已经停止了对它的使用, 但是这在下一章节有详细的文档。

##### Fiber reconciler
> “fiber” reconciler 是一个新尝试，致力于解决 stack reconciler 中固有的问题，同时解决一些历史遗留问题。Fiber 从 React 16 开始变成了默认的 reconciler。

---
### 二、什么是异步渲染，为什么要使用它
在react 16以前，对我们的react组件的更新和渲染是以`深度优先`、`堆栈`和`同步`的方式进行的，它会一直执行到栈空为止。

> 大量的同步计算任务阻塞了浏览器的 UI 渲染。默认情况下，JS 运算、页面布局和页面绘制都是运行在浏览器的主线程当中，他们之间是互斥的关系。如果 JS 运算持续占用主线程，页面就没法得到及时的更新。当我们调用setState更新页面的时候，React 会遍历应用的所有节点，计算出差异，然后再更新 UI。整个过程是一气呵成，不能被打断的。如果页面元素很多，整个过程占用的时机就可能超过 16 毫秒，就容易出现掉帧的现象。

> 就是当一次更新或者一次加载开始以后，diff virtual dom并且渲染的过程是一口气完成的。如果组件层级比较深，相应的堆栈也会很深，长时间占用浏览器主线程，一些类似`用户输入`、`鼠标滚动`等操作得不到响应。借Lin的两张图，视频 [A Cartoon Intro to Fiber - React Conf 2017](https://www.youtube.com/watch?v=ZCuYPiUIONs)

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

> “fiber” reconciler 是一个新尝试，致力于解决 stack reconciler 中固有的问题，同时解决一些历史遗留问题。Fiber 从 React 16 开始变成了默认的 reconciler。

> 你可以在[`这里`](https://github.com/acdlite/react-fiber-architecture)和[`这里`](https://medium.com/react-in-depth/inside-fiber-in-depth-overview-of-the-new-reconciliation-algorithm-in-react-e1c04700ef6e)，深入了解 React Fiber 架构。虽然这已经在 React 16 中启用了，但是 `async 特性还没有默认开启`。

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

---
### 四、什么是 Fiber

在这之前我们先了解一下React的内部结构，可以分为3层：
1. Virtual DOM 层，也就是我们常说的虚拟DOM，用来对页面DOM进行描述。
2. Reconciler 层，负责调用组件生命周期方法，进行Diff运算等。
3. Renderer 层，根据不同平台，渲染出相应的页面，比较常见的是ReactDOM和ReactNative

这次改动最大的当属Reconciler层了，React团队也给他起了个新名字，也就是`Fiber Reconciler`。为了加以区分，以前的Reconciler被命名为`Stack Reconciler`。

Fiber其实指的是一种数据结构，他可以用一个纯JS对象来表示:
```javascript
const fiber = {
    stateNode,    // 节点实例
    child,        // 子节点
    sibling,      // 兄弟节点
    return,       // 父节点
}
```

在Fiber真正执行前需要有一个调度器 (Scheduler) 来进行任务分配。任务的优先级有六种：
1. synchronous，与之前的Stack Reconciler操作一样，同步执行
2. task，在next tick之前执行
3. animation，下一帧之前执行
4. high，在不久的将来立即执行
5. low，稍微延迟执行也没关系
6. offscreen，下一次render时或scroll时才执行
优先级高的任务（如键盘输入）可以打断优先级低的任务（如Diff）的执行，从而更快的生效。

> React v16.3.2的优先级，不再这么划分，分为三类：NoWork、sync、async，前两类可以认为是同步任务，需要在当前tick完成，过期时间为null，最后一类异步任务会计算一个expirationTime，在workLoop中，根据过期时间来判断是否进行下一个分片任务，scheduleWork中更新任务优先级，也就是更新这个expirationTime。至于这个时间怎么计算，可以[查看源码](https://github.com/facebook/react/blob/v16.3.2/packages/react-reconciler/src/ReactFiberExpirationTime.js)。

Fiber Reconciler 在执行过程中，会分为 2 个阶段, Reconciliation Phase和Commit Phase。
- Reconciliation Phase，找出要做的更新工作（Diff Fiber Tree）生成 Fiber 树，得出需要更新的节点信息。这一步是一个渐进的过程，`计算结果可以被缓存，可以被打断`。
- Commit Phase，将需要更新的节点一次过批量更新，为了防止页面抖动，`这个过程不能被打断`。

> `componentWillMount` `componentWillReceiveProps` `componentWillUpdate` 几个生命周期方法，在Reconciliation Phase被调用，有被打断的可能（时间用尽等情况），所以可能被多次调用。其实 `shouldComponentUpdate` 也可能被多次调用，只是它只返回`true`或者`false`，没有副作用，可以暂时忽略。

阶段一可被打断的特性，让优先级更高的任务先执行，从框架层面大大降低了页面掉帧的概率。
![upload successful](/images/pasted-2.png)


### 五、参考资料：

[1]. [React Fiber 原理介绍](https://segmentfault.com/a/1190000018250127?utm_source=tag-newest)

[2]. [React-从源码分析React Fiber工作原理](https://blog.csdn.net/qiqingjin/article/details/80118669)

[3]. [深入理解React16之：（一）.Fiber架构](https://www.jianshu.com/p/bf824722b496)