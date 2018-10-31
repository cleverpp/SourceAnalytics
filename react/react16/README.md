## 基于React v16.3.2进行源码分析与学习
### 核心概念
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

![组件的挂载和更新](https://github.com/cleverpp/SourceAnalytics/blob/master/react/react16/images/reactdom_render.png)

#### 组件的更新 this.setState

上图中的红色箭头线展示this.setState流程，与render的流程区别在于：setState的调用对象非根节点，node.return!==null，因此需要循环回归到根节点才开始执行requestWork。
#### 生命周期
1. reconiliation阶段

    在beginWork执行时，会根据当前workInProgress.tag 执行对应的组件的方法，其中涉及到声明周期的只有updateClassComponent

![reconiliation阶段的生命周期](https://github.com/cleverpp/SourceAnalytics/blob/master/react/react16/images/lifecycle-reconcile.png)

2. commint阶段

    1. 在commitRoot的commitBeforeMutationLifecycles阶段，tag=ClassComponent时会执行生命周期函数getSnapshotBeforeUpdate。
    2. 在commitRoot的commitAllHostEffects阶段，如果当前操作是Deletion则执行commitDeletion，此处会执行生命周期componentWillUnmount
    3. 在commitRoot的commitAllLifeCycles阶段，根据当前Fiber.tag来进行不同的处理，tag=ClassComponent时，如果是挂载则执行componentDidMount，更新执行componentDidUpdate，报错则执行componentDidCatch。
#### react diff
主要在reconcileChildFibers阶段进行diff

1. 单个元素，调用reconcileSingleElement
    1. key值不同，则deleteChild，effectTag为Deletion。
    2. key相等且type相等，删除旧子节点的兄弟节点，复用旧节点并返回。
    3. key相等且type不相等，删除旧子节点及兄弟节点，跳出循环，不能复用，则直接新建Fiber实例，并返回
2. 单个Portal元素，调用reconcileSinglePortal
3. string或者number，调用reconcileSingleTextNode
4. Array（Fragment），调用reconcileChildrenArray

注：2，3，4 同1的逻辑类似。

#### 异步渲染


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
4. [React16源码之React Fiber架构](https://juejin.im/post/5b7016606fb9a0099406f8de) 【推荐】
5. [React Fiber架构](https://zhuanlan.zhihu.com/p/37095662)
6. [关于React v16.3 新生命周期](https://juejin.im/post/5aca20c96fb9a028d700e1ce)

### 其它
1. packages/shared/ReactDOMFrameScheduling.js ：react自己实现了requestIdleCallback