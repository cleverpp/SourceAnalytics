## redux源码分析[v4.0.0]
```
import {createStore, combineReducers, applyMiddleware} from 'redux';
import reducers from './reducers/index';//all reduces
import thunk from 'redux-thunk'; //dispatch(function)


//创建一个 Redux store 来以存放应用中所有的 state，应用中应有且仅有一个 store。
var store = createStore(
    combineReducers(reducers),
    applyMiddleware(thunk)
);

/**
 * log states
 * @type {Unsubscribe}
 */
let unsubscribe = store.subscribe(() => { //监听状态的变化
        console.log("unsubscribe");
        console.log(store.getState());
    }
);
```
### applyMiddleware
1. 函数式编程currying：一种使用匿名单参数函数来实现多参数函数的方法。
2. 函数式编程compose：组合函数，让其函数数组从右到左执行
    ```
    export default function compose(...funcs) {
      if (funcs.length === 0) {
        return arg => arg
      }

      if (funcs.length === 1) {
        return funcs[0]
      }

      return funcs.reduce((a, b) => (...args) => a(b(...args)))
    }
    ```
3. 共享store，在applyMiddleware执行的过程中，store还是旧的，但是因为闭包的存在，applyMiddleware完成后，所有的middleware内部拿到的store是最新且相同的。
4. middleware的形式，以react-thunk为例，三层嵌套函数：
    ```
    export default ({ dispatch, getState }) => next => action => {
                       if (typeof action === 'function') {
                         return action(dispatch, getState, extraArgument);
                       }

                       return next(action);
                     };
    ```
5. 中间件的功能：增强dispatch

6. 源码注解
```
// 执行applyMiddleware(m1,m2)后返回了俩层嵌套函数，其中middlewares=[m1,m2]
export default function applyMiddleware(...middlewares) {
  return createStore => (...args) => {
    // 创建store，此时dispatch未增强
    const store = createStore(...args)

    // 声明空dispatch，方便闭包内的middleware能共享最新且相同的dispatch
    let dispatch = () => {
      throw new Error(
        `Dispatching while constructing your middleware is not allowed. ` +
          `Other middleware would not be applied to this dispatch.`
      )
    }

    const middlewareAPI = {
      getState: store.getState,
      dispatch: (...args) => dispatch(...args)  // 传入middleware，方便共享最新且相同的dispatch
    }

    // chain = [f1=next=>action=>{1...next(action)...1}, f2=next=>action=>{2...next(action)...2}]
    const chain = middlewares.map(middleware => middleware(middlewareAPI))

    // compose(...chain)(store.dispatch) = f1(f2(store.dispatch))
    // f1next = f2(store.dispatch) = action=>{2...store.dispatch(action)...2}
    // f1(f1next) = action=>{1...{2...store.dispatch(action)...2}...1}
    dispatch = compose(...chain)(store.dispatch)

    return {
      ...store,
      dispatch
    }
  }
}
```
7. example
```
function createStore(){ return {getState:function(){console.log('getState');},dispatch:function(){console.log('orignal dispatch')}}};

var m1 = store=>next=>action=>{console.log('m1 begin'); next(action);console.log('m1 end')};
var m2 = store=>next=>action=>{console.log('m2 begin'); next(action);console.log('m2 end')};

var enhanceStore = applyMiddleware(m1,m2)(createStore)();
// 示例1，演示洋葱模型
enhanceStore.dispatch({type:'test',params:{a:1}})

var m3 = ({dispatch,getState})=>next=>action=>{ console.log('m3 begin'); if(typeof action === 'function') {return action(dispatch,getState)};   return next(action); console.log('m3 end');}

enhanceStore = applyMiddleware(m1,m2,m3)(createStore)();
// 示例2，演示return类型
enhanceStore.dispatch({type:'test',params:{a:1}})

// 示例3，演示action为函数时
function action(dispatch,getState){ dispatch({type:'test',params:{a:1}});}
enhanceStore.dispatch(action);
```
执行结果如下：
```
m1 begin
m2 begin
orignal dispatch
m2 end
m1 end

m1 begin
m2 begin
m3 begin
orignal dispatch   // return了，所以不会执行m3 end.
m2 end
m1 end

m1 begin
m2 begin
m3 begin
m1 begin
m2 begin
m3 begin
orignal dispatch
m2 end
m1 end
m2 end
m1 end

```

