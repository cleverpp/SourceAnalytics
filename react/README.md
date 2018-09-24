# React源码分析（React 16.5.2）
[react github](https://github.com/facebook/react)

[react中文文档](https://react.docschina.org/)

[react英文文档](https://reactjs.org/)

分析源码目录的package.json，使用了Rollup模块打包器。
```
 "scripts": {
    "build": "npm run version-check && node ./scripts/rollup/build.js",
    ....
 }
```

## [react-dom](https://github.com/facebook/react/tree/master/packages/react-dom)
```
ReactDOM.render(element, container[, callback])
```

## [react](https://github.com/facebook/react/tree/master/packages/react)

## [react-reconciler](https://github.com/facebook/react/tree/master/packages/react-reconciler)

# 源码阅读边角料
1. Object.freeze
2. [Rollup](https://www.rollupjs.com/guide/zh):JavaScript 模块打包器，可以将小块代码编译成大块复杂的代码
3. [Flow](https://flow.org/en/)的使用:Flow is a static type checker for your JavaScript code

