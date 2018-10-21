# React源码分析
[react github](https://github.com/facebook/react)

[react中文文档](https://react.docschina.org/)

[react英文文档](https://reactjs.org/)

## React16之前，例如：react15.4.2
### 核心概念
#### ReactElement
1. 描述react中的虚拟的DOM节点的对象
2. render中的JSX语法实际上是调用了ReactElement.createElement来实现生成一个ReactElement对象
3. ReactElement主要包含了这个DOM节点的类型（type）、属性（props）和子节点（children）
4. 不同的type在挂载时会被实例化为不同的ReactComponent，type为string时实例化为ReactDOMComponent，为class或function时实例化为ReactCompositeComponent
#### ReactComponent
1. mountComponent方法实现组件的挂载
2. updateComponent方法实现组件的更新
3. setState方法，将新的state放入更新队列中，并触发更新
#### 组件的挂载 ReactDOM.render
1. ReactElement.createElement 会递归的实现由一个个ReactElement组成虚拟DOM
2. instantiateReactComponent时根据ReactElement的type实例化不同的ReactComponent
3. ReactTextComponent会将文本插入到DOM中
4. ReactDOMComponent，会递归mountComponent，最终转化为普通html插入到DOM中。
5. ReactCompositeComponent， 先执行生命周期componentWillMount和render，然后递归mountComponent，最后执行生命周期componentDidMount。
#### 组件的更新 this.setState
#### react diff

### 参考及学习笔记
1. [React源码解析(一):组件的实现与挂载](https://juejin.im/post/5983dfbcf265da3e2f7f32de) 核心要点如下：    
    1. 组件是一个ReactElement类型的对象，组件与组件的父子关系通过props.children来关联。
    2. ReactDOM.render(component,mountNode)，不同类型的component对应不同的React内部四大类封装组件，记为componentInstance。而后将其作为参数传入mountComponentIntoNode方法中，由此获得组件对应的HTML，记为变量markup。将真实的DOM的属性innerHTML设置为markup，即完成了DOM插入。
    3. 封装成四大类型组件（ReactEmptyComponent、ReactTextComponent、ReactDOMComponent、ReactCompositeComponent）的过程中，赋予了封装组件mountComponet方法， 执行该方法会触发组件的生命周期，从而解析出HTML

2. [React源码解析(二):组件的类型与生命周期](https://juejin.im/post/59ca03b9518825177c60d10b) 核心要点如下：
    1. ReactEmptyComponent，实际注入的是ReactDOMEmptyComponent，不存在生命周期，最终插入到DOM中的是空。
    2. ReactTextComponent，实际注入的是ReactDOMTextComponent，不存在生命周期，最终插入到DOM中的是文本。
    3. ReactDOMComponent，实际注入的是ReactDOMComponent，不存在生命周期，最终插入到DOM中的是一些具有tag的元素```<tag>tagContent</tag>```
    4. ReactCompositeComponent，有生命周期。componentWillMount、render，componentDidMount。其中父组件与子组件的生命周期的关系如下图：
    ```
    /**
     * ------------------ The Life-Cycle of a Composite Component ------------------
     *
     * - constructor: Initialization of state. The instance is now retained.
     *   - componentWillMount
     *   - render
     *   - [children's constructors]
     *     - [children's componentWillMount and render]
     *     - [children's componentDidMount]
     *     - componentDidMount
     *
     *       Update Phases:
     *       - componentWillReceiveProps (only called if parent updated)
     *       - shouldComponentUpdate
     *         - componentWillUpdate
     *           - render
     *           - [children's constructors or receive props phases]
     *         - componentDidUpdate
     *
     *     - componentWillUnmount
     *     - [children's componentWillUnmount]
     *   - [children destroyed]
     * - (destroyed): The instance is now blank, released by React and ready for GC.
     *
     * -----------------------------------------------------------------------------
     */
    ```

3. [React源码解析(三):详解事务与更新队列](https://juejin.im/post/59cc4c4bf265da0648446ce0) 核心要点如下：
    1. Transaction的作用:可以理解为一种中间件的设计模式，利用wrappers在anyMethod执行前和执行后插入一些功能。
    ```
    <pre>
    *                       wrappers (injected at creation time)
    *                                      +        +
    *                                      |        |
    *                    +-----------------|--------|--------------+
    *                    |                 v        |              |
    *                    |      +---------------+   |              |
    *                    |   +--|    wrapper1   |---|----+         |
    *                    |   |  +---------------+   v    |         |
    *                    |   |          +-------------+  |         |
    *                    |   |     +----|   wrapper2  |--------+   |
    *                    |   |     |    +-------------+  |     |   |
    *                    |   |     |                     |     |   |
    *                    |   v     v                     v     v   | wrapper
    *                    | +---+ +---+   +---------+   +---+ +---+ | invariants
    * perform(anyMethod) | |   | |   |   |         |   |   | |   | | maintained
    * +----------------->|-|---|-|---|-->|anyMethod|---|---|-|---|-|-------->
    *                    | |   | |   |   |         |   |   | |   | |
    *                    | |   | |   |   |         |   |   | |   | |
    *                    | |   | |   |   |         |   |   | |   | |
    *                    | +---+ +---+   +---------+   +---+ +---+ |
    *                    |  initialize                    close    |
    *                    +-----------------------------------------+
    * </pre>

    ```
    2. setState的流程
    ![](https://user-gold-cdn.xitu.io/2018/3/1/161df93690f3abac?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)
    3. 将需要更新的组件放入dirtyComponents，如何处理dirtyComponents？（ReactUpdates.flushBatchedUpdates）
        > flushBatchedUpdates方法循环遍历所有的dirtyComponents，又通过事务的形式调用runBatchedUpdates方法，因为源码较长所以在这里直接说明该方法所做的两件事：一是通过执行updateComponent方法来更新组件;二是若setState方法传入了回调函数则将回调函数存入callbackQueue队列。
4. [React源码解析(四):事件系统](https://juejin.im/post/5a0cf54ff265da43333df2c4) 核心要点如下：  
    1. React并不像原生事件一样将事件和DOM一一对应，而是将所有的事件都绑定在网页的document，通过统一的事件监听器处理并分发，找到对应的回调函数并执行。
    2. React对事件进行统一而不是分散的存储与管理，捕获事件后内部生成合成事件提高浏览器的兼容度，执行回调函数后再进行销毁释放内存，从而大大提高网页的响应性能。

5. [深入理解react（源码分析）](https://github.com/lanjingling0510/blog/issues/1)
6. [React源码分析 - 组件初次渲染](https://juejin.im/post/5a92e02d6fb9a0633d71f7f7)
7. [React源码分析 - 事件机制](https://juejin.im/post/5a92e120f265da4e9e307fea)
8. [React源码分析 - 组件更新与事务](https://juejin.im/post/5a92e234f265da4e6f1801e6)
9. [React源码分析 - 生命周期](https://juejin.im/post/5a92e2d65188257a6426bad0)
10. [React源码分析 - Diff算法](https://juejin.im/post/5aa163df518825557b4c4f0a)
11. [react源码剖析——（一）生命周期的管理艺术](https://www.jianshu.com/p/18523cdaf893)
12. [react源码剖析——（二）解密setState机制](https://www.jianshu.com/p/0cdaafe2a26e)
13. [react源码剖析——（三）不可思议的React diff算法](https://www.jianshu.com/p/ca44182828ae)
14. [React源码剖析——（四）新引擎React Fiber](https://www.jianshu.com/p/420b5c030e98)

## React16后，例如：react16.3.2
### 核心概念
#### React Fiber

### 参考及学习笔记
1. 

# 源码阅读边角料
1. Object.freeze
2. [Rollup](https://www.rollupjs.com/guide/zh):JavaScript 模块打包器，可以将小块代码编译成大块复杂的代码
3. [Flow](https://flow.org/en/)的使用:Flow is a static type checker for your JavaScript code
4. JS中的依赖注入（DI:dependency injection）：src/renderers/dom/shared/ReactDefaultInjection.js
5. 什么是幂等（idempotent）:在编程中一个幂等操作的特点是其任意多次执行所产生的影响均与一次执行的影响相同。

