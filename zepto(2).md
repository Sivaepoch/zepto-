
#### 1.  $.type $.isArray $.isFunction $.isNumeric $.isPlainObject $.isWindow
*  **判断对象的方法介绍**
在zepto源码中，使用了**Object.prototype.toString.call()**方法判断对象的类型，以下简单介绍下此方法的大致情况
```javascript
//null
Object.prototype.toString.call(null);//”[object Null]”
//undefined
Object.prototype.toString.call(undefined);//”[object Undefined]”
//string
Object.prototype.toString.call(“aaa”);//”[object String]”
//number
Object.prototype.toString.call(111);//”[object Number]”
//boolean
Object.prototype.toString.call(true);//”[object Boolean]”
//函数
Function fn(){console.log(“xixi”);}
Object.prototype.toString.call(fn);//”[object Function]”
//数组类型
var arr = [1,2，,3,4];
Object.prototype.toString.call(arr);//”[object Array]”
//日期
var date = new Date();
Object.prototype.toString.call(date);//”[object Date]”
//自定义类型
//不能判断aa是不是AA的实例，要用instanceof判断
function AA(a, b) {
    this.a = a;
    this.b = b;
}
var aa = new AA("xixi", 'haha');
Object.prototype.toString.call(aa); //”[object Object]”
Object.prototype.toString.call(aa); //”[object Object]”
//正则
var aa = /^\w$/
Object.prototype.toString.call(aa); //”[object RegExp]”
```
* 在zepto中，先定义了一个空对象`class2type`，并取出对象函数自带的`toString`方法，即为`Object.prototype.toString.call()`
```javascript
class2type = {},
toString = class2type.toString,
```
然后填充`class2type`的值；
```javascript
$.each("Boolean Number String Function Array Date RegExp Object Error".split(" "), function(i, name) {
    class2type["[object " + name + "]"] = name.toLowerCase()
    //最终结果
    //class2type[”[object Boolean]”] = boolean,
    //class2type[”[object Number]”] = number,
    //class2type[”[object String]”] = string,
    // ...
  })
```
* 准备工作做完，就可以进行对象的判断了；

    * 首先封装一个判断方法，根据这个方法的返回值来判断对象的类型；
    ```java
    //比如，如果要判断function aa(){};则返回class2type[Object.prototype.toString.call(aa)]==class2type[”[object Function]”]==function；所以就判断出为函数
    function type(obj) {
     //如果obj为null或者undefined,返回字符串'null','undefined';否则返回class2type[]或者object
          return obj == null ? String(obj) :class2type[toString.call(obj)] || "object"
    }
    ```
    ```java
    $.type = type
    ```
    * 下面根据上面的封装方法返回值判断类型；
  
    	* 判断函数
        ```javascript
        function isFunction(value) {
            return type(value) == "function"
          }
        ```
        ```javascript
        $.isFunction = isFunction
        ```
        * 判断window;根据widow自己的属性来判断；window.window = window;
        ```javascript
        function isWindow(obj) {
            return obj != null && obj == obj.window
          }
        ```
        ```javascript
        $.isWindow = isWindow
        ```
        * 判断document
        ```javascript
        function isDocument(obj) {
            return obj != null && obj.nodeType == obj.DOCUMENT_NODE
          }
        ```
        * 判断是否为对象
        ```javascript
        function isObject(obj) {
            return type(obj) == "object"
          }
        ```
        * 对于通过字面量定义的对象和new Object的对象返回true，new Object时传参数的返回false
        ```javascript
        function isPlainObject(obj) {
            return isObject(obj) && !isWindow(obj) && obj.__proto__ == Object.prototype
          }
        ```
        ```javascript
        $.isPlainObject = isPlainObject
        ```
        * 判断数组
        ```javascript
        function isArray(value) {
            return value instanceof Array
          }
        ```
        ```javascript
        $.isArray = isArray
        ```
        * 判断类数组
        ```javascript
        function likeArray(obj) {
            return typeof obj.length == 'number'
          }
        ```
        * 空对象
        ```javascript
        $.isEmptyObject = function(obj) {
            var name
            for (name in obj) return false
            return true
          }
        ```
        * 有限数值或者字符串表达的数字
        ```javascript
        $.isNumeric = function(val) {
            var num = Number(val), type = typeof val
            return val != null && type != 'boolean' &&
              (type != 'string' || val.length) &&
              !isNaN(num) && isFinite(num) || false
          }
        ```

#### 2. $.camelCase 
```javascript
camelize = function(str){ 
	return str.replace(/-+(.)?/g,
    	function(match, chr){ return chr ? chr.toUpperCase() : '' }
    ) 
}
```
```javascript
$.camelCase = camelize
```
* 用法
	```javascript
    $.camelCase('hello-there') //=> "helloThere"
	$.camelCase('helloThere')  //=> "helloThere"
    ```
* `str.replcace(a,b)`

	将str中的a替换成b;上面代码中将b用了函数返回值来表达;

