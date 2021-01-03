# JavaScript 高级程序设计
## 第 1 章 JavaScript 简介
* JavaScript 历史回顾
* JavaScript 是什么
* JavaScript 与 ECMAScript 的关系
* JavaScript 的不同版本
### 1.1 JavaScript 简史
### 1.2 JavaScript 实现
* 核心（ECMAScript）：由 ECMA-262 定义，提供核心语言功能
* 文档对象模型（DOM）：提供操作网页内容的方法和接口
* 浏览器对象模型（BOM）：提供与浏览器交互的方法和接口
#### 1.2.1 ECMAScript
由 ECMA-262 定义的 ECMAScript 与 Web 浏览器没有依赖关系。实际上，这门语言并不包含输入和输出定义。ECMA-262 定义的只是这门语言的基础，而在此基础之上可以构建更完善的脚本语言。我们常见的 Web 浏览器只是 ECMAScript 实现可能的宿主环境之一。宿主环境不仅提供基本的ECMAScript 实现，同时也会提供该语言的扩展，一遍语言与环境之间对接交互。而这些扩展----如 DOM，则利用 ECMAScript 的核心类型和语法提供更多更具体的功能，以便实现针对环境的操作。其他的宿主环境包括 Node（一种服务端 JavaScript 平台）和 Adobe Flash。

