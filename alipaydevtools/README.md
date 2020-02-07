# 支付宝小程序IDE 探析
```
// package.json
{
  "name": "小程序开发者工具",
  "version": "1.5.7",
  "description": "小程序开发者工具",
  "main": "vol_modules.asar/node_modules/@alipay/volans-source/out/bootstrap.js",
  "files": [
    "out",
    "package-lock.json"
  ],
  "volans_dynamic_bundle_exclude": [
    "out/vendor",
    "node_modules"
  ],
  "volans_build_asar": {
    "enable": true,
    "global_node_modules": {
      "nsfw": "^1.1.1",
      "node-ripgrep": "^0.6.0-patch.1",
      "@ali/vscode-ripgrep": "^1.3.1-release",
      "typescript": "3.6.2"
    }
  },
  "config": {
    "commitizen": {
      "path": "./node_modules/@alipay/vol-conventional-changelog"
    },
    "electron_mirror": "https://npm.alibaba-inc.com/mirrors/electron/"
  },
  ......
}
```
# 源码分析
## 入口文件

