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



15.3.5
Function 的实例的属性
 除了必要的内部属性之外，每个函数实例还有一个 [[Call]] 内部属性并且在大多数情况下使用不同版本的 [[Get]] 内部属性。函数实例根据怎样创建的（见 8.6.2 ,13.2, 15, 15.3.4.5）可能还有一个 [[HasInstance]] 内部属性 , 一个 [[Scope]] 内部属性 , 一个 [[Construct]] 内部属性 , 一个 [[FormalParameters]] 内部属性 , 一个 [[Code]] 内部属性 , 一个 [[TargetFunction]] 内部属性 , 一个 [[BoundThis]] 内部属性 , 一个 [[BoundArgs]] 内部属性。

 [[Class]] 内部属性的值是 "Function"。