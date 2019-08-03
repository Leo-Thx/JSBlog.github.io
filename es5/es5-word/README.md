## 引用规范类型
为什么要说这个类型，这个是官方给出的一种内部结构类型，用来说明 delete，typeof，赋值运算符这些运算符的行为。引用就是已经解析过的命名绑定(在词法环境组件生成时)，包含以下几个部分：  
1. base(基值)：undefined、Object、Boolean、String、Number、环境记录项
2. referenced name(引用名称)
3. strict reference(严格引用标志)
### GetValue(v) 操作
*获取对应基值下所引用名称的值*
>简要步骤如下(具体可以自行查看规范：8.7 引用规范类型):  
>1. Type(v)不是引用，直接返回V
>2. 调用GetBase(v)，获取基值base
>3. 如果基值是undefined，报错
>4. 如果基值是Object、String、Number、Boolean进行对应的取值操作(获取对象属性，或进行拆箱)
>5. 如果是环境记录项，则获取环境记录项中的标识符绑定的值



## void
_语法：void 表达式_

这个单词运算符我们极少用到，本质不是变更表达式的值，而是处理表达式的返回值为undefined
```js
var temp = 100;
temp;   // 这个单值表达式是有值的，100
void temp;  // 表示忽略表达式的返回值，返回undefined
```
*这里我有必要说明一下自执行的函数表达式的几种形式*
1. (function(){}()) 和 (function(){})()
>有真正了解其中的区别了吗(请仔细琢磨下面两句话)：  
    (...()): 这种表示强制函数进行调用，并返回函数表达式调用后的值  
    (...)(): 这种表示强制返回函数表达式，并进行函数调用
2. 有见到写法如下的吗：
```js
void function(){}();    //or !function(){}(); 
new function(){}();     // 创建一个匿名对象
new new Function('return {key: 123}')();    // 难受吧 object{key: 123}
```
>然后前面或者+ -... 等一些乱七八糟的写法，本质就是这些都是 **运算符与运算单元** 的组合形式，只要你明白 _表达式_ 与 _表达式的运算_ 以及 _语句_ 之间的关系，我想你在看优秀源码的时候大体都能知道，不会被一些难以理解的语法所打败。源码一方面是体现语法，一方面是体现设计思维的地方。

__有个不成熟的小建议：当你拿捏不准的时候，多用()运算符分隔__

## in 11.8.7
_如果对 **引用类型** 和 **GetValue** 没有过多的了解，建议往前面看一看_

语法：RelationalExpression in ShiftExpression

官方说明：
>产生式 RelationalExpression : RelationalExpression in ShiftExpression 按照下面的过程执行:
>1. 令 lref 为解释执行 RelationalExpression 的结果 .  
>2. 令 lval 为 GetValue(lref).  
>3. 令 rref 为解释执行 ShiftExpression 的结果 .  
>4. 令 rval 为 GetValue(rref).  
>5. 如果 Type(rval) 不是 Object ，抛出一个 TypeError 异常 .  
>6. 返回以参数 ToString(lval). 调用 rval 的 [[HasProperty]] 内置方法的结果

简要分析一下：
1. 获取in操作符 左侧表达式 和 右侧表达式 的值
2. 右侧不是对象则报错(这里解释了基本类型装箱为什么会报错)
    ```js
    var num = 1;
    console.info("toString" in num);        // in 不能用在基本类型上
    ```
3. 调用HasProperty判断自身或原型链上是否有对应的属性(无论enumerable)

## instanceof 11.8.6
语法：leftExpression instanceof rightExpression
这个我就不把官方规范弄出来了，因为冗长的执行步骤也记不住。

1. 如果右侧表达式没有[[HasInstance]]则报错 （在标准内置 ECMAScript 对象中只有 Function 对象实现 [[HasInstance]]），简单来说，右侧应该是一个函数
    ```js
    console.info(({}) instanceof Object);   // true
    console.info("string" instanceof String);   // false
    console.info(({}) instanceof true);   // error
    ```
2. 调用[[HasInstance]]方法 - F.[[HasInstance]]\(V\) 查找原型链
>1. 如果 V 不是Object类型，则返回false : "string" instanceof String
>2. 获取F的原型对象 O = F.prototype, 如果 O 不是对象，报错
>3. 获取 V 内部属性[[prototype]]， 即 V = V.\_\_proto__ 
>4. 判断 如果 O 与 V 是同一个对象，返回true，否则返回false
>5. 重复 3 - 4  

```js
    function A(){ this.name = 'A instance'; }
    function B(){ this.name = 'B instance'; }

    var a = new A();
    console.info(a instanceof A);

    // 简单处理一下原型委托
    function Temp(){ this.name = 'Temp instance'; };
    Temp.prototype = A.prototype;
    B.prototype = new Temp;
    B.prototype.constructor = B;

    var b = new B();
    console.info(b instanceof A);
```
**总结一下：即查找左侧的原型链上是否含有右侧的原型对象**


## new 13.2.2[[Construct]]
语法：new NewExpression
1. 一些判断直接略过，比如表达式是不是函数，有没有[[Construct]]属性等
2. 执行[[Construct]]操作， 令 F 为表达式的构造函数
    * 创建一个新的ECMAScript原生对象 obj
    * 设置 obj.[[Class]] = "Object", obj[[Extensible]] = true;
    * 获取 proto = F.prototype, 设定 obj.[[prototype]] = proto, 即obj.\_\_proto__ = proto;
    * 以 obj 为this值，调用 F ， result = F.call(this, args);
    * 如果result是对象类型，返回result，否则返回obj

_这个没有必要多说_

## delete 11.4.1
语法：delete UnaryExpression 删除某个属性

多余不说，只说一些注意的地方：
1. 严格模式下，如果右侧是未处理的绑定，则会报错
    ```js
    (function(){'use strict'; delete a})();
    // Uncaught SyntaxError: Delete of an unqualified identifier in strict mode.
    ```
2. 严格模式下，右侧是环境记录项，对不可变绑定一会报错
    ```js
    void function(){
        'use strict';
        var temp = function fnName(){
            delete fnName;  // 错误同上，新增的不可变绑定在数据类型中有提到过，详情参考上一篇文章
        }
        temp();
    }();
    ```
3. 严格模式下，不可配置的[[Configurable]]为false，执行会报错
    ```js
    ;(function(){
        'use strict';
        var obj = { canDelete : 'canDelete' };
        Object.defineProperty(obj, 'unDelete', {
            value: 'unDelete'
        });
        delete obj.canDelete;
        delete obj.unDelete;    // error
    }());
    ```

## typeof 11.4.3

|  类型   | 结果  |
|  ----  | ----  |
| Undefined  | "undefined" |
| Null  | "object" \ "null?" |
| Boolean  | "boolean" |
| Number  | "number" |
| String  | "string" |
| Object(原生，且没有实现[[call]])  | "object" |
| Object(原生或宿主，且实现[[call]])   | "function" |

