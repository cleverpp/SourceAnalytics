# 微信开发者工具 探析
```
// package.json
{
    "forceVendor": true,
    "main": "html/index.html",
    "name": "微信web开发者工具",
    "productName": "微信开发者工具",
    "appname": "wechatwebdevtools",
    "version": "1.02.1810190",
    "window": {
        "id": "init",
        "title": "微信开发者工具 v1.02.1810190",
        "icon": "app/images/logo2.png",
        "toolbar": true,
        "width": 334,
        "height": 474,
        "show": false,
        "transparent": false,
        "frame": false
    },
    "inject_js_start": "js/extensions/inject/documentstart/index.js",
    "inject_js_end": "js/extensions/inject/devtools/index.js",
    "chromium-args": "--disable-http2 --disable-devtools --ignore-gpu-blacklist  --enable-experimental-web-platform-features  -ignore-certificate-errors",
    "node-remote": [
        "http://*.open.weixin.qq.com/*",
        "https://*.mp.weixin.qq.com/*",
        "http://127.0.0.1:*/*"
    ],
    "webview": {
        "partitions": [
            {
                "name": "devtools",
                "accessible_resources": [
                    "<all_urls>"
                ]
            },
            {
                "name": "remotedebugdevtools",
                "accessible_resources": [
                    "<all_urls>"
                ],
                "web_accessible_resources": [
                    "<all_urls>"
                ]
            },
            {
                "name": "gitmanager",
                "accessible_resources": [
                    "<all_urls>"
                ],
                "web_accessible_resources": [
                    "<all_urls>"
                ]
            },
            {
                "name": "simulator",
                "accessible_resources": [
                    "<all_urls>"
                ]
            },
            {
                "name": "editor",
                "accessible_resources": [
                    "<all_urls>"
                ]
            },
            {
                "name": "appservice",
                "accessible_resources": [
                    "<all_urls>"
                ]
            },
            {
                "name": "htmlwebview",
                "accessible_resources": [
                    "<all_urls>"
                ]
            }
        ]
    },
    "product_string": "wechatwebdevtools"
}
```

基于package.json文件中，我们了解到微信开发者工具的7大功能区：
  1. devtools（开发者工具的调试工具）
  2. remotedebugdevtools（远程真机调试）
  3. gitmanager（基于git的版本管理）
  4. simulator（左侧模拟器，即时显示编码效果）
  5. editor（右侧编辑器，编码区域）
  6. appservice
  7. htmlwebview
  
除此之外，还需要关注两个注入js
  1. "inject_js_start": "js/extensions/inject/documentstart/index.js",
  2. "inject_js_end": "js/extensions/inject/devtools/index.js",
  
# 源码分析
## 入口 html/index.html，js/core/index.js
1. hack() 禁用拖拽、滚轮缩放
2. initGlobal() global变量的一些初始化，global.worker懒加载、获取查询参数（包括：appid、projectname、projectpath等信息）
3. initMenu() 
4. init() 

    4.1 响应close事件：关闭所有webview、关闭除主窗口Win外的其它窗口、等待serverWindowSync.clientWindows.size=0时主窗口退出
    
    4.2 根据启动参数中是否有inspect来决定是否打开调试窗口chrome://inspect/#devices
    
    4.3 根据是否多账号模式来决定加载不同的js
    
    
在js/core/index.js 中require了一下几个js：tools、global.worker.bbsLogWorker、background
### tools
```
......
module.exports = {
    getAvailablePort: async function() {...},
    getType: function(a) {...},
    getAppConfig: function() {...},
    checkUpdateApp: async function() {...},
    openInspectWin: function() {...},
    getVersionNum: function b(a) {...},
    compareLibVersion: function(a, b) {...},
    compareSemVer: function c(a, b) {...},,
    quit: function e() {...},
    rmSync: function f(a) {...},
    mvSync: function g(a, b) {...},
    cpSync: function j(a, b) {...},
    mkdirSync: function h(a) {...},
    getQuery: function(a='') {...},
    normalizePath: function(a='') {...},
    generateMD5: function(a) {...},
    chooseFile: async function(a) {...},
    fillArray: function(b, c) {...}
}
```

### global.worker.bbsLogWorker
```
self.onmessage = event => {
  if (event.data) {
    switch (event.data.msgType) {
      case 'evaluate': {
        if (event.data.msgData) {
          if (event.data.msgData.message) {
            evaluate(event.data.msgData.message, event.data.msgData.context, event.data.msgData.ext)
          }
        }
        break
      }
      case 'refreshSession': {
        config = JSON.parse(JSON.stringify(originalConfig || '[]'))
        break
      }
      case 'updateConfig': {
        originalConfig = event.data.msgData
        config = JSON.parse(JSON.stringify(originalConfig || '[]'))
        break
      }
    }
  }
}
```
一个全局消息监听者

