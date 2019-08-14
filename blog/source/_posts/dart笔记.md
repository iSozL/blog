---
title: dart笔记
date: 2019-07-16 14:06:46
tags: 笔记|dart
categories: 笔记|dart
---
dart程序的入口
{% codeblock %}
main() {
  ...;
}
void main() {
  ...; // 没有返回值 
}
{% endcodeblock %}
注释方式和JavaScript类似

---
定义变量
---

可以指定类型也可以不指定
{% codeblock %}
var string = 'myName';
{% endcodeblock %}
dart自动推断类型和JavaScript相似，可以定义任意类型，与JavaScript不同的是，dart有类型校验
{% codeblock %}
String name = 'liHua';
{% endcodeblock %}
也可以指定变量的类型

---
定义常量
---

dart有两个关键字来定义常量，一个是const，一个是final，前者在js中就已经接触到很多了，它在一开始就必须为定义的常量赋值，而final比const强大一些，它不需要在定义常量的时候就为它赋值，但是只能赋值一次，不然也不叫常量了，final与const最大的区别在于，final是惰性初始化的，即在第一次使用前才初始化，来看下面一段代码
{% codeblock %}
main() {
  const a = new DateTime.now(); // error
  final b = new DateTime.now(); // correct
}
{% endcodeblock %}
这就体现了const与final的最大不同处

---
String类型
---

定义多行字符串
{% codeblock %}
String myname = 
 '''
   hello
   welcome to BeiJing
 ''';
{% endcodeblock %}
用三对单引号或者三对双引号

---
字符串拼接
{% codeblock %}
main() {
  String str1 = '你好';
  String str2 = 'dart';
  print("$str1,$str2"); // 你好，dart
}
{% endcodeblock %}
或者
{% codeblock %}
main() {
  String str1 = '你好';
  String str2 = 'dart';
  print(str1 + "," + $str2); // 你好，dart
}
{% endcodeblock %}

---
数值类型
---
int: 必须是整型
double: 既可以是整型也可以是浮点型

---
Boolean
---

值为true或者false
{% codeblock %}
bool flag = true;
{% endcodeblock %}

---
集合类型
---
定义list的方法
1.
{% codeblock %}
var l1 = ['张三', '李四', '王五'];
{% endcodeblock %}
2.
{% codeblock %}
var l1 = new List();
  l1.add('张三');
  l1.add('李四');
  l1.add('王五');
{% endcodeblock %}
3.泛型
{% codeblock %}
var l3 = new List<String>();
  l3.add('张三');
  ...
{% endcodeblock %}

---
Maps型
---

定义maps型和访问，在dart中只能通过Person['age']的形式访问maps类型，不可以像js那样通过Person.age的形式访问
{% codeblock %}
void main() {
  var Person = {
      "name": "张三",
      "age": 20
    };
    var Person2 = new Map();
    Person2["name"] = "李四";
    print(Person["age"]); // 20
    print(Person2); // {name: 李四}
}
{% endcodeblock %}

---
类型判断
---
is 用来判断类型
{% codeblock %}
void main() {
  var name = '李华';
  if(name is String) {
    print("name是String类型,$name");
  } else {
    print("name不是String类型");
  }
} // name是String类型, 李华
{% endcodeblock %}

---
List常用属性和方法
---
属性：
length // 长度
reversed // 翻转
isEmpty // 是否为空
isNotEmpty // 是否不为空
方法：
add // 添加
addAll // 拼接数组
indexOf // 查找 输入具体值
remove // 删除 输入具体值
removeAt // 删除 输入索引值
fillRange // 修改
join // 把list转化为字符串

---
dart中的类
---
使用类的属性和方法必须先实例化
{% codeblock %}
class Person {
  String name = "张三";
  int age = 18;
  void printInfo() {
    print("name: $name, age: $age");
  }
}
void main() {
  // 实例化
  var person = new Person();
  person.printInfo();
}
{% endcodeblock %}

dart类中构造函数
{% codeblock %}
class Person {
  String name = "张三";
  int age = 18;
  // 构造函数
  Person() {
    print("这是构造函数，在实例化的时候触发");
  }
  void printInfo() {
    print("name: $name, age: $age");
  }
  void changeInfo(int age) {
    this.age = age;
  }
}
void main() {
  var person = new Person();
  person.changeInfo(19);
  person.printInfo();
}
{% endcodeblock %}
命名构造函数
{% codeblock %}
class Person {
  String name;
  int age;
  Person(this.name, this.age) {
    print("$name, $age");
  }
  Person.printInfo() {
    print("我是命名构造函数");
  }
}
void main() {
  var person = new Person("张三", 17);
  var named = new Person.printInfo();
}
// 张三, 17
// 我是命名构造函数
{% endcodeblock %}
dart类中的静态方法和属性，非静态方法可以访问静态成员和非静态成员，静态属性不能访问非静态成员
{% codeblock %}
class Person {
  static String name = "张三";
  int age = 19;
  static void getInfo() {
    print("静态方法");
  }
}
void main() {
  print(Person.name); // 张三
  Person.getInfo(); // 我是静态方法
}
{% endcodeblock %}
dart中的对象运算符
判断：?
{% codeblock %}
class Person {
  String name = "张三";
  int age = 19;
  getInfo() {
    print(["$name---$age"]);
  }
}
void main() {
  Person p;
  p?.getInfo(); // 判断p对象是否存在---空
  Person per = new Person();
  per?.getInfo(); // 张三---19
}
{% endcodeblock %}
判断类型：is
print(per is Person); // true
级联操作：..
Person p = new Person();
p..name = "李四"
 ..age = 20
 ..getInfo();
等同于
Person p = new Person();
p.name = "李四";
p.age = 20;
p.getInfo();

