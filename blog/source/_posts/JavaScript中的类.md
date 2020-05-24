---
title: JavaScript中的类
date: 2020-04-08 19:24:00
tags: JavaScript
categories: JavaScript
---

# JavaScript中的类

在class概念引入之前，js通过原型对象来实现类和类的继承，在ECMAScript 6出现class的概念之后，才算是告别了直接通过原型对象来模拟类和类继承，但class也只是基于JavaScript原型继承的语法糖，并没有引入新的对象继承模式，所以理解原型以及原型继承是非常重要的。通过class来创建对象，可以让代码更为简洁，复用性更高。

### 类声明

ECMAScript 6中定义一个类非常的简单，看下面的代码

```javascript
class Animal {
  constructor(name) {
  	this.name = name;
  }
  sayHi() {
    console.log(`hello,${this.name}`);
  }
}
Animal.prototype.constructor === Animal; // true
let dog = new Animal('dog');
dog.sayHi(); // hello, dog
```

constructor方法用来创建和初始化对象，而且一个类中有且只能有一个consctuctor方法，默认为constructor(){}。dog就是Animal实例化的对象。

### ES5中创建类

在前面说到，class只是基于现有的JavaSript实现类的语法糖，那先让我们简单使用ES5中的方法来模拟类，并实例化。

```javascript
function Animal(name) {
  this.name = name;
}
Animal.prototype.sayHi = function() {
  console.log(`hello,${this.name}`);
}
let dog = new Animal('dog');
dog.sayHi(); // hello, dog
```

可以看出，class中的constructor()方法就相当于Animal()构造函数，而在class中定义属性就相当于直接在原型对象上定义属性。我们不妨这样试一试：

``` javascript
class Animal {
  constructor(name) {
    this.name = name;
  }
  sayHi() {
    console.log(`hello, ${this.name}`);
  }
}
let dog = new Animal('dog');
dog.sayHi(); // hello, dog

dog.__proto__ === Animal.prototype; // true

dog.__proto__.sayHi = function() {
  console.log(`hi, ${this.name}`);
}
dog.sayHi(); // hi, dog
```

可以看出class还是依靠原型对象来实现类的。

###　Class的继承

class使用extends关键字来创建子类

```javascript
class Animal {
  constructor(name) {
    this.name = name;
  }
  sayHi() {
    console.log(`hello, ${this.name}`);
  }
}

class Dog extends Animal {
  bark() {
    console.log(`喵喵喵`);
  }
}

let wangcai = new Dog('旺财');
wangcai.bark(); // 喵喵喵
wangcai.sayHi(); // hello, 旺财
```

但如果在子类中定义了constructor方法，必须先调用super()才能使用this，因为子类并没有this对象，而是继承父类的this对象，所以super必须在使用this关键字之前使用：

```javascript
class Animal {
  constructor(name) {
    this.name = name;
  }
  sayHi() {
    console.log(`hello, ${this.name}`);
  }
}

class Dog extends Animal {
  constructor(name, sound) {
  	this.name = name;
  	this.sound = sound;
  };
  bark() {
    console.log(this.sound);
  }
}

let wangcai = new Dog('旺财', '喵喵喵');
wangcai.bark(); // referenceError

class Dog extends Animal {
  constructor(name, sound) {
  	super(name);
  	this.sound = sound;
  };
  bark() {
    console.log(this.sound);
  }
}
let wangcai = new Dog('旺财', '喵喵喵');
wangcai.bark(); // 喵喵喵
```

super不仅可以调用父类的constructor函数，还可以调用父类上的方法：

```javascript
class Animal {
  constructor(name) {
    this.name = name;
  }
  sayHi() {
    console.log(`hello, ${this.name}`);
  }
}

class Dog extends Animal {
  bark() {
    super.sayHi();
  }
}

let wangcai = new Dog('旺财');
wangcai.bark(); // hello, 旺财
```

### 静态方法

所有在类中定义的方法会被实例继承，但是有时候我们并不想所有实例都能继承某个方法，这时候，static关键字就能达到你的目的，在声明方法前加上static关键字，这个方法就不会被实例继承，而是直接通过类来调用，它被叫做静态方法，如下：

```javascript
class Animal {
  constructor(name) {
    this.name = name;
  }
  sayHi() {
    console.log(`hello, ${this.name}`);
  }
  static bark() {
  	console.log('喵喵喵');
  }
}
let dog = new Animal('dog');

dog.bark(); // TypeError
Animal.bark(); // 喵喵喵
```

静态方法虽然不能被实例继承，但是可以被子类继承，但是子类的实例依旧没有继承它：

```javascript
class Animal {
  static bark() {
  	console.log('喵喵喵');
  }
}
class Dog extends Animal{
    
}
Dog.bark(); // 喵喵喵
let dog = new Dog();
dog.bark(); // TypeError
```

### 实例属性

实例属性必须定义在类方法中：

```javascript
class Animal {
  constructor(name) {
    this.name = name; // 实例属性
  }
}
```

静态属性和原型属性避必须在类的外面定义：

```javascript
Animal.age = 18; // “静态属性”
Animal.prototype.sex = 'male'; // 原型属性
```

实例属性顾名思义，就是对象独有的方法/属性，静态属性就是位于Class本身的属性，但是ES6中明确说明Class只有静态方法，没有静态属性，原型属性也很容易理解也很容易看出，就是位于原型链上的属性/方法。

参考资料

- [MDN Classes](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Classes)