### background 【主要业务逻辑入口】
区分了多账号模式登录和非多账号模式

1. 多账号模式登录
    - 文档readyState==='compolete'或load事件后开始处理
    - 获取当前有效端口，并赋值给global.proxyPort，该端口作为参数用于启动代理服务器f.startProxyServer 【ProxyServer.md】
    - 再次获取当前有效端口，并赋值给global.messageCenterPort，该端口作为参数启动服务e.start(p) 【WebSocketServer.md】
    - 如果是macOS，则执行h.init(global.Win) 【初始化主窗口menu】
    - 处理异常。非开发者模式时，process.on('uncaughtException'、window.addEventListener('error'、global.contentWindow.addEventListener('error'，开发者模式时只处理了uncaughtException
    - **dispatch({type: m.USER_UPDATE_INFO, userInfo: q})**
    - **采用Object.defineProperties方式赋值global.userinfo，方便及时获取或设置最新的用户信息**
    - 加载模块s并执行s.init()，实际执行的是创建了一个BroadcastChannel，其中BROADCAST_CHANNEL_NAME为ACTION_TRANSFER
    - 执行渲染，ReactDOM.render(element, container)，React.createElement(type,[props],[...children])
        ```
        // b:react-dom, a:react, c:react-redux的Provider，d: redux的store，l：React.Component【0-入口组件-2.md】
        b.render(a.createElement(c, {
            store: d
        }, a.createElement(l, null)), global.contentDocument.getElementById('container'))
        ```
2. 非多账号模式
    - 文档readyState==='compolete'或load事件后开始处理
    - 获取当前有效端口，并赋值给global.proxyPort，该端口作为参数用于启动代理服务器f.startProxyServer 【ProxyServer.md】
    - 再次获取当前有效端口，并赋值给global.messageCenterPort，该端口作为参数启动服务e.start(p) 【WebSocketServer.md】
    - 如果是macOS，则执行h.init(global.Win) 【初始化主窗口menu】
    - **处理启动参数含--inspect，则打开调试窗口chrome://inspect/#devices**
    - **处理启动参数含--cli**
    - **显示主窗口**
    - **获取当前有效端口，并赋值给global.cliPort，该端口作为参数启动CLI Server： n.start(a, t) 【CLIServer.md】**
    - 处理异常。非开发者模式时，process.on('uncaughtException'、window.addEventListener('error'、global.contentWindow.addEventListener('error'，开发者模式时只处理了uncaughtException
    - **异步获取系统相关信息，例如:os、cpu、mem等**
    - ***区分开发模式和开发模式，加载模块u，并进行初始化u.init()，实际执行的是创建了一个BroadcastChannel，其中BROADCAST_CHANNEL_NAME为WINDOW_SYNC，并监听消息。***
    - **监听切换locale事件：q.onChangeLocale((a,b)=>u.notifyChangeLocale(b))**
    - 执行渲染，ReactDOM.render(element, container)，React.createElement(type,[props],[...children])
        ```
        // b:react-dom, a:react, c:react-redux的Provider，f: redux的store，e：React.Component【0-入口组件-1.md】
        b.render(a.createElement(c, {
            store: f
        }, a.createElement(e, null)), global.contentDocument.getElementById('container')),
        ```
    - 区分开发模式和开发模式，加载模块，并进行初始化init()，实际执行的是创建了一个BroadcastChannel，其中BROADCAST_CHANNEL_NAME为ACTION_TRANSFER
    - **获取当前版本号，并从缓存（文件存储）中获取lastVersion进行比较，如果比lastVersion大，则更新缓存，并执行l('update_success_report') 【待进一步分析】**

## 从用户操作流程一步一步解析
### 0-入口组件-1【非多账号模式】
8e433dffaa20c3a7331f9aeecb1221b0.js
### 0-入口组件-2【多账号模式】
60fdb5a14c198acde3823b610d29f71f.js
### 1-扫码登录_LOGIN【MAIN_WINDOW_TYPE.LOGIN】
2dba3da3f9fb5ee64049d4556bb9519d.js
### 2_工程入口_ENTRANCE【MAIN_WINDOW_TYPE.ENTRANCE】
0deb78a56cbcc9f9f3b0823d28892e90.js

## 从各个功能进行解析
### 预览
### 真机调试
