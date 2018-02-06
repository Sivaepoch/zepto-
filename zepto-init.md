## zepto.init()
#### 1.  首先是第一种情况,selector为空

既然是反向分析,那我们先看看这句话的代码;

`if (!selector) return zepto.Z()`
    
 这里的返回值为`zepto.Z();`那我们继续往上找zepto.Z()函数

```javascript
    zepto.Z = function(dom, selector) {
      return new Z(dom, selector)
    }
```
这个函数仍然拥有一个返回值,**Z函数的实例**,同样的道理,我们继续去找`Z()` ;   

```javascript
    function Z(dom, selector) {
      var i, len = dom ? dom.length : 0
      for (i = 0; i < len; i++) this[i] = dom[i]
      this.length = len
      this.selector = selector || ''
    }
```
根据以上代码可以分析出,当没有参数时,会得到一个`length:0,selector:''`的对象.
![QQ截图20170608110900.png](http://upload-images.jianshu.io/upload_images/1909602-a7394f13b43e4fb3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 2.  当selector为字符串的时候,又分为三种情况;
	同样的,我们先看看这句话的代码
    
```javascript
else if (typeof selector == 'string') {
    selector = selector.trim()
}
```
  *  第一种,当selector为html片段
    ```javascript
    if (selector[0] == '<' && fragmentRE.test(selector))
          dom = zepto.fragment(selector, RegExp.$1, context), selector = null
    ```
这里有两个知识点:  

(1) `fragmentRE.test(selector)`  

这里的fragmentRE是Zepto函数在之前定义的一段正则;  

```javascript
//<div>erfwef</div>  取出<div>
fragmentRE = /^\s*<(\w+|!)[^>]*>/,
```

(2) `zepto.fragment(selector, RegExp.$1, context)`  

*  `RegExp.$1`
     `RegExp.$1`为`RegExp`的一个属性,指的是与正则表达式匹配的第一个 子匹配(以括号为标志)字符串;
     例子:  

    ```javascript
        var r= /^(\d{4})-(\d{1,2})-(\d{1,2})$/;
        r.exec('1985-10-15');
        s1=RegExp.$1;
        s2=RegExp.$2;
        s3=RegExp.$3;
        alert(s1+" "+s2+" "+s3)//结果为1985 10 15
    ```
* zepto.fragment()函数

    ```javascript
    //对应上面的代码,这里第一个参数是selector,就是我们在写代码时的$('xxx')中的xxx,
    //name为RegExp.$1,即正则匹配的第一个()里的东西,就是标签元素,例如 div p  h1等
    zepto.fragment = function(html, name, properties) {
          var dom, nodes, container
          // singleTagRE仍为之前定义的变量
          //singleTagRE = /^<(\w+)\s*\/?>(?:<\/\1>|)$/, 匹配值如下截图
          //如html传入值为<p></p>,匹配singleTagRE,则创建<p></p>,并调用$('<p></p>')
          if (singleTagRE.test(html)) dom = $(document.createElement(RegExp.$1))
          //如果不匹配
          if (!dom) {
            //这是一段修复代码,将<div/>之类的不正常的代码修复成<div></div>;
            //具体的下面再讲解
            if (html.replace) html = html.replace(tagExpanderRE, "<$1></$2>")
            //如果没有标签名,,给他一个标签,fragmentRE = /^\s*<(\w+|!)[^>]*>/,
            if (name === undefined) name = fragmentRE.test(html) && RegExp.$1
            //containers = {tr': document.createElement('tbody'),tbody': table, 'thead': table, 'tfoot': table,td': tableRow, 'th': tableRow,'*': document.createElement('div')},
            //如果name值不在container范围内,则标签名为div
            if (!(name in containers)) name = '*'
            //创建容器
            container = containers[name]
            //把html片段放入到容器中
            container.innerHTML = '' + html
            //这里调用了$.each();一会再详细讲解,这里是涉及到哪个函数我就去解析哪个函数
            //emptyArray = [], slice = emptyArray.slice,
            //所以这里的slice.call即为Array.prototype.slice.call(),能将具有length属性的对象转成数组;
            dom = $.each(slice.call(container.childNodes), function(){
            //删除
              container.removeChild(this)
            })
          }
          //如果properties为对象
          if (isPlainObject(properties)) {
          //$(dom)将dom转化成zepto对象；是为了方便调用其他方法；
            nodes = $(dom)
            $.each(properties, function(key, value) {
            //methodAttributes = ['val', 'css', 'html', 'text', 'data', 'width', 'height', 'offset'],
            //如果设置的key在methodAttributes内，则直接调用zepto上的方法；
              if (methodAttributes.indexOf(key) > -1) nodes[key](value)
              else nodes.attr(key, value)
            })
          }
          //<p><span></span></p>返回[p,span]
          return dom
}
```  

 以上代码出现了singleTagRE;这里推荐一个正则查询工具:[http://regexpal.isbadguy.com/](http://regexpal.isbadguy.com)
    *  singleTagRE		
    
    ![QQ截图20170608135118.png](http://upload-images.jianshu.io/upload_images/1909602-9b1238722c96a596.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
    * tagExpanderRE  
    
     ![QQ截图20170608142535.png](http://upload-images.jianshu.io/upload_images/1909602-046f51adf996d191.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
    * fragmentR        
        
    ![QQ截图20170608173421.png](http://upload-images.jianshu.io/upload_images/1909602-0b7fd77e150953ef.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

*  第二种 当context有值
    ```javascript
     else if (context !== undefined) return $(context).find(selector)
    ```
    这里涉及到一个方法find,是$.fn中的方法,之后做统一分析;
    
*  第三种 没有context;
    ```javascript
     else dom = zepto.qsa(document, selector)
    ```
      ```javascript
    	zepto.qsa = function(element, selector){
        	var found,
            //判断是不是ID
            maybeID = selector[0] == '#',
            //判断是不是css
            maybeClass = !maybeID && selector[0] == '.',
            //看是不是class和id名,如果是,将'#'或者'.'去除,然后赋值给nameOnlt;
            //否则,直接将值赋值;
            nameOnly = maybeID || maybeClass ? selector.slice(1) : selector,
            //simpleSelectorRE = /^[\w-]*$/,
            //匹配字母数字下划线和减号的组合;
            isSimple = simpleSelectorRE.test(nameOnly)
            //如果有内置getElementById方法,并且是id名;
        return (element.getElementById && isSimple && maybeID) ?
        	//则返回element.getElementByID(nameOnly)
          ( (found = element.getElementById(nameOnly)) ? [found] : [] ) :
          //反之的话,再做一次判断
          //若element不为元素节点,document,DocumentFragment时;为空,
          (element.nodeType !== 1 && element.nodeType !== 9 && element.nodeType !== 11) ? [] :
          //否则,将节点转换成数组;
          slice.call(
          //这里是一个三元运算符里套着另一个三元运算符;
            isSimple && !maybeID && element.getElementsByClassName ?
            //当为class,则调用element.getElementsByClassName(nameOnly) 
                  maybeClass ? element.getElementsByClassName(nameOnly) :
                    //否则调用tagName;
                  element.getElementsByTagName(selector) :
           //这个否则是最外层的判断;
              element.querySelectorAll(selector)
          )
      }
      ```

## 3. 当传入的值为函数时,则在dom加载后执行它;
```javascript
else if (isFunction(selector)) return $(document).ready(selector)
```
## 4. 如果selector为Z的实例对象.则返回他自己;
```javascript
else if (zepto.isZ(selector)) return selector
```
##  5. 最后,又分为5种情况;
* 如果selector为数组;
	```javascript
    //
    if (isArray(selector)) dom = compact(selector)
    ```
    这里用到了一个compact方法;
    ```javascript
    //这里调用了一个filter方法,是在$.fn内,以后统一分析;
    //这个函数是去除数组中的null和undefined;
    function compact(array) { return filter.call(array, function(item){ return item != null }) }
    ```
    所以当为数组的时候,去除数组中的null和undefined;
    
* selector为对象
    ```javascript
    else if (isObject(selector))
        dom = [selector], selector = null
    ```
    如果selector为对象,将对象变为一个数组;
* selector为html片段;则将其转换成dom
    ```javascript
    else if (fragmentRE.test(selector))
        dom = zepto.fragment(selector.trim(), RegExp.$1, context), selector = null
    ```
*  有context的时候
    ```javascript
    else if (context !== undefined) return $(context).find(selector)
    ```
* 没有context
    ```javascript
    else dom = zepto.qsa(document, selector)
    ```