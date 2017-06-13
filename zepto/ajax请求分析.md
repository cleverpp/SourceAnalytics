## $.ajax(options)
关键步骤
1. options.global=true（默认），则发送自定义事件：ajaxStart
2. 判断是否跨域，设置options.crossDomain
3. 序列化options.data，如果options.processData=true（默认），则使用$.param将data序列化成形如：xx=xx&xx=xx；
4. 判断options.dataType是否为jsonp，正则表达式判断为：```hasPlaceholder = /\?.+=\?/.test(settings.url)```
5. 判断options.cache，如果未false。则url后加上时间戳，形如：http://www.xxx.com?_=201706131128890
6. 如果是jsonp类型，则调用$.ajaxJSONP(settings, deferred)
7. 设置请求头：X-Requested-With、Accept、MIME、Content-Type
8. 核心代码：使用

知识点：
1. [XMLHttpRequest MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest)
2. [XMLHttpRequest W3School](http://www.w3school.com.cn/xml/xml_http.asp)
3. [你真的会使用XMLHttpRequest吗？](https://segmentfault.com/a/1190000004322487)
