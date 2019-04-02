# react-native-web 源码分析

## 入口
```
// index.js
import {Platform, AppRegistry} from 'react-native';
import App from './App';
import {name as appName} from './app.json';

AppRegistry.registerComponent(appName, () => App);

if (Platform.OS === 'web') {
    AppRegistry.runApplication(appName, {
        // 启动时传给 App 组件的属性
        initialProps: {},
        // 渲染 App 的 DOM 容器
        rootTag: document.getElementById('react-root')
    });
}
```
### AppRegistry
1. AppRegistry.registerComponent

    执行完该函数后，得到一个runnables[appName]对象，该对象有两个函数属性getApplication、run。
  
2. AppRegistry.runApplication

    

## 以View组件为例

## 如何扩展



# 参考
1. [React Native转web方案：react-native-web](https://juejin.im/post/5b79397be51d45389153b060)
