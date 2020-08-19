# vue2.6.11 源码分析
任何项目的源码分析，都应该从package.json文件开始
1. 使用rollup进行构建
2. main：dist/vue.runtime.common.js
3. module: dist/vue.runtime.esm.js
4. dist/vue.runtime.esm.js 对应的web入口文件为src/platforms/web/entry-runtime.js
5. 在源码构建脚本scripts/config.js，进行了以下构建的分类：
  - platform区分：web、weex
  - 构建模块规范区分：cjs(commonJS)、esm(ES6 Module)、browser、umd
  - 构建功能区分：仅runtime、仅compiler、runtime+compiler
  - 构建环境区分：dev、prod
  - 对于web，还区分了csr(客户端或浏览器端渲染)、ssr(服务端渲染)

Vue.js 的源码都在 src 目录下，其目录结构如下。
```
src
├── compiler        # 编译相关 
├── core            # 核心代码 
├── platforms       # 不同平台的支持
├── server          # 服务端渲染
├── sfc             # .vue 文件解析
├── shared          # 共享代码
```
## 入口分析
从入口文件src/platforms/web/entry-runtime.js 或 src/platforms/web/entry-runtime-with-compiler.js中可以看到Vue的来源
```
import Vue from './runtime/index'
```
## 设计思路

## 扩展机制
