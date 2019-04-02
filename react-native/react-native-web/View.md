# View组件的核心内容如下：
1. filterSupportedProps(this.props) 只保留支持的属性
2. StyleSheet.compose 组合样式
3. 处理hitSlop属性
4. createElement('div', supportedProps)， View对应的web中的div。
5. export default applyLayout(applyNativeMethods(View))

## applyNativeMethods

## applyLayout
