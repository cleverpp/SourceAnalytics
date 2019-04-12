# createElement
在View组件中，最后将View组件转化为div的调用
```
return createElement('div', supportedProps);

// const createElement = (component, props, ...children) => {......}
```

# 核心功能如下
1. 无障碍功能处理：AccessibilityUtil.propsToAccessibilityComponent(props)
2. createDOMProps
3. adjustProps(domProps)
4. React.createElement(Component, domProps, ...children)

## 无障碍功能处理：AccessibilityUtil.propsToAccessibilityComponent(props)
```
  let accessibilityComponent;
  if (component && component.constructor === String) {  // 一种判断component为字符串的方法
    accessibilityComponent = AccessibilityUtil.propsToAccessibilityComponent(props);
  }
  
// propsToAccessibilityComponent.js
const roleComponents = {
  article: 'article',
  banner: 'header',
  complementary: 'aside',
  contentinfo: 'footer',
  form: 'form',
  link: 'a',
  list: 'ul',
  listitem: 'li',
  main: 'main',
  navigation: 'nav',
  region: 'section'
};

const emptyObject = {};

const propsToAccessibilityComponent = (props: Object = emptyObject) => {
  // special-case for "label" role which doesn't map to an ARIA role
  if (props.accessibilityRole === 'label') {
    return 'label';
  }

  const role = propsToAriaRole(props);
  if (role) {
    if (role === 'heading') {
      const level = props['aria-level'] || 1;
      return `h${level}`;
    }
    return roleComponents[role];
  }
};
```
其中propsToAriaRole(props)，将props中无障碍属性（accessibilityComponentType,accessibilityRole,accessibilityTraits）处理为web对无障碍功能的支持属性，详细可了解[ARIA - Accessibility | MDN](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA)

简单的说，如果RN中设置了无障碍属性，那么View应该不是简单的转换为div了，而是会转化为语义化的标签，例如：article、footer、section等。

## createDOMProps
```
const domProps = createDOMProps(Component, props);
```
在createDOMProps逻辑中
1. 处理无障碍相关的属性转化
2. 处理disabled
3. 处理是否可focus
4. 处理style【重点】
  ```
  const { className, style } = styleResolver(reactNativeStyle);  // 核心代码
  if (className && className.constructor === String) {
    domProps.className = props.className ? `${props.className} ${className}` : className;
  }
  if (style) {
    domProps.style = style;
  }
  ```
  其中styleResolver(reactNativeStyle)=
5. nativeID转化为web元素的id
