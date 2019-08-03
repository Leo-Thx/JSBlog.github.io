## [引用规范类型](http://www.ecma-international.org/ecma-262/5.1/index.html#sec-8.7)
为什么要说这个类型，且部分运算符需要这个已经定义好的规范类型。
这个是官方给出的一种内部结构类型，用来说明 delete，typeof，赋值运算符这些运算符的行为。引用就是已经解析过的命名绑定(在词法环境组件生成时)，包含以下几个部分：  
1. base(基值)：undefined、Object、Boolean、String、Number、环境记录项，如果base是undefined，则表示该标识符绑定是个无法解析的绑定
2. referenced: name(引用名称)
3. strict reference: 严格模式标志

### 部分抽象操作
- GetBase(V)。 返回引用值 V 的基值组件。
- GetReferencedName(V)。 返回引用值 V 的引用名称组件。
- IsStrictReference(V)。 判断引用值 V 是否是严格的。
- HasPrimitiveBase(V)。 如果基值是 Boolean, String, Number，那么返回 true。即基值是基本数据类型
- IsPropertyReference(V)。 如果基值是个对象或 HasPrimitiveBase(V) 是 true，那么返回 true；否则返回 false。即base 不是环境记录项和undefined
- IsUnresolvableReference(V)。 如果基值是 undefined 那么返回 true，否则返回 false。

### [GetValue(v)](http://www.ecma-international.org/ecma-262/5.1/index.html#sec-8.7.1)
*获取对应基值下所引用名称的值*
>1. Type(v)不是引用，直接返回V
>2. 调用GetBase(v)，获取基值base
>3. 如果基值是undefined，报错
>4. 如果基值是Object、String、Number、Boolean
>> - 如果基值不是String、Number、Boolean，则进行对应的 Object的属性值获取
>> - 将 base 作为 this 值，传递 GetReferencedName(V) 为参数，调用 get 内部方法，返回结果
>5. 如果是环境记录项，则获取环境记录项中的标识符绑定的值


## [操作符 void](http://www.ecma-international.org/ecma-262/5.1/index.html#sec-11.4.2)
_语法：void 表达式_

这个单词运算符我们极少用到，本质不是变更表达式的值，而是处理表达式的返回值为undefined，因为表达式都是有值的。
```js
var temp = 100;
temp;   // 这个单值语句是有值的，100
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
new new Function('return {key: 123}')();    // object{key: 123}
```
>然后前面或者+ -... 等一些乱七八糟的写法，本质就是这些都是 **运算符与运算单元** 的组合形式，只要你明白 _表达式_ 与 _表达式的运算_ 以及 _语句_ 之间的关系，我想你在看优秀源码的时候大体都能知道，不会被一些难以理解的语法所打败。源码一方面是体现语法，一方面是体现设计的地方。

__有个不成熟的小建议：当你拿捏不准的时候，多用()运算符分隔__

## [操作符 in](http://www.ecma-international.org/ecma-262/5.1/index.html#sec-11.8.7)
_如果对 **引用类型** 和 **GetValue** 没有过多的了解，建议往前面看一看_

语法：RelationalExpression in ShiftExpression

官方说明：
>产生式 RelationalExpression : RelationalExpression in ShiftExpression 按照下面的过程执行:
>1. 令 lref 为解释执行 RelationalExpression 的结果 .  
>2. 令 lval 为 GetValue(lref).  即获取左侧运算元的值
>3. 令 rref 为解释执行 ShiftExpression 的结果 .  
>4. 令 rval 为 GetValue(rref).  即获取右侧运算元的值
>5. 如果 Type(rval) 不是 Object ，抛出一个 TypeError 异常 .  
>6. 返回以参数 ToString(lval). 调用 rval 的 [[HasProperty]] 内置方法的结果
    >> 1. rval[[HasProperty]](ToString(lval)) 即判断右侧是否有左侧的属性, 调用[[GetProperty]]
    >> 2. [[GetProperty]](P) 解释如下：
    >>> 1. 如果O.[[GetOwnProperty]](P) 有值，就直接返回，即判断自己有没有属性 P
    >>> 2. 否则令 proto 为 O 的 [[Prototype]] 内部属性值, 即获取原型
    >>> 3. 递归调用 proto.[[GetProperty]](P)

简要分析一下：
1. 获取in操作符 左侧表达式 和 右侧表达式 的值
2. 右侧不是对象则报错(这里解释了基本类型装箱为什么会报错)
    ```js
    var num = 1;
    console.info("toString" in num);        // in 不能用在基本类型上
    ```
