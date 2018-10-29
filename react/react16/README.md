## 基于React v16.3.2进行源码分析与学习
### 核心概念
#### 重点源码
#### 组件的挂载 ReactDOM.render
1. 创建ReactRoot
```
ReactRoot._internalRoot:FiberRoot = {
    current:FiberNode {
        tag:HostRoot,
        stateNode:ReactRoot._internalRoot
        ...
    },
    containerInfo:DOMContainer,
    ...
}
```
2. renderRoot 是reconciliation阶段，通过循环进行深度优先搜索（DFS）来构建了Fiber tree 和 workInProgress tree

    1. performUnitOfWork从根节点（HostRoot）开始沿着子节点一直到无子节点
    2. completeUnitOfWork从无子节点开始回溯，先回溯兄弟节点，再回溯父节点
    3. 如果是兄弟节点，则继续执行performUnitOfWork。
    4. 如果是父节点，则判断父节点是否有兄弟节点，有则执行3，否则继续执行4

3. completeRoot 是提交阶段，根据收集到的effectTag，进行对应的操作。
    1. commitBeforeMutationLifeCycles ：此处执行生命周期函数getSnapshotBeforeUpdate，此时尚未加载或更新到dom中
    2. commitAllHostEffects : 此处根据effectTag决定执行 Placement, PlacementAndUpdate, Update, Deletion 等操作
    3. commitLifeCycles ：此处执行生命周期componentDidMount 或 componentDidUpdate

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