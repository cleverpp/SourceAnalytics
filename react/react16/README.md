## 基于React v16.3.2进行源码分析与学习
### 核心概念
#### 重点源码
1. ReactRoot， FiberNode
2. ReactWork
3. ReactFiber
4. ReactFiberReconciler
    1. createContainer
    2. updateContainer
#### 组件的挂载 ReactDOM.render
#### 组件的更新 this.setState
#### 生命周期
#### react diff

### 参考及学习笔记
1. [[译] React 16 带来了什么以及对 Fiber 的解释](https://juejin.im/post/59de1b2a51882578c70c0833)
2. [浅谈React16框架 - Fiber](https://zhuanlan.zhihu.com/p/43394081)
3. [完全理解React Fiber](http://www.ayqy.net/blog/dive-into-react-fiber/)
    1. 生命周期函数被分为2个阶段：第1阶段的生命周期函数可能会被多次调用，默认以low优先级（后面介绍的6种优先级之一）执行，被高优先级任务打断的话，稍后重新执行
    ```
    // 第1阶段 render/reconciliation
    componentWillMount
    componentWillReceiveProps
    shouldComponentUpdate
    componentWillUpdate

    // 第2阶段 commit
    componentDidMount
    componentDidUpdate
    componentWillUnmount
    ```
    2. fiber tree 以及 workInProgress tree
4. [React16源码之React Fiber架构](https://juejin.im/post/5b7016606fb9a0099406f8de)
5. [React Fiber架构](https://zhuanlan.zhihu.com/p/37095662)
6. [关于React v16.3 新生命周期](https://juejin.im/post/5aca20c96fb9a028d700e1ce)

### 其它
1. packages/shared/ReactDOMFrameScheduling.js ：react自己实现了requestIdleCallback