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
// 执行applyMiddleware(thunk)后返回了俩层嵌套函数，其中middlewares=[thunk]
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

enhanceStore.dispatch({type:'test',params:{a:1}})
```
执行结果如下：（符合洋葱模型）
```
m1 begin
m2 begin
orignal dispatch
m2 end
m1 end
```

### createStore

### combineReducers

## react-redux

## 参考