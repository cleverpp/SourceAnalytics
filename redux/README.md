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
### createStore

### combineReducers

## react-redux

## 参考