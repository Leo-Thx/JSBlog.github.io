_本篇主要对函数知识做一些补充_

为什么对function要单独拿出来说一下，因为在JS里面，函数是可能被忽视较多的地方

1. 函数是做面向对象程序设计的基石
2. 函数是作用域的主要体现
3. 是函数式编程的基础

_所以搞清楚函数的来源和部分应用能让自己对js理解的更加通透。_

## 函数来源、与Object的关系
一个新的对象 Function (_这里F是大写的_)，个人觉得Function可能是JS中最核心的一个对象。

### Function.prototype
Function也是一个函数，一个用来生成函数的函数，参数列表中只有最后一个参数是函数体，前面的都是参数
```js
var say = new Function('name', 'console.info(name, "world")');   
say('hello');    // hello, world
//一般加个new，不加也是可以的: Function('name', 'console.info(name, "world")')('hello')
```
既然是函数，就会有prototype属性，表示该函数关联的原型对象，_该属性是一个函数_，不像其他一样是一个对象。
该属性上定义着普通function可以直接使用的函数，比如：call、apply
```js
console.info( typeof Function.prototype );  // function
console.info( typeof Function.prototype.call );

Function.prototype.say = function(){
    console.info('Function.prototype.say...');
};
function fn(){}
fn.say();   // Function.prototype.say...
```

### Function 与 function
_上面说到了，Function.prototype上定义的操作，在普通函数上就能直接使用，那么到底和function是什么关系呢？_

__function 其实是 Function 的实例对象__
```js
function fn(){}
console.info( fn.__proto__ === Function.prototype );    // true
console.info( Function.prototype.constructor );         // Function
console.info( Function.__proto__ === Function.prototype );  // true
```
\_\_proto__在ES5中是一个隐藏的内部属性[当前对象的原型引用指向]，但在ES6中已经规范化了，fn的__proto__指向Function.prototype，故apply、call、bind方法等在普通函数上都能直接使用。

Function.__proto__是指向它自己的原型，给我的理解即 Function 也是由自己构造出来的(这里没有查阅相关文档)
```
Function.protoytpe.constructor => Function;
Function.__proto__ => Function.prototype;
```


### Function 与 Object
Object是所有字面量对象的模版，为JS对象重要的组成部分。而重要的是Object也是 ___函数___，那么Object是来自哪里呢，由于对象是一系列键值对的组合而成的字典，那么Object上也有对应的属性，所以我认为从这个角度来看，函数也是对象。比如：Object.defineProperty, Object.keys ...，当然我们自身也可以在上面添加自己的定义的属性和方法。
```js
console.info( Object.__proto__ === Function.prototyp ); // true
console.info( Function.prototype.__proto__ === Object.prototype );  // true
```
可以看出，Object也是由Function构造出来的，Function的原型关联着Object的原型，所以Object.prototype上定义的toString、valueOf等方法在Function.prototype上也能使用。所以，我认为Function是js中最重要的一个对象。

## [函数定义](http://www.ecma-international.org/ecma-262/5.1/index.html#sec-13)
_说一下函数的定义过程_

形式：
1. 函数声明：function Identifier(FormalParameterList){FunctionBody}
2. 匿名函数表达式：function(FormalParameterList){FunctionBody}
3. 命名函数表达式：function Identifier(FormalParameterList){FunctionBody}

下列只列举了一些关键步骤：
指定 FormalParameterList 为可选的 参数列表，指定 FunctionBody 为 函数体，指定 Scope 为词法环境，Strict 为布尔标记，按照如下步骤构建函数对象：

1. 创建一个新的 ECMAScript 原生对象，令 F 为此对象。
2. 设定 F 的 [[Class]] 内部属性为 "Function"。
3. 设定 F 的 [[Prototype]] 内部属性为标准内置 Function 对象的 prototype 属性。 F.\_\_proto__ = Function.protoytpe;
4. 依照 13.2.2 描述，设定 F 的 [[Construct]] 和 [[HasInstance]] 内部属性。
5. 设定 F 的 [[Scope]] 内部属性为 Scope 的值。(Scope是形成作用域链的基础)。
6. 处理参数，设置length, arguments等属性
7. 令*proto*为**new Object**调用结果，处理constructor、prototype属性
    - 以*constructor*为名字调用**proto**上**DefinePrototype**内部方法 value:F  
    - 以*prototype*为名字调用**F**上**DefinePrototype**内部方法 value:proto
8. 返回 F。

_注：每个函数都会自动创建一个 prototype 属性，以满足函数会被当作构造器的可能性。_
___命名函数表达式，只是在第一步之前新建一个词法环境用来存储函数名称___
![初始化草图](./image/data_type.png)


## 函数与作用域
作用域定义为在只某个范围内访问某个变量。ES5中块级作用域只有catch，但也是运行时层面的处理。但是函数可以构建新的作用域，实现私有数据的封装和访问控制。
```js
var _num = 1;
var someFunction = function(){
    _num = 2;       // 这里可以访问到，原因与 function.[[Scope]]有关，下篇文章会有相应的说明
    var _num2 = 3;  // 
};
someFunction();
_num;   // 2
// _num2;  // error 准确来说是 右值 导致的错误


if( condition ) {
    (function(){
        // 这里可以模拟块级作用域
    }());
}
```
此时_num2被限制在someFunction内部，外部无法访问，也就达到了数据的私有操作。


## 函数式的部分说明
- 函数是第一型的
    >第一型，表示与其他基本类型是等同的，是一个值，既可以参与预算，也能做参数传递
- 函数内部可以保存数据
    ```js
    var set, get;
    var someFn = function(arg){
        set = function( v ){
            arg = v;
        };
        get = function() {
            return arg;
        };
    }
    someFn();
    // set
    // get
    ```
- 函数对外无副作用
    >函数的内部的任何操作不影响外部状态，即不改变外部的任何职，只能通过参数承担



<!-- Function 的实例的属性
 除了必要的内部属性之外，每个函数实例还有一个 [[Call]] 内部属性并且在大多数情况下使用不同版本的 [[Get]] 内部属性。函数实例根据怎样创建的（见 8.6.2 ,13.2, 15, 15.3.4.5）可能还有一个 [[HasInstance]] 内部属性 , 一个 [[Scope]] 内部属性 , 一个 [[Construct]] 内部属性 , 一个 [[FormalParameters]] 内部属性 , 一个 [[Code]] 内部属性 , 一个 [[TargetFunction]] 内部属性 , 一个 [[BoundThis]] 内部属性 , 一个 [[BoundArgs]] 内部属性。

 [[Class]] 内部属性的值是 "Function"。 -->


 参考资料：  
    《JAVASCRIPT 语言精髓与编程实践》  
    《JavaScript 高级程序设计》  
    《JavaScript 权威指南》 
    《ES5文档》(http://www.ecma-international.org/ecma-262/5.1/index.html)