---
layout: post
title: Groovy学习之-Groovy Development Kit（GDK）-IO操作
category: [groovy]
description: Groovy为I/O操作提供了许多帮助方法，虽然你可以在Groovy中用标准Java代码来实现I/O操作，不过Groovy提供了大量的方便的方式来操作File、Stream、Reader等等。
date: 2017-07-05
author: 化作春泥
---

> 本文首发于[《简书》](http://www.jianshu.com/p/6dac50fc564e)，转载请保留链接 : )

_Groovy学习目录-[传送门](http://www.jianshu.com/p/5a4ce49da37f)_

Groovy为I/O操作提供了许多帮助方法，虽然你可以在Groovy中用标准Java代码来实现I/O操作，不过Groovy提供了大量的方便的方式来操作File、Stream、Reader等等。

## 读取文件
读取文本文件并打印每一行文本
```java
new File(baseDir, 'haiku.txt').eachLine{ line ->
    println line
}
```
`eachLine`方法是Groovy为File类自动添加的方法，同时提供多个变体方法，比如你想知道读取的行数，你可以使用它的变体方法，如下
```java
new File(baseDir, 'haiku.txt').eachLine{ line, nb ->
    println "Line $nb: $line"
}
```
如果出于某种原因eachLine抛出异常，该方法能够确保资源正确地关闭，这适用于所有Groovy添加的I/O资源方法

例如在某些情况下,你更喜欢使用Reader访问I/O资源，但你仍然受益于Groovy的自动资源管理。在下一个示例中，即使发生了异常，Reader也将会被关闭。
```java
def count = 0, MAXSIZE = 3
new File(baseDir,"haiku.txt").withReader { reader ->
    while (reader.readLine()) {
        if (++count > MAXSIZE) {
            throw new RuntimeException('Haiku should only have 3 verses')
        }
    }
}
```
如果你需要收集文本文件的每一行到一个列表中，你可以这样做：
```java
def list = new File(baseDir, 'haiku.txt').collect {it}
```
或者你甚至可以用as操作符来讲文件内容转为String数组
```java
def array = new File(baseDir, 'haiku.txt') as String[]
```
多少次你不得不获得一个文件的内容到一个byte[]，它需要多少代码？Groovy使得这件事非常的容易。
```java
byte[] contents = file.bytes
```
处理I/O并不局限于处理文件。事实上,很多的操作依赖于输入/输出流,这就是为什么Groovy添加大量的支持方法,正如你所看到的文档。

例如，你可以很容易从一个文件获得一个InputStream
```java
def is = new File(baseDir,'haiku.txt').newInputStream()
// do something ...
is.close()
```
但是你可以看到，这种方式需要你手动关闭流。会为你留意到，在Groovy中使用withInputStream通常是更好的方式。
```java
new File(baseDir,'haiku.txt').withInputStream { stream ->
    // do something ...
}
```

## 写文件
当然，一些时候你并不想读取文件，而是要写文件。其中一个方式就是使用Writer：
```java
new File(baseDir,'haiku.txt').withWriter('utf-8') { writer ->
    writer.writeLine 'Into the ancient pond'
    writer.writeLine 'A frog jumps'
    writer.writeLine 'Water’s sound!'
}
```
但是对于这样一个简单的示例中,使用<<操作符就足够了：
```java
new File(baseDir,'haiku.txt') << '''Into the ancient pond
A frog jumps
Water’s sound!'''
```
当然我们并不总是处理文本内容,所以您可以使用Writer或者直接写入字节，例如：
```java
file.bytes = [66,22,11]
```
当然你也可以直接处理输出流。例如,下面就是如何创建一个输出流写入一个文件：
```java
def os = new File(baseDir,'data.bin').newOutputStream()
// do something ...
os.close()
```
但是你可以看到它需要你手动关闭输出流。会为你留意到，在Groovy中使用withOutputStream通常是更好的方式。
```java
new File(baseDir,'data.bin').withOutputStream { stream ->
    // do something ...
}
```

## 遍历文件树
在脚本中，在文件树种查找一些特定文件并处理这些文件是很常见的事情。Groovy提供了多种方法来做到这一点。例如你可以为一个文件夹中的每个文件直行一些操作：
```java
//在目录的每一个文件直行闭包代码
dir.eachFile { file ->                      
    println file.name
}

//在目录中符合匹配模式的文件直行闭包代码
dir.eachFileMatch(~/.*\.txt/) { file ->     
    println file.name
}
```
通常你需要处理一个更深的目录结构的文件，这种情况下你可以使用`eachFileRecurse`
```java
//在指定目录递归查找，并在每一个文件和目录直行闭包代码
dir.eachFileRecurse { file ->                      
    println file.name
}

//在指定目录递归查找，并在每一个文件直行闭包代码
dir.eachFileRecurse(FileType.FILES) { file ->      
    println file.name
}
```
对于更复杂的遍历技术可以使用`traverse`方法，它你需要设置一个特殊的标志指示这个遍历要做什么。
```java
dir.traverse { file ->
    if (file.directory && file.name=='bin') {
    		//如果当前file是一个目录或者它的名字是 bin ，则停止遍历
        FileVisitResult.TERMINATE                   
    } else {
    		//打印文件名并继续遍历
        println file.name
        FileVisitResult.CONTINUE                    
    }

}
```

## 数据和对象
在Java中使用`java.io.DataOutputStream`和`java.io.DataInputStream`进行序列化和反序列化并不少见，Groovy将使它更容易处理。例如,您可以把数据序列化到一个文件中并反序列化它使用以下代码：
```java
boolean b = true
String message = 'Hello from Groovy'
// Serialize data into a file
file.withDataOutputStream { out ->
    out.writeBoolean(b)
    out.writeUTF(message)
}
// ...
// Then read it back
file.withDataInputStream { input ->
    assert input.readBoolean() == b
    assert input.readUTF() == message
}
```
类似的，如果您想要序列化的数据实现了Serializable接口，你可以将它作为一个Object输出流处理，例如：
```java
Person p = new Person(name:'Bob', age:76)
// Serialize data into a file
file.withObjectOutputStream { out ->
    out.writeObject(p)
}
// ...
// Then read it back
file.withObjectInputStream { input ->
    def p2 = input.readObject()
    assert p2.name == p.name
    assert p2.age == p.age
}
```

## 执行外部程序
前一节中描述的是使用Groovy处理文件多么容易。然而在系统管理等领域或devops通常需要与外部processes沟通。Groovy提供了一个简单的方法来执行命令行processes。仅仅需要些命令字符串再调用`execute()`方法即可。如。在 *nix机（安装了合适的 *nix命令工具的windows机），你这样执行：
```java
//在外部process执行`ls`命令
def process = "ls -l".execute()
//读取文本并打印
println "Found text ${process.text}"
```
`execute()`方法返回一个`java.lang.Process`对象，并允许处理它的in/out/err流和退出值，从Process检查等。
如，这是和上面一样的命令
```java
//执行`ls`命令
def process = "ls -l".execute()             
//遍历命令进程的输入流
process.in.eachLine { line ->               
	  //打印每一行输入流
    println line                            
}
```
值得注意的是，in是一个输入流，对应于命令的标准输出。 out指的是可以向Process发送数据的流（标准输入）。
记住许多命令是shell内置函数，需要特殊处理。所以，如果你想要在一个Windows机器上列出一个目录下的所有文件：
```java
def process = "dir".execute()
println "${process.text}"
```
你会收到一个`IOException`说无法运行程序`“dir”：CreateProcess error = 2`，系统找不到指定的文件。这是因为`dir`内置在Windows shell（`cmd.exe`）中，不能作为简单的可执行文件运行。 相反，你需要写：
```java
def process = "cmd /c dir".execute()
println "${process.text}"
```
此外，由于此功能当前使用`java.lang.Process`底层，必须考虑该类的缺陷。尤其是，这个类的javadoc说：
```
由于一些本地平台仅为标准输入和输出流提供有限的缓冲区大小，因此无法及时写入输入流或读取子过程的输出流可能导致子过程阻塞甚至死锁
```
正因为如此，Groovy提供了一些额外的帮助方法，使得流处理更容易。
这里是如何从你的Process中`gobble(未翻译)`所有的输出（包括错误流输出）：
```java
def p = "rm -f foo.tmp".execute([], tmpDir)
p.consumeProcessOutput()
p.waitFor()
```
还有`consumeProcessOutput`的变体，使用`StringBuffer`，`InputStream`，`OutputStream`等...有关完整的列表，请阅读[GDK API for java.lang.Process](http://docs.groovy-lang.org/latest/html/groovy-jdk/java/lang/Process.html)
此外，这些是一个`pipeTo命令`（映射到`|`以允许重载），它使一个进程的输出流被传送到另一个进程的输入流。
这里有一些使用的例子：
```java
proc1 = 'ls'.execute()
proc2 = 'tr -d o'.execute()
proc3 = 'tr -d e'.execute()
proc4 = 'tr -d i'.execute()
proc1 | proc2 | proc3 | proc4
proc4.waitFor()
if (proc4.exitValue()) {
    println proc4.err.text
} else {
    println proc4.text
}
```
消耗错误
```java
def sout = new StringBuilder()
def serr = new StringBuilder()
proc2 = 'tr -d o'.execute()
proc3 = 'tr -d e'.execute()
proc4 = 'tr -d i'.execute()
proc4.consumeProcessOutput(sout, serr)
proc2 | proc3 | proc4
[proc2, proc3].each { it.consumeProcessErrorStream(serr) }
proc2.withWriter { writer ->
    writer << 'testfile.groovy'
}
proc4.waitForOrKill(1000)
println "Standard output: $sout"
println "Standard error: $serr"
```

