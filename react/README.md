# React源码分析
[react github](https://github.com/facebook/react)

[react中文文档](https://react.docschina.org/)

[react英文文档](https://reactjs.org/)

源码结构概览：[codebase overview](https://reactjs.org/docs/codebase-overview.html)

## [React15]()
1. grunt进行构建
2. Stack Reconciliation :更新时会锁住整个主线程，影响页面的事件响应、动画等。
3. 主要数据结构：ReactElement、ReactComponent 等构建vDom tree（自上而下的简单树结构）。
4. 生命周期
    > ![](https://pic2.zhimg.com/80/v2-60e8649b6b3aa27c91be6243b3c03db1_hd.jpg)

## [React16]()
1. rollup进行构建
2. Fiber Reconciliation :实现了一个基于优先级和requestIdleCallback的循环任务调度算法。更新异步化，可中断，可管理更新的优先级
3. 主要数据结构：ReactElement、Fiber 构建的fiber tree 以及 workInProgress tree。（基于单链表的树结构）
4. 生命周期发生变化
    1. 增加了新生命周期：getDerivedStateFromProps、getSnapshotBeforeUpdate、componentDidCatch
    2. componentWillMount、componentWillReceiveProps、componentWillUpdate变得不安全，不推荐使用了，后续版本中会废弃。
5. 异步渲染：


# 源码阅读边角料
1. Object.freeze
2. [Rollup](https://www.rollupjs.com/guide/zh):JavaScript 模块打包器，可以将小块代码编译成大块复杂的代码
3. [Flow](https://flow.org/en/)的使用:Flow is a static type checker for your JavaScript code
4. JS中的依赖注入（DI:dependency injection）：src/renderers/dom/shared/ReactDefaultInjection.js
5. 什么是幂等（idempotent）:在编程中一个幂等操作的特点是其任意多次执行所产生的影响均与一次执行的影响相同。
6. requestIdleCallback和requestAnimationFrame。

