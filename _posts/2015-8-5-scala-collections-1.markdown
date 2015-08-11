---
layout: post
title: Scala tuple & case class
category: scala
--- 
### Scala 笔记3 tuple & case class

**tuple**:穷人的`case class`

Tuple 对于 scala 和python 来说是截然不同的概念.
python的tuple 就像 immutable list, 除了`immutable`之外和list 用法差不多.

而对于Scala, `tuple`是很不一样的东西,它不是固化的列表, 更像是穷人的`case class`.
Scala的 List 的所有元素,必须是同一类型, 或者 接口, 或者有共同的基类/虚类.

> 修正: 类型为 List[Any] 的列表,可以包含多种类型并且没有继承关系的元素. 

而 tuple 里面的元素可以属于不同的类型, 不必继承某个接口,基类. 
tuple  最多只能包含22 个不同类型的元素. 如果你打开scala的标准库, 查看tuple 的实现, 会发现 从Tuple1 到 Tuple22 这22个类. 作者认为这么多就够用了.
tuple 不是用来遍历的, 相反 它可以用来包装一些属性,比如用途广泛的键值对. 只用到了 tuple2.

上面提到了`case class`, 也就是升级版的`tuple`,也可以用来包装属性 ,而且 同样最多只能包含22个不同的元素. 就是 case class 
case class 当然是一种特殊的 类 ,自带了构造器, 生成实例时可以省略 关键字 `new`. 可以通过属性名字访问元素,而tuple 只能用从 1开始的下标访问元素 形如: `pairs._1` 这样写出来的代码可读性不好.
好在你可以使用 scala 淫荡的 模式匹配.
对于 

```scala
case class Person(name:String, age:Int)
```

可以

```scala
val jude = Person("zhutou",25)
jude match{
 case Person(name, age) => name +" is "+age
}
```

而对于一个类型为 (String, Int) 的tuple 来说, 也可以这样

```scala
val pre = ("Haha", 386)
pre match {
case (name, id) => name+" is "+id
}
```


