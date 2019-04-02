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
# 核心功能主要有：
1. compose
2. create
3. faltten
