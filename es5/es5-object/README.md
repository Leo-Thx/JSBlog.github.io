_本篇对面向对象做一部分知识补充，其他可以参照以下提供的参考资料_

## 描述
___类/继承描述了一种代码的组织结构形式--一种在软件中对真实世界中问题领域的建模方法。
面向对象编程强调的是数据和操作数据的行为本质上是相互关联的，因此好的设计就是把数据以及和它相关的行为打包(或者说是封装)起来.___

_个人简解：面向对象将某个有共有属性的事物进行抽象，将公有的数据进行封装，并建立某个数据结构，已经在此结构上附加对应的数据操作，力求达到解耦和复用。_

- 封装
> Java语言有对应的关键字，比如：privated、protected等关键字进行修饰，以限定属性或行为的使用范围，是由语法层面处理的。但是JavaScript中只能通过 _变量作用域_ 实现。
- 继承
> 常见面向对象语言上，都有类似 extends 的关键字完成抽象类型到具体类型的派生，但至少在ES6的语法糖出现之前，通过 _原型委托_ 实现。
- 多态
> 由于JavaScript是弱类型的语言，类型的处理只能在运行时体现出来，所以没有类似Java一样，强制对某个对象设计只属于它的可操作类。JavaScript是否有某个操作只取决于它有没有对应的操作，并不取决于它是否是某个类型，我觉得JavaScript本就是多态的。

## 普通创建形式
```js
function Person( name, age ){
    this.name = name;
    this.age = age;
    this.sayName = function(){
        console.info('hello，我是小米');
    };
}
var p = new Person('小米', 20);
p.sayName();
```

## 原型继承
_原型继承也只是利用原型链的查找_
```js
function Person(){}
Object.assign(Person.prototype, {
    name: '小米',
    age: 20,
    sayName: function(){
        console.info('hello，我是', this.name);
    }
});

function Student(){}
Student.prototype = new Person();

var st = new Student;
st.sayName();
```
某个函数prototype上定义的属性都是共享的，通过对该函数进行构造调用，得的对象都会有一个属性关联到该函数的原型对象上，属性的查找也就是原型链的遍历。

_📍 单纯使用这种，导致在创建对象时，无法向超类构造函数调用时传递参数。_

## 借用构造函数
_通过call和apply的方式间接的调用函数_
```js
function SuperType(name){
    this.colors = ['red', 'green'];
    this.name = name;
    // this.sayHello = function(){};
}
function SubType( name ){
    SuperType.call(this, name);
}

var obj = new SubType('subType');
```
_📍 通过在构造函数中，手动调用超类型的构造函数，并绑定this为当前构造实例。但单独使用，无法复用在超类型上定义的各种函数。_

## 组合继承
_组合继承就是将二者结合起来_
```js
function SuperType(name){
    this.name = name;
}
SuperType.prototype.sayName = function(){
    console.info(this.name);
}

function SubType(name, age){
    SuperType.call(this, name);
    this.age = age;
}
SubType.prototype = new SuperType();
SubType.prototype.sayAge = function(){
    console.info(this.age);
};

```
_📍 在继承的时候，调用了两次构造函数，如果超类消耗比较大，那么可能会造成额外的浪费。_

## 原型式继承
_通过中间对象进行间接引用_
```js
function inherit( o ){
    function F(){}
    F.prototype = o;
    return new F;
}
var stu = inherit({
    name: '',
    colors: []
});
```
_以某个对象为基础，进行原型上的关联，但这也意味着 colors 属性会被共享_

## 寄生组合继承
```js
function inherit(Sub, Super){
    function F(){}
    F.prototype = Super.prototype;
    Sub.prototype = new F;
    Sub.prototype.constructor = Sub;
    Sub.prototype.Super = Super;
}

function SuperType(name){
    this.name = name;
}
SuperType.prototype.sayName = function(){
    console.info(this.name);
}

function SubType(name, age){
    // SuperType.call(this, name);
    this.Super.call(this, name);
    this.age = age;
}

inherit(SubType, SuperType);
/*
1. 令新的函数 F 的原型对象指向Super ；F.prototype = Super.prototype;
2. Sub.prototype = new F;   即生成F的实例，但是F实例的原型又关联着Super的原型，故Sub的原型通过 F的实例，和 Super的实例进行关联
    Sub.prototye -> F[instance].__proto__ -> F.prototype -> Super.prototype
3. 最后重置Sub原型上的constructor属性为自己
*/

SubType.prototype.sayAge = function(){
    console.info(this.age);
};
```
EXT中Base.js也有类似的操作:
```js
extend: function(parent) {
    var parentPrototype = parent.prototype,	//
        basePrototype, prototype, i, ln, name, statics;

    prototype = this.prototype = Ext.Object.chain(parentPrototype);	// this.prototype > obj -> parent.prototype
    prototype.self = this;	// 保留当前的函数

    this.superclass = prototype.superclass = parentPrototype;	// 当期构造器和原型中都存入父类的原型

    if (!parent.$isClass) {	// 如果不是Class的类型
        basePrototype = Ext.Base.prototype;	// 
        // 将Base中的原型 拷贝到当前的原型里面
        for (i in basePrototype) {
            if (i in prototype) {
                prototype[i] = basePrototype[i];
            }
        }
    }
}
```

_📑可以参考《JavaScript 框架设计》第五章类工厂部分有更多详细的资料_


参考资料：  
    《You Don't Know JS: Types & Grammar》上卷  
    《JAVASCRIPT 语言精髓与编程实践》  
    《JavaScript 高级程序设计》  
    《JavaScript 权威指南》  
    《JavaScript 面向对象编程指南》