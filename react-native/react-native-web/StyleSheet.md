# API - StyleSheet分析
```
onst StyleSheet = {
  absoluteFill,
  absoluteFillObject,
  compose(style1, style2) {
    if (style1 && style2) {
      return [style1, style2];
    } else {
      return style1 || style2;
    }
  },
  create(styles) {
    const result = {};
    Object.keys(styles).forEach(key => {
      if (process.env.NODE_ENV !== 'production') {
        StyleSheetValidation.validateStyle(key, styles);
      }
      const id = styles[key] && ReactNativePropRegistry.register(styles[key]);
      result[key] = id;
    });
    return result;
  },
  flatten: flattenStyle,
  hairlineWidth: 1
};

export default StyleSheet;
```
1. absoluteFillObject : 默认绝对定位的样式（ position absolute and zero positioning ）
```
const absoluteFillObject = {
  position: 'absolute',
  left: 0,
  right: 0,
  top: 0,
  bottom: 0
};
```
2. absoluteFill : 它的值是absoluteFillObject对象在ReactNativePropRegistry中注册的id值。
```
const absoluteFill = ReactNativePropRegistry.register(absoluteFillObject);
```
3. compose ： 组合样式，要么两个参数样式以数组形式返回，要么返回其中之一
4. create ： 将每个样式对象在ReactNativePropRegistry中注册，返回的对象中存储的不是样式，而是样式对象对应的id
5. faltten
```
function getStyle(style) {
  if (typeof style === 'number') {
    return ReactNativePropRegistry.getByID(style);
  }
  return style;
}

function flattenStyle(style: ?StyleObj): ?Object {
  if (!style) {
    return undefined;
  }

  if (process.env.NODE_ENV !== 'production') {
    invariant(style !== true, 'style may be false but not true');
  }

  if (!Array.isArray(style)) {
    // $FlowFixMe
    return getStyle(style);
  }

  const result = {};
  for (let i = 0, styleLength = style.length; i < styleLength; ++i) {
    const computedStyle = flattenStyle(style[i]);
    if (computedStyle) {
      for (const key in computedStyle) {
        const value = computedStyle[key];
        result[key] = value;
      }
    }
  }
  return result;
}

export default flattenStyle;
```
- 样式是number型，则应该是id，通过ReactNativePropRegistry.getByID(style)来获取具体的样式
- 样式是数组，递归调用数组中的每一项，然后保存在对象result中，这样保证key值相同的样式，以数组后面的为准。即后面非赋值会覆盖前面的赋值。
- 样式是其它类型，例如object，则直接返回该object。
