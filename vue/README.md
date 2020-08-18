# vue2.6.11 源码分析
任何项目的源码分析，都应该从package.json文件开始
1. 使用rollup进行构建
2. main：dist/vue.runtime.common.js
3. module: dist/vue.runtime.esm.js
4. dist/vue.runtime.esm.js 对应的web入口文件为src/platforms/web/entry-runtime.js
