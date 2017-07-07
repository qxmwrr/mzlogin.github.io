---
layout: post
title: Groovy学习之-Groovy Development Kit（GDK）-实用工具
category: [groovy]
keywords: [groovy, java]
date: 2017-07-05
author: 化作春泥
---

> 本文首发于[《简书》](http://www.jianshu.com/p/7c3bd8a67dc8)，转载请保留链接 : )

_Groovy学习目录-[传送门](http://www.jianshu.com/p/5a4ce49da37f)_

## ConfigSlurper

`ConfigSlurper`是一个用于读取以Groovy脚本形式定义的配置文件的实用程序类。 类似于Java `* .properties`文件的情况，`ConfigSlurper`允许使用点符号。 但此外，它允许Closure作用域配置值和任意对象类型。

~~~groovy
def config = new ConfigSlurper().parse('''
    app.date = new Date()  //点符号的用法
    app.age  = 42
    app {                  //使用Closure作用域作为点符号的替代
        name = "Test${42}"
    }
''')

assert config.app.date instanceof Date
assert config.app.age == 42
assert config.app.name == 'Test42'
~~~

从上面的例子可以看出，`parse`方法可以用来检索`groovy.util.ConfigObject`实例。` ConfigObject`是一个专门的`java.util.Map`实现，它返回配置的值或一个新的`ConfigObject`实例，但永远不为`null`。

```groovy
def config = new ConfigSlurper().parse('''
    app.date = new Date()
    app.age  = 42
    app.name = "Test${42}"
''')

assert config.test != null//`config.test`还没有被指定，它被调用时返回一个`ConfigObject`。
```

在`点符号`是配置变量名称的一部分的情况下，可以使用单引号或双引号对其进行转义。

```groovy
def config = new ConfigSlurper().parse('''
    app."person.age"  = 42
''')

assert config.app."person.age" == 42
```

此外，`ConfigSlurper`还支持`environments `。 `environments `方法可以用来移交一个本身可能由几个部分组成的`Closure`实例。 假设我们想为开发环境创建一个特定的配置值。 当创建ConfigSlurper实例时，我们可以使用`ConfigSlurper(String)`构造函数来指定目标环境。

```groovy
def config = new ConfigSlurper('development').parse('''
  environments {
       development {
           app.port = 8080
       }

       test {
           app.port = 8082
       }

       production {
           app.port = 80
       }
  }
''')

assert config.app.port == 8080
```

`ConfigSlurper`环境不限于任何特定的环境名称。 它只依赖于`ConfigSlurper`客户端代码支持和解释什么值。

`environments`方法是内置的，但`registerConditionalBlock`方法可用于注册除`environments `名称之外的其他方法名称。

```groovy
def slurper = new ConfigSlurper()
slurper.registerConditionalBlock('myProject', 'developers')   //一旦新block被注册，ConfigSlurper可以解析它。

def config = slurper.parse('''
  sendMail = true

  myProject {
       developers {
           sendMail = false
       }
  }
''')

assert !config.sendMail
```

对于Java集成，`toProperties`方法可用于将`ConfigObject`转换为可能存储到`* .properties`文本文件的`java.util.Properties`对象。 请注意，将配置值添加到新创建的`Properties`实例时，将其转换为`String`实例。

```groovy
def config = new ConfigSlurper().parse('''
    app.date = new Date()
    app.age  = 42
    app {
        name = "Test${42}"
    }
''')

def properties = config.toProperties()

assert properties."app.date" instanceof String
assert properties."app.age" == '42'
assert properties."app.name" == 'Test42'
```

## Expando

`Expando`类可用于创建动态可扩展对象。 尽管它的名称叫Expando，它不使用类`ExpandoMetaClass`。 每个`Expando`对象表示一个独立的动态制作的实例，可以在运行时使用属性（或方法）进行扩展。
```groovy
def expando = new Expando()
expando.name = 'John'

assert expando.name == 'John'
```

当动态属性注册`Closure`代码块时，会发生特殊情况。 一旦`Closure`被注册，它就可以被调用，因为它将通过方法调用完成。

```groovy
def expando = new Expando()
expando.toString = { -> 'John' }
expando.say = { String s -> "John says: ${s}" }

assert expando as String == 'John'
assert expando.say('Hi') == 'John says: Hi'
```

## 可观察的 list, map and set

Groovy带有可观察的 list, map and set。 当添加，删除或更改元素时，每个集合都会触发`java.beans.PropertyChangeEvent`事件。 注意，`PropertyChangeEvent`不仅发出某个事件已经发生的信号，此外，它保存关于属性名称和某个属性已经改变到的旧/新值的信息。

根据发生的更改的类型，可观察的集合可能触发更专门的`PropertyChangeEvent`类型。 例如，向可观察列表中添加元素会触发`ObservableList.ElementAddedEvent`事件。

```groovy
def event                                       //声明一个PropertyChangeEventListener捕获被触发的事件
def listener = {
    if (it instanceof ObservableList.ElementEvent)  {  //ObservableList.ElementEvent及其子类型与此侦听器相关
        event = it
    }
} as PropertyChangeListener


def observable = [1, 2, 3] as ObservableList    //注册监听器
observable.addPropertyChangeListener(listener)  //从给定列表创建一个ObservableList

observable.add 42                               //触发ObservableList.ElementAddedEvent事件

assert event instanceof ObservableList.ElementAddedEvent

def elementAddedEvent = event as ObservableList.ElementAddedEvent
assert elementAddedEvent.changeType == ObservableList.ChangeType.ADDED
assert elementAddedEvent.index == 3
assert elementAddedEvent.oldValue == null
assert elementAddedEvent.newValue == 42
```

请注意，添加元素实际上会触发两个事件。 第一个是类型  `ObservableList.ElementAddedEvent`，第二个是一个普通的`PropertyChangeEvent`，通知监听器属性`size`的更改。

`ObservableList.ElementClearedEvent`事件类型是另一个有趣的事件类型。 每当删除多个元素时，例如调用`clear()`时，它保存从列表中删除的元素。

```groovy
def event
def listener = {
    if (it instanceof ObservableList.ElementEvent)  {
        event = it
    }
} as PropertyChangeListener


def observable = [1, 2, 3] as ObservableList
observable.addPropertyChangeListener(listener)

observable.clear()

assert event instanceof ObservableList.ElementClearedEvent

def elementClearedEvent = event as ObservableList.ElementClearedEvent
assert elementClearedEvent.values == [1, 2, 3]
assert observable.size() == 0

```
为了获得所有支持的事件类型的概述，我们鼓励读者查看JavaDoc文档或正在使用的observable集合的源代码。

`ObservableMap`和`ObservableSet`带有我们在本节中对`ObservableList`所看到的相同的概念。