ECMA-262 标准没有参照 Web 浏览器，它规定了这门语言的下列组成成分：
* 语法
* 类型
* 语句
* 关键字
* 保留字
* 操作符
* 对象
#### 1.2.2 文档对象模型（DOM）
#### 1.2.3 浏览器对象（BOM）
### 1.3 JavaScript 版本
### 1.4 小结
## 第 2 章 在 HTML 中使用 JavaScript
* 使用 \<script> 元素
* 嵌入脚本与外部脚本
* 文档模式对 JavaScript 的影响
* 考虑禁用 JavaScript 的场景
### 2.1 \<script> 元素
HTML 4.01 为 \<script> 定义了下列 6 个属性：
* async: 可选。表示应该立即下载脚本，但不应妨碍页面中的其他操作，比如下载其他资源或等待加载其他脚本。支队外部脚本文件有效。
* charset: 可选。表示通过 src 属性指定的代码的字符集。由于大多数浏览器会忽略它的值，因此这个属性很少有人用。
* defer: 可选。表示脚本可以延迟到文档完全被解析和显示之后再执行。只对外部脚本文件有效。
* language: 已废弃。原来用于表示编写代码使用的脚本语言（如JavaScript、JavaScript1.2 或 VBScript）。大多数浏览器会忽略这个属性，因此也没有必要再用了。
* src: 可选。表示包含要执行代码的外部文件。
* type: 可选。可以看成是 language 的替代熟悉感；表示编写代码使用的脚本语言的内容类型（也成为 MIME 类型）。虽然 text/javascript 和 text/ecmascript 都已经不被推荐使用，但人们一直以来使用的都还是 text/javascript。实际上，服务器在传送 JavaScript 文件时使用的 MIME 类型通常是 application/x-javascript，但在 type 中设置这个值却可能导致脚本被忽略。另外，在非 IE 浏览器中还可以使用以下值：application/javascript 和 application/ecmascript。考虑到约定俗成和最大限度的浏览器兼容性，目前 type 属性的值依旧还是 text/javascript。不过这个属性不是必须的，如果没有指定这个属性，则其默认值仍为 text/javascript。
#### 2.1.1 标签的位置
```
<!DOCTYPE html>
<html>
    <head>
        <title>Example HTML Page</title>
        </head>
    <body>
        <!-- 这里放内容 -->
        <script type="text/javascript" src="example1.js"></script>
        <script type="text/javascript" src="example2.js"></script>
    </body>
</html>
```
#### 2.1.2 延迟脚本
```
<!DOCTYPE html>
<html>
    <head>
        <title>Example HTML Page</title>
        <script type="text/javascript" defer="defer" src="example1.js"></script>
        <script type="text/javascript" defer="defer" src="example2.js"></script>
    </head>
    <body>
        <!-- 这里放内容 -->
    </body>
</html>
<!--
    在这个例子中，虽然我们把 <script> 元素放在了文档的 <head> 元素中，但其中包含的脚本将延迟到浏览器遇到 </html> 标签后再执行。HTML5 规范要求脚本按照它们出现的先后顺序执行，因此第一个延迟脚本会先于第二个延迟脚本执行，而这两个脚本会先于 DOMContentLoaded 事件执行。在现实当中，延迟脚本并不一定会按照顺序执行，也不一定会在 DOMContentLoaded 事件触发前执行，因此最好只包含一个延迟脚本。
-->
```
#### 2.1.3 异步脚本
```
<!DOCTYPE html>
<html>
    <head>
        <title>Example HTML Page</title>
        <script type="text/javascript" async src="example1.js"></script>
        <script type="text/javascript" async src="example2.js"></script>
    </head>
    <body>
        <!-- 这里放内容 -->
    </body>
</html>
```
#### 2.1.4 在 XHTML 中的用法
```
<!-- 报错 -->
<script type="text/javascript">
    function compare(a, b) {
        if (a < b) {
            alert("A is less than B");
        } else if (a > b) {
            alert("A is greater than B");
        } else {
            alert("A is equal to B");
        }
    }
</script>
```
```
<script type="text/javascript">
    function compare(a, b) {
        if (a &lt; b) {
            alert("A is less than B");
        } else if (a > b) {
            alert("A is greater than B");
        } else {
            alert("A is equal to B");
    }
}
</script>

<script type="text/javascript"><![CDATA[
    function compare(a, b) {
        if (a < b) {
            alert("A is less than B");
        } else if (a > b) {
            alert("A is greater than B");
        } else {
            alert("A is equal to B");
        }
    }
]]></script>

<script type="text/javascript">
//<![CDATA[
    function compare(a, b) {
        if (a < b) {
            alert("A is less than B");
        } else if (a > b) {
            alert("A is greater than B");
        } else {
            alert("A is equal to B");
        }
    }
//]]>
</script>
```
#### 2.1.5 不推荐使用的语法
```
<script><!--
    function sayHi(){
        alert("Hi!");
    }
//--></script>
```
### 2.2 嵌入代码与外部文件
在 HTML 中嵌入 JavaScript 代码虽然没有问题，但一般认为最好的做法还是尽可能使用外部文件来包含 JavaScript 代码。
* 可维护性：遍及不同 HTML 页面的 JavaScript 会造成维护问题。但把所有 JavaScript 文件都放在一个文件夹中，维护起来就轻松多了。而且开发人员因此也能够在不触及 HTML 标记的情况下，集中精力编辑 JavaScript 代码。
* 可缓存：浏览器能够根据具体的设置缓存链接的所有外部 JavaScript 文件。也就是说，如果有两个页面都是用同一个文件，那么这个文件只需下载一次。因此，最终结果就是能够加快页面加载速度。
* 适应未来：通过外部文件来包含 JavaScript 无须使用前面提到 XHTML 或注释 hack。HTML 和 XHTML 包含外部文件的语法是相同的。
### 2.3 文档模式
* 标准模式
* 准标准模式
* 混杂模式
### 2.4 <noscript> 元素
```
<html>
    <head>
        <title>Example HTML Page</title>
        <script type="text/javascript" defer="defer" src="example1.js"></script>
        <script type="text/javascript" defer="defer" src="example2.js"></script>
    </head>
    <body>
        <noscript>
            <p>本页面需要浏览器支持（启用） JavaScript。
        </noscript>
    </body>
</html>
```
### 2.5 小结
## 第 3 章 基本概念
* 语法
* 数据类型
* 流控制语句
* 函数
### 3.1 语法
#### 3.1.1 区分大小写
#### 3.1.2 标志符
#### 3.1.3 注释
#### 3.1.4 严格模式
```
"use strict"
```
```
function doSomething(){
    "use strict";
    //函数体
}
```
#### 3.1.5 语句
### 3.2 关键字和保留字
```
关键字
break       do          instanceof  typeof
case        else        new         var
catch       finally     return      void
continue    for         switch      while
debugger    function    this        with
default     if          throw
delete      in          try
```
```
保留字
abstract    enum        int         short
boolean     export      interface   static
byte        extends     long        super
char        final       native      synchronized
class       float       package     throws
const       goto        private     transient
debugger    implements  protected   volatite
double      import      public
```
```
第 5 版把在非严格模式下运行时的保留字缩减为下列这些：
class       enum        extends     super
const       export      import
在严格模式下，第 5 版还对以下保留字施加了限制：
implements  package     public
interface   private     static
let         protected   yield
```
### 3.3 变量
### 3.4 数据类型
#### 3.4.1 typeof 操作符
#### 3.4.2 Undefined 类型
对于尚未声明过的变量，只能执行一项操作，即使用 typeof 操作符检测其数据类型
> 即使未初始化的变量会自动被赋予 undefined 值，但显式地初始化变量依然是明智的选择。如果能做到这一点，那么当 typeof 操作符返回“undefined”时，我们就知道被检测的变量还没有被生命，而不是尚未初始化。
#### 3.4.3 Null 类型
```
alert(null == undefined); //true
```
#### 3.4.4 Boolean 类型
数据类型|转换为 true|转换为false
---|---|---
Boolean|true|false
String|任何非空字符|""
Number|任何非0数字值|0 和 NaN
Object|任何对象|null
Undefined|NA|undefined
#### 3.4.5 Number 类型
1、浮点数值  
浮点数在某些情况会自动转为整数，以节省内存。  
2、数值范围  
如果某次计算的结果得到了一个超出 JavaScript 数值范围的值，那么这个数值将被自动转换成特殊的 Infinity 值。具体来说，如果这个数值是负数，则会被转换成-Infinity（负无穷），如果这个数值是正数，则会被转换成 Infinity（正无穷）。
> isFinite()：判断是否位于最小与最大数值之间。  

