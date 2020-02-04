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
