---
layout: post
title: Groovy学习之-与Java的不同之处
categories: [groovy]
description: Groovy试图对Java开发人员尽可能自然。 我们试图在设计Groovy时遵循最小惊讶原则，特别是对于来自Java背景的Groovy开发人员。
keywords: [groovy, java]
date: 2017-07-04
author: 化作春泥
---

> 本文首发于[《简书》](http://www.jianshu.com/p/0de590d08532)，转载请保留链接 : )

## 前言

_Groovy学习目录-[传送门](http://www.jianshu.com/p/5a4ce49da37f)_

Groovy试图对Java开发人员尽可能自然。 我们试图在设计Groovy时遵循[最小惊讶原则](https://en.wikipedia.org/wiki/Principle_of_least_astonishment)，特别是对于来自Java背景的Groovy开发人员。

这里我们列出了Java和Groovy之间的所有主要区别。

## 默认 imports
所有这些包和类都是默认导入的，您不必使用显式import语句来使用它们：
```groovy
java.io.*
java.lang.*
java.math.BigDecimal
java.math.BigInteger
java.net.*
java.util.*
groovy.lang.*
groovy.util.*
```

## Multi-methods
在Groovy中，将在运行时选择将被调用的方法。 这称为运行时分派或`Multi-methods`。 这意味着将基于运行时参数的类型来选择方法。 在Java中，则是根据声明的类型，在编译时选择方法。

下面的代码，以Java代码编写，可以在Java和Groovy中编译，但它的行为会有所不同：
```groovy
int method(String arg) {
    return 1;
}
int method(Object arg) {
    return 2;
}
Object o = "Object";
int result = method(o);
```
在Java中, 您讲得到：
```groovy
assertEquals(2, result);
```
而在Groovy中：
```groovy
assertEquals(1, result);
```
这是因为Java将使用静态信息类型，即`o`被声明为`Object `，而Groovy将在运行时选择该方法被实际调用时。 因为它是用`String`调用的，所以调用`String`版本。

## 数组初始化
在Groovy中，{...}块是为闭包而保留的。 这意味着您不能使用以下语法创建数组_literal_：
```groovy
int [] array = {1，2，3}
```
实际上你必须使用：
```groovy
int [] array = [1,2,3]
```

## 包范围可见性
在Groovy中，在字段上省略修饰符不会像Java中一样变成` package-private`字段：
```groovy
class Person {
     字符串名称
}}
```
相反，它用于创建一个属性，也就是说一个私有字段，一个关联的`getter`和一个关联的`setter`。

可以通过使用@PackageScope注释来创建一个`package-private`字段：
```groovy
class Person {
     @PackageScope字符串名称
}}
```

## 自动资源管理块
Groovy不支持Java 7中的ARM（自动资源管理）块。 相反，Groovy提供了依赖闭包的各种方法，它们具有相同的效果，同时更加惯用。 例如：
```java
Path file = Paths.get("/path/to/file");
Charset charset = Charset.forName("UTF-8");
try (BufferedReader reader = Files.newBufferedReader(file, charset)) {
    String line;
    while ((line = reader.readLine()) != null) {
        System.out.println(line);
    }

} catch (IOException e) {
    e.printStackTrace();
}
```
可以这样写：
```groovy
new File('/path/to/file').eachLine('UTF-8') {
   println it
}
```
或者，如果你想要一个更接近Java的版本：
```groovy
new File('/path/to/file').withReader('UTF-8') { reader ->
   reader.eachLine {
       println it
   }
}
```

## 内部类
匿名内部类和嵌套类的实现遵循Java方式，但是你不应该拿出Java语言规范，并且对不同的东西不断地摇头。 它的实现看起来很像`groovy.lang.Closure`，同时有一些好处和一些差异。 例如访问私有字段和方法可能成为一个问题，但另一方面，局部变量不必是final的。
1. 静态内部类
这里有一个静态内部类的例子：

```groovy
class A {
    static class B {}
}

new A.B()
```
 静态内部类的使用是最好的支持。 如果你需要一个内部类，你最好使用静态的。
2. 匿名内部类

```groovy
import java.util.concurrent.CountDownLatch
import java.util.concurrent.TimeUnit

CountDownLatch called = new CountDownLatch(1)

Timer timer = new Timer()
timer.schedule(new TimerTask() {
    void run() {
        called.countDown()
    }
}, 0)

assert called.await(10, TimeUnit.SECONDS)
```
3. 创建非静态内部类的实例
在Java中，你可以这样做：

```groovy
public class Y {
    public class X {}
    public X foo() {
        return new X();
    }
    public static X createX(Y y) {
        return y.new X();
    }
}
```
Groovy不支持`y.new X()`语法。 相反，你必须用`new X(y)`，如下面的代码：
```groovy
public class Y {
    public class X {}
    public X foo() {
        return new X()
    }
    public static X createX(Y y) {
        return new X(y)
    }
}
```
注意，Groovy支持使用一个参数调用方法，而不提供参数。 然后，参数的值为null。 基本上，相同的规则适用于调用构造函数。 有一个危险，例如你会写new X()，而不是new X(this)。 由于这也可能是正常的方式，我们还没有找到一个好的方法来防止这个问题。

## Lambdas
Java 8支持lambdas和方法引用：
```java
Runnable run = () -> System.out.println("Run");
list.forEach(System.out::println);
```
Java 8 lambdas可以或多或少被认为是匿名内部类。 Groovy不支持该语法，但是可以使用闭包：
```groovy
Runnable run = { println 'run' }
list.each { println it } // or list.each(this.&println)
```

## GStrings
由于双引号字符串字面量被解释为`GString`，如果具有包含美元字符的`String`的类是使用Groovy和Java编译器编译的，Groovy可能会失败并产生编译错误或产生细微不同的代码。

通常，Groovy将在`GString`和`String`之间自动转换，如果API声明参数的类型，小心接受`Object`参数的Java API，然后检查实际的类型。

## 字符串和字符
Groovy中的单引号用于`String`，双引号结果是`String`或`GString`，取决于文字中是否有插值。
```groovy
assert 'c'.getClass()==String
assert "c".getClass()==String
assert "c${1}".getClass() in GString
```
只有在赋给`char`类型的变量时，Groovy会自动将单字符`String`转换为`char`。 当调用类型为`char`的参数的方法时，我们需要显式转换或确保该值已预先转换。
```groovy
char a='a'
assert Character.digit(a, 16)==10 : 'But Groovy does boxing'
assert Character.digit((char) 'a', 16)==10

try {
  assert Character.digit('a', 16)==10
  assert false: 'Need explicit cast'
} catch(MissingMethodException e) {
}
```
Groovy支持两种类型的转换，在转换为`char`的情况下，在转换multi-char 时存在微妙的差别。 Groovy风格的转换是更宽松的，将采取第一个字符，而C风格的转换将失败，异常。
```groovy
// for single char strings, both are the same
assert ((char) "c").class==Character
assert ("c" as char).class==Character

// for multi char strings they are not
try {
  ((char) 'cx') == 'c'
  assert false: 'will fail - not castable'
} catch(GroovyCastException e) {
}
assert ('cx' as char) == 'c'
assert 'cx'.asType(char) == 'c'
```

## 原始和封装
因为Groovy使用Objects来做每一件事，它对原始的引用[自动包装](http://docs.groovy-lang.org/latest/html/documentation/core-object-orientation.html#_primitive_types)。 因此，它不遵循Java的行为扩展优先于装箱。 这里有一个使用int的例子
```groovy
int i
m(i)

//这是Java将调用的方法，因为扩展优先于拆箱。
void m(long l) {
  println "in m(long)"
}

//这是Groovy实际调用的方法，因为所有的基本引用都使用它们的包装类。
void m(Integer i) {
  println "in m(Integer)"
}
```

## `==`的行为
在Java中`==`表示对象的原始类型或标识的相等性。 在Groovy `==`翻译为`a.compareTo(b)== 0`，如果他们是可比较的，否则`a.equals(b)`。 如果要检查身份，有`is`方法，例如` a.is(b)`。

## 转换
[详情请点击超链接](http://www.groovy-lang.org/differences.html#_conversions)

## 额外的关键字
Groovy中还要比Java多几个关键字。 不要将它们用于变量名称等。
* `as`
* `def`
* `in`
* `trait`