3、NaN：这个数值用于表示一个本来要返回数值的操作数未返回数值的情况（这样就不会抛出错误了）。
* 首先，任何涉及 NaN 的操作（例如 NaN/10）都会返回 NaN，这个特点在多步计算中有可能导致问题。其次， NaN 与任何值都不相等，包括 NaN 本身。例如，下面的代码会返回 false：alert(NaN == NaN); //false
* 这个函数接受一个参数，该参数可以是任何类型，而函数会帮我们确定这个参数是否“不是数值”。
```
alert(isNaN(NaN));      //true
alert(isNaN(10));       //false（ 10 是一个数值）
alert(isNaN("10"));     //false（可以被转换成数值 10）
alert(isNaN("blue"));   //true（不能转换成数值）
alert(isNaN(true));     //false（可以被转换成数值 1）
```
4、数值转换
* Number()
* parseInt()
* parseFloat()
#### 3.4.6 String 类型
1、字符字面量  
> 转义字符

2、字符串的特点  
> 对变量进行赋值，需要销毁原来的对象，然后使用新值填充变量。

3、转换为字符串
* xxx.toString()
* String(xxx)
#### 3.4.7 Object 类型
* constructor：保存着用于创建当前对象的函数。
* hasOwnProperty(propertyName)：用于检查给定的属性在当前对象实例中是否存在。
* isPrototypeOf(object)：用于检查传入的对象是否是传入对象的原型。
* propertyIsEnumerable(propertyName)：用于检查给定的属性是否能够使用 for-in 语句来枚举。
* toLocaleString()：返回对象的字符串表示，该字符串与执行环境的地区对应。
* toString()：返回对象的字符串表示。
* valueOf()：返回对象的字符串、数值或布尔值表示。通常与 toString() 方法的返回值相同。
### 3.5 操作符
* 算数操作符，如加号、减号
* 位操作符
* 关系操作符
* 相等操作符
#### 3.5.1 一元操作符
1、递增和递减操作符  
2、一元加和减操作符
#### 3.5.2 位操作符
1、按位非（NOT）  
> ~

2、按位与（AND）  
> &

3、按位或（OR）  
> |

