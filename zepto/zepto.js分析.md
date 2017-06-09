## domready
$(function(){}) : $()函数调用，参数为函数funcion(){}
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

注意点：
1. DOMContentLoaded事件在许多Webkit浏览器以及IE9上都可以使用, 此事件会在DOM文档准备好以后触发, 包含在HTML5标准中. 对于支持此事件的浏览器, 直接使用DOMContentLoaded事件是最简单最好的选择.IE6,7,8都不支持DOMContentLoaded事件
2. addEventListener也不兼容IE9以下。
