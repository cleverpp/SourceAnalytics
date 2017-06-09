## $()之DOM加载完成事件监听
处理业务逻辑的js代码通常是在dom加载完成后开始处理，通常是这样写的：$(function(){})，即调用函数$()，参数为函数funcion(){}。

来看一下源码中对这一语句的处理逻辑：
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
## $()之DOM选择
一个常用的场景是通过选择器selector选择目标元素集合，可以指定上下文context。$(selector, [context])   ⇒ collection
例如：
```
$('div')  //=> 所有页面中得div元素
$('#foo') //=> ID 为 "foo" 的元素
```
来看一下源码中对这类语句的处理
1. 如果指定了context，则$(selector,context)=$(context).find(selector)
      1. $(context)，context可以为css选择器，dom，或者Zepto集合对象，最终返回的也都是Zepto集合对象
      2. zepto.Z.find(selector) = $.fn.find(selector)
```
$.fn = {
......
find: function(selector){
      var result, $this = this
      if (!selector) result = $()
      else if (typeof selector == 'object')
        result = $(selector).filter(function(){
          var node = this
          return emptyArray.some.call($this, function(parent){
            return $.contains(parent, node)
          })
        })
      else if (this.length == 1) result = $(zepto.qsa(this[0], selector))
      else result = this.map(function(){ return zepto.qsa(this, selector) })
      return result
    },
......
}
```
      3. $.fn.filter
```
filter: function(selector){
      if (isFunction(selector)) return this.not(this.not(selector))
      return $(filter.call(this, function(element){
        return zepto.matches(element, selector)
      }))
    },
```
      4. Array.map  返回一个新的Zepto对象集合。
2. 如果未指定context，则$(selector) = zepto.Z(zepto.qsa(document, selector), selector)
      1. zepto.qsa(document, selector) , 返回一个HTML数组
```
  simpleSelectorRE = /^[\w-]*$/;
  zepto.qsa = function(element, selector){
    var found,
        maybeID = selector[0] == '#',
        maybeClass = !maybeID && selector[0] == '.',
        nameOnly = maybeID || maybeClass ? selector.slice(1) : selector, // Ensure that a 1 char tag name still gets checked
        isSimple = simpleSelectorRE.test(nameOnly)
    return (element.getElementById && isSimple && maybeID) ? // Safari DocumentFragment doesn't have getElementById
      ( (found = element.getElementById(nameOnly)) ? [found] : [] ) :
      (element.nodeType !== 1 && element.nodeType !== 9 && element.nodeType !== 11) ? [] :
      slice.call(
        isSimple && !maybeID && element.getElementsByClassName ? // DocumentFragment doesn't have getElementsByClassName/TagName
          maybeClass ? element.getElementsByClassName(nameOnly) : // If it's simple, it could be a class
          element.getElementsByTagName(selector) : // Or a tag
          element.querySelectorAll(selector) // Or it's not simple, and we need to query all
      )
  }
```
      2. zepto.Z([HTMLElement],selector) = new Z([HTMLElement],selector) = {0:HTMLElement,1:HTMLElement,....,selecotr:selector字符串}, 即最终返回的是一个Zepto集合对象。

知识点：
1. 选择器类型
      - 简单选择器simpleSelectorRE = /^[\w-]*$/， 包括类型选择器（元素选择器）、类选择器、ID选择器
      - 属性选择器：种特殊类型的选择器，它根据元素的 属性和属性值来匹配元素。它们的通用语法由方括号([]) 组成，其中包含属性名称，后跟可选条件以匹配属性的值
      - 伪类(:)、伪元素(::)选择器
      - 组合选择器、多选择器
2. [nodeType](https://developer.mozilla.org/zh-CN/docs/Web/API/Node/nodeType)
3. getElementById、getElementsByClassName、getElementsByTagName、querySelectorAll: https://www.zhihu.com/question/24702250 , http://www.jianshu.com/p/4159485b035d 
4. [Array.slice.call](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/slice) :slice 方法可以用来将一个类数组（Array-like）对象/集合转换成一个数组。你只需将该方法绑定到这个对象上

