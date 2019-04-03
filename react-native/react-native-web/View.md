# View组件
View 是一个支持 Flexbox 布局、样式、一些触摸处理、和一些无障碍功能的容器

# 源码分析
1. filterSupportedProps(this.props) 只保留支持的属性
2. StyleSheet.compose 组合样式
3. 处理hitSlop属性
4. createElement('div', supportedProps)， View对应的web中的div。
5. export default applyLayout(applyNativeMethods(View))

## 处理hitSlop属性
例如：
```
<View hitSlop={{top: 10, bottom: 10, left: 0, right: 0}}></View>
```
经过rnweb处理后,View转换成div，且div下增加了子元素span，大致如下代码一样的效果：
```
<div>
  <span style="position:absolute;top:-10px;bottom:-10px;left:0;right:0;"></span>
</div>
```
span的作用范围是整个屏幕

## applyNativeMethods
在NativeMethodsMixin中定义了一些和web平台相关的方法，如果转换后的View组件中没有这些方法，则添加这些方法。举例其中一个方法focus：
```
const NativeMethodsMixin = {
  /**
   * Requests focus for the given input or view.
   * The exact behavior triggered will depend the type of view.
   */
  focus() {
    UIManager.focus(findNodeHandle(this));
  },
};

const UIManager = {
  focus(node) {
    try {
      const name = node.nodeName;
      // A tabIndex of -1 allows element to be programmatically focused but
      // prevents keyboard focus, so we don't want to set the value on elements
      // that support keyboard focus by default.
      if (node.getAttribute('tabIndex') == null && focusableElements[name] == null) {
        node.setAttribute('tabIndex', '-1');
      }
      node.focus();
    } catch (err) {}
  },
}
```
其中findNodeHandle是调用了ReactDOM.findDOMNode(component)，找到当前组件挂载后所对应的dom元素， 然后调用该元素的原生focus方法。

## applyLayout
对组件的生命周期componentDidMount、componentDidUpdate、componentWillUnmount进行了增强，从而可以处理onLayout属性（当组件挂载或者布局变化的时候调用）。

以componentDidMount为例：
```
Component.prototype.componentDidMount = safeOverride(
    componentDidMount,
    function componentDidMount() {
      this._layoutState = emptyObject;
      this._isMounted = true;
      if (this.props.onLayout) {
        observe(this);
      }
    }
);
```
其中safeOverride(original,next)做的事情是，如果original存在则先执行original再执行next，否则只执行next

observer(this)，该函数实际使用了[window.ResizeObserver](https://zhuanlan.zhihu.com/p/41418813)和_handleLayout来监听该组件对应的DOM元素的resize变化，发生了变化则执行onLayout({nativeEvent: { layout: {x, y, width, height}}})
