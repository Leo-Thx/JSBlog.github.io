# 概念介绍
## 可执行代码
ECMAScript中有三种可执行的代码：
1. 全局代码
2. Eval执行的代码块: eval函数参数接收到的程序文本
3. 函数代码：包含在函数体中的程序文本

## 词法环境
词法环境是一个用于定义 __特定变量__ 和 __函数标识符__ 在ECMAScript __词法嵌套结构上__ _关联关系_ 的规范类型。一般由一个 __环境记录项__ 和 一个可能为空的 __外部词法环境引用__ 组成。

>环境记录项：用来记录当前词法环境中创建的标识符绑定。包括 标识符声明、参数声明等等  
>外部词法环境引用：用来表示词法环境的嵌套关系 __(关系到作用域链的形成)__

环境记录项又可以细分两个：

>__声明式环境记录项__：定义那些将标识符与语言值直接绑定的 ECMA 脚本语法元素 - 用来绑定作用域内的一些标识符，比如变量声明、函数声明、catch等。  
>__对象式环境记录项__：定义那些将标识符与具体对象的属性绑定的 ECMA 脚本元素，比如with语句。  
    每个对象式环境记录项都会有一个对应的绑定对象，该记录项中的一系列标识符会与对象式环境记录项的属性建立对应关系。__将其绑定对象合为函数调用时的隐式 this 对象的值__。

### 词法环境运算
搞清楚以下两个就可以：
1. 获取标识符的引用类型：GetIdentifierReference(lex, name, strict)
    >参数lex：词法环境，name：标识符，strict：严格模式  
    >1. 如果lex为null，返回基值为**undefined**的引用类型
    >2. 令 envRec = lex.环境记录项  
    >3. 判断envRec中是否有name的标识符绑定  
    >4. 如果存在该标识符的绑定，返回一个引用类型的对象，envRec为基值  
        >_引用类型在数据类型中有说明_
    >5. 如果不存在  
        >5.1 outer = lex.外部词法环境引用  
        >5.2 以outer,name,strict调用GetIdentifierReference

2. 创建新的词法环境
    * 创建声明式词法环境 NewDeclarativeEnvironment(E)
        >1. env = 新建一个词法环境
        >2. envRec = 新建一个声明式词法环境记录项
        >3. env.[环境记录项] = envRec, 且不包含任何标识符绑定;
        >4. env.[外部词法环境引用] = E (不一定是赋值，这里说明一下);
        >5. 返回env
    * 创建对象式词法环境 NewObjectEnvironment(O, E)
        >1. env = 新建一个词法环境
        >2. envRec = 新建一个声明式词法环境记录项
        >3. env.[环境记录项] = envRec, 设置O为环境数据中的 **绑定对象**;
        >4. env.[外部词法环境引用] = E (不一定是赋值，这里说明一下);
        >5. 返回env

        ```js
        // 伪代码演示一下
        function LexicalEnvironments(){}    // 词法环境
        function EnvironmentRecord(){}  // 环境数据

        function NewDeclarativeEnvironment( E ){
            var env = new LexicalEnvironments;
            var envRec = new EnvironmentRecord;
            env.environmentRecord = envRec;
            env.outerLexicalEnvironment = E;
            return env;
        }

        function NewObjectEnvironment(O, E){
            var env = new LexicalEnvironments;
            var envRec = new EnvironmentRecord;
            env.environmentRecord = envRec;
            env.bindObject = O;
            env.provideThis = true;
            env.outerLexicalEnvironment = E;
            return env;
        }
        ```

