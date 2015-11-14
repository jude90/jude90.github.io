---
title: PartialFunction的二三事
layout: post
date: 2015-08-12 16:06:02
---

`partialfunction` , 可以翻译成偏函数。当我好不容易理解 python中的柯里化， 学会使用高阶函数和闭包来生成一个偏函数以后，我遇到了scala 的 `PartialFunction` 。于是问题来了。

- 这究竟是什么鬼！？
- 它是闭包吗？
- 它是柯里化的产物吗？
- 它和`Partial Application` 是什么关系！？

twitter 的教程里，给了这样一个例子。

```scala
val one: PartialFunction[Int, String] = { case 1 => "one" }

scala> one.isDefinedAt(1)
res0: Boolean = true

scala> one.isDefinedAt(2)
res1: Boolean = false
```

根据twitter的说法，普通的函数只会限定类型，而偏函数可以限制该类型的定义域。比如上面的 one函数，只能接收 `1:Int` , 试着调用 `one(3)`,会得到报错：

```scala
scala> one(3)
scala.MatchError: 3 (of class java.lang.Integer)at scala.PartialFunction$$anon$1.apply(PartialFunction.scala:248)
...
```
partialFucntion 的用法总是与模式匹配 有关，既然可以用`case` 也就可以用 `case guard`, 比如：

```scala
val tripleOdds: PartialFunction[Int, Int] = {
      case x: Int if (x % 2) != 0 => x * 3
    }
```

一个包含 `case`的函数体，只是 `PartialFunction` 的简写形式，在scala koans 给的例子里面， 完整的声明大概是这样

```scala
val doubleEvens: PartialFunction[Int, Int] = new PartialFunction[Int, Int] {
      //States that this partial function will take on the task
    def isDefinedAt(x: Int) = x % 2 == 0
      //What we do if this does partial function matches
    def apply(v1: Int) = v1 * 2
    }
```

`isDefinedAt` 返回布尔值，用来判断参数是否属于定义域。
`apply`函数就是实际执行的函数体。

说了这么多。我们为什么需要这个挑剔、爱报错的函数? 大概是为了实现函数的组合吧。
容我先装个逼。在Haskell 的世界里， 将函数组合成一个新的函数，就像呼吸一样自然。如果你有函数 `g`和 函数`f` ,想要生成一个函数 ，执行`f(g(arg))`这样的计算。你可以声明

```haskell
h = f.g
h args == f (g args) 
```

将你自己的函数组成管道，数据或者对象将在管道里传递。简洁，直观。
如果我们想在复杂的scala中实现同样的管道式编程，就需要借助一些组合函数。

**orElse 和 andThen**
先定义两个函数，他们的类型都是 `PartialFunction`

```scala
val doubleEvens: PartialFunction[Int, Int] = {
      case x: Int if (x % 2) == 0 => x * 2
    }
    val tripleOdds: PartialFunction[Int, Int] = {
      case x: Int if (x % 2) != 0 => x * 3
    }
```
 定义另外两个函数
 
```scala
val addFive = (x: Int) => x + 5
val whatToDo = doubleEvens orElse tripleOdds andThen addFive //Here we chain the partial function
```    

`addFive` 是一个普通的匿名函数。 `whatToDo` 是由前面三个函数组合而成的复合函数。
`orElse` 函数是`PartialFunction` 定义的方法，其参数必须也是 `PartialFunction`。
当左边的函数匹配失败，则会调用`orElse`右边的函数。
`andThen` 也是`PartialFunction` 定义的方法，他的参数是普通的`Function1`类型。

组合函数，管道式编程是函数式编程美学的一部分，内容博大精深，这里不再展开。

#### 总结
Scala 的 `PartialFunction` 是一个用来实现组合函数的`trait` 。虽然名字很坑，但是功能很强大。`PartialFunction`  实现了`lift` 方法，可以将失败的值包装成`Option`，而非报错 。 看到这里，`monad`已经呼之欲出？对我而言，普通的管道式编程已经很受用了。这些装逼的奥义，以后再研究吧。