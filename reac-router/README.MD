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
5. 传递给需渲染的component的props={match, location, history, staticContext }
### Switch



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

## react-router-native

## react-router-config

## react-router and redux
