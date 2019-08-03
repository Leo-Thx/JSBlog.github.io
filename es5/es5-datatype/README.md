# 对数据类型的补充说明
_本篇主要是对ES5中常见的数据类型进行一些补充和说明_

## 基本数据类型
经常被问及一些基本的数据类型，我也不打算深入研究，某个数据究竟在内存中是怎么样存储的。JavaScript是一门弱类型的语言，就个人理解而言：JavaScript变量的类型取决于变量值的类型。

你需要知道的一些基本数据类型
- number
- string
- boolean
- undefined
- null

_使用运算符 **typeof** 对以上进行运算你会发现除了**null**为object之外，其他都是对应的字符串形式_
_number，string，boolean都有其对应的包装函数 Number，String，Boolean_
### 使用方式
```js
// 直接量形式使用即可
var _num = 1;
var _str = "string";
var _bool = true;
var _null = null;
var _undefined = undefined;
```
_对number的说明：由于数字是可以带小数的，故还可能有以下形式_
```js
var num = .1;   // or: var num = 1.;
```

## 引用数据类型
对C有了解的可能就会知道，有些变量存储单元是对应的直接值，而有些存储单元存储的某个内存地址，地址所指向的位置就是其真实的数据存储的位置（_以上不考虑堆栈中的其他数据和状态所表示的结构_）。

### 基本类型和引用类型的异同
我没有特别详尽的语言去概述基本类型和引用类型的异同
- 基本类型是按值访问的，且不能更改基本类型的值，比如你不可能把5改成6吧
- 基本类型是在序列中直接可以进行大小比较的，比如 5 > 4
- 基本类型是存储在栈内存中的

* 引用类型是存储在栈内存和堆内存中的，是按引用访问的
* 引用类型的值是可变的
    ```js
    var obj = {};
    obj.name = "newName"
    ```
* 引用类型相互比较的时候，只能比较地址是否一样

_如果我们自己不搞事情，typeof可以区分function和其他引用类型，通过instanceof运算符也能推断出某个对象是否是某个构造函数的实例，通过Object.prototype.toString能精确获取。但是在ES6中，如果自己要放炸弹：Symbol.hasInstance -> instanceof，Symbol.toStringTag -> Object.prototype.toString，也请尽量考虑一下爆炸的范围_
```js
// 盗用jQuery中的各种类型校验函数
jQuery.each("Boolean Number String Function Array Date RegExp Object".split(" "), function(i, name) {
	class2type[ "[object " + name + "]" ] = name.toLowerCase();
});
```

### 常见引用类型

#### Object 普通对象
我们经常看到的多数是Object的实例，即直接由函数Object构造调用而来的实例
```js
var obj = {};   //直接量形式
obj = new Object();
obj = Object(); // 这种形式对不同的参数会有不同的处理，参见以下说明
// 以下方式我也写一下
obj = eval('({})');
obj = new Function('return {}')();
```
#### Date 日期操作
常使用的日期的操作，这个我觉得查文档可能会更好，我比较喜欢moment.js这个库。

#### RegExp
正则表达式：var expression = /pattern/flags; 具体也自己查文档更好，推荐查看犀牛书。我也把自己的记录分享出来
```
ES3中规定，一个正则表达式直接量会在执行到它时转换为RegExp对象，同一段代码所表示的正则表达式直接量的每次运算都返回同一个对象
ES5与ES3相反，每次都返回新的对象

直接量字符
    ^ $ . ? * + = ! : | \ / () [] {}	\t \n...
字符类
    [...] [^...] . \w \W \s \S \d \D [\b]
重复
    {n, m} {n,} {n} * + ?(可选)
    非贪婪重复：尽可能的多的匹配字符，若不需要，则只需要在上述重复后面添加 '?'
    注意：正则表达式的模式匹配总是会寻找字符串中第一个可能匹配的位置
        例: /a+?b/ -> 'aaab'	该匹配从第一个字符开始，所以全匹配
选择、分组、引用
    | 用于分隔供选择的字符，选择项的尝试匹配是从左到右，直到发现匹配项，一旦左边匹配，就会忽略右边
    () 
        用于将单独的项组合成子表达式；
        在完整的模式中定义子模式，可以进行抽取
        表达式(?:...)只是分组，而不会生成引用，即分组不会编码
指定匹配位置
    \b匹配单词边界，位于\w \W之间的边界or单词与字符串的开始或结尾之间的边界(匹配发生的合法的位置) ^ $
    \B将把匹配的锚点定位在不是单词边界的地方 /\B[Ss]cript/ == JavaScript|postscript !=script|Scripting
    任意表达式都可以作为锚点条件
        (?=) 先行断言
        (?!) 负先行断言
    锚字符
        ^		匹配字符串开头，多行检索中，匹配一行的开头
        $		..........结尾
        \b 		匹配一个单词边界，注意 [\b]匹配的是退格符
        \B 		匹配非单词边界位置
        (?=p)	零宽正向先行断言，要求接下来的字符都与p匹配，但不能包括匹配p的那些字符
        (?!p)	零宽负向........，要求............不与p匹配，...
```

