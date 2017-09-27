## swiper源码分析 之 基础知识
### 被动事件监听器：Passive event listeners
```
// Test via a getter in the options object to see 
// if the passive property is accessed
var supportsPassive = false;
try {
  var opts = Object.defineProperty({}, 'passive', {
    get: function() {
      supportsPassive = true;
    }
  });
  window.addEventListener("test", null, opts);
} catch (e) {}

// Use our detect's results. 
// passive applied if supported, capture will be false either way.
elem.addEventListener(
  'touchstart',
  fn,
  supportsPassive ? { passive: true } : false
); 
```
