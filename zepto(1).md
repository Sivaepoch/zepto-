## 一 整体结构
* 为了防止全局变量污染,zepto使用的是立即执行函数,写法结构为:
```javascript
(function(global,factory){
    if(typeof define === 'function' && define.amd)
        define(function(){return factory(global)})
    else
        factory(global)
}(this,function(window){
	return Zepto
}))
```
* 其中框架的实现方式都是在立即执行函数的第二个函数内部实现的.
定义了一个核心函数`Zepto`,也是个立即执行函数.
```javascript
var Zepto =(function(){})()
//将Zepto赋值给window.Zepto;这样其他地方就可以调用zepto的方法了
window.Zepto = Zepto
//当$变量未被占用时,将Zepto赋值给$
window.$ === undefined && (window.$ = Zepto)
```
* Zepto函数是整个框架源码的核心部分.这里只对这一部分进行分析.下面来看看这里究竟是怎么样的一个过程.

    我们在使用zepto的时候.经常会用到$();我们顺着这个过程去找代码
    *  首先$()中的$是什么呢,从下面代码可以明白$是Zepto函数的返回值,这里要注意Zepto是大写的;Zepto里定义了一个变量zepto,两者是不同的;
    ```javascript
        var Zepto =(function(){
              ...
              zepto.Z.prototype = Z.prototype = $.fn
              $.zepto = zepto
              return $
        })()
	```

    *  知道了$;我们再去找$();此函数的返回值为zepto.init
    
    ```javascript
    	$ = function(selector, context){
            return zepto.init(selector, context)
       }
    ```
    
    *  同样,我们接着去找zepto.init()函数,返回值为zepto.Z()
    
    ```javascript
        zepto.init = function(selector, context) {
              ...
              return zepto.Z(dom, selector)
        }
    ```
    
    *  再接着找zepto.Z();返回值为Z函数的实例化
    
    ```javascript
        zepto.Z = function(dom, selector) {
            return new Z(dom, selector)
        }
    ```
    
    *  最后找Z函数;
    
    ```javascript
        function Z(dom, selector) {
          var i, len = dom ? dom.length : 0
          for (i = 0; i < len; i++) this[i] = dom[i]
          this.length = len
          this.selector = selector || ''
        }
    ```

## 二 分析
大致的流程如上,下面我根据以上的过程去分析代码.

以下为zepto源码节选:

```javascript
$ = function(selector, context){
    return zepto.init(selector, context)
}
```

```javascript
zepto.init = function(selector, context) {
    var dom
    if (!selector) return zepto.Z()
    else if (typeof selector == 'string') {
      selector = selector.trim()
      if (selector[0] == '<' && fragmentRE.test(selector))
        dom = zepto.fragment(selector, RegExp.$1, context), selector = null
      else if (context !== undefined) return $(context).find(selector)
      else dom = zepto.qsa(document, selector)
    }
    else if (isFunction(selector)) return $(document).ready(selector)
    else if (zepto.isZ(selector)) return selector
    else {
      if (isArray(selector)) dom = compact(selector)
      else if (isObject(selector))
        dom = [selector], selector = null
      else if (fragmentRE.test(selector))
        dom = zepto.fragment(selector.trim(), RegExp.$1, context), selector = null
      else if (context !== undefined) return $(context).find(selector)
      else dom = zepto.qsa(document, selector)
    }
    return zepto.Z(dom, selector)
}
```
根据以上代码,可以将zepto.init()函数分为以下几种情况:

![qqqq.png](http://upload-images.jianshu.io/upload_images/1909602-ca69c61182980e94.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

