# cordova原理分析『H5方向』
帮助H5应用通过调用插件的方式使用native功能，例如：拍照、获取本地照片，经纬度信息、存储、传感器等信息
## H5端的使用方式
1. 针对不同平台（android or ios）加载对应的cordova.js 文件【android3.6.4，ios3.6.3】
2. 监听deviceready事件【User event fired to indicate that Cordova is ready】
```
  isCordovaReady = false;
  onCordovaReady = function(callback){
    if(isCordovaReady){
        callback();
    }else {
        document.addEventListener('deviceready', function () {
            isCordovaReady = true;
            callback();
        }, false);
  }
```
3. 在onCordovaReady中使用自定义的插件
```
  // success: 成功回调, fail:失败回调, service插件名, action方法名, actionArgs：传递的参数
  onCordovaReady(function(){
    window.cordova.exec(success,fail,service,action,actionArgs)
  })
```
## js如何调用android native

## js如何调用ios native
1. jsToNativeModes有以下7中，一般用IFRAME_NAV
```
  jsToNativeModes = {
        IFRAME_NAV: 0,
        XHR_NO_PAYLOAD: 1,
        XHR_WITH_PAYLOAD: 2,
        XHR_OPTIONAL_PAYLOAD: 3,
        IFRAME_HASH_NO_PAYLOAD: 4,
        // Bundling the payload turns out to be slower. Probably since it has to be URI encoded / decoded.
        IFRAME_HASH_WITH_PAYLOAD: 5,
        WK_WEBVIEW_BINDING: 6
    }
```
2. cordova维护一个callback对象，key为callbackId，value为回调函数组成的数组
```
    if (successCallback || failCallback) {
        callbackId = service + cordova.callbackId++;  //cordovacallbackId=Math.floor(Math.random() * 2000000000)
        cordova.callbacks[callbackId] =
            {success:successCallback, fail:failCallback};
    }
```
3. 处理传递的参数actionArgs,最终actionArgs是一个数组。
```
function massageArgsJsToNative(args) {
    if (!args || utils.typeName(args) != 'Array') {
        return args;
    }
    var ret = [];
    args.forEach(function(arg, i) {
        if (utils.typeName(arg) == 'ArrayBuffer') {
            ret.push({
                'CDVType': 'ArrayBuffer',
                'data': base64.fromArrayBuffer(arg)
            });
        } else {
            ret.push(arg);
        }
    });
    return ret;
}

actionArgs = actionArgs || []
actionArgs = massageArgsJsToNative(actionArgs)
```
4. 生成command并入队列
```
var command = [callbackId, service, action, actionArgs];
commandQueue.push(JSON.stringify(command));
```
5. 根据不同的桥接模式bridgeMode，发送消息
```
if (bridgeMode === jsToNativeModes.WK_WEBVIEW_BINDING) {
        window.webkit.messageHandlers.cordova.postMessage(command);
    } else {
        // If we're in the context of a stringByEvaluatingJavaScriptFromString call,
        // then the queue will be flushed when it returns; no need for a poke.
        // Also, if there is already a command in the queue, then we've already
        // poked the native side, so there is no reason to do so again.
        if (!isInContextOfEvalJs && commandQueue.length == 1) {
            switch (bridgeMode) {
            case jsToNativeModes.XHR_NO_PAYLOAD:
            case jsToNativeModes.XHR_WITH_PAYLOAD:
            case jsToNativeModes.XHR_OPTIONAL_PAYLOAD:
                pokeNativeViaXhr();    // 新建一个XMLHttpRequest，并发送一个HEAD请求，并将commondQueue以json串的形式放在请求头cmds上。
                break;
            default: // iframe-based.
                pokeNativeViaIframe(); // 创建iframe，通过hash值来传递commondQueue 或 execIframe.src = "gap://ready"
            }
        }
    }
```
6. 猜测ios端接收到commondQueue后，执行插件的实际
