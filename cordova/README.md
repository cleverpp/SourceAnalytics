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
## js如何调用ios native
1. jsToNativeModes有以下7种，一般用IFRAME_NAV
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
2. cordova维护一个callback对象，key为callbackId，value为回调函数组成的对象
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
6. ios端通过UIWebViewDelegate（iframe方式）或 NSURLProtocol拦截（xhr方式）方式接收到commondQueue后，执行插件的实际功能，然后怎么样向js发送消息呢？
    1. ios原生支持native调用js，即通过UIWebView的stringByEvaluatingJavaScriptFromString方法
    2. ios端执行完插件逻辑后，会调用iOSExec.nativeCallback
    ```
    iOSExec.nativeCallback = function(callbackId, status, message, keepCallback) {
      return iOSExec.nativeEvalAndFetch(function() {
        var success = status === 0 || status === 1;
        var args = convertMessageToArgsNativeToJs(message);  // 将CDVType类型的native数据转换为
        cordova.callbackFromNative(callbackId, success, status, args, keepCallback);
      });
    };
    
    iOSExec.nativeEvalAndFetch = function(func) {
      // This shouldn't be nested, but better to be safe.
      isInContextOfEvalJs++;
      try {
        func();
        return iOSExec.nativeFetchMessages();
      } finally {
        isInContextOfEvalJs--;
      }
    };
    ```
    3. 最终调用的是cordova.callbackFromNative(callbackId, success, status, args, keepCallback)
    ```
      /**
     * Called by native code when returning the result from an action.
     */
    callbackFromNative: function(callbackId, success, status, args, keepCallback) {
        var callback = cordova.callbacks[callbackId];
        if (callback) {
            if (success && status == cordova.callbackStatus.OK) {  // cordova.callbackStatus.OK = 1;
                callback.success && callback.success.apply(null, args);
            } else if (!success) {
                /**
                 * 遇到如下错误(status)以后，将status作为参数返回失败回调 fail(err), err.cordovaRetStatus 就是失败状态原因（status）
                 *
                 * CLASS_NOT_FOUND_EXCEPTION: 2,
                   ILLEGAL_ACCESS_EXCEPTION: 3,
                   INSTANTIATION_EXCEPTION: 4,
                   MALFORMED_URL_EXCEPTION: 5,
                   ...
                 *
                 */
                this.errorRetStatus = status;

                callback.fail && callback.fail.apply(null, args);
            }

            // Clear callback if not expecting any more results
            if (!keepCallback) {
                delete cordova.callbacks[callbackId];
            }
        }
    },
    ```
## js如何调用android native
1. jsToNativeModes有以下2种,默认使用JS_OBJECT
```
  jsToNativeModes = {
        PROMPT: 0,   // cordova重载了OnJsPrompt方法
        JS_OBJECT: 1   // addjavascriptinterface（Android Webview的API），这样可以调用以@JavascriptInterface注解的java方法
  },
```
2. 同ios一样，维护callback对象，key为callbackId，value为回调函数组成的对象；
3. 处理actionArgs，首先必须是个数组，并将参数序列化argsJson = JSON.stringify(args)
4. js向android发送消息
    ```
    var messages = nativeApiProvider.get().exec(bridgeSecret, service, action, callbackId, argsJson); // 
    // If argsJson was received by Java as null, try again with the PROMPT bridge mode.
    // This happens in rare circumstances, such as when certain Unicode characters are passed over the bridge on a Galaxy S2.  See CB-2666.
    if (jsToNativeBridgeMode == jsToNativeModes.JS_OBJECT && messages === "@Null arguments.") {
      androidExec.setJsToNativeBridgeMode(jsToNativeModes.PROMPT);
      androidExec(success, fail, service, action, args);
      androidExec.setJsToNativeBridgeMode(jsToNativeModes.JS_OBJECT);
      return;
    } else {
      androidExec.processMessages(messages, true);
    }
    ```   
   
    1. 如果是JS_OBJECT方式，那么nativeApiProvider.get().exec= 安卓端源码中注解了@JavascriptInterface的 exec方法
    2. 如果是PROMPT方式，那么nativeApiProvider.get().exec 为如下方法：
      ```
        exec: function(bridgeSecret, service, action, callbackId, argsJson) {
          return prompt(argsJson, 'gap:'+JSON.stringify([bridgeSecret, service, action, callbackId]));
        },
      ```
5. native向js发送消息：安卓会直接调用：androidExec.processMessages(messages, true); 最终通过解析messages来获得回传参数，并执行成功回调、失败回调
## 参考
1. [浅析 Cordova for iOS](http://zhenby.com/blog/2013/05/16/cordova-for-ios/)
2. 
