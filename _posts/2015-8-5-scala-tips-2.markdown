---
layout: post
title: Scala tips 2
category: scala
---
### Scala 笔记2

**class**
一个简单的scala 类声明

```scala
class Point(xc: Int, yc: Int){
var x: Int = xc
var y: Int = yc 

def move(dx: Int, dy:Int){
 x = x + dx
 y = y + dy
}
}
```


Point 类有一个构造器，可以通过 `new Point(1,2)` 来创建一个 Point 实例。
在声明构造器的时候如果使用了 `val` 或者`var`，会让类自动生成 setter 和getter.
具体来讲，如果 声明

```scala
class Person(val name: String)
val jude = new Person("jude") 
jude.name should be "jude"
```

如果 声明

```scala
class Person(var name: String)
val per = new Person("Juan")
per.name should be "Juan"
per.name = "jude"
per.name should be "jude"
```

**伴生对象**
 如果一个单例对象(Object) 和一样 Class 同名，那么它就是这个Class的 伴生对象。伴生对象的功能很多，可以用来实现工厂方法。

伴生对象的方法，类似于java 的static method。Class的实例，可以访问 Object 中 的成员变量，而且所有实例共享一个成员变量。实例也可以调用伴生对象的成员方法。
反过来，伴生对象的方法可以访问实例的私有变量。





**构造器函数**
构造器函数可以没有明确的声明,` class 类 { ... } `花括号里面的内容都属于构造函数. 构造器函数也可以接收参数,参数列表在类名字后面.

**trait**
类似于 Java的 interface . 声明一个 trait

```scala
trait Haha {
val say :String
val speak = "Too young Too simple"
}
```

声明一个类继承它 

```scala 
class President extends Haha {
val say = { () => println(" Some Time naive !")
}
val hu = new Prsident
hu.say() should print " Some Time naive !"    
```

如果 注释掉 President 内部 实现的 方法`say` , 类声明就会失败.
trait 提供了一种强制约束, 继承了trait的类必须 实现 trait 里面没有实现或者赋值的 方法和对象. 
而 `Haha` 里面实现了speak 成员,所以不需要在 类声明里给他赋值.

trait 的一个 局限是它的构造器不能携带参数列表. 如果你想继承构造器的参数, 你需要继承一个虚类而不是 trait.