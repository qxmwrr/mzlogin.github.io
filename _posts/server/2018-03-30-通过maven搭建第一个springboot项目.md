---
layout: post
title: Maven搭建第一个SpringBoot项目
categories: [Server]
description: 通过IDEA，可以很轻松的直接创建一个基于SpringBoot开发的项目，但是对于小白来说，并不知道项目是由哪些依赖搭建起来的，通过本文可以简单了解这些知识
keywords: Server, SpringBoot Maven
date: 2018-03-30
author: 化作春泥
---

# 环境搭建
## `pom.xml`配置

通过IDEA创建一个Maven工程，创建完成后目录结构如下

<img src="/images/posts/server/project_dir.jpg" width="60%" alt="Project Dir"/>

`pom.xml`内容如下

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>cn.qiuximing</groupId>
    <artifactId>project1</artifactId>
    <version>1.0-SNAPSHOT</version>
   
</project>
```

### 添加`<parent>`

基于maven的spring boot项目中，通过使用`spring-boot-starter-parent`，可以获得一些更加合理的默认配置，它提供了一下特性：

* 默认使用Java8
* 使用UTF-8编码
* 一个引用管理功能，在dependencies里的部分配置可以不用填写version信息，这些version 信息会从spring-boot-dependencies里面继承。
* 智能资源过滤
* 智能插件配置（exec plugin, surefire, Git commit ID, shade）
* 能够识别application.properties和application.yml类型文件，同时也能支持profile-specific类型文件（application-dev.properties和application-dev.yml），因此能更好的配置不同的开发环境以及生产环境

Maven项目之间通过`<parent>`可以实现继承关系，示例如下

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.10.RELEASE</version>
    <relativePath/>
</parent>
```

### 添加`spring-boot-starter-web`

starter主要用来简化依赖，比如我们之前做MVC时要引入日志组件，就需要找到log4j的版本然后引用，有了starter之后，我们引入`spring-boot-starter-web`，log4j就自动引入了，不需要关心版本这些问题。

`pom.xml`中添加`spring-boot-starter-web`代码如下：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

### 添加`plugin`

添加spring-boot相关插件，代码如下：

```xml
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
</plugin>
```

### 最后`pom.xml`文件如下

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>cn.qiuximing</groupId>
    <artifactId>mymoney</artifactId>
    <version>1.0-SNAPSHOT</version>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.10.RELEASE</version>
        <relativePath/>
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

## 创建Application类

创建MyApplication类，注意Application类需要放到package下，不能直接放到main->java目录下，代码如下：

```java
package cn.qiuximing.mymoney;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class MyMoneyApplication {

    public static void main(String[] args) {
        SpringApplication.run(MyMoneyApplication.class, args);
    }
}
```

### 创建controller，实现hello spring boot

创建Controller，代码如下：

```java
package cn.qiuximing.mymoney.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class IndexController {

    @GetMapping("/hello")
    public String index() {
        return "hello spring boot";
    }
}
```

最终目录结构如下

<img src="/images/posts/server/project_finish" width="70%" alt="目录结构"/>


### 添加 Run/Debug Configurations启动项

如图：

<img src="/images/posts/server/run_config" width="70%" alt="启动项配置" />

最后运行，浏览器输入 http://localhost:8080/hello。开启你的spring boot之旅吧。