4、按位异或（XOR）  
> ^

5、左移（<<）  
6、有符号的右移（>>）
> 高位填充符号位

7、无符号的右移（>>>）
> 高位填充0
#### 3.5.3 布尔操作符
1、逻辑非
> !  

2、逻辑与
> &&

逻辑与操作可以应用于任何类型的操作数，而不仅仅是布尔值。在有一个操作数不是布尔值的情况下，逻辑与操作就不一定返回布尔值；此时，它遵循下列规则：
* 如果第一个操作数是对象，则返回第二个操作数；
* 如果第二个操作数是对象，则只有在第一个操作数的求值结果为 true 的情况下才会返回该对象；
* 如果两个操作数都是对象，则返回第二个操作数；
* 如果有一个操作数是 null，则返回 null；
* 如果有一个操作数是 NaN，则返回 NaN；
* 如果有一个操作数是 undefined，则返回 undefined。
3、逻辑或
> ||

与逻辑与操作相似，如果有一个操作数不是布尔值，逻辑或也不一定返回布尔值；此时，它遵循下列规则：
* 如果第一个操作数是对象，则返回第一个操作数；
* 如果第一个操作数的求值结果为 false，则返回第二个操作数；
* 如果两个操作数都是对象，则返回第一个操作数；
* 如果两个操作数都是 null，则返回 null；
* 如果两个操作数都是 NaN，则返回 NaN；
* 如果两个操作数都是 undefined，则返回 undefined。

####  3.5.4 乘性操作符
1、乘法  
在处理特殊值的情况下，乘法操作符遵循下列特殊的规则：
* 如果操作数都是数值，执行常规的乘法计算，即两个正数或两个负数相乘的结果还是正数，而如果只有一个操作数有符号，那么结果就是负数。如果乘积超过了 ECMAScript 数值的表示范围，则返回 Infinity 或 -Infinity；
* 如果有一个操作数是 NaN，则结果是 NaN；
* 如果是 Infinity 与 0 相乘，则结果是 NaN；
* 如果是 Infinity 与非 0 数值相乘，则结果是 Infinity 或 -Infinity ，取决于有符号操作数的符号；
* 如果是 Infinity 与 Infinity 相乘，则结果是 Infinity；
* 如果有一个操作数不是数值，则在后台调用 Number() 将其转换为数值，然后再应用上面的规则。

2、除法  
与乘法操作符类似，除法操作符对特殊的值也有特殊的处理规则。这些规则如下：
* 如果操作数都是数值，执行常规的除法计算，即两个正数或两个负数相除的结果还是正数，而如果只有一个操作数有符号，那么结果就是负数。如果商超过了 ECMAScript 数值的表示范围，
则返回 Infinity 或-Infinity；
* 如果有一个操作数是 NaN，则结果是 NaN；
* 如果是 Infinity 被 Infinity 除，则结果是 NaN；
* 如果是零被零除，则结果是 NaN；
* 如果是非零的有限数被零除，则结果是 Infinity 或-Infinity，取决于有符号操作数的符号；
* 如果是 Infinity 被任何非零数值除，则结果是 Infinity 或 -Infinity，取决于有符号操作数的符号；
* 如果有一个操作数不是数值，则在后台调用 Number()将其转换为数值，然后再应用上面的规则。

3、求模  
与另外两个乘性操作符类似，求模操作符会遵循下列特殊规则来处理特殊的值：
* 如果操作数都是数值，执行常规的除法计算，返回除得的余数；
* 如果被除数是无穷大值而除数是有限大的数值，则结果是 NaN；
* 如果被除数是有限大的数值而除数是零，则结果是 NaN；
* 如果是 Infinity 被 Infinity 除，则结果是 NaN；
* 如果被除数是有限大的数值而除数是无穷大的数值，则结果是被除数；
* 如果被除数是零，则结果是零；
* 如果有一个操作数不是数值，则在后台调用 Number()将其转换为数值，然后再应用上面的规则。
#### 3.5.5 加性操作符
1、加法  
如果两个操作符都是数值，执行常规的加法计算，然后根据下列规则返回结果：
* 如果有一个操作数是 NaN，则结果是 NaN；
* 如果是 Infinity 加 Infinity，则结果是 Infinity；
* 如果是-Infinity 加-Infinity，则结果是-Infinity；
* 如果是 Infinity 加-Infinity，则结果是 NaN；
* 如果是+0 加+0，则结果是+0；
* 如果是-0 加-0，则结果是-0；
* 如果是+0 加-0，则结果是+0。