## 执行环境
解释执行全局代码、使用 eval 函数输入的代码、调用ECMA创建的函数 会创建并进入一个新的执行环境。控制流进入到执行环境时，会创建 __this绑定__，定义变量环境和初始化词法环境，执行 [_声明式绑定初始化_](http://ecma-international.org/ecma-262/5.1/#sec-10.5)。

### 组成
>1. 词法环境组件: 用来解析该执行环境内的创建的标识符引用
>2. 变量环境组件: 用来保存当前执行环境中创建的标识绑定，包括变量声明、函数声明
>3. this绑定组件: 指定该执行环境中代码执行时 this 所关联的值


### 全局环境
全局环境是一个唯一的词法环境，在任何ECMA代码执行前创建它，环境数据是对象式的声明记录项，外部词法环境引用为null，全局对象作为当前执行环境的绑定对象。

### eval执行环境
1. 如果是非严格模式代码：
>1. 将 __词法环境组件__ 设置为当前执行环境下的 _词法环境组件_
>2. 将 __变量环境组件__ 设置为当前执行环境下的 _变量环境组件_
>3. 将 __this绑定__ 设置为当前执行环境下的 _this绑定组件_
2. 如果是严格模式下：
>1. 调用NewDeclarativeEnvironment 新建一个词法环境strictVarEnv
>2. 将 __词法环境组件__ 设置为当前执行环境下的 _strictVarEnv_
>3. 将 __变量环境组件__ 设置为当前执行环境下的 _strictVarEnv_
3. 使用eval代码执行 [_声明式绑定初始化_](http://ecma-international.org/ecma-262/5.1/#sec-10.5)。

```js
// 有什么区别呢
(function(){
    'use strict';
    eval('var a = 1;'); // 即在严格模式下，绑定和初始化都在一个新的词法环境中进行
    console.info(typeof a); // undefined
})();

(function(){
    eval('var a = 1;');
    console.info(typeof a); // number
})();
```

### 函数执行环境
1. 如果函数代码是在严格模式下，如果调用的this为null或undefined，设置为全局对象
2. 以函数[[Scope]]内部属性为参数调用NewDeclarativeEnvironment，得到结果localEnv
3. 设置词法环境组件为 localEnv
4. 设置变量环境组件为 localEnv
5. 执行函数代码

_如果不知道[[Scope]]是啥，请看数据类型中函数对象创建过程的简要说明_

# ES5常见的一些作用域
简单理解就是可以存储变量的值，并且能在之后对该值进行访问和修改。

## 全局作用域
全局作用域是唯一的、也是最外层的，在浏览器中，除了提供内置原生对象以外，还会提供宿主对象对全局环境进行初始化。

## eval作用域
参考Eval执行环境的运算，注意严格模式下，变量及函数绑定将在一个新的 变量环境组件 中被初始化，该 变量环境组件 仅可被 eval 代码访问。

_也就是可以动态延长作用域，改变当前词法环境组件对变量的解析_


## with作用域
当执行with语句时，以当前词法环境调用NewObjectEnvironment获得新的词法环境，并以该词法环境执行with代码，严格模式下是不能用的，导致这个语句用的及其的少的原因吧。
>注意点：  
    >1. 严格模式不能使用
    >2. 只读属性不能更改
    >3. 原型上访问器属性不能更改, 普通属性会在本对象声明
```js
var proto = { 
    get superKey(){ return 'superValue' },
    set superKey(val){
        console.info('set superKey: ', val);
    },
    superKey2 : 'superValue2'
};
var obj = Object.create(proto);
obj.key = '123';
Object.defineProperty(obj, 'age', {
    value: 12
});
with(obj){
    key = 1;
    value = "ageString";
    superKey = "with set";
    superKey2 = "withSetSuperKey2";
}
console.info(obj, obj.hasOwnProperty('superKey'));
// obj: {key, value, superKey2} false
```

## 块级作用域catch
是的，你没有看错，ES5是存在块级作用域的
官方规范如下：
>产生式 Catch : catch ( Identifier ) Block 按照下面的过程执行 :
>1. 令 C 为传给这个产生式的参数 .
>2. 令 oldEnv 为运行中执行环境的 LexicalEnvironment.
>3. 令 catchEnv 为以 oldEnv 为参数调用 NewDeclarativeEnvironment 的结果
>4. 以 Identifier 字符串值为参数调用 catchEnv 的 CreateMutableBinding 具体方法。
>5. 以 Identifier, C, false 为参数调用 catchEnv 的 SetMutableBinding 具体方法。注：这种情况下最后一个参数无关紧要。
>5. 设定运行中执行环境的 LexicalEnvironment 为 catchEnv.
>6. 令 B 为解释执行 Block 的结果 .
>7. 设定运行中执行环境的 LexicalEnvironment 为 oldEnv.
>8. 返回 B.

## 对 new Function 的说明
就记住一点：Scope为全局环境，即外部词法引用为全局环境


# 作用域链和自由变量寻址
相信上面这些东西弄明白了，作用域链和自由变量查询也就很简单了。
作用域链的构成其实本质还是由外部词法环境引用构成。

自由变量寻址也就是从当前词法环境组件解析开始，一层一层的往外查询，关键是理解函数对象创建时[[Scope]]的处理。

# 再谈闭包
到2019年，我也从事两年前端了，其他的不知道，但是闭包的问题屡见不鲜。

高级程序设计定义：闭包是定义在一个外部函数内部，并且能够访问（存取）外部函数中自由变量的函数。

《You Don't Know JS: Types & Grammar》上卷：当函数可以记住并访问所在的词法作用域时，就产生了闭包，即使函数是在当前词法作用域之外执行。

_抱歉这本书名我忘记了，但是很感谢作者_
>闭包是一个运行期的概念，有以下特征：
>1. 闭包作为与函数成对的数据，在执行过程中处于激活状态
>2. 闭包在函数在结束后，保持运行过程中的最终数据状态

---
个人觉得比较有用的部分，浅薄的见解
>1. 保护数据私有安全
>2. 防止作用域污染
>3. 部分高阶编程的基础
