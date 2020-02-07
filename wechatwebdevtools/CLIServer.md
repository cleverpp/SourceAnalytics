# CLI Server
1. 启动http server，其中requestListen中实现了一下几个接口
    - /updatePort ， 更新CLI port
    - /open ，打开主窗口
    - /quit ， 退出主窗口
    - /login ，登录
    - /loginresult ，查询登录结果
    - /preview ，上传代码并生成预览二维码
    - /upload ， 上传代码
    - /test ，执行自动测试
    - 判断是否为测试模式.如果是,则初始化...
      ```
      global.CLI.isTestMode && i.init()
      ```
 2. 发请求
    - `http://127.0.0.1:${f}/updatePort?port=${c}`
    - `http://127.0.0.1:${f}/wechatdevtools/${encodeURIComponent(global.CLI.id)}/pid/${process.pid}`
    - `http://127.0.0.1:${f}/wechatdevtools/${encodeURIComponent(global.CLI.id)}/updateport/${c}`
    
## 预览功能
### 上传代码
有以下几个过程：
  - 构建build
    + checkfile
    + compilejs
    + compileotherfile
  - 打包pack
  - 上传upload
    + 上传服务（Bata版）：https://servicewechat.com/wxa-dev-new/commitsource?user-version=${c}&user-desc=${a}&uuid=${d}&gzip=1
    + 上传服务： https://servicewechat.com/wxa-dev/commitsource?user-version=${c}&user-desc=${a}&uuid=${d}&gzip=1

### 生成二维码
