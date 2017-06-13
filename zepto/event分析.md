## 创建事件$.Event(type,props)
使用document.createEvent创建事件，并增加了以下三个方法：
1. event.isDefaultPrevented ：如果preventDefault()被该事件的实例调用，那么返回true。 这可作为跨平台的替代原生的 defaultPrevented属性，如果 defaultPrevented缺失或在某些浏览器下不可靠的时候。
2. event.isImmediatePropagationStopped ：如果stopImmediatePropagation()被该事件的实例调用，那么返回true。Zepto在不支持该原生方法的浏览器中实现它，  （例如老版本的Android）。
3. event.isPropagationStopped ：如果stopPropagation()被该事件的实例调用，那么返回true。

知识点
1. 创建事件：document.createEvent
2. [Event - Web API 接口| MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/Event)

## 事件监听on
常用方式如下：

1. on(type, [selector], function(e){ ... })   ⇒ self

2. on(type, [selector], [data], function(e){ ... })   ⇒ self v1.1+

3. on({ type: handler, type2: handler2, ... }, [selector])   ⇒ self

4. on({ type: handler, type2: handler2, ... }, [selector], [data])   ⇒ self v1.1+

```
  $.fn.on = function(event, selector, data, callback, one){
    var autoRemove, delegator, $this = this
    if (event && !isString(event)) {  // 处理方式3和方式4
      $.each(event, function(type, fn){
        $this.on(type, selector, data, fn, one)
      })
      return $this
    }

    if (!isString(selector) && !isFunction(callback) && callback !== false)
      callback = data, data = selector, selector = undefined
    if (callback === undefined || data === false)
      callback = data, data = undefined

    if (callback === false) callback = returnFalse

    return $this.each(function(_, element){
      if (one) autoRemove = function(e){
        remove(element, e.type, callback)
        return callback.apply(this, arguments)
      }

      if (selector) delegator = function(e){
        var evt, match = $(e.target).closest(selector, element).get(0)
        if (match && match !== element) {
          evt = $.extend(createProxy(e), {currentTarget: match, liveFired: element})
          return (autoRemove || callback).apply(match, [evt].concat(slice.call(arguments, 1)))
        }
      }

      add(element, event, callback, data, selector, delegator || autoRemove)
    })
  }
```
其中add(element, event, callback, data, selector, delegator || autoRemove)本质是通过element.addEventListener来实现的。

## 事件监听的其它形式
1. bind = on
```
$.fn.bind = function(event, data, callback){
    return this.on(event, data, callback)
  }
```
2. delegate = on
```
$.fn.delegate = function(selector, event, callback){
    return this.on(event, selector, callback)
  }
```
3. live = delegate, 其中selector指定为document.body。
```
$.fn.live = function(event, callback){
    $(document.body).delegate(this.selector, event, callback)
    return this
  }
```
4. one = on
```
$.fn.one = function(event, selector, data, callback){
    return this.on(event, selector, data, callback, 1)
  }
```
## 事件移除off
对应事件监听on。
```
  $.fn.off = function(event, selector, callback){
    var $this = this
    if (event && !isString(event)) {
      $.each(event, function(type, fn){
        $this.off(type, selector, fn)
      })
      return $this
    }

    if (!isString(selector) && !isFunction(callback) && callback !== false)
      callback = selector, selector = undefined

    if (callback === false) callback = returnFalse

    return $this.each(function(){
      remove(this, event, callback, selector)
    })
  }
```
其中remove(this, event, callback, selector)，本质是通过element.removeEventListener移除事件。
unbind（对应bind）、undelegate（对应delegate），die（对应live），都是通过调用off来实现的。
## 事件的触发trigger
```
  ......
  focus = { focus: 'focusin', blur: 'focusout' },
  ......
  $.fn.trigger = function(event, args){
    event = (isString(event) || $.isPlainObject(event)) ? $.Event(event) : compatible(event)
    event._args = args
    return this.each(function(){
      // handle focus(), blur() by calling them directly
      if (event.type in focus && typeof this[event.type] == "function") this[event.type]()
      // items in the collection might not be DOM elements
      else if ('dispatchEvent' in this) this.dispatchEvent(event)
      else $(this).triggerHandler(event, args)
    })
  }
```
对于focus、blur事件，直接调用,如果当前选择元素有dispatchEvent方法，则调用改方法。否则执行triggerHandler，改方法阻止冒泡。
