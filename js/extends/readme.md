# 组合寄生式继承

```js
/**
 * 1. 将原型指向了父类的副本
 * 2. 构造器指向了自身
 */
function inheritPrototype(subType, superType){
    var protoType = Object.create(superType.prototype);    // 创建对象
    protoType.constructor = subType;                       // 增强对象
    subType.prototype = protoType;                         // 指定对象
}
function SuperType(name){
    this.name = name;
    this.colors = ["red", "blue", "green"];
}
SuperType.prototype.sayName = function(){
    alert(this.name);
}

function SubType(name, age){
    SuperType.call(this, name);　　
    
    this.age = age;
}
inheritPrototype(SubType, SuperType)
SubType.prototype.sayAge = function(){
    alert(this.age);
}

var instance = new SubType("Bob", 18);
instance.sayName();
instance.sayAge();
```

`inheritPrototype` 函数接收两个参数：子类型构造函数和超类型构造函数。
1. 创建超类型原型的副本。
2. 为创建的副本添加constructor属性，弥补因重写原型而失去的默认的 `constructor` 属性
3. 将新创建的对象（即副本）赋值给子类型的原型
这种方法只调用了一次 `SuperType` 构造函数，`instanceof` 和 `isPrototypeOf()` 也能正常使用。

