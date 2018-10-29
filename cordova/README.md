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
  // success: 成功回调, fail:失败回调, pluginName插件名, actionName方法名, params：传递的参数
  onCordovaReady(function(){
    window.cordova.exec(success,fail,"pluginName","actionName",[params])
  })
```
## js to android native