#### 3. $.contain
```javascript
//为了判断某个节点是不是另一个节点的后代,浏览器引入了contains()方法;
$.contains = document.documentElement.contains ?
//如果浏览器支持contains()方法,就执行这个函数
    function(parent, node) {
      return parent !== node && parent.contains(node)
    } :
    //否则循环判断
    function(parent, node) {
      while (node && (node = node.parentNode))
        if (node === parent) return true
      return false
}
```
* 检查父节点是否包含给定的dom节点，如果两者是相同的节点，则返回` false`。

#### 4. $.each
```javascript
$.each = function(elements, callback){
    var i, key
    if (likeArray(elements)) {
      for (i = 0; i < elements.length; i++)
        if (callback.call(elements[i], i, elements[i]) === false) return elements
    } else {
      for (key in elements)
        if (callback.call(elements[key], key, elements[key]) === false) return elements
    }

    return elements
}
```
* 遍历,将每次循环的值作为callback的上下文;如果callback返回的结果为false,遍历停止;

#### 5. $.extend
```javascript
function extend(target, source, deep) {
    for (key in source)
    //如果是神拷贝,source[key]是对象或者数组
      if (deep && (isPlainObject(source[key]) || isArray(source[key]))) {
      //source[key]是对象,而target[key]不是对象
        if (isPlainObject(source[key]) && !isPlainObject(target[key]))
          target[key] = {}
      //source[key]是数组,而target[key]不是数组
        if (isArray(source[key]) && !isArray(target[key]))
          target[key] = []
          //递归
        extend(target[key], source[key], deep)
      }
      else if (source[key] !== undefined) target[key] = source[key]
  }
```

```javascript
$.extend = function(target){
    var deep, args = slice.call(arguments, 1)
    //如果传入的第一个参数为布尔值
    if (typeof target == 'boolean') {
      deep = target
     //将除第一个参数外的参数赋值给target
      target = args.shift()
    }
    //遍历参数,将参数都复制到target上;
    args.forEach(function(arg){ extend(target, arg, deep) })
    return target
  }
```
* 我们首先看一下用法;
```javascript
$.extend(target, [source, [source2, ...]])   ⇒ target
$.extend(true, target, [source, ...])   ⇒ target v1.0+
```
```javascript
var target = { one: 'patridge' },
source = { two: 'turtle doves' }
$.extend(target, source)//{one: "patridge", two: "turtle doves"}
```
* `target`代表被拓展的对象,`source`为源对象;`deep`代表是深复制还是浅复制;

	*  对于字符串来说,浅复制是对值的复制,,对于对象来说,浅复制是对对象地址的复制,两者共同指向同一个地址,其中一个改变,另一个对象也会随之改变,而深复制是在堆区中开辟一个新的,两个对象则指向不同的地址,相互独立;
	
* `slice.call(arguments, 1)`选取传入参数的第一个参数到最后一个参数;

#### 5. $.inArray
```javascript
$.inArray = function(elem, array, i){
    return emptyArray.indexOf.call(array, elem, i)
}
```
* 用法
```javascript
$.inArray("abc",["bcd","abc","edf","aaa"]);//=>1
$.inArray("abc",["bcd","abc","edf","aaa"],1);//=>1
$.inArray("abc",["bcd","abc","edf","aaa"],2);//=>-1
```
* 返回数组中指定元素的索引值，如果没有找到该元素则返回-1。


#### 6. $.map
```javascript
$.map = function(elements, callback){
    var value, values = [], i, key
    if (likeArray(elements))
      for (i = 0; i < elements.length; i++) {
        value = callback(elements[i], i)
        if (value != null) values.push(value)
      }
    else
      for (key in elements) {
        value = callback(elements[key], key)
        if (value != null) values.push(value)
      }
    return flatten(values)
 }
```
* `element`为类数组对象或者对象;

	*  如果为类数组对象的话,用for循环
	*  如果为对象的话,用fo.....in....循环
	*  再将索引和值传给callback,
	*  如果callback的返回值不是null或者undefined,存入新的数组
	*  最后将数组扁平化flatten()---数组降维,二维数组转为一维数组

* `flatten()`

	`function flatten(array) { return array.length > 0 ? $.fn.concat.apply([], array) : array }`
    
    这里巧妙应用了`apply`方法,`apply`方法的第一个元素会被当作`this`,第二个元素被做为传入的参数`arguments`;
    例如:`concat.apply([],[[1],[2],[3]]),`等价于`[].concat([1],[2],[3])`,输出的值为`[1,2,3]`,就实现了数组的扁平化;

#### 7. $.noop
```javascript
$.noop = function() {}
```
* 引用空函数

#### 8. $.parseJSON
```javascript
if (window.JSON) $.parseJSON = JSON.parse
```
* 原生JSON.parse的别名；接受一个JSON字符串，返回解析后的javascript对象。

#### 9. $.trim
```javascript
$.trim = function(str) {
	return str == null ? "" : String.prototype.trim.call(str)
}
```
* 去除字符串开头和结尾的空格