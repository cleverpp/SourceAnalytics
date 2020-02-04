在对主入口进行分析后，了解到核心业务逻辑中，首先获取有效端口并启动代理服务器，这篇主要对其进行分析。
```
module.exports = {
  startProxyServer,
  updateProxy,
  clearProxyCache,
  getProxyForURL
}
```
# startProxyServer
1. 获取配置信息
```
let {
    port,
    rule,
    type = 'DIRECT',
    hostname = '127.0.0.1'
  } = options
```
2. 清除代理缓存clearProxyCache
```
const clearProxyCache = () => {
  proxyCache = {}
}
```
3. 处理rule.forceLocalProxy,当其为数组时创建本地代理
```
proxyLocal.makeProxyLocal(port, rule.forceLocalProxy)
rule.shouldUseLocalResponse = (req, reqData) => {...}  //根据请求url，判断是否使用本地代理响应
```
4. 创建代理服务器实例
```
new proxy.proxyServer({
    port,
    hostname,
    rule,
})
```
# proxyLocal
主要看一下proxyLocal.makeProxyLocal(port, rule.forceLocalProxy)，注释显示强制代理到本地
```
let proxyLocalCache


// 强制代理到本地
const makeProxyLocal = (port, rules = []) => {
  let res = []
  rules.forEach(item => {
    let type = Object.prototype.toString.call(item)
    if (type == '[object RegExp]') {
      res.push(`${item}.test(url)`)
    } else if (type == '[object String]') {
      res.push(`url.indexOf('${item}') === 0`)
    }
  })

  let forceDirectReg = /^(http|ws)s?\:\/\/(localhost|127.0.0.1)/

  proxyLocalCache = `if (${res.join('||')}) {
    return 'PROXY 127.0.0.1:${port}'
  }

  if (${forceDirectReg}.test(url)){
    return 'DIRECT'
  }`
}


module.exports = {
  makeProxyLocal,
  get config(){
    return proxyLocalCache
  },
  set config(value) {
    proxyLocalCache = value
  }
}
```
# proxy
   ```
      function(callback) {
        self.httpProxyServer = http.createServer(requestHandler.userRequestHandler)
        callback(null)
      },
   ```
以上说明proxy是http server。其中核心逻辑是requestHandler.userRequestHandler部分
1. 获取请求信息
2. 判断是本地响应还是远程响应userRule.shouldUseLocalResponse(req, reqData)
3. 远程响应
  - 默认的userRules是不会处理远程响应的，如果需要处理远程响应，需调用requestHandler.setRules(newRules)
  - 追溯rules，发现只覆盖了：forceLocalProxy，dealLocalResponse
4. 根据上下文，只会处理本地响应
  - 从请求头user-agent中获取token，并进行验证C.validateSessionToken(c[1], B.UA_TOKEN)
  - 判断请求url的类型然后走不同的分支
    + /appservice ： getAppServiceSource，获取appservice相关的源码文件
    + /calibration
    + /\__pageframe__ : getWebviewSource, 获取webview相关的源码
    + /editor : getEditorSource
    + /trace
    + /widgetwebview ：getWidgetWebviewResource
    + /widgetservice ：getWidgetServiceResource
    + /game  ： getGamePageResource
    + /aboutblank
    + /favicon.ico
    + https://clients1.google.com/tbproxy/af/
    + http://aboutblank
    + /^https?\:\/\/wxfile.open.weixin.qq.com\// 或者 http://tmp/ 或者 http://store/ 或者 http://usr/  ： getLocalIdResponse
    + /^https?\:\/\/.*?\.game.open.weixin.qq.com/  ：getGamePageResource
    + /usr  ：getUsrFileResponse
    + /ideplugin 或 /^https?\:\/\/.*?\.?pluginservice.open.weixin.qq.com/
    + /experience
 ## appservice
 1. \__asdebug__/asdebug.js
 响应内容为 文件./extensions/appservice/index.js， 初步推测是jsbridge
 2. \__workerasdebug__/workerasdebug.js
 响应内容为 文件./extensions/worker/index.js，推测作用是消息传递
 3. \__workerasdebug__/weixinworker.js
 响应内容为 文件./extensions/worker/weixinworker.tpl.js，消息监听作用
 4. ...
