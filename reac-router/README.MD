# [react-router](https://github.com/ReactTraining/react-router)
1. 源码分析 基于v4.2.2
2. 使用了lerna管理packages，可以让你在主项目下管理多个子项目，从而解决了多个包互相依赖，且发布时需要手动维护多个包的问题
3. 编译cjs和es模式时，使用了babel；编译umd模式时，使用了rollup
4. 基础知识 [history](https://github.com/ReactTraining/history) 库， 主要目的是创建history对象
  - createBrowserHistory, 支持[HTML5 history API](https://developer.mozilla.org/en-US/docs/Web/API/History_API)
  - createMemoryHistory, 适用于非DOM的环境，例如ReactNative
  - createHashHistory, legacy browser support

## react-router
### MemoryRouter(适用于非DOM的环境，例如ReactNative）
源码同下HashRouter，区别在于使用了history/createMemoryHistory创建的history，以及支持的属性略有不同
```
static propTypes = {
    initialEntries: PropTypes.array,
    initialIndex: PropTypes.number,
    getUserConfirmation: PropTypes.func,
    keyLength: PropTypes.number,
    children: PropTypes.node
}
```
### StaticRouter(适用于server-side rendering scenarios)
1. 通过legacy context，向子组件传递router.statciContext
```
  static childContextTypes = {
    router: PropTypes.object.isRequired
  }

  getChildContext() {
    return {
      router: {
        staticContext: this.props.context
      }
    }
  }
```
2. 不支持传递history属性，每次渲染时创建history
```
  render() {
    const { basename, context, location, ...props } = this.props

    const history = {
      createHref: this.createHref,
      action: 'POP',
      location: stripBasename(basename, createLocation(location)),
      push: this.handlePush,
      replace: this.handleReplace,
      go: staticHandler('go'),
      goBack: staticHandler('goBack'),
      goForward: staticHandler('goForward'),
      listen: this.handleListen,
      block: this.handleBlock
    }

    return <Router {...props} history={history}/>
  }
```
3. 通过context记录push、replace，不支持go、goBack、goForward、listen、block。
### Router
该组件是HashRouter、BrowserRouter等的底层Router实现，其最终渲染的是属性children的内容，且只支持children数目为1. 通常children即项目业务的入口组件
1. 支持2个属性：history，children
2. 使用了legacy context，既从父组件中通过this.context.router获取router，也向子组件传递丰富后的router
  ```
  static contextTypes = {
    router: PropTypes.object
  }

  static childContextTypes = {
    router: PropTypes.object.isRequired
  }

  getChildContext() {
    return {
      router: {
        ...this.context.router,
        history: this.props.history,
        route: {
          location: this.props.history.location,
          match: this.state.match
        }
      }
    }
  }
  ```
3. render
  ```
  render() {
    const { children } = this.props
    return children ? React.Children.only(children) : null
  }
  ```
4. this.state.match
  ```
  state = {
    match: this.computeMatch(this.props.history.location.pathname)
  }

  computeMatch(pathname) {
    return {
      path: '/',
      url: '/',
      params: {},
      isExact: pathname === '/'
    }
  }
  ```
### Route，当location.pathname匹配上path时，渲染该path对应的component
1. 支持的属性
  ```
  static propTypes = {
    computedMatch: PropTypes.object, // private, from <Switch>
    path: PropTypes.string,
    exact: PropTypes.bool,
    strict: PropTypes.bool,
    sensitive: PropTypes.bool,
    component: PropTypes.func,
    render: PropTypes.func,
    children: PropTypes.oneOfType([
      PropTypes.func,
      PropTypes.node
    ]),
    location: PropTypes.object
  }
  ```
2. 使用了legacy context，既从父组件(一般是Router)中通过this.context.router获取router，也向子组件传递丰富后的router
  ```
  static contextTypes = {
    router: PropTypes.shape({
      history: PropTypes.object.isRequired,
      route: PropTypes.object.isRequired,
      staticContext: PropTypes.object
    })
  }

  static childContextTypes = {
    router: PropTypes.object.isRequired
  }

  getChildContext() {
    return {
      router: {
        ...this.context.router,
        route: {
          location: this.props.location || this.context.router.route.location,
          match: this.state.match
        }
      }
    }
  }
  ```
3. component、render、children 不应该同时出现，优先级依次是component、render、children。
4. matchPath 采用了 path-to-regexp来实现匹配， 其中返回的match对象如下所示：
  ```
  {
    path, // the path pattern used to match
    url: path === '/' && url === '' ? '/' : url, // the matched portion of the URL
    isExact, // whether or not we matched exactly
    params: keys.reduce((memo, key, index) => {   // 通过正则获取到的路由的动态参数部分，例如/far/:bar, 则会有params={bar:'xxx'}
      memo[key.name] = values[index]
      return memo
    }, {})
  }
  ```
5. 传递给需渲染的component(或render、children)的props={match, location, history, staticContext }
### Switch
1. children 是Route 或者是 Redirect
2. 遍历children，找出第一个匹配的，并渲染它。
### Redirect
```
perform() {
    const { history } = this.context.router
    const { push, to } = this.props

    if (push) {
      history.push(to)
    } else {
      history.replace(to)
    }
}
```
### withRouter
1. [使用了hoist-non-react-statics来自动复制 高阶组件中被包裹的组件 的静态方法](https://reactjs.org/docs/higher-order-components.html#static-methods-must-be-copied-over)。
2. 在component组件上封装一层Route组件，从而可以传递props={match, location, history}等属性给component
3. [为什么 withRouter 高阶组件应该 处于最外层？](https://zhuanlan.zhihu.com/p/36895827)
>因为 withRouter 本身的作用是为了给那些需要操作 route 的组件提供 location 等属性。如果一个组件本身就已经能拿到这些属性了，那再使用 withRouter 就是一种浪费了。通过Route包裹的组件，就实在没必要包裹一层 withRouter 了
4. [可以解决re-render时，可能会被PureComponent和shouldComponentUpdate阻断更新的问题](https://github.com/ReactTraining/react-router/blob/master/packages/react-router/docs/guides/blocked-updates.md)
5. 设置withRouter的属性wrappedComponentRef，可以传递给被包裹组件。

### Prompt
主要功能是调用history.block，其中when属性决定是否调用，message是调用block时提示的内容
```
  enable(message) {
    if (this.unblock)
      this.unblock()

    this.unblock = this.context.router.history.block(message)
  }
  componentWillMount() {
    invariant(
      this.context.router,
      'You should not use <Prompt> outside a <Router>'
    )

    if (this.props.when)
      this.enable(this.props.message)
  }
```

## react-router-dom
### HashRouter
```
import warning from 'warning'
import React from 'react'
import PropTypes from 'prop-types'
import createHistory from 'history/createHashHistory'
import Router from './Router'

/**
 * The public API for a <Router> that uses window.location.hash.
 */
class HashRouter extends React.Component {
  static propTypes = {
    basename: PropTypes.string,
    getUserConfirmation: PropTypes.func,
    hashType: PropTypes.oneOf([ 'hashbang', 'noslash', 'slash' ]),
    children: PropTypes.node
  }

  history = createHistory(this.props)

  componentWillMount() {
    warning(
      !this.props.history,
      '<HashRouter> ignores the history prop. To use a custom history, ' +
      'use `import { Router }` instead of `import { HashRouter as Router }`.'
    )
  }

  render() {
    return <Router history={this.history} children={this.props.children}/>
  }
}

export default HashRouter
```
首先，HashRouter是React的组件，其子组件是Router。 其次，源码的核心在于使用了history/createHashHistory创建的history
### BrowserRouter
源码与HashRouter基本相同，区别在于使用了history/createBrowserHistory创建的history，以及支持的属性不同。
```
static propTypes = {
    basename: PropTypes.string,
    forceRefresh: PropTypes.bool,
    getUserConfirmation: PropTypes.func,
    keyLength: PropTypes.number,   // 生成key的长度
    children: PropTypes.node
}
```
### Link
对a标签的一个封装，使用history的push或replace来进行页面跳转
### NavLink
对Link组件的一个封装(增加了父组件Route来对path的是否匹配进行判断)，支持activeClassName(和/或activeStyle)，以及isActive的逻辑判断。 如果active则应用activeClassName(和/或activeStyle)对应的样式。
## react-router-native

## react-router-config
静态路由配置助手，ReactRoute V4并不需要一个集中的路由配置，react-router-config适用于以下几种场景：
1. 服务端加载数据 或 渲染下一屏前 等类似钩子函数作用的场景 Loading data on the server or in the lifecycle before rendering the next screen
2. 按name进行路由跳转 Linking to routes by name
3. 静态分析 Static analysis

注意事项：
1. Route仅接受component属性，不接render或children
2. routes属性用来定义子路由
3. 支持key属性来性能优化
### matchRoutes(routes, pathname)
返回匹配pathname的所有{route,match}(包括嵌套的子route）
### renderRoutes(routes, extraProps = {})
使用Switch来渲染routes(即子路由),支持给子路由统一增加额外属性extraProps
```
// example
const Child = ({ route }) => (
  <div>
    <h2>Child</h2>
    {/* child routes won't render without this */}
    {renderRoutes(route.routes, { someProp: "these extra props are optional" })}
  </div>
);
```
## react-router-redux (version:5.0.0-alpha.6)(后续版本中已经deprecated)
1. 提供了ConnectedRouter用来从react-redux的Provider中获取store，并监听location的变化及时更新this.store.location
```
import React, { Component } from 'react'
import PropTypes from 'prop-types'
import { Router } from 'react-router'

import { LOCATION_CHANGE } from './reducer'

class ConnectedRouter extends Component {
  static propTypes = {
    store: PropTypes.object,
    history: PropTypes.object,
    children: PropTypes.node
  }

  static contextTypes = {
    store: PropTypes.object    // 可以通过this.context.store 来实时获取store
  }

  handleLocationChange = location => {
    this.store.dispatch({
      type: LOCATION_CHANGE,
      payload: location
    })
  }

  componentWillMount() {
    const { store:propsStore, history } = this.props
    this.store = propsStore || this.context.store
    this.handleLocationChange(history.location)   // 初始化this.store.location
  }

  componentDidMount() {
    const { history } = this.props  // 此处的history需与routerMiddleware(history)为同一个，BrowserHistory或HashHistory等
    this.unsubscribeFromHistory = history.listen(this.handleLocationChange)  // location发生变化时及时更新this.store.location
  }

  componentWillUnmount() {
    if (this.unsubscribeFromHistory) this.unsubscribeFromHistory()
  }

  render() {
    return <Router {...this.props} />
  }
}

export default ConnectedRouter

```
2. 提供了routerReducer用来修改this.state.router
```
export function routerReducer(state = initialState, { type, payload } = {}) {
  if (type === LOCATION_CHANGE) {
    return { ...state, location: payload }
  }

  return state
}
```
3. 提供了push、replace、go、goBack、goForward等action，它们的action.type均为CALL_HISTORY_METHOD
4. 提供了routerMiddleware用来加强redux
```
import { CALL_HISTORY_METHOD } from './actions'

/**
 * This middleware captures CALL_HISTORY_METHOD actions to redirect to the
 * provided history object. This will prevent these actions from reaching your
 * reducer or any middleware that comes after this one.
 */
export default function routerMiddleware(history) {
  return () => next => action => {
    if (action.type !== CALL_HISTORY_METHOD) {
      return next(action)
    }

    const { payload: { method, args } } = action
    history[method](...args)
  }
}
```
当action === CALL_HISTORY_METHOD时，直接操作history，history会监听location的变化并触发this.store.location的变化

5. 基础使用方法如下所示：
```
import React from 'react'
import ReactDOM from 'react-dom'

import { createStore, combineReducers, applyMiddleware } from 'redux'
import { Provider } from 'react-redux'

import createHistory from 'history/createBrowserHistory'
import { Route } from 'react-router'

import { ConnectedRouter, routerReducer, routerMiddleware, push } from 'react-router-redux'

import reducers from './reducers' // Or wherever you keep your reducers

// Create a history of your choosing (we're using a browser history in this case)
const history = createHistory()

// Build the middleware for intercepting and dispatching navigation actions
const middleware = routerMiddleware(history)

// Add the reducer to your store on the `router` key
// Also apply our middleware for navigating
const store = createStore(
  combineReducers({
    ...reducers,
    router: routerReducer
  }),
  applyMiddleware(middleware)
)

// Now you can dispatch navigation actions from anywhere!
// store.dispatch(push('/foo'))

ReactDOM.render(
  <Provider store={store}>
    { /* ConnectedRouter will use the store from Provider automatically */ }
    <ConnectedRouter history={history}>
      <div>
        <Route exact path="/" component={Home}/>
        <Route path="/about" component={About}/>
        <Route path="/topics" component={Topics}/>
      </div>
    </ConnectedRouter>
  </Provider>,
  document.getElementById('root')
)
```
6. [为什么deprecated？](https://reacttraining.com/react-router/web/guides/redux-integration)
>Our recommendation is not to keep your routes in your Redux store at all. Reasoning:
>1. Routing data is already a prop of most of your components that care about it. Whether it comes from the store or the router, your component’s code is largely the same.
>2. In most cases, you can use Link, NavLink and Redirect to perform navigation actions. Sometimes you might also need to navigate programmatically, after some asynchronous task that was originally initiated by an action. For example, you might dispatch an action when the user submits a login form. Your thunk, saga or other async handler then authenticates the credentials, then it needs to somehow navigate to a new page if successful. The solution here is simply to include the history object (provided to all route components) in the payload of the action, and your async handler can use this to navigate when appropriate.
>3. Route changes are unlikely to matter for time travel debugging. The only obvious case is to debug issues with your router/store synchronization, and this problem goes away if you don’t synchronize them at all.
But if you feel strongly about synchronizing your routes with your store, you may want to try [Connected React Router](https://github.com/supasate/connected-react-router), a third party binding for React Router v4 and Redux.

备注：Connected React Router 感觉像是从react-router-redux中分离出去继续维护了。
