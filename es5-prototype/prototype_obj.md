### 题外话
原型是JavaScript中非常重要的一环，几乎绝大部分面向对象编程精要都以其为蓝本进行深层次的规划。

# 原型
原型是一种机制，是一种属性查询机制，每个对象都有一个[[Prototype]]的内部属性，间接可以用__proto__属性体现 (不谈 _Object.create(null)_)，这个属性理论上都会被赋予一个非空的值，以此来构成原型链。对于以下代码：

```js
var obj = {
    key: 'value'
};
obj.key;
```

对于 __obj.key__ 来说，即[属性表达式](http://ecma-international.org/ecma-262/5.1/#sec-11.2.1)调用，获取对应的 _引用类型_，并执行 [[[GetValue]]](http://ecma-international.org/ecma-262/5.1/#sec-8.7.1) 操作，执行对象内部 [[[Get]]](http://ecma-international.org/ecma-262/5.1/#sec-8.12.3) 操作，判断是否存在于自身，如果没有，进而执行 [[[GetProperty]]](http://ecma-international.org/ecma-262/5.1/#sec-8.12.2) 递归寻找其原型，直至找到对应的属性 或 没有。

```js
var proto = {
    pKey: 'pVal'
};
var obj = Object.create(proto, {
    key: {
        value: 'value'
    }
});
console.info(obj);
// 我就不截图了， obj.__proto__ = proto = {pKey: 'pVal'}
```
JavaScript的原型链是有尽头的，尽头是 Object.prototype.\_\_proto__ = null，就像Java一样，类继承的源头就是Object [_我当初了解的是这样_]。

# 对象
对象是JavaScript中的基石，原型作为JavaScript中完成面向对象的重要理论，封装由闭包体现私有和保护；继承的功能由原型实质性承担；至于多态，JavaScript是弱类型的，本身就已经达到了多态的基本要求吧。

面向对象讲究将某个有共有属性的事物进行抽象，将公有的数据进行封装，并建立某个数据结构，已经在此结构上附加对应的数据操作，力求达到解耦和复用。

## 创建对象
```js
var obj = {};   // 直接量创建
function _create(p){    // 间接等同于Object.create
    // 这里不做任何必要的判断
    var f = function(){}
    f.prototype = p;
    return new f;
}
obj = new Object(); // 函数构造创建
```
## 属性的操作
通过对算符 [] 和 . 进行表达式的处理，ES5增加了对象属性的一些描述。

### 数据属性
我们经常在某个对象上定义的属性都是数据属性，基本由 value，enumerable、writable、configurable 四个特性组成
```js
var obj = {key: 'keyValue' };
console.info(Object.getOwnPropertyDescriptor(obj, 'key'));
// {value: "keyValue", writable: true, enumerable: true, configurable: true}
// 基本通过直接量定义的对应的值都是true
```

含义：
>value: 表示属性代表的值  
>enumerable: 表示是否可以进行枚举

```js
var obj = {key: 'keyValue' };
console.info('key' in obj, Object.keys(obj)); // true ["key"]

Object.defineProperty(obj, 'key', {
    enumerable: false
});
console.info(Object.keys(obj)); // []
for(var attr in obj){console.info(attr)}    // for...in也无法遍历到
```

>writable: 表示是否可以进行赋值操作

```js
var obj = {key: 'keyValue' };
Object.defineProperty(obj, 'key', {
   writable : false
});
obj.key = "newValue";
console.info(obj);  // 依旧是 keyValue, 注意在严格模式下，对只读属性重新赋值会报错
```

>configurable: 是否可以进行配置，该属性一旦设置为false，就无法在继续更改为true

```js
var obj = {key: 'keyValue' };
Object.defineProperty(obj, 'key', {
   configurable : false
});
obj.key = "newValue";
Object.defineProperty(obj, 'key', {  // 不能在重新定义其他特性，但是writable可以改为false
    writable: false
});
obj.key = "newValue2";

console.info(Object.getOwnPropertyDescriptor(obj, 'key'));
// 结果：{value: "newValue", writable: false, enumerable: true, configurable: false} 
```

### 访问器属性
除了enumerable、configurable两个特性一致之外，其他两个分别有 set 和 get 所替代

```js
var obj = {
    set key(val){},
    get key(){}
}
```

### ES5新增对象的扩展
以下操作不可逆，只会影响到自己，不会影响到原型，谨慎使用

Object.preventExtensions()/Object.isExtensible()

Object.seal()/Object.isSealed()	// 不能添加新的属性、已有属性不可删除或配置，已有可写属性可以设置

Object.freeze()/Object.isFrozen // 所有属性不可配置，所有数据属性设置为只读，如果有setter存储器，则不受影响[可以设值]


# 解释一下function
自定义的对象通过某个函数进行构造调用得到，函数对象的生成步骤就不说了(_相关可以查询数据类型中对function的说明_)，重述一下new运算符
## 重述一下new运算符 13.2.2[[Construct]]
语法：new NewExpression
1. 一些判断直接略过，比如表达式是不是函数，有没有[[Construct]]属性等
2. 执行[[Construct]]操作， 令 F 为表达式的构造函数
    * 创建一个新的ECMAScript原生对象 obj
    * 设置 obj.[[Class]] = "Object", obj[[Extensible]] = true;
    * 获取 proto = F.prototype, 设定 obj.[[prototype]] = proto, 即obj.\_\_proto__ = proto;
    * 以 obj 为this值，调用 F ， result = F.call(this, args);
    * 如果result是对象类型，返回result，否则返回obj

## function.prototype

```js
function Class(){}
var obj = new Class();

Class.prototype.sayHello = function(){
    console.info('我是新增在原型上的 sayHello方法')
};

obj.sayHello();
```

通过对new操作的概述，实际上当构造一个对象的时候，对象的内部属性[[prototype]]会与函数的prototype对象进行关联，进而由于原型查找机制，完成 "类" 上面添加属性，构造对象可以访问的现象。

__对象只与构造函数对象原型有关，与构造函数本身没有关联__


# 最后放一张图吧
![JS原型](./img/js_pro.jpg)

