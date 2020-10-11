# constructor

```ecmascript 6
class Animal {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }
  say(words) {
    console.log(words);
  }
}

class Dog extends Animal {
  
}

class Cat extends Animal {
  // 子类继承父类后，那么，在子类的`constructor`里必须在第一行使用`super`
  constructor() {
    super();
  }
}

class Bird extends Animal {
  // 构造器传入`name`和`age`后，this.name和this.age一般不为undefined
  constructor(name, age) {
    super(name, age);
  }
}
```