#### Array 数组类型
数组类型：存放一组值的数据结构，对其基本的操作和函数需要牢记于心

_构造函数的说明：参数如果只有一个，且是数字则表示长度，其他则作为元素，ES6中Array.of和Array.from对其进行了补充的操作_

>会改变数组的操作：pop/push、shift/unshift、splice  
>遍历：forEach、some、every、filter、map、reduce/reduceRight  
>其他操作：slice、concat、sort、indexOf、lastIndexOf

___filter、map、reduce是函数式中针对集合编程的基础：数据流+高阶函数___

#### Function 函数类型
ES5中所见的函数都由Function构造调用而来，依据官方说明，函数的产生式有如下形式
1. 函数声明：function Identifier(FormalParameterList){FunctionBody}
2. 匿名函数表达式：function(FormalParameterList){FunctionBody}
3. 命名函数表达式：function Identifier(FormalParameterList){FunctionBody}
>**对命名函数表达式的简要说明（此处牵扯到一部分词法环境的知识）**：
>1. 在构造函数对象前，以当前词法环境组件Lexical为参数新建一个声明式的词法环境组件funcEnv
>2. 在新建的词法环境中的环境记录项中创建不可变的绑定Identifier，用来在函数体内部做自由变量寻址使用

>函数对象构建过程说明(简化版)
>1. 指定当前词法环境为**Scope**（Scope是形成作用域链的基础）
>2. 新建一个原生ECMAScript对象**F**
>3. 设置**F**内部属性*class*为 *"Function"*，内部属性*prototype*为*Function.prototype*，内部属性*Scope*为**Scope**的值
>4. 处理FormalParameterList参数列表，设置length，arguments等也会定义
>5. 令*proto*为**new Object**调用结果  
    - 以*constructor*为名字调用**proto**上**DefinePrototype**内部方法 value:F  
    - 以*prototype*为名字调用**F**上**DefinePrototype**内部方法 value:proto
>6. new Function构造产生式，Scope指定为全局环境

___命名函数表达式，只是在第一步之前新建一个词法环境用来存储函数名称(此处所有只适用于ES5)___

![初始化草图-真的是草图](./image/data_type.png)


#### 基本包装类型
为了便于操作基本类型的值，ECMAScript提供了三种特殊的引用类型：Number、String、Boolean
```js
var _num = new Number(1);   // 千万不要省略new运算符，否则是做类型转换
var _str = new String("string");
var _bool = new Boolean(true);

// 对Object函数调用时的说明
console.info(Object(1));
console.info(Object("string"));
console.info(Object(true));
// 对以上三种基本数据，会返回对应的包装类型

// 当参数是null、undefined时直接生成一个新的对象
console.info(Object(null));
console.info(Object(undefined));

// 当参数是引用类型时直接返回该引用
console.info(Object({key: 'value'}));
```

探究自动包装的特征  
_后台隐式创建对应包装对象，在这个实例上调用指定方法，最后销毁这个实例_
```js
String.prototype.getSelf = function(){
    return this;
};
var result = "string".getSelf();
console.info(result, typeof result);    //String{"string"} "object"
// delete String.prototype.getSelf;

var num = 1;
console.info(num instanceof Number);    // false
// console.info("toString" in num);        // in 不能用在基本类型上

Number.prototype.getFn = function(){}
// 12.6.4 for-in 语句
// 产生式 IterationStatement : for ( LeftHandSideExpression in Expression ) Statement 按照下面的过程执行 :
// 1. 令 exprRef 为解释执行 Expression 的结果 .
// 2. 令 experValue 为 GetValue(exprRef).
// 3. 如果 experValue 是 null 或 undefined，返回 (normal, empty, empty).
// 4. 令 obj 为 ToObject(experValue).
for(var attr in 1){
    console.info(attr); // getFn
}
 // 对此的说明存在于 规范12.10中对产生式with中进行数据类型转换
with(1){
    console.info(getFn);    // 输出函数
}
```
_总结：当基本类型数据在做存取运算时，会发生自动转换为包装对象的操作_

#### 内置对象类型：global/window、Math
提供的对应的可操作方法 请自行查阅文档

## valueOf、toString、toPrimitive的说明
ToPrimitive抽象操作：将参数转为非对象类型，检查该值是否有 valueOf() 方法。如果有并且返回基本类型值，就使用该值进行强制类型转换。如果没有就使用 toString()的返回值（如果存在）来进行强制类型转换。
```js
var _valueOf = Number.prototype.valueOf;
Number.prototype.valueOf = function(){
    console.info("valueOf", typeof this);    //"object"
    var result = _valueOf.call(this);
    console.info("valueOf-result", result, typeof result);  // 原始值
    // return this;
    return result;
};
var _toString = Number.prototype.toString;
Number.prototype.toString = function(){
    console.info("toString", typeof this);
    var result = _toString.call(this);

    console.info("toString-result", result, typeof result); // 原始值的字符串形式
    return _toString.call(this);
};
var _num = new Number(true);
console.warn(100 + _num);
```
>object -> string  
    1. 调用toString方法，如果返回一个原始值，将这个值转为String并返回  
    2. 没有toString or 返回的不是原始值，调用valueOf  
    3. JavaScript无法从toString或者valueOf方法获取一个原始值，抛出一个类型异常  