不过，如果有一个操作数是字符串，那么就要应用如下规则：
* 如果两个操作数都是字符串，则将第二个操作数与第一个操作数拼接起来；
* 如果只有一个操作数是字符串，则将另一个操作数转换为字符串，然后再将两个字符串拼接起来。

2、减法
与加法操作符类似， ECMAScript 中的减法操作符在处理各种数据类型转换时，同样需要遵循一些特殊规则，如下所示：
* 如果两个操作符都是数值，则执行常规的算术减法操作并返回结果；
* 如果有一个操作数是 NaN，则结果是 NaN；
* 如果是 Infinity 减 Infinity，则结果是 NaN；
* 如果是-Infinity 减-Infinity，则结果是 NaN；
* 如果是 Infinity 减-Infinity，则结果是 Infinity；
* 如果是-Infinity 减 Infinity，则结果是-Infinity；
* 如果是+0 减+0，则结果是+0；
* 如果是+0 减-0，则结果是-0；
* 如果是-0 减-0，则结果是+0；
* 如果有一个操作数是字符串、布尔值、 null 或 undefined，则先在后台调用 Number()函数将其转换为数值，然后再根据前面的规则执行减法计算。如果转换的结果是 NaN，则减法的结果就是 NaN；
* 如果有一个操作数是对象，则调用对象的 valueOf()方法以取得表示该对象的数值。如果得到的值是 NaN，则减法的结果就是 NaN。如果对象没有 valueOf()方法，则调用其 toString()
方法并将得到的字符串转换为数值。
#### 3.5.6 关系操作符
与 ECMAScript 中的其他操作符一样，当关系操作符的操作数使用了非数值时，也要进行数据转换或完成某些奇怪的操作。以下就是相应的规则。
* 如果两个操作数都是数值，则执行数值比较。
* 如果两个操作数都是字符串，则比较两个字符串对应的字符编码值。
* 如果一个操作数是数值，则将另一个操作数转换为一个数值，然后执行数值比较。
* 如果一个操作数是对象，则调用这个对象的 valueOf()方法，用得到的结果按照前面的规则执行比较。如果对象没有 valueOf()方法，则调用 toString()方法，并用得到的结果根据前面的规则执行比较。
* 如果一个操作数是布尔值，则先将其转换为数值，然后再执行比较。
#### 3.5.7 相等操作符
1、相等和不相等  
在转换不同的数据类型时，相等和不相等操作符遵循下列基本规则：
* 如果有一个操作数是布尔值，则在比较相等性之前先将其转换为数值——false 转换为 0，而
true 转换为 1；
* 如果一个操作数是字符串，另一个操作数是数值，在比较相等性之前先将字符串转换为数值；
* 如果一个操作数是对象，另一个操作数不是，则调用对象的 valueOf()方法，用得到的基本类型值按照前面的规则进行比较；

这两个操作符在进行比较时则要遵循下列规则。
* null 和 undefined 是相等的。
* 要比较相等性之前，不能将 null 和 undefined 转换成其他任何值。
* 如果有一个操作数是 NaN，则相等操作符返回 false，而不相等操作符返回 true。重要提示：
即使两个操作数都是 NaN，相等操作符也返回 false；因为按照规则， NaN 不等于 NaN。
* 如果两个操作数都是对象，则比较它们是不是同一个对象。如果两个操作数都指向同一个对象，则相等操作符返回 true；否则，返回 false。

2、 全等和不全等
#### 3.5.8 条件操作符
#### 3.5.9 赋值操作符
#### 3.5.10 逗号操作符
### 3.6 语句
#### 3.6.1 if 语句
