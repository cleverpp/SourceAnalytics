## domready
使用zepto库，在处理业务逻辑的js中，入口通常是这样写的：$(function(){})，及调用函数$()，参数为函数funcion(){}。以下来看一下源码中对这一语句的处理逻辑：
1. $(function(){}) = $(document).ready(function(){})
2. $(document) = zepto.Z = new Z([document],null) = {0:document, lenght:1, selector:""}
3. zepto.Z.prototype = Z.prototype = $.fn
4. $(function(){}) = $.fn.ready(function(){})
```
readyRE = /complete|loaded|interactive/;
$.fn = {
......
ready: function(callback){
      // need to check if document.body exists for IE as that browser reports
      // document ready when it hasn't yet created the body element
      if (readyRE.test(document.readyState) && document.body) callback($)
      else document.addEventListener('DOMContentLoaded', function(){ callback($) }, false)
      return this
      },
......
}
```
知识点：
1. [document.readyState](https://developer.mozilla.org/zh-CN/docs/Web/API/Document/readyState)
2. [DOMContentLoaded事件](https://developer.mozilla.org/zh-CN/docs/Web/Events/DOMContentLoaded)
3. [事件DOMContentLoaded和load的区别](http://www.jianshu.com/p/d851db5f2f30)
4. [兼容所有浏览器的 DOM 载入事件](http://harttle.com/2016/05/14/binding-document-ready-event.html)

注意点：
1. DOMContentLoaded事件在许多Webkit浏览器以及IE9上都可以使用, 此事件会在DOM文档准备好以后触发, 包含在HTML5标准中. 对于支持此事件的浏览器, 直接使用DOMContentLoaded事件是最简单最好的选择.IE6,7,8都不支持DOMContentLoaded事件
2. addEventListener也不兼容IE9以下。
