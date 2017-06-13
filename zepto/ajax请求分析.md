## $.ajax(options)
关键步骤
1. options.global=true（默认），则发送自定义事件：ajaxStart
2. 判断是否跨域，设置options.crossDomain
3. 序列化options.data，如果options.processData=true（默认），则使用$.param将data序列化成形如：xx=xx&xx=xx；
4. 判断options.dataType是否为jsonp，正则表达式判断为：```hasPlaceholder = /\?.+=\?/.test(settings.url)```
5. 判断options.cache，如果未false。则url后加上时间戳，形如：http://www.xxx.com?_=201706131128890
6. 如果是jsonp类型，则调用$.ajaxJSONP(settings, deferred)
7. 设置请求头：X-Requested-With、Accept、MIME、Content-Type等
8. 监听xhr.onreadystatechange，xhr.readyState=4表示请求完成。根据xhr.status判断响应成功或失败，成功则触发自定义事件：ajaxSuccess，失败则触发自定义事件：ajaxError
9. xhr.open
10. 复制options
11. xhr.setRequestHeader
12. 处理options.timeout>0的情况，设置setTimeout回调进行xhr.abort()
13. xhr.send(settings.data ? settings.data : null)

知识点：
1. [XMLHttpRequest MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest)
2. [XMLHttpRequest W3School](http://www.w3school.com.cn/xml/xml_http.asp)
3. [你真的会使用XMLHttpRequest吗？](https://segmentfault.com/a/1190000004322487)

## $.ajaxJSONP(options,deferred)
注意：参数defferred是$.Deffered,属于deffered模块范围内的。
1. 通过设置options.jsonpCallback来设置jsonp回调函数，如果没有设置，则回调函数名为：Zeptoxxxxxx
2. 核心处理即为创建script元素，并设置script.src = options.url.replace(/\?(.+)=\?/, '?$1=' + callbackName)
3. 监听成功、失败事件：$(script).on('load error', function(e, errorType){......}

## $.ajax的三种简写方式
```
  $.get = function(/* url, data, success, dataType */){
    return $.ajax(parseArguments.apply(null, arguments))
  }

  $.post = function(/* url, data, success, dataType */){
    var options = parseArguments.apply(null, arguments)
    options.type = 'POST'
    return $.ajax(options)
  }

  $.getJSON = function(/* url, data, success */){
    var options = parseArguments.apply(null, arguments)
    options.dataType = 'json'
    return $.ajax(options)
  }
```