### combineReducers
1. 执行combineReducers(reducers)得到的是个combination(state = {}, action)，该函数返回一个以reducer函数名作为其属性，reducer执行结果作为其值的一个对象，即全局state
2. 每一个action，都会被所有的reducer函数执行，且注意action是对象引用，因此如果其中一个reducer改变了action，会影响后续的reducer
3. 以下源码去掉了一些不影响逻辑的代码，并加上了注释
```
export default function combineReducers(reducers) {
  const reducerKeys = Object.keys(reducers)  // reducers是一个对象，reducerKeys是每一个reducer函数名
  const finalReducers = {}
  // for循环进行了浅copy，将reducers复制到finalReducers
  for (let i = 0; i < reducerKeys.length; i++) {
    const key = reducerKeys[i]

    if (typeof reducers[key] === 'function') {
      finalReducers[key] = reducers[key]
    }
  }
  const finalReducerKeys = Object.keys(finalReducers)

  return function combination(state = {}, action) {

    let hasChanged = false
    const nextState = {}
    for (let i = 0; i < finalReducerKeys.length; i++) {
      const key = finalReducerKeys[i]
      const reducer = finalReducers[key]
      const previousStateForKey = state[key] // 储存当前store中的state状态
      const nextStateForKey = reducer(previousStateForKey, action) // 执行reducer函数得到新state
      if (typeof nextStateForKey === 'undefined') {
        const errorMessage = getUndefinedStateErrorMessage(key, action)
        throw new Error(errorMessage)
      }
      nextState[key] = nextStateForKey  // store中的state对象的属性即为reducer函数名
      hasChanged = hasChanged || nextStateForKey !== previousStateForKey  // hasChanged只要循环中有一次reducer产生了新的state则为true
    }
    return hasChanged ? nextState : state
  }
}
```

### createStore
1. 第二个参数默认为preloadedState，如果为函数，例如applyMiddleware(thunk)，则返回的是使用中间件增强过的store
    ```
      if (typeof preloadedState === 'function' && typeof enhancer === 'undefined') {
        enhancer = preloadedState
        preloadedState = undefined
      }

      if (typeof enhancer !== 'undefined') {
        if (typeof enhancer !== 'function') {
          throw new Error('Expected the enhancer to be a function.')
        }

        return enhancer(createStore)(reducer, preloadedState)
      }
    ```
2. 由dispatch({ type: ActionTypes.INIT })开始，对state tree进行初始化
3. dispatch，每一次dispatch执行都会执行所有的reducer，不管状态是否变化，如果有订阅callback，均会执行。
    ```
    function dispatch(action) {
        if (!isPlainObject(action)) {   //仅支持普通对象，不支持Promise, an Observable, a thunk, or something else
          throw new Error(
            'Actions must be plain objects. ' +
              'Use custom middleware for async actions.'
          )
        }

        if (typeof action.type === 'undefined') {
          throw new Error(
            'Actions may not have an undefined "type" property. ' +
              'Have you misspelled a constant?'
          )
        }

        if (isDispatching) { // 当前是否正在处理中，这个检查主要是为了避免在reducer中执行dispatch的情况
          throw new Error('Reducers may not dispatch actions.')
        }

        try {
          isDispatching = true

          // currentReducer 即combineReducers执行后返回的combination(state = {}, action)函数
          currentState = currentReducer(currentState, action)
        } finally {
          isDispatching = false
        }

        const listeners = (currentListeners = nextListeners)
        // 如果有使用store.subscribe订阅callback，则执行callback
        for (let i = 0; i < listeners.length; i++) {
          const listener = listeners[i]
          listener()
        }

        return action
      }
    ```