3. 调用HasProperty判断自身或原型链上是否有对应的属性(无论enumerable)

## [操作符 instanceof](http://www.ecma-international.org/ecma-262/5.1/index.html#sec-11.8.6)
语法：leftExpression instanceof rightExpression

1. 令 lref 为解释执行 RelationalExpression 的结果 .
2. 令 lval 为 GetValue(lref).
3. 令 rref 为解释执行 ShiftExpression 的结果 .
4. 令 rval 为 GetValue(rref).
5. 如果 Type(rval) 不是 Object，抛出一个 TypeError 异常 .
6. 如果 rval 没有 [[HasInstance]] 内置方法，抛出一个 TypeError 异常.(在标准内置ECMAScript对象中只有Function对象实现)，右侧应该是一个函数
    ```js
    console.info(({}) instanceof Object);   // true
    console.info("string" instanceof String);   // false
    console.info(({}) instanceof true);   // error
    ```
7. 调用[[HasInstance]]方法 - F.[[HasInstance]]\(V\) 查找原型链
>1. 如果 V 不是Object类型，则返回false : "string" instanceof String
>2. 获取F的原型对象 O = F.prototype, 如果 O 不是对象，报错
>3. 获取 V 内部属性[[prototype]]， 即 V = V.\_\_proto__ 
>4. 判断 如果 O 与 V 是同一个对象，返回true，否则返回false
>5. 重复 3 - 4  

```js
    function A(){ this.name = 'A instance'; }
    function B(){ this.name = 'B instance'; }

    var a = new A();
    console.info(a instanceof A);   // true

    // 简单处理一下原型委托
    function Temp(){ this.name = 'Temp instance'; };
    Temp.prototype = A.prototype;
    B.prototype = new Temp;
    B.prototype.constructor = B;

    var b = new B();
    console.info(b instanceof A);   // true
```
**总结一下：即查找左侧的原型链上是否含有右侧的原型对象**


## [操作符 new](http://www.ecma-international.org/ecma-262/5.1/index.html#sec-11.2.2)

语法：new NewExpression
1. 一些判断直接略过，比如表达式是不是函数，有没有[[Construct]]属性等
2. 执行[[[Construct]]](http://www.ecma-international.org/ecma-262/5.1/index.html#sec-13.2.2)操作， 令F为表达式的构造函数
    * 创建一个新的ECMAScript原生对象 obj
    * 设置 obj.[[Class]] = "Object", obj.[[Extensible]] = true;
    * 获取 proto = F.prototype, 设定 obj.[[prototype]] = proto, 即obj.\_\_proto__ = proto;
    * 以 obj 为this值，调用 F ， result = F.call(this, args);
    * 如果result是对象类型，返回result，否则返回obj;


## [操作符 delete](http://www.ecma-international.org/ecma-262/5.1/index.html#sec-11.4.1)
语法：delete UnaryExpression 删除某个属性

多余不说，只说一些注意的地方：
1. 严格模式下，如果右侧是未处理的绑定(_即base的值是undefined_)，则会报错
    ```js
    (function(){'use strict'; delete a})();
    // Uncaught SyntaxError: Delete of an unqualified identifier in strict mode.
    ```
2. 严格模式下，右侧是环境记录项，对不可变绑定一会报错
    ```js
    void function(){
        'use strict';
        var temp = function fnName(){
            delete fnName;  // 错误同上，因为函数名是不能更改的绑定
        }
        temp();
    }();
    ```
3. 严格模式下，不可配置的[[Configurable]]为false，执行会报错
    ```js
    (function(){
        'use strict';
        var obj = { canDelete : 'canDelete' };
        Object.defineProperty(obj, 'unDelete', {
            value: 'unDelete'
        });
        delete obj.canDelete;
        delete obj.unDelete;    // error
    }());
    ```

## [操作符 typeof](http://www.ecma-international.org/ecma-262/5.1/index.html#sec-11.4.3)
语法：typeof UnaryExpression;
1. 令 val 为 右侧表达式的值
2. 如果 val 是引用类型
    - 如果基值是Undefined，返回 'undefined'
    - 否则 令 val = GetVal(val)，获取引用的值
3. 根据下表操作

|  类型   | 结果  |
|  ----  | ----  |
| Undefined  | "undefined" |
| Null  | "object" |
| Boolean  | "boolean" |
| Number  | "number" |
| String  | "string" |
| Object(原生，且没有实现[[call]])  | "object" |
| Object(原生或宿主，且实现[[call]])   | "function" |


参考资料：  
    《ES5文档》(http://www.ecma-international.org/ecma-262/5.1/index.html)
