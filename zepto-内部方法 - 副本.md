> 根据第一篇整体框架,可以知道$()方法返回的Z函数的实例化;而Z的原型又指向$.fn;所以经过$()处理过的对象,都可以使用$.fn中的方法;
这一篇就记录一下读$.fn的笔记;

```javascript
zepto.Z.prototype = Z.prototype = $.fn
```

##   一 内部函数

##  1. zepto.match
```javascript
  //判断一个元素是否匹配给定的选择器
zepto.matches = function(element, selector) {
    //如果selector,element没值或者element是普通节点
    if (!selector || !element || element.nodeType !== 1) return false
    var matchesSelector = element.matches || element.webkitMatchesSelector ||
                          element.mozMatchesSelector || element.oMatchesSelector ||
                          element.matchesSelector
    if (matchesSelector) return matchesSelector.call(element, selector)
    // fall back to performing a selector:
    var match, parent = element.parentNode, temp = !parent
    if (temp) (parent = tempParent).appendChild(element)
    match = ~zepto.qsa(parent, selector).indexOf(element)
    temp && tempParent.removeChild(element)
    return match
 }
```
* 如果浏览器支持`matchesSelector`方法,则用此方法进行判断

	![QQ截图20170610142612.png](http://upload-images.jianshu.io/upload_images/1909602-a202c2b7ef7a87a7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
	![QQ截图20170610171446.png](http://upload-images.jianshu.io/upload_images/1909602-122d0d1192c45887.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 如果浏览器不支持`matchesSelector`,则定义`match`和`parent`变量,其中`parent`为`element`的父元素,如果没有父元素,则创建一个`div`容器当作其父元素

	`tempParent = document.createElement('div'),`
    
    ` match = ~zepto.qsa(parent, selector).indexOf(element)`
    
    `qsa()`函数之前已经说明,若`element`不存在`zepto.qsa(parent, selector).`中,则返回-1,~-1即为0,即`match`最终的值为`false`
    
    若`element`存在,则返回非零的值,转换为布尔值为`true`
    

##   二 $.fn
##   **constructor**
`constructor: zepto.Z,`
* 这就涉及到原型和构造函数的知识.之后会当作一个专题来记录;这里只做简单介绍;
```javascript
var person = function(name){
this.name = name;
}
var siva = new Person('Siva');
siva .__proto__ == person.prototype //true
person.prototype.constructor == person //true
```
由于`zepto.Z.prototype = $.fn` ;因此$.fn的构造函数即为`zepto.Z`

##    forEach
`forEach: emptyArray.forEach,`
* 之前已经说过,emptyArray为定义的一个空数组,所以这里的forEach即为Array.prototype.forEach,可以理解为数组自带的forEach方法;

##    reduce,push,sort,splice,indexOf
```javascript
reduce: emptyArray.reduce,
push: emptyArray.push,
sort: emptyArray.sort,
splice: emptyArray.splice,
indexOf: emptyArray.indexOf,
```
* 道理同上

##  get
```javascript
get: function(idx){
//如果不传参,将Z对象转为数组, slice.call()在第三篇已经提到过;数组自有的slice;
//如果传参小于0,就将传的参数转换为idx加上传入对象的长度;即为倒数返回;
//例如传入的值为-1;则选取最后一个元素
  return idx === undefined ? slice.call(this) : this[idx >= 0 ? idx : idx + this.length]
},
```
在分析这句话之前,我们先看看get的用法;

![QQ截图20170610142612.png](http://upload-images.jianshu.io/upload_images/1909602-940f5c812c7711f6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![QQ截图20170610142642.png](http://upload-images.jianshu.io/upload_images/1909602-b525241ef348e3a6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



##  toArray
```javascript
//将Z对象集合转换成数组
toArray: function(){ return this.get() },
```
* 不传参调用`get()`

##   concat
```javascript
concat: function(){
  var i, value, args = []
  for (i = 0; i < arguments.length; i++) {
    value = arguments[i]
    //如果其中传入的参数为Z对象,则将其转为数组;并将值传给arg
    args[i] = zepto.isZ(value) ? value.toArray() : value
  }
  //apply方法将this强制绑定到调用concat的对象身上
  //如果调用concat的是Z对象,则将其转换成数组
  return concat.apply(zepto.isZ(this) ? this.toArray() : this, args)
},
```
* 添加元素到一个`Zepto`对象集合形成一个新数组。如果参数是一个数组，那么这个数组中的元素将会合并到Zepto对象集合中。
* `arguments`为传入的参数组合;

* 这个方法并没有像上面的reduce,push,sort,splice,等直接复制Array.prototype的属性,是因为调用此事件的对象并不一定为数组,如果调用的对象为`$();`则是类数组对象,而不是真正的数组,如果直接调用数组的`concat`,则会将`$();`当作数组的一个item合并起来,所以此方法需要重新封装

##  slice
```javascript
slice: function(){
//同上,将this强制绑定到调用的对象身上;
  return $(slice.apply(this, arguments))
},
```
##  ready
```javascript
ready: function(callback){
//判断dom是否ready,如果已经ready,直接执行函数
  if (readyRE.test(document.readyState) && document.body) callback($)
  //否则注册监听器;
  else document.addEventListener('DOMContentLoaded', function(){ callback($) }, false)
  return this
},
```
* ` readyRE = /complete|loaded|interactive/,`

* document.readyState返回当前文档的状态,该属性返回四个值;分别为:
	*  uninitialized - 还未开始载入
	*  loading - 载入中
	*  interactive - 已加载，文档与用户可以开始交互 ---仅DOM加载完成，不包括样式表，图片，flash,触发DOMContentLoaded事件
	*  complete - 载入完成 ---页面上所有的DOM，样式表，脚本，图片，flash都已经加载完成了,开始触发load事件

##   size
```javascript
size: function() {
  return this.length
},
```
* 返回所选集合的length属性的值;

##   each
```javascript
each: function(callback){
  emptyArray.every.call(this, function(el, idx){
    return callback.call(el, idx, el) !== false
  })
  //可以使用链式操作
  return this
},
```
* 方法内部调用了`every`,当`callback`的值返回`false`的时候就会中止循环;

##   eq
```javascript
eq: function(idx){
  return idx === -1 ? this.slice(idx) : this.slice(idx, + idx + 1)
},
```
* 当idx为-1的时候.取最后一个;

##   first
```javascript
first: function(){
  var el = this[0]
  return el && !isObject(el) ? el : $(el)
},
```
* this[0] 取集合中的第一个值;
* `return el && !isObject(el) ? el : $(el)`  如果集合中第一个数据为对象的话返回本身,否则转换成zepto对象

##  last
```javascript
last: function(){
  var el = this[this.length - 1]
  return el && !isObject(el) ? el : $(el)
},
```
* 同上

##   add
```javascript
add: function(selector,context){
  return $(uniq(this.concat($(selector,context))))
},
```
```javascript
uniq = function(array){ 
	return filter.call(array, 
    function(item, idx){ return array.indexOf(item) == idx })
}
```
* 添加元素到当前匹配的元素集合中
* `uniq`中的`filter`在之前已经定义；`filter = emptyArray.filter`,即为`Array.prototype.filter`

	* `Array.prototype.filter`的用法
		```javascript
        var words = ["spray", "limit", "elite", "exuberant", "destruction", "present"];
        var longWords = words.filter(function(word){
          return word.length > 6;
        })
        // Filtered array longWords is ["exuberant", "destruction", "present"]
        ```
* `uniq`的作用是给数组去重，如果数组中的数据在数组中的位置不等于索引值，说明这个数据在数组中出现过两次以上，我们再利用`filter`函数，只取出位置和索引值相同的那个；

## map
```javascript
map: function(fn){
  return $($.map(this, function(el, i){ return fn.call(el, i, el) }))
},
```

##   remove
```javascript
remove: function(){
  return this.each(function(){
    if (this.parentNode != null)
      this.parentNode.removeChild(this)
  })
},
```
* 从其父节点中删除当前集合中的元素，有效的从dom中移除。

##   is
```javascript
is: function(selector){
  return this.length > 0 && zepto.matches(this[0], selector)
},
```
* 判断当前元素集合中的第一个元素是否符css选择器。
* `match`在开头已经写过了；

##  not
```javascript
not: function(selector){
  var nodes=[]
  //当selector为函数时，但是safari下的typeof odeList也是function，
  //所以这里需要再加一个判断selector.call !== undefined
  if (isFunction(selector) && selector.call !== undefined)
    this.each(function(idx){
    //当selector.call(this,idx)返回结果为false，记录；
      if (!selector.call(this,idx)) nodes.push(this)
    })
  else {
  //当selector为字符串的时候，筛选满足selector的记录；
    var excludes = typeof selector == 'string' ? this.filter(selector) :
    //当selector为类数组，将其变为数组，否则，执行$()
      (likeArray(selector) && isFunction(selector.item)) ? slice.call(selector) : $(selector)
    this.forEach(function(el){
    //筛选出不在excludes中的数据
      if (excludes.indexOf(el) < 0) nodes.push(el)
    })
  }
  //以上得到的是数组，将其转换成zepto对象；
  return $(nodes)
},
```