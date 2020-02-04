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
## proxyLocal

## proxy
   ```
      function(callback) {
        self.httpProxyServer = http.createServer(requestHandler.userRequestHandler)
        callback(null)
      },
   ```
以上说明proxy是http server。其中核心逻辑是requestHandler.userRequestHandler部分
1. 获取请求信息
2. 判断是本地响应还是远程响应userRule.shouldUseLocalResponse(req, reqData)
3. 本地响应,读取的是文件req.anyproxy_map_local
   ```
   dealLocalResponse: function(req, reqBody, callback) {
    if (req.method == "OPTIONS") {
      callback(200, mergeCORSHeader(req.headers), "")
    } else if (req.anyproxy_map_local) {
      fs.readFile(req.anyproxy_map_local, function(err, buffer) {
        if (err) {
          callback(200, {}, "[AnyProxy failed to load local file] " + err)
        } else {
          callback(200, {}, buffer)
        }
      })
    }
   },
   ```
4. 远程响应
  - 默认的userRules是不会处理远程响应的，如果需要处理远程响应，需调用requestHandler.setRules(newRules)
  - 追溯rules，发现只覆盖了：forceLocalProxy，dealLocalResponse



