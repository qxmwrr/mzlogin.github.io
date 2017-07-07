---
layout: post
title: Groovy学习之-运行时元编程
category: [groovy]
description: Groovy语言支持两种类型的元编程：运行时元编程和编译时元编程。 第一个允许在运行时改变类模型和程序的行为，而第二个只在编译时发生。 两者都有利弊，我们将在本节详细介绍。
keywords: [groovy, java]
date: 2017-07-05
author: 化作春泥
---

> 本文首发于[《简书》](http://www.jianshu.com/p/5a4ce49da37f)，转载请保留链接 : )

_Groovy学习目录-[传送门](http://www.jianshu.com/p/5a4ce49da37f)_

元编程(Metaprogramming)->[百度百科](http://baike.baidu.com/item/元编程)

Groovy语言支持两种类型的元编程：运行时元编程和编译时元编程。 第一个允许在运行时改变类模型和程序的行为，而第二个只在编译时发生。 两者都有利弊，我们将在本节详细介绍。

## 运行时元编程
使用运行时元编程，我们可以在运行时截取，注入，合成类和接口的方法。 为了更深入地理解Groovy MOP，我们需要了解Groovy对象和Groovy的方法处理。 在Groovy中，我们使用了三种对象：POJO，POGO和Groovy Interceptors。 Groovy允许对所有类型的对象进行元编程，但是是以不同的方式进行的。

* POJO - 一个常规的Java对象，它的类是用Java或任何其他运行在JVM上的编程语言编写的。
* POGO - 一个Groovy对象，它的类是用Groovy编写的。 它扩展了`java.lang.Object`并在默认情况下实现了`groovy.lang.GroovyObject`接口。
* Groovy Interceptor - 一个Groovy对象，它实现了`groovy.lang.GroovyInterceptable`接口并具有方法拦截功能，我们将在`GroovyInterceptable`部分讨论。

对于每个方法调用，Groovy检查对象是POJO还是POGO。 对于POJO，Groovy从groovy.lang.MetaClassRegistry获取它的MetaClass，并将方法调用委托给它。 对于POGO，Groovy需要更多步骤，如下图所示：

![图1. Groovy拦截机制](http://upload-images.jianshu.io/upload_images/3806049-1ba4962d5be63898.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## GroovyObject接口
[groovy.lang.GroovyObject](http://docs.groovy-lang.org/2.4.7/html/gapi/index.html?groovy/lang/GroovyObject.html)是Groovy中的主要接口，就像`Object`类是Java中的主要接口一样。` GroovyObject`在[groovy.lang.GroovyObjectSupport](http://docs.groovy-lang.org/2.4.7/html/gapi/index.html?groovy/lang/GroovyObjectSupport.html)类中有一个默认实现，它负责将调用传递给[groovy.lang.MetaClass](http://docs.groovy-lang.org/2.4.7/html/gapi/index.html?groovy/lang/MetaClass.html)对象。 `GroovyObject`源代码如下：

```groovy
package groovy.lang;

public interface GroovyObject {
    Object invokeMethod(String name, Object args);
    Object getProperty(String propertyName);
    void setProperty(String propertyName, Object newValue);
    MetaClass getMetaClass();
    void setMetaClass(MetaClass metaClass);
}
```

#### invokeMethod
根据[运行时元编程](http://www.groovy-lang.org/metaprogramming.html#_runtime_metaprogramming)中的模式，当您调用的方法不存在于Groovy对象上时，将调用此方法。 下面是一个重写`invokeMethod()`方法的简单示例：

```groovy
class SomeGroovyClass {

    def invokeMethod(String name, Object args) {
        return "called invokeMethod $name $args"
    }

    def test() {
        return 'method exists'
    }
}

def someGroovyClass = new SomeGroovyClass()

assert someGroovyClass.test() == 'method exists'
assert someGroovyClass.someMethod() == 'called invokeMethod someMethod []'
```

#### get/setProperty

对属性的每个读取访问都可以通过覆盖当前对象的`getProperty()`方法来拦截。 这里有一个简单的例子：
```groovy
class SomeGroovyClass {

    def property1 = 'ha'
    def field2 = 'ho'
    def field4 = 'hu'

    def getField1() {
        return 'getHa'
    }

    def getProperty(String name) {
        if (name != 'field3')
            return metaClass.getProperty(this, name) //将请求转发到getter以获取除field3之外的所有属性。
        else
            return 'field3'
    }
}

def someGroovyClass = new SomeGroovyClass()

assert someGroovyClass.field1 == 'getHa'
assert someGroovyClass.field2 == 'ho'
assert someGroovyClass.field3 == 'field3'
assert someGroovyClass.field4 == 'hu'
```

您可以通过覆盖`setProperty()`方法来拦截对属性的写访问权限：
```groovy
class POGO {

    String property

    void setProperty(String name, Object value) {
        this.@"$name" = 'overridden'
    }
}

def pogo = new POGO()
pogo.property = 'a'

assert pogo.property == 'overridden'
```

#### get/setMetaClass
您可以访问对象的`metaClass`或设置您自己的`MetaClass`实现以更改默认拦截机制。 例如，您可以编写自己的`MetaClass`接口实现，并将其分配给对象，从而更改拦截机制：
```groovy
// getMetaclass
someObject.metaClass

// setMetaClass
someObject.metaClass = new OwnMetaClassImplementation()
```

您可以在[GroovyInterceptable](http://www.groovy-lang.org/metaprogramming.html#_groovyinterceptable)主题中找到其他示例。

## get/setAttribute
此功能与`MetaClass`实现相关。 在默认实现中，您可以访问字段而不调用它们的getter和setter。 下面的示例演示了这种方法：

```groovy
class SomeGroovyClass {

    def field1 = 'ha'
    def field2 = 'ho'

    def getField1() {
        return 'getHa'
    }
}

def someGroovyClass = new SomeGroovyClass()

assert someGroovyClass.metaClass.getAttribute(someGroovyClass, 'field1') == 'ha'
assert someGroovyClass.metaClass.getAttribute(someGroovyClass, 'field2') == 'ho'
```

```groovy
class POGO {

    private String field
    String property1

    void setProperty1(String property1) {
        this.property1 = "setProperty1"
    }
}

def pogo = new POGO()
pogo.metaClass.setAttribute(pogo, 'field', 'ha')
pogo.metaClass.setAttribute(pogo, 'property1', 'ho')

assert pogo.field == 'ha'
assert pogo.property1 == 'ho'
```

## methodMissing
Groovy支持`methodMissing`的概念。 此方法与`invokeMethod`的不同之处在于，它仅在失败的方法分派的情况下被调用，当没有找到给定名称和/或给定参数的方法时：
```groovy
class Foo {

   def methodMissing(String name, def args) {
        return "this is me"
   }
}

assert new Foo().someUnknownMethod(42l) == 'this is me'
```
通常当使用`methodMissing`时，可以缓存结果，以便下次调用相同的方法。

例如，考虑GORM中的动态查找器。 这些是根据`methodMissing`来实现的。 代码类似这样：
```groovy
class GORM {

   def dynamicMethods = [...] // an array of dynamic methods that use regex

   def methodMissing(String name, args) {
       def method = dynamicMethods.find { it.match(name) }
       if(method) {
          GORM.metaClass."$name" = { Object[] varArgs ->
             method.invoke(delegate, name, varArgs)
          }
          return method.invoke(delegate,name, args)
       }
       else throw new MissingMethodException(name, delegate, args)
   }
}
```
注意如果我们找到一个方法来调用，随后就会使用[ExpandoMetaClass](http://www.groovy-lang.org/metaprogramming.html#metaprogramming_emc)动态地注册一个新的方法。 这是为了下次更有效率的调用相同的方法。 这种使用`methodMissing`的方法不像`invokeMethod`一样，这种方式在第二次调用时并不产生昂贵开销。

## propertyMissing
Groovy支持`propertyMissing`的概念，用于拦截那些失败的属性解析尝试。 在getter方法的情况下，`propertyMissing`接受一个包含属性名称的`String`参数：
```groovy
class Foo {
   def propertyMissing(String name) { name }
}

assert new Foo().boo == 'boo'
```
仅当Groovy运行时找不到给定属性的getter方法时才调用`propertyMissing(String)`方法。

对于setter方法，可以添加另一个`propertyMissing`定义，它接受一个附加的value参数：
```groovy
class Foo {
   def storage = [:]
   def propertyMissing(String name, value) { storage[name] = value }
   def propertyMissing(String name) { storage[name] }
}

def f = new Foo()
f.foo = "bar"

assert f.foo == "bar"
```
与`methodMissing`一样，最佳做法是在运行时动态注册新属性，以提高总体查找性能。

`methodMissing`和`propertyMissing`方法处理静态方法和属性可以通过[ExpandoMetaClass](http://www.groovy-lang.org/metaprogramming.html#metaprogramming_emc)添加。

## GroovyInterceptable
[groovy.lang.GroovyInterceptable](http://docs.groovy-lang.org/2.4.7/html/gapi/index.html?groovy/lang/GroovyInterceptable.html)接口是用于通知Groovy运行时的扩展`GroovyObject`的标记接口，所有方法都应通过Groovy运行时的方法分派器机制拦截。
```groovy
package groovy.lang;

public interface GroovyInterceptable extends GroovyObject {
}
```

当Groovy对象实现`GroovyInterceptable`接口时，对任何方法的调用都会调用`invokeMethod()`方法。 下面你可以看到一个这种类型的对象的简单例子：
```groovy
class Interception implements GroovyInterceptable {

    def definedMethod() { }

    def invokeMethod(String name, Object args) {
        'invokedMethod'
    }
}
```
下一段代码是一个测试，显示对现有和不存在的方法的调用都将返回相同的值。
```groovy
class InterceptableTest extends GroovyTestCase {

    void testCheckInterception() {
        def interception = new Interception()

        assert interception.definedMethod() == 'invokedMethod'
        assert interception.someMethod() == 'invokedMethod'
    }
}
```

我们不能使用默认groovy方法，如`println`，因为这些方法被注入到所有groovy对象中，所以它们也会被拦截。

如果我们要拦截所有方法调用，但不想实现`GroovyInterceptable`接口，我们可以在对象的`MetaClass`上实现`invokeMethod()`。 此方法适用于POGO和POJO，如此示例所示：
```groovy
class InterceptionThroughMetaClassTest extends GroovyTestCase {

    void testPOJOMetaClassInterception() {
        String invoking = 'ha'
        invoking.metaClass.invokeMethod = { String name, Object args ->
            'invoked'
        }

        assert invoking.length() == 'invoked'
        assert invoking.someMethod() == 'invoked'
    }

    void testPOGOMetaClassInterception() {
        Entity entity = new Entity('Hello')
        entity.metaClass.invokeMethod = { String name, Object args ->
            'invoked'
        }

        assert entity.build(new Object()) == 'invoked'
        assert entity.someMethod() == 'invoked'
    }
}
```
有关`MetaClass`的其他信息，请参阅[MetaClasses](http://www.groovy-lang.org/metaprogramming.html#_metaclasses)部分。

## Categories分类
有些情况下，如果想为一个无法控制的类添加额外的方法，Categories就派上用场了。 为了启用此功能，Groovy实现了一个从Objective-C借用的功能，称为Categories。

Categories使用所谓的Category 类实现。Category 类是特殊的类，因为它需要满足用于定义扩展方法的某些预定义规则。

系统中包括几个Categories，用于向类添加功能，以使这些类在Groovy环境中更易于使用：

* [groovy.time.TimeCategory](http://docs.groovy-lang.org/2.4.7/html/gapi/index.html?groovy/time/TimeCategory.html)
* [groovy.servlet.ServletCategory](http://docs.groovy-lang.org/2.4.7/html/gapi/index.html?groovy/servlet/ServletCategory.html)
* [groovy.xml.dom.DOMCategory](http://docs.groovy-lang.org/2.4.7/html/gapi/index.html?groovy/xml/dom/DOMCategory.html)

默认情况下不启用Category 类。 要使用Category 类中定义的方法，需要使用由GDK提供并可从每个Groovy对象实例中获取的`use`方法：
```groovy
use(TimeCategory)  {
    println 1.minute.from.now       //TimeCategory给Integer添加了方法
    println 10.hours.ago

    def someDate = new Date()       //TimeCategory将方法添加到Date
    println someDate - 3.months
}
```

`use`方法将Category 类作为其第一个参数，将闭包代码块作为第二个参数。 在`Closure`内可以访问category的方法。 从上面的例子可以看出，甚至像`java.lang.Integer`或`java.util.Date`这样的JDK中的类也可以用用户定义的方法来丰富功能。

一个category不需要直接暴露给用户代码，如下：
```groovy
class JPACategory{
    //让我们在不修改JPA EntityManager的前提下，增强JPA EntityManager的功能
    static void persistAll(EntityManager em , Object[] entities) { //添加一个接口用于保存所有entities
        entities?.each { em.persist(it) }
    }
}

def transactionContext = {
    EntityManager em, Closure c ->
    def tx = em.transaction
    try {
        tx.begin()
        use(JPACategory) {
            c()
        }
        tx.commit()
    } catch (e) {
        tx.rollback()
    } finally {
        //清理你的资源
    }
}

// 用户代码，他们总是忘记关闭异常中的资源，有些甚至忘记提交，让我们不要依赖他们。
EntityManager em; //可能注射
transactionContext (em) {
   em.persistAll(obj1, obj2, obj3)
   // let's do some logics here to make the example sensible
   //让我们在这里做一些逻辑，使示例理智
   em.persistAll(obj2, obj4, obj6)
}
```

当我们看一下`groovy.time.TimeCategory`类，我们看到扩展方法都被声明为`static`方法。 事实上，这是类方法必须满足的要求之一，它的方法被成功地添加到`use`代码块中的类中：
```groovy
public class TimeCategory {

    public static Date plus(final Date date, final BaseDuration duration) {
        return duration.plus(date);
    }

    public static Date minus(final Date date, final BaseDuration duration) {
        final Calendar cal = Calendar.getInstance();

        cal.setTime(date);
        cal.add(Calendar.YEAR, -duration.getYears());
        cal.add(Calendar.MONTH, -duration.getMonths());
        cal.add(Calendar.DAY_OF_YEAR, -duration.getDays());
        cal.add(Calendar.HOUR_OF_DAY, -duration.getHours());
        cal.add(Calendar.MINUTE, -duration.getMinutes());
        cal.add(Calendar.SECOND, -duration.getSeconds());
        cal.add(Calendar.MILLISECOND, -duration.getMillis());

        return cal.getTime();
    }

    // ...
}
```
另一个要求是静态方法的第一个参数必须定义该方法一旦被激活时附加的类型。 其他参数是方法将作为参数的正常参数。

由于参数和静态方法约定，category方法定义可能比正常的方法定义更不直观。 作为替代Groovy带有一个`@Category`注解，在编译时将加了注解的类转换为category 类。
```groovy
class Distance {
    def number
    String toString() { "${number}m" }
}

@Category(Number)
class NumberCategory {
    Distance getMeters() {
        new Distance(number: this)
    }
}

use (NumberCategory)  {
    assert 42.meters.toString() == '42m'
}
```

应用`@Category`注解具有能够使用没有目标类型作为第一个参数的实例方法的优点。 目标类型class作为注释的参数提供。

在[compile-time metaprogramming section](http://www.groovy-lang.org/metaprogramming.html#xform-Category)中，有一个关于`@Category`的完整章节

## MateClass
待定
#### 自定义MateClass
待定
* 委托MateClass
待定
* Magic package (Maksym Stavytskyi)
待定

#### 每个metaclass实例
待定

#### ExpandoMetaClass
Groovy带有一个特殊的`MetaClass`，它就是`ExpandoMetaClass`。 它是特别的，它允许通过使用一个整洁的闭包语法动态添加或更改方法，构造函数，属性，甚至静态方法。

应用这些修改在模拟([mock](http://baike.baidu.com/item/mock测试))或桩([stub](https://www.zhihu.com/question/24844900))情况下特别有用，如[测试指南](http://docs.groovy-lang.org/latest/html/documentation/core-testing-guide.html#testing_guide_emc)中所示。

每个`java.lang.Class`由Groovy提供，并有一个特殊的`metaClass`属性，它将提供对`ExpandoMetaClass`实例的引用。 然后，此实例可用于添加方法或更改已有现有方法的行为。

默认情况下`ExpandoMetaClass`不执行继承。 要启用它，您必须在应用程序启动之前调用`ExpandoMetaClass＃enableGlobally()`，例如在main方法或servlet引导中。

以下部分详细介绍了`ExpandoMetaClass`如何在各种情况下使用。

###### 方法

一旦通过调用`metaClass`属性访问`ExpandoMetaClass`，可以使用左移`<<`或`=`运算符来添加方法。

注意，左移位运算符用于附加一个新方法。 如果类或接口声明了具有相同名称和参数类型的公共方法，包括继承自父类和父接口但不包括在运行时添加到`metaClass`的那些方法，那么将抛出异常。 如果要替换由类或接口声明的方法，可以使用`=`运算符。

运算符通过一个Closure代码块实例，将功能应用于metaClass的不存在的属性上。

```groovy
class Book {
   String title
}

Book.metaClass.titleInUpperCase << {-> title.toUpperCase() }

def b = new Book(title:"The Stand")

assert "THE STAND" == b.titleInUpperCase()
```

上面的例子显示了如何通过访问`metaClass`属性并使用`<<`或`=`运算符来分配一个`Closure`代码块来将新方法添加到类中。 `Closure`参数被解释为方法参数。 无参数方法可以使用`{-> ...}`语法添加。

###### 属性

`ExpandoMetaClass`支持两种机制来添加或覆盖属性。

首先，它支持通过向`metaClass`的属性赋值来声明一个可变属性：

```groovy
class Book {
   String title
}

Book.metaClass.author = "Stephen King"
def b = new Book()

assert "Stephen King" == b.author
```

另一种方法是通过使用添加实例方法的标准机制来添加getter和/或setter方法。
```groovy
class Book {
  String title
}
Book.metaClass.getAuthor << {-> "Stephen King" }

def b = new Book()

assert "Stephen King" == b.author
```

在上面的源代码示例中，属性由闭包指定，并且是只读属性。 可以添加一个等效的setter方法，但是该属性值需要被存储以备将来使用。 这可以如下面的示例所示完成。

```groovy
class Book {
  String title
}

def properties = Collections.synchronizedMap([:])

Book.metaClass.setAuthor = { String value ->
   properties[System.identityHashCode(delegate) + "author"] = value
}
Book.metaClass.getAuthor = {->
   properties[System.identityHashCode(delegate) + "author"]
}
```

这不是唯一的方式。 例如，在servlet容器中，一种方式可能是将值作为请求属性存储在当前执行的请求中（如在Grails中的某些情况下所做的那样）。

###### 构造函数

可以通过使用特殊的构造函数属性来添加构造函数。 可以使用`<<`或`=`运算符来分配`Closure`代码块。 当代码在运行时执行时，`Closure`参数将成为构造函数参数。

```groovy
class Book {
    String title
}
Book.metaClass.constructor << { String title -> new Book(title:title) }

def book = new Book('Groovy in Action - 2nd Edition')
assert book.title == 'Groovy in Action - 2nd Edition'
```

当添加构造函数时要小心，因为它很容易陷入堆栈溢出问题。

###### 静态方法

可以使用与实例方法相同的技术添加静态方法，并在方法名称之前添加静态限定符。

```groovy
class Book {
   String title
}

Book.metaClass.static.create << { String title -> new Book(title:title) }

def b = Book.create("The Stand")
```

###### 借用方法

使用`ExpandoMetaClass`，可以使用Groovy的方法指针语法从其他类中借用方法。

```groovy
class Person {
    String name
}
class MortgageLender {
   def borrowMoney() {
      "buy house"
   }
}

def lender = new MortgageLender()

Person.metaClass.buyHouse = lender.&borrowMoney

def p = new Person()

assert "buy house" == p.buyHouse()
```

###### 动态方法名称

由于Groovy允许使用Strings作为属性名，这反过来允许您在运行时动态创建方法和属性名。 要创建具有动态名称的方法，只需使用引用属性名称的语言特性作为字符串。

```groovy
class Person {
   String name = "Fred"
}

def methodName = "Bob"

Person.metaClass."changeNameTo${methodName}" = {-> delegate.name = "Bob" }

def p = new Person()

assert "Fred" == p.name

p.changeNameToBob()

assert "Bob" == p.name
```

相同的概念可以应用于静态方法和属性。

动态方法名称的一个应用程序可以在Grails Web应用程序框架中找到。 “动态编解码器”的概念通过使用动态方法名称来实现。

`HTMLCodec` 类

```groovy
class HTMLCodec {
    static encode = { theTarget ->
        HtmlUtils.htmlEscape(theTarget.toString())
    }

    static decode = { theTarget ->
    	HtmlUtils.htmlUnescape(theTarget.toString())
    }
}
```

上面的示例显示了编解码器实现。 Grails有各种编解码器实现，每个在一个类中定义。 在运行时，应用程序类路径中将有多个编解码器类。 在应用程序启动时，框架向某些元类添加encodeXXX和decodeXXX方法，其中XXX是编解码器类名称的第一部分（例如，encodeHTML）。 这种机制在下面显示的一些Groovy伪代码中：

```groovy
def codecs = classes.findAll { it.name.endsWith('Codec') }

codecs.each { codec ->
    Object.metaClass."encodeAs${codec.name-'Codec'}" = { codec.newInstance().encode(delegate) }
    Object.metaClass."decodeFrom${codec.name-'Codec'}" = { codec.newInstance().decode(delegate) }
}


def html = '<html><body>hello</body></html>'

assert '<html><body>hello</body></html>' == html.encodeAsHTML()
```

###### 运行时发现

在运行时，知道在执行该方法时存在什么其他方法或属性通常是有用的。   `ExpandoMetaClass`在撰写本文时提供了以下方法：

* `getMetaMethod`
* `hasMetaMethod`
* `getMetaProperty`
* `hasMetaProperty`

你为什么不能使用反射？ 因为Groovy是不同的，它有方法是“真正的”方法，同时方法只在运行时可用。 这些有时（但不总是）表示为MetaMethods。 MetaMethod告诉你在运行时可用的方法，因此你的代码可以适应。

当覆盖`invokeMethod`，`getProperty`和/或`setProperty`时，这是特别有用的。

###### GroovyObject 方法

`ExpandoMetaClass`的另一个特点是它允许重写方法`invokeMethod`，`getProperty`和`setProperty`，所有这些都可以在`groovy.lang.GroovyObject`类中找到。

以下示例显示如何覆盖`invokeMethod`：
```groovy
class Stuff {
   def invokeMe() { "foo" }
}

Stuff.metaClass.invokeMethod = { String name, args ->
   def metaMethod = Stuff.metaClass.getMetaMethod(name, args)
   def result
   if(metaMethod) result = metaMethod.invoke(delegate,args)
   else {
      result = "bar"
   }
   result
}

def stf = new Stuff()

assert "foo" == stf.invokeMe()
assert "bar" == stf.doStuff()
```

`Closure`代码的第一步是查找给定名称和参数的`MetaMethod`。 如果方法可以找到一切都很好，它被委托。 如果不是，则返回一个虚拟值。


## 扩展模块
#### 扩展现有类
扩展模块允许您向现有类添加新方法，包括预编译的类，如JDK中的类。 这些新方法与通过元类或使用类定义的方法不同，在全局范围内可用。 例如，当您写：

标准扩展方法

```groovy
def file = new File(...)
def contents = file.getText('utf-8')
```

`File`类中不存在`getText`方法。 但是，Groovy可以使你调用它，因为它在一个特殊的类`ResourceGroovyMethods`中定义：

ResourceGroovyMethods.java

```groovy
public static String getText(File file, String charset) throws IOException {
    return IOGroovyMethods.getText(newReader(file, charset));
}
```

您可能会注意到，扩展方法是在辅助类（其中定义了各种扩展方法）中使用静态方法定义的。 `getText`方法的第一个参数对应于接收者，而附加参数对应于扩展方法的参数。 所以这里，我们在`File`类上定义一个名为 _getText_ 的方法（因为第一个参数是`File`类型），它接受一个参数作为参数（编码`String`）。

创建扩展模块的过程很简单：

* 编写一个类似上面的扩展类
* 写一个模块描述符文件

然后你必须让扩展模块对Groovy可见，这就像在classpath上有扩展模块类和描述符一样简单。 这意味着您可以选择：

* 直接在类路径上提供类和模块描述符
* 或将您的扩展模块捆绑到一个jar中以便重用

扩展模块可以向类添加两种方法：

* 实例方法（要在类的实例上调用）
* 静态方法（要在类本身上调用）

#### 实例方法

要向现有类添加实例方法，您需要创建扩展类。 例如，假设您要在`Integer`中添加一个`maxRetries`方法，该方法接受一个闭包并执行它最多n次，直到没有抛出异常。 要做到这一点，你只需要写如下：

MaxRetriesExtension.groovy

```groovy
class MaxRetriesExtension {       //                              扩展类
    static void maxRetries(Integer self, Closure code) {        //静态方法的第一个参数对应于消息的接收者，也就是说扩展实例
        int retries = 0
        Throwable e
        while (retries<self) {
            try {
                code.call()
                break
            } catch (Throwable err) {
                e = err
                retries++
            }
        }
        if (retries==0 && e) {
            throw e
        }
    }
}
```

然后，在声明你的扩展类之后，你可以这样调用：

```groovy
int i=0
5.maxRetries {
    i++
}
assert i == 1
i=0
try {
    5.maxRetries {
        throw new RuntimeException("oops")
    }
} catch (RuntimeException e) {
    assert i == 5
}
```

#### 静态方法

还可以向类添加静态方法。 在这种情况下，静态方法需要在其自己的文件中定义。 静态和实例扩展方法不能出现在同一个类中。

StaticStringExtension.groovy

```groovy
class StaticStringExtension {                //静态扩展类                   
    static String greeting(String self) {      //静态方法的第一个参数对应于正在扩展并且未使用的类                     
        'Hello, world!'
    }
}
```

在这种情况下，您可以直接在`String`类上调用它：
```groovy
assert String.greeting() == 'Hello, world!'
```

#### 模块描述符

要使Groovy能够加载您的扩展方法，必须声明扩展帮助器类。 您必须在`META-INF/services`目录中创建一个名为`org.codehaus.groovy.runtime.ExtensionModule`的文件：

org.codehaus.groovy.runtime.ExtensionModule

```
moduleName=Test module for specifications
moduleVersion=1.0-test
extensionClasses=support.MaxRetriesExtension
staticExtensionClasses=support.StaticStringExtension
```

模块描述符需要4个键：

* _moduleName_：模块的名称
* _moduleVersion_：您的模块的版本。 请注意，版本号仅用于检查您不加载同一模块在两个不同的版本。
* _extensionClasses_：实例方法的扩展辅助类列表。 您可以提供几个类，假定它们是逗号分隔的。
* _staticExtensionClasses_：静态方法的扩展辅助类列表。 您可以提供几个类，假定它们是逗号分隔的。

请注意，模块不需要定义静态辅助函数和实例辅助函数，并且可以向单个模块添加多个类。 您还可以在单个模块中扩展不同的类，而不会有问题。 甚至可能在单个扩展类中使用不同的类，但建议按扩展类将扩展方法分组。

#### 扩展模块和类路径

值得注意的是，你不能使用一个在使用它的代码同时编译的扩展。 这意味着要使用扩展，它必须在类路径上可用，作为编译类，在使用它的代码被编译之前。 通常，这意味着您不能将测试类与扩展类本身放在同一个源单元中。 因为一般来说，测试源与正常源分离并在构建的另一步骤中执行，这不是问题。

#### 与类型检查的兼容性

与Category不同，扩展模块与类型检查兼容：如果它们在类路径中找到，那么类型检查器知道扩展方法，并且不会在调用它们时抱怨。 它也与静态编译兼容。