## react-redux
```
// app.js
  ReactDOM.render(
      <Provider store={store}>
        <Component />
      </Provider>,
    document.getElementById('root')
  )


// xxxContainer.js
import React from 'react';
import {connect} from 'react-redux';
// v4 新增
import {withRouter} from 'react-router-dom'
//page
import Page from 'Page.jsx';

function mapStateToProps(state, ownProps) {
  return {
    a: state.a,
    b: state.b
  }
}

function mapDispatchToProps(dispatch, ownProps) {
  return {
    setA: (params) => dispatch(actionCreator(params)),
    setB: (params) => dispatch(actionCreator(params))
  }
}

export default withRouter(connect(mapStateToProps, mapDispatchToProps)(AllCoupon));

```
### Provider
一个React组件，使用了[Legacy Context](https://reactjs.org/docs/legacy-context.html),
这样任意子组件只需定义contextTypes即可以获取到store。
```
export function createProvider(storeKey = 'store', subKey) {
    const subscriptionKey = subKey || `${storeKey}Subscription`

    class Provider extends Component {
        getChildContext() {
          return { [storeKey]: this[storeKey], [subscriptionKey]: null }
        }

        constructor(props, context) {
          super(props, context)
          this[storeKey] = props.store;  //获取props.store并挂载到当前实例
        }

        render() {
          return Children.only(this.props.children)  // provider仅接受一个子组件
        }
    }

    if (process.env.NODE_ENV !== 'production') {
      Provider.prototype.componentWillReceiveProps = function (nextProps) {
        if (this[storeKey] !== nextProps.store) {
          warnAboutReceivingStore()
        }
      }
    }

    Provider.propTypes = {
        store: storeShape.isRequired,
        children: PropTypes.element.isRequired,
    }
    Provider.childContextTypes = {
        [storeKey]: storeShape.isRequired,
        [subscriptionKey]: subscriptionShape,
    }

    return Provider
}

export default createProvider()
```

### connect
1. connect接受接受4个参数：mapStateToProps，mapDispatchToProps，mergeProps，extraOptions。
2. mapStateToProps: 函数，返回的对象是从redux的state提取的，并把它当做props传给当前组件。
3. mapDispatchToProps：函数(也可以是对象)，返回的对象是能改变redux的state的方法（即dispatch(action)），并把它当做props传给当前组件。
4. connect最终返回的是一个高阶组件(HOC)，该高阶组件定义了contextTypes可以获取到Provider中提供的store。
    1. construct
        1. 传递store.getState()到mapStateToProps(state,ownProps),获得想要传递到目标组件的stateProps
        2. 传递store.dispatch到mapDispatchToProps(dispatch,ownProps)，获得想要传递到目标组件的dispatchProps
        3. mergeProps(stateProps, dispatchProps, ownProps)合并三种props并传递给目标组件，即WrappedComponent。
        4. 注册subscription，并使用legacy context的方式传递subscription给任意子组件
    2. render：传递合并后的props到WrappedComponent
        ```
        return createElement(WrappedComponent, this.addExtraProps(selector.props))
        ```
    3. componentDidMount
        1. this.store.subscribe(this.onStateChange) 注册监听store的变化
        2. 再一次获取store.getState()并计算最新的props
    4. onStateChange
        1. 再一次获取store.getState()并计算最新的props
        2. 通知订阅者
        3. 调用this.setState({})，主要目的是触发再次调用render，使得新的props可以传递给WrappedComponent
    5. componentWillReceiveProps： 再一次获取store.getState()并计算最新的props

## 更多
1. redux-saga
2. react-router-redux（for react-router 2.x， 3.x）  或  connected-react-router （for react-router 4.x）
3. 高阶reducer解决以下问题：reducer的复用、reducer的增强
4. 表单问题：redux-form-utils、redux-form
5. redux性能优化：1）利用纯函数的缓存特性，减少不必要的计算。2) 数据Immutable，redux-immutable

## 参考
1. 深入React技术栈