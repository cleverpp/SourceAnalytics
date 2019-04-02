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
            ```
            render() {
                const { children, WrapperComponent } = this.props;
                let innerView = (
                    <View
                        children={children}
                        key={this.state.mainKey}
                        pointerEvents="box-none"
                        style={styles.appContainer}
                    />
                );

                if (WrapperComponent) {
                    innerView = <WrapperComponent>{innerView}</WrapperComponent>;
                }

                return (
                    <View pointerEvents="box-none" style={styles.appContainer}>
                        {innerView}
                    </View>
                );
            }
            ```

综上分析，执行完index.js文件的内容后，实际上我们得到大概是这样的内容：
```
 <View pointerEvents="box-none" style={styles.appContainer}>
     <View
        children={<App/>}
        key={this.state.mainKey}
        pointerEvents="box-none"
        style={styles.appContainer}
      />
 </View>
```
## 组件及API的实现（src/index.js）
分以下几种类型：
1. top-level API，例如：createElement、render 等
2. modules
3. APIs，即已实现的APIs
4. components，即已实现的components
5. propTypes
6. compat (components)，即未实现的components，例如：DatePickerIOS，对于这种组件只提供了不会报错的最简组件实现UnimplementedView
7. compat (apis)，即尉氏县的APIs，例如：AlertIOS，对于这种API只是个emptyObject

## 如何扩展



# 参考
1. [React Native转web方案：react-native-web](https://juejin.im/post/5b79397be51d45389153b060)