>object -> number  
    1. 调用valueOf，如果返回一个原始值，转为number并返回  
    2. 调用toString，如果返回一个原始值，转为number并返回  
    3. 否则抛出异常  

|  类型   | valueOf返回值  | 备注 |  toStrig返回值  |
|  ----  | ----  | ----  | ----  | ----  |
| Boolean  | 对应的基本类型 |  | 对应基本类型值的字符串形式 |
| Number  | 对应的基本类型 | |对应基本类型值的字符串形式 |
| String  | 对应的基本类型 | |对应基本类型值的字符串形式 |
| Array  | 数组 | 引用类型 | Array.toString一致 |
| Function  | 函数 | 引用类型 | function的字符串表示形式 |
| Object  | 对象 | 引用类型 | '[object Object]' |
| RegExp  | 正则对象 | 引用类型 | 字面量字符串形式 |
| Error  | error对象 | 引用类型 | 暂无 |

## 数据类型变换
一般转换形式：

ToString：非字符串到字符串的转换
>普通对象，除非自定义toString函数，否则调用内置的toString进行转换；其他对象强制类型转换，请参考上述

ToNumber：
>true: 1, false: 0, "":0, null: 0, undefined: NaN  
对象类型转换，则调用抽象操作ToPrimitive

ToBoolean：
>所有假值 ：undefined null false 0 -0 NaN "" 为false，非假值都是true

### 显示强制类型转换
1. 字符串和数字的相互转换：Number、String(_没有new关键字_)
2. 显示函数解析数字字符串：parseInt/parseFloat
3. 显示转换为布尔值：Boolean构造函数，运算符!、&&、||都是直接转换

### 隐式类型转换
1. 字符串和数字之间的隐式类型转换  
    >如果某个操作数是字符串或者能够通过以下步骤转换为字符串的话， + 将进行拼接操作。  
    >如果其中一个操作数是对象（包括数组）  
        >>首先对其调用ToPrimitive 抽象操作（规范 9.1 节），  
        >>该抽象操作再调用 [[DefaultValue]] （规范 8.12.8节），以数字作为上下文 

    >运算符+和String转换区别  
        >>ToPrimitive抽象操作 在运算符+时，先调用valueOf，然后调用toString
            而String显示转换时，直接调用toString
2. 布尔值到数字隐式类型转换
    true -> 1
    false -> 0
3. 隐式强制类型转换为布尔值  
    if/for/while/do..while的条件判断  
    ?..:..	&& ||
4. || 和 &&  
    选择器运算符，返回值并不一定是布尔类型，而是两个操作数其中一个的值

### 宽松相等和严格相等
== 允许在相等比较中进行强制类型转换，而 === 不允许
>11.9.3.1 的最后定义了对象（包括函数和数组）的宽松相等 == 。两个对象指向同一个值时即视为相等，不发生强制类型转换  
11.9.3 节中还规定， == 在比较两个不同类型的值时会发生隐式强制类型转换，会将其中之一或两者都转换为相同的类型后再进行比较  

>1.1 字符串和数字  
    (1) 如果 Type(x) 是数字， Type(y) 是字符串，则返回 x == ToNumber(y)的结果。  
    (2) 如果 Type(x) 是字符串， Type(y) 是数字，则返回 ToNumber(x) == y的结果。
  
>1.2 其他类型和布尔值  
    (1) 如果 Type(x) 是布尔类型，则返回 ToNumber(x) == y 的结果；  
    (2) 如果 Type(y) 是布尔类型，则返回 x == ToNumber(y) 的结果。  

>1.3 null和undefined比较  
    (1) 如果 x 为 null ， y 为 undefined ，则结果为 true 。  
    (2) 如果 x 为 undefined ， y 为 null ，则结果为 true 。  

>1.4 对象和非对象比较  
    (1) 如果 Type(x) 是字符串或数字， Type(y) 是对象，则返回 x == ToPrimitive(y)的结果；  
    (2) 如果 Type(x) 是对象， Type(y) 是字符串或数字，则返回 ToPromitive(x) == y的结果。  
封装对象会自动进行拆封

_简记：boolean->number, string->number, null等于undefined, 对象类型转为基本类型_
```js
console.info([] == 0);  // true
console.info(false == []);  // true
console.info(null == 0);    // false
console.info([] + 1);   // "1"
console.info({} + 1);   // 1 ：因为js在执行过程中语句优先，{}被解析为代码块
console.info([] + {}, {} + []); // 多少自己慢慢可以分得清楚了
```

### 抽象关系比较
比较双方首先调用 ToPrimitive ，如果结果出现非字符串，就根据 ToNumber 规则将双方强制类型转换为数字来进行比较。
	如果比较双方都是字符串，则按字母顺序来进行比较

