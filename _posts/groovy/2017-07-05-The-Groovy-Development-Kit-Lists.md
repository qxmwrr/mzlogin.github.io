---
layout: post
title: Groovy学习之-Groovy Development Kit（GDK）-集合操作
category: [groovy]
description: Groovy为各种集合类型提供native支持，包括List，Map和Ranges。 其中大多数基于Java集合类型，并使用Groovy开发包中提供的其他方法进行了装饰。
keywords: [groovy, java]
date: 2017-07-05
author: 化作春泥
---


> 本文首发于[《简书》](http://www.jianshu.com/p/c591c139efbc)，转载请保留链接 : )

_Groovy学习目录-[传送门](http://www.jianshu.com/p/5a4ce49da37f)_

Groovy为各种集合类型提供native支持，包括[List](http://www.groovy-lang.org/groovy-dev-kit.html#Collections-Lists)，[Map](http://www.groovy-lang.org/groovy-dev-kit.html#Collections-Maps)和[Ranges](http://www.groovy-lang.org/groovy-dev-kit.html#Collections-Ranges)。 其中大多数基于Java集合类型，并使用[Groovy开发包](http://www.groovy-lang.org/gdk.html)中提供的其他方法进行了装饰。

## Lists

#### List 字面值
您可以按如下所示创建列表。 请注意，[]是空列表表达式。
```groovy
def list = [5, 6, 7, 8]
assert list.get(2) == 7
assert list[2] == 7
assert list instanceof java.util.List

def emptyList = []
assert emptyList.size() == 0
emptyList.add(5)
assert emptyList.size() == 1
```
每个列表表达式都是创建[java.util.List](http://docs.oracle.com/javase/8/docs/api/java/util/List.html)的实例。

当然，一个list可以用作构造另一个list的源：
```java
def list1 = ['a', 'b', 'c']
//构造一个新的List，这个List和list1有相同的items
def list2 = new ArrayList<String>(list1)

assert list2 == list1 // == 检测每一个对应的item，判断它们是否相同

// clone() 也是可以使用的
def list3 = list1.clone()
assert list3 == list1
```
list是objects的有序集合：
```java
def list = [5, 6, 7, 8]
assert list.size() == 4
assert list.getClass() == ArrayList     //所使用的列表的具体类型

assert list[2] == 7                     // 索引是从0开始的
assert list.getAt(2) == 7               // 同[]运算符
assert list.get(2) == 7                 // 替代方法
list[2] = 9
assert list == [5, 6, 9, 8,]           //结果通过

list.putAt(2, 10)                       //等效于 list[2] = 10
assert list == [5, 6, 10, 8]
assert list.set(2, 11) == 10            // 赋值并返回原值
assert list == [5, 6, 11, 8]

assert ['a', 1, 'a', 'a', 2.5, 2.5f, 2.5d, 'hello', 7g, null, 9 as byte]
//元素可以是不同类型; 允许重复
assert [1, 2, 3, 4, 5][-1] == 5             // 允许负数index，从list尾部开始计数
assert [1, 2, 3, 4, 5][-2] == 4
assert [1, 2, 3, 4, 5].getAt(-2) == 4       // getAt() 可以使用负数index
try {
    [1, 2, 3, 4, 5].get(-2)                 // 但是get()方法不允许使用负数index
    assert false
} catch (e) {
    assert e instanceof ArrayIndexOutOfBoundsException
}
```

#### List作为布尔表达式
可以将列表计算为`boolean`：
```java
// 空list = false
assert ![]

//所有有内容的list都 = true
assert [1] && ['a'] && [0] && [0.0] && [false] && [null]
```

#### List迭代
迭代列表的元素通常是通过调用`each`和`eachWithIndex`方法，它们对列表的每个项执行代码：
```java
[1, 2, 3].each {
    println "Item: $it"//it是对应于当前元素的隐式参数
}
['a', 'b', 'c'].eachWithIndex { it, i -> //it是当前元素, i是索引位置
    println "$i: $it"
}
```
除了迭代之外，通过将每个元素转换为其他元素来创建新的List通常是很有用的。 这个操作，通常称为映射，在Groovy中通过`collect`方法完成：
```java
assert [1, 2, 3].collect { it * 2 } == [2, 4, 6]

//简洁语法
assert [1, 2, 3]*.multiply(2) == [1, 2, 3].collect { it.multiply(2) }

def list = [0]
//可以给“collect”传入list参数，收集元素的列表
assert [1, 2, 3].collect(list) { it * 2 } == [0, 2, 4, 6]
assert list == [0, 2, 4, 6]
```

####list操作
######过滤和搜索
[Groovy开发工具包](http://www.groovy-lang.org/gdk.html)包含许多集合的方法，通过这些方法增强标准集合的功能，其中一些如下所示：
```groovy
assert [1, 2, 3].find { it > 1 } == 2           // 找出第一个符合条件的元素
assert [1, 2, 3].findAll { it > 1 } == [2, 3]   //找出所有符合条件的元素
assert ['a', 'b', 'c', 'd', 'e'].findIndexOf {      // 找出符合条件的第一个元素的index
    it in ['c', 'e', 'g']
} == 2

assert ['a', 'b', 'c', 'd', 'c'].indexOf('c') == 2  // 返回index
assert ['a', 'b', 'c', 'd', 'c'].indexOf('z') == -1 // index返回-1意味着没有找到结果
assert ['a', 'b', 'c', 'd', 'c'].lastIndexOf('c') == 4

assert [1, 2, 3].every { it < 5 }               // 如果每一个元素都符合条件则返回true
assert ![1, 2, 3].every { it < 3 }
assert [1, 2, 3].any { it > 2 }                 // 如果有一个元素符合条件就返回true
assert ![1, 2, 3].any { it > 3 }

assert [1, 2, 3, 4, 5, 6].sum() == 21                // 所有元素求和
assert ['a', 'b', 'c', 'd', 'e'].sum {
    it == 'a' ? 1 : it == 'b' ? 2 : it == 'c' ? 3 : it == 'd' ? 4 : it == 'e' ? 5 : 0
    // 求和的时候可以自定义元素的值
} == 15
assert ['a', 'b', 'c', 'd', 'e'].sum { ((char) it) - ((char) 'a') } == 10
assert ['a', 'b', 'c', 'd', 'e'].sum() == 'abcde'
assert [['a', 'b'], ['c', 'd']].sum() == ['a', 'b', 'c', 'd']

// 可以提供初始值
assert [].sum(1000) == 1000
assert [1, 2, 3].sum(1000) == 1006

assert [1, 2, 3].join('-') == '1-2-3'           // 每个元素之间添加字符串
assert [1, 2, 3].inject('counting: ') { str, item -> 
    str + item                     // 减少操作
} == 'counting: 123'
assert [1, 2, 3].inject(0) { count, item ->
    count + item
} == 6
```

这里是用于在集合中查找最大和最小值的惯用Groovy代码：
```groovy
def list = [9, 4, 2, 10, 5]
assert list.max() == 10
assert list.min() == 2

// 单字符的list也可以查找最大值和最小值
assert ['x', 'y', 'a', 'z'].min() == 'a'

// 我们可以用Closure闭包来描述元素的大小
def list2 = ['abc', 'z', 'xyzuvw', 'Hello', '321']
assert list2.max { it.size() } == 'xyzuvw'
assert list2.min { it.size() } == 'z'
```
除了闭包之外，您还可以使用`Comparator`来定义比较条件：
```groovy
Comparator mc = { a, b -> a == b ? 0 : (a < b ? -1 : 1) }

def list = [7, 4, 9, -6, -1, 11, 2, 3, -9, 5, -13]
assert list.max(mc) == 11
assert list.min(mc) == -13

Comparator mc2 = { a, b -> a == b ? 0 : (Math.abs(a) < Math.abs(b)) ? -1 : 1 }


assert list.max(mc2) == -13
assert list.min(mc2) == -1

assert list.max { a, b -> a.equals(b) ? 0 : Math.abs(a) < Math.abs(b) ? -1 : 1 } == -13
assert list.min { a, b -> a.equals(b) ? 0 : Math.abs(a) < Math.abs(b) ? -1 : 1 } == -1
```

######添加或删除元素
我们可以使用`[]`分配一个新的空List，使用`<<`为List添加项目：
```groovy
def list = []
assert list.empty

list << 5
assert list.size() == 1

list << 7 << 'i' << 11
assert list == [5, 7, 'i', 11]

list << ['m', 'o']
assert list == [5, 7, 'i', 11, ['m', 'o']]

//在<<表达式最前端的list是目标list
assert ([1, 2] << 3 << [4, 5] << 6) == [1, 2, 3, [4, 5], 6]

//使用leftShift方法等价于使用 <<
assert ([1, 2, 3] << 4) == ([1, 2, 3].leftShift(4))
```
我们可以通过多种方式添加到List中：
```groovy
assert [1, 2] + 3 + [4, 5] + 6 == [1, 2, 3, 4, 5, 6]
// 等价于调用plus方法
assert [1, 2].plus(3).plus([4, 5]).plus(6) == [1, 2, 3, 4, 5, 6]

def a = [1, 2, 3]
a += 4      //创建了一个新的List
a += [5, 6]
assert a == [1, 2, 3, 4, 5, 6]

assert [1, *[222, 333], 456] == [1, 222, 333, 456]
assert [*[1, 2, 3]] == [1, 2, 3]
assert [1, [2, 3, [4, 5], 6], 7, [8, 9]].flatten() == [1, 2, 3, 4, 5, 6, 7, 8, 9]

def list = [1, 2]
list.add(3)
list.addAll([5, 4])
assert list == [1, 2, 3, 5, 4]

list = [1, 2]
list.add(1, 3) //在索引1前面插入元素3
assert list == [1, 3, 2]

list.addAll(2, [5, 4]) //在索引2前面插入元素[5,4]
assert list == [1, 3, 5, 4, 2]

list = ['a', 'b', 'z', 'e', 'u', 'v', 'g']
list[8] = 'x' // []运算符根据需要使列表增长
// 如果需要插入null
assert list == ['a', 'b', 'z', 'e', 'u', 'v', 'g', null, 'x']
```
然而，重要的是List上的`+`运算符不会改变List本身。 与`<<`相比，`+`运算符会创建一个新的列表，这通常不是你想要的，并可能导致性能问题。

[Groovy开发包](http://www.groovy-lang.org/gdk.html)还包含一些方法，使您可以通过元素值轻松地从列表中删除元素：
```groovy
assert ['a','b','c','b','b'] - 'c' == ['a','b','b','b']
assert ['a','b','c','b','b'] - 'b' == ['a','c']
assert ['a','b','c','b','b'] - ['b','c'] == ['a']

def list = [1,2,3,4,3,2,1]
list -= 3           //从原始list创建一个新的list，并删除元素3
assert list == [1,2,4,2,1]
assert ( list -= [2,4] ) == [1,1]
```
也可以通过引用其索引来删除元素，在这种情况下，列表会改变：
```groovy
def list = [1,2,3,4,5,6,2,2,1]
assert list.remove(2) == 3          //删除第三个元素并返回第三个元素的值
assert list == [1,2,4,5,6,2,2,1]
```
如果你只想删除列表中具有相同值的第一个元素，而不是删除所有元素，则调用`remove`方法：
```groovy
def list= ['a','b','c','b','b']
assert list.remove('c')             // 删除元素'c'如果删除成功返回true
assert list.remove('b')             // 删除第一个找到的元素'b'，如果删除成功返回true
assert ! list.remove('z')           // 返回false，因为没有任何元素删除
assert list == ['a','b','b']
```
删除列表中的所有元素可以通过调用`clear`方法来完成：
```groovy
def list= ['a',2,'c',4]
list.clear()
assert list == []
```

######设置操作
[Groovy开发工具包](http://www.groovy-lang.org/gdk.html)还包括一些方法，使得它易于推理：
```groovy
assert 'a' in ['a','b','c']             // 如果元素'a'在list中返回true
assert ['a','b','c'].contains('a')      // 等价于java中的`contains`方法
assert [1,3,4].containsAll([1,4])       // `containsAll` 将检测每一个待查元素，如果都包含在list中，返回true

assert [1,2,3,3,3,3,4,5].count(3) == 4  // 返回元素3在列表中包含的数量
assert [1,2,3,3,3,3,4,5].count {
    it%2==0                             // 返回符合断言的元素在列表中包含的数量
} == 2

assert [1,2,4,6,8,10,12].intersect([1,3,6,9,12]) == [1,6,12]//返回两个列表的交集

assert [1,2,3].disjoint( [4,6,9] )//两个列表是互斥的，返回true
assert ![1,2,3].disjoint( [2,4,6] )
```
######排序
使用List经常会遇到排序。 Groovy提供了各种选项来排序List，从使用闭包到comparators，如下例所示：
```groovy
assert [6, 3, 9, 2, 7, 1, 5].sort() == [1, 2, 3, 5, 6, 7, 9]

def list = ['abc', 'z', 'xyzuvw', 'Hello', '321']
assert list.sort {
    it.size()
} == ['z', 'abc', '321', 'Hello', 'xyzuvw']

def list2 = [7, 4, -6, -1, 11, 2, 3, -9, 5, -13]
assert list2.sort { a, b -> a == b ? 0 : Math.abs(a) < Math.abs(b) ? -1 : 1 } ==
        [-1, 2, 3, 4, 5, -6, 7, -9, 11, -13]

Comparator mc = { a, b -> a == b ? 0 : Math.abs(a) < Math.abs(b) ? -1 : 1 }

// 仅限于JDK 8+
// list2.sort(mc)
// assert list2 == [-1, 2, 3, 4, 5, -6, 7, -9, 11, -13]

def list3 = [6, -3, 9, 2, -7, 1, 5]

Collections.sort(list3)
assert list3 == [-7, -3, 1, 2, 5, 6, 9]

Collections.sort(list3, mc)
assert list3 == [1, 2, -3, 5, 6, -7, 9]
```

######复制元素
[Groovy开发工具包](http://www.groovy-lang.org/gdk.html)还利用了操作符重载，以提供允许复制列表元素的方法：
```groovy
assert [1, 2, 3] * 3 == [1, 2, 3, 1, 2, 3, 1, 2, 3]
assert [1, 2, 3].multiply(2) == [1, 2, 3, 1, 2, 3]
assert Collections.nCopies(3, 'b') == ['b', 'b', 'b']

// 来自JDK的nCopies具有与列表的乘法不同的语义
assert Collections.nCopies(2, [1, 2]) == [[1, 2], [1, 2]] //而不是[1,2,1,2]
```

##Maps
####Map 字面值
在Groovy中，可以使用map语法创建map（也称为关联数组）：`[:]`：
```groovy
def map = [name: 'Gromit', likes: 'cheese', id: 1234]
assert map.get('name') == 'Gromit'
assert map.get('id') == 1234
assert map['name'] == 'Gromit'
assert map['id'] == 1234
assert map instanceof java.util.Map

def emptyMap = [:]
assert emptyMap.size() == 0
emptyMap.put("foo", 5)
assert emptyMap.size() == 1
assert emptyMap.get("foo") == 5
```
默认情况下，map的key是字符串：`[a:1]`等价于`['a':1]`。 如果您定义一个名为a的变量，并且您希望a的值为您的map的key，这可能会令人困惑。 如果是这种情况，则必须通过添加括号来转义键，如以下示例所示：
```groovy
def a = 'Bob'
def ages = [a: 43]
assert ages['Bob'] == null // 找不到`Bob`
assert ages['a'] == 43     // 因为 `a` 是字面值（文本）!

ages = [(a): 43]            // 将`a`用()括起来
assert ages['Bob'] == 43   // 这时就能找到'Bob'的值了
```
除了map 字面值外, 也可以通过clone方法得到一个map的副本：
```groovy
def map = [
        simple : 123,
        complex: [a: 1, b: 2]
]
def map2 = map.clone()
assert map2.get('simple') == map.get('simple')
assert map2.get('complex') == map.get('complex')
map2.get('complex').put('c', 3)
assert map.get('complex').get('c') == 3
```
如前面的示例所示，生成的map是原始map的浅拷贝

####Map属性符号
Maps也像bean一样，所以你可以使用属性符号在`Map`中get/set项，只要键是有效的Groovy标识符的字符串即可：
```groovy
def map = [name: 'Gromit', likes: 'cheese', id: 1234]
assert map.name == 'Gromit'     // 同map.get('Gromit')
assert map.id == 1234

def emptyMap = [:]
assert emptyMap.size() == 0
emptyMap.foo = 5
assert emptyMap.size() == 1
assert emptyMap.foo == 5
```

注意：根据设计，`map.foo`会一直在map中查找这个key。这意味着`foo.class`将在不包含`class`键的map上返回null。如果你是想返回foo的类型，那么你必须使用foo.getClass()方法：
```groovy
def map = [name: 'Gromit', likes: 'cheese', id: 1234]
assert map.class == null
assert map.get('class') == null
assert map.getClass() == LinkedHashMap // 这很可能是你想要的结果

map = [1      : 'a',
       (true) : 'p',
       (false): 'q',
       (null) : 'x',
       'null' : 'z']
assert map.containsKey(1) // 数字1不是一个标识符，所以得这样调用
assert map.true == null
assert map.false == null
assert map.get(true) == 'p'
assert map.get(false) == 'q'
assert map.null == 'z'
assert map.get(null) == 'x'
```

####Map迭代
照例，在[Groovy开发工具包](http://www.groovy-lang.org/gdk.html)中，Map上的惯用迭代使用`each`和`eachWithIndex`方法。 值得注意的是，使用Map字面值符号创建的Map是有序的，也就是说，如果对Map条目进行迭代，将保证条目将按照它们在Map中添加的顺序返回。
```groovy
def map = [
        Bob  : 42,
        Alice: 54,
        Max  : 33
]

// `entry` is a map entry
map.each { entry ->
    println "Name: $entry.key Age: $entry.value"
}

// `entry` is a map entry, `i` the index in the map
map.eachWithIndex { entry, i ->
    println "$i - Name: $entry.key Age: $entry.value"
}

// Alternatively you can use key and value directly
map.each { key, value ->
    println "Name: $key Age: $value"
}

// Key, value and i as the index in the map
map.eachWithIndex { key, value, i ->
    println "$i - Name: $key Age: $value"
}
```

####Map操作
######添加或删除元素
向Map中添加元素可以使用`put`方法，下标运算符或使用`putAll`：
```groovy
def defaults = [1: 'a', 2: 'b', 3: 'c', 4: 'd']
def overrides = [2: 'z', 5: 'x', 13: 'x']

def result = new LinkedHashMap(defaults)
result.put(15, 't')
result[17] = 'u'
result.putAll(overrides)
assert result == [1: 'a', 2: 'z', 3: 'c', 4: 'd', 5: 'x', 13: 'x', 15: 't', 17: 'u']
```
删除Map的所有元素可以通过调用clear方法来完成：
```groovy
def m = [1:'a', 2:'b']
assert m.get(1) == 'a'
m.clear()
assert m == [:]
```
使用Map字面值语法生成的Map使用object的`equals`和`hashcode`方法。 这意味着你不应该使用哈希码随时间变化的object，否则你将无法获得相关的值。

还值得注意的是，您不应该使用`GString`作为Map的键，因为`GString`的哈希码与等效`String`的哈希码不同：
```groovy
def key = 'some key'
def map = [:]
def gstringKey = "${key.toUpperCase()}"
map.put(gstringKey,'value')
assert map.get('SOME KEY') == null
```

######Keys, values and entries
我们可以检查视图中的keys, values, and entries：
```groovy
def map = [1:'a', 2:'b', 3:'c']

def entries = map.entrySet()
entries.each { entry ->
  assert entry.key in [1,2,3]
  assert entry.value in ['a','b','c']
}

def keys = map.keySet()
assert keys == [1,2,3] as Set
```

由于操作的成功直接取决于正在操作的Map的类型，因此视图（不管是map的entry，key还是value）返回的变化值是非常不鼓励的。 特别地，Groovy依赖于来自JDK的集合，通常不能保证集合可以通过`keySet`，`entrySet`或`values`安全地操作。

######过滤和搜索
[Groovy开发包](http://www.groovy-lang.org/gdk.html)包含与[List](http://www.groovy-lang.org/groovy-dev-kit.html#List-Filtering)中类似的过滤，搜索和收集方法：
```groovy
def people = [
    1: [name:'Bob', age: 32, gender: 'M'],
    2: [name:'Johnny', age: 36, gender: 'M'],
    3: [name:'Claire', age: 21, gender: 'F'],
    4: [name:'Amy', age: 54, gender:'F']
]

def bob = people.find { it.value.name == 'Bob' } // 查找单个entry
def females = people.findAll { it.value.gender == 'F' }

//两个都是返回entries，但是您可以使用collect来检索年龄例如
def ageOfBob = bob.value.age
def agesOfFemales = females.collect {
    it.value.age
}

assert ageOfBob == 32
assert agesOfFemales == [21,54]

// 但您也可以使用键/对值作为闭包的参数but you could also use a key/pair value as the parameters of the closures
def agesOfMales = people.findAll { id, person ->
    person.gender == 'M'
}.collect { id, person ->
    person.age
}
assert agesOfMales == [32, 36]

// `every` 如果所有entries都匹配规则，则返回true
assert people.every { id, person ->
    person.age > 18
}

// `any` 如果任何一个entry匹配规则，则返回true
assert people.any { id, person ->
    person.age == 54
}
```

######分组
我们可以使用一些标准将List分组到Map中：
```groovy
assert ['a', 7, 'b', [2, 3]].groupBy {
    it.class
} == [(String)   : ['a', 'b'],
      (Integer)  : [7],
      (ArrayList): [[2, 3]]
]

assert [
        [name: 'Clark', city: 'London'], [name: 'Sharma', city: 'London'],
        [name: 'Maradona', city: 'LA'], [name: 'Zhang', city: 'HK'],
        [name: 'Ali', city: 'HK'], [name: 'Liu', city: 'HK'],
].groupBy { it.city } == [
        London: [[name: 'Clark', city: 'London'],
                 [name: 'Sharma', city: 'London']],
        LA    : [[name: 'Maradona', city: 'LA']],
        HK    : [[name: 'Zhang', city: 'HK'],
                 [name: 'Ali', city: 'HK'],
                 [name: 'Liu', city: 'HK']],
]
```

##Ranges
Ranges允许您创建顺序值List。 这些可以用作`List`，因为[Range](http://docs.groovy-lang.org/latest/html/api/groovy/lang/Range.html)扩展了[java.util.List](http://docs.oracle.com/javase/8/docs/api/java/util/List.html)。

使用`..`符号定义的范围是包含性的（即List包含from和to值）。

使用`.. <`符号定义的范围是半开的，它们包括第一个值，但不是最后一个值。
```groovy
// 全包含的range
def range = 5..8
assert range.size() == 4
assert range.get(2) == 7
assert range[2] == 7
assert range instanceof java.util.List
assert range.contains(5)
assert range.contains(8)

// 半开的range
range = 5..<8
assert range.size() == 3
assert range.get(2) == 7
assert range[2] == 7
assert range instanceof java.util.List
assert range.contains(5)
assert !range.contains(8)

//获取range的端点而不使用索引
range = 1..10
assert range.from == 1
assert range.to == 10
```

请注意，int类型的Range实现了高效率，创建了一个包含from和to值的轻量级Java对象。

Ranges可以用于实现java.lang.Comparable以进行比较的任何Java对象，并且还有方法`next()`和`previous()`返回range中的下一个/上一个项目。 例如，您可以创建一系列String元素：
```groovy
// 全包括range
def range = 'a'..'d'
assert range.size() == 4
assert range.get(2) == 'c'
assert range[2] == 'c'
assert range instanceof java.util.List
assert range.contains('a')
assert range.contains('d')
assert !range.contains('e')
```
您可以使用经典的for循环在一个range上进行迭代：
```groovy
for (i in 1..10) {
    println "Hello ${i}"
}
```
但是或者，您可以通过使用`each`方法迭代Range，在更加Groovy惯用的风格中实现相同的效果：
```groovy
(1..10).each { i ->
    println "Hello ${i}"
}
```
Range也可以用在switch语句中：
```groovy
switch (years) {
    case 1..10: interestRate = 0.076; break;
    case 11..25: interestRate = 0.052; break;
    default: interestRate = 0.037;
}
```

##集合的语法增强
####GPath支持
由于对List和Map的属性符号支持，Groovy提供了语法糖，使得处理嵌套集合变得非常容易，如下例所示：
```groovy
def listOfMaps = [['a': 11, 'b': 12], ['a': 21, 'b': 22]]
assert listOfMaps.a == [11, 21] //GPath 标记
assert listOfMaps*.a == [11, 21] //扩展点符号

listOfMaps = [['a': 11, 'b': 12], ['a': 21, 'b': 22], null]
assert listOfMaps*.a == [11, 21, null] // 适用于空值
assert listOfMaps*.a == listOfMaps.collect { it?.a } //等价符号
// 但这只会收集非空值
assert listOfMaps.a == [11,21]
```

####扩展运算符
扩展运算符可以用于将集合“内联”到另一个集合中。 它是语法糖，通常避免调用`putAll`并促进一行的实现：
```groovy
assert [ 'z': 900,
         *: ['a': 100, 'b': 200], 'a': 300] == ['a': 300, 'b': 200, 'z': 900]
//在map定义中的扩展map符号
assert [*: [3: 3, *: [5: 5]], 7: 7] == [3: 3, 5: 5, 7: 7]

def f = { [1: 'u', 2: 'v', 3: 'w'] }
assert [*: f(), 10: 'zz'] == [1: 'u', 10: 'zz', 2: 'v', 3: 'w']
//函数参数中的扩展map符号
f = { map -> map.c }
assert f(*: ['a': 10, 'b': 20, 'c': 30], 'e': 50) == 30

f = { m, i, j, k -> [m, i, j, k] }
//使用具有混合未命名和命名参数的展开map符号
assert f('e': 100, *[4, 5], *: ['a': 10, 'b': 20, 'c': 30], 6) ==
        [["e": 100, "b": 20, "c": 30, "a": 10], 4, 5, 6]
```

####星号“*”运算符
“星点”运算符是一个快捷运算符，允许您对集合的所有元素调用方法或属性：
```groovy
assert [1, 3, 5] == ['a', 'few', 'words']*.size()

class Person {
    String name
    int age
}
def persons = [new Person(name:'Hugo', age:17), new Person(name:'Sandra',age:19)]
assert [17, 19] == persons*.age
```

####使用下标运算符切片
您可以使用下标表达式将其索引到list，数组和map中。 有趣的是，字符串在这种情况下被视为特殊类型的集合：
```groovy
def text = 'nice cheese gromit!'
def x = text[2]

assert x == 'c'
assert x.class == String

def sub = text[5..10]
assert sub == 'cheese'

def list = [10, 11, 12, 13]
def answer = list[2,3]
assert answer == [12,13]
```
请注意，您可以使用Range提取集合的一部分：
```groovy
list = 100..200
sub = list[1, 3, 20..25, 33]
assert sub == [101, 103, 120, 121, 122, 123, 124, 125, 133]
```
下标运算符可用于更新现有集合（对于不可变的集合类型）：
```groovy
list = ['a','x','x','d']
list[1..2] = ['b','c']
assert list == ['a','b','c','d']
```
值得注意的是，允许使用负索引，从集合的末尾更容易提取：
您可以使用负指数从List，array，String等结尾计数。
```groovy
text = "nice cheese gromit!"
x = text[-1]
assert x == "!"

def name = text[-7..-2]
assert name == "gromit"
```
最终，如果使用向后Range（开始索引大于结束索引），则答案将反过来。
```groovy
text = "nice cheese gromit!"
name = text[3..1]
assert name == "eci"
```

##增强的集合方法
除了List，Map或Ranges，Groovy还提供了许多额外的过滤，收集，分组，计数等方法，这些方法可以直接在集合上使用，或者更容易迭代。

特别是，我们邀请您阅读[Groovy开发包API文档](http://www.groovy-lang.org/gdk.html)，具体来说：

* 添加到`Iterable`的方法可以从[这里](http://docs.groovy-lang.org/latest/html/groovy-jdk/java/lang/Iterable.html)找到
* 添加到`Iterator`的方法可以从[这里](http://docs.groovy-lang.org/latest/html/groovy-jdk/java/util/Iterator.html)找到
* 添加到`Collection`的方法可以从[这里](http://docs.groovy-lang.org/latest/html/groovy-jdk/java/util/Collection.html)找到
* 添加到`List`的方法可以从[这里](http://docs.groovy-lang.org/latest/html/groovy-jdk/java/util/List.html)找到
* 添加到`Map`的方法可以从[这里](http://docs.groovy-lang.org/latest/html/groovy-jdk/java/util/Map.html)找到


_ps: literals这个单词应该翻译成什么更合适些呢，有朋友知道的话可以留言List literals 我翻译成了 List字面值 显然是不对的_

