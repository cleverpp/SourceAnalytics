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
  const reactNativeStyle = [
    component === 'a' && resetStyles.link,
    component === 'button' && resetStyles.button,
    role === 'heading' && resetStyles.heading,
    component === 'ul' && resetStyles.list,
    role === 'button' && !disabled && resetStyles.ariaButton,
    pointerEvents && pointerEventsStyles[pointerEvents],
    providedStyle,
    placeholderTextColor && { placeholderTextColor }
  ];
  const { className, style } = styleResolver(reactNativeStyle);  // 核心代码
  if (className && className.constructor === String) {
    domProps.className = props.className ? `${props.className} ${className}` : className;
  }
  if (style) {
    domProps.style = style;
  }
  ```
  其中styleResolver(reactNativeStyle)=(new ReactNativeStyleResolver()).resolve(reactNativeStyle)
  
  深入分析，其处理方式为：
  - 将样式对象的属性一一创建为一个新的className，并以该className、prop、value生成一条原子css规则，并将该条规则插入到WebStyleSheet(“react-native-stylesheet”);
    ```
    injectDeclaration(prop, value): string {
    const val = normalizeValue(value);
    let className = this.getClassName(prop, val);  // 先去缓存中查找
    if (!className) {
      className = createClassName(prop, val);// 缓存中没有则创建className
      this._addToCache(className, prop, val);// 添加到缓存中，缓存中可以基于className获得样式属性及值，也可以通过样式属性及值获取className
      const rules = createAtomicRules(`.${className}`, prop, value);
      rules.forEach(rule => {
        this._sheet.insertRuleOnce(rule); //并将该条规则插入到WebStyleSheet(“react-native-stylesheet”)中
      });
    }
    return className;
    }
    ```
    创建className的规则为：基于该规则，同样的样式属性以及值相同，其生成的className是一样的。
    ```
    const createClassName = (prop, value) => {
      const hashed = hash(prop + normalizeValue(value));
      return process.env.NODE_ENV !== 'production' ? `rn-${prop}-${hashed}` : `rn-${hashed}`;
    };
    ```
  - 将该组件上所有的className放到数组prop.classList中，并将其转化为字符串最后作为该组件的prop.className.
  
5. nativeID转化为web元素的id
## adjustProps(domProps)
主要是处理以下几种事件类型的属性：
```
const eventHandlerNames = {
  onBlur: true,
  onClick: true,
  onClickCapture: true,
  onContextMenu: true,
  onFocus: true,
  onResponderRelease: true,
  onTouchCancel: true,
  onTouchCancelCapture: true,
  onTouchEnd: true,
  onTouchEndCapture: true,
  onTouchMove: true,
  onTouchMoveCapture: true,
  onTouchStart: true,
  onTouchStartCapture: true
};
```
如果是鼠标(mouse)事件，则normalizeMouseEvent，如果Touch事件，则normalizeTouchEvent。并将正规化的事件作为参数传递给事件函数
