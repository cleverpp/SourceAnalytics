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
```
const runnables = {};
export default class AppRegistry {
  ......
  static registerComponent(appKey: string, componentProvider: ComponentProvider): string {
    runnables[appKey] = {
      getApplication: appParameters =>
        getApplication(
          componentProviderInstrumentationHook(componentProvider),
          appParameters ? appParameters.initialProps : emptyObject,
          wrapperComponentProvider && wrapperComponentProvider(appParameters)
        ),
      run: appParameters =>
        renderApplication(
          componentProviderInstrumentationHook(componentProvider),
          appParameters.initialProps || emptyObject,
          appParameters.rootTag,
          wrapperComponentProvider && wrapperComponentProvider(appParameters),
          appParameters.callback
        )
    };
    return appKey;
  }
  ......
  static runApplication(appKey: string, appParameters: Object): void {
    .......
    runnables[appKey].run(appParameters);
  }
  .......
```
1. AppRegistry.registerComponent

    执行完该函数后，得到一个runnables[appName]对象，该对象有两个函数属性getApplication、run。
  
2. AppRegistry.runApplication

    实际执行的是runnables[appName].run(appParameters)
    其中appParammeters即{initialProps:{}, rootTag:document.getElementById('react-root')}
    
3. renderApplication.js
    
    该js中包含getApplication、renderApplication的实现。
    ```
        export default function renderApplication<Props: Object>(
            RootComponent: ComponentType<Props>,
            initialProps: Props,
            rootTag: any,
            WrapperComponent?: ?ComponentType<*>,
            callback?: () => void
        ) {
            invariant(rootTag, 'Expect to have a valid rootTag, instead got ', rootTag);

            renderFn(
                <AppContainer WrapperComponent={WrapperComponent} rootTag={rootTag}>
                    <RootComponent {...initialProps} />
                </AppContainer>,
                rootTag,
                callback
            );
        }
    ```
    1. 参数分析
        - RootComponent = componentProviderInstrumentationHook(componentProvider) = componentProviderInstrumentationHook(()=>App), 最终得到的结果是RootComponent = App。
        - WrapperComponent 为null，可以通过AppRegistry.setWrapperComponentProvider函数进行设置。
    2. 函数体分析
        - renderFn即 ReactDOM.render 或 ReactDOM.hydrate
        - AppContainer组件：使用了[Legacy Context](https://react.docschina.org/docs/legacy-context.html),其属性rootTag可以实现跨组件传递，它的任何一层的子组件可以通过定义contextTypes来获取rootTag。

## 以View组件为例

## 如何扩展



# 参考
1. [React Native转web方案：react-native-web](https://juejin.im/post/5b79397be51d45389153b060)
