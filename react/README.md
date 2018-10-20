# React源码分析
[react github](https://github.com/facebook/react)

[react中文文档](https://react.docschina.org/)

[react英文文档](https://reactjs.org/)

## React16之前，例如：react15.4.2
### 核心概念
1. Transaction
2. 
### 参考
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
    1. 

## React16后，例如：react16.3.2

# 源码阅读边角料
1. Object.freeze
2. [Rollup](https://www.rollupjs.com/guide/zh):JavaScript 模块打包器，可以将小块代码编译成大块复杂的代码
3. [Flow](https://flow.org/en/)的使用:Flow is a static type checker for your JavaScript code
4. JS中的依赖注入（DI:dependency injection）：src/renderers/dom/shared/ReactDefaultInjection.js
5. 什么是幂等（idempotent）:在编程中一个幂等操作的特点是其任意多次执行所产生的影响均与一次执行的影响相同。

