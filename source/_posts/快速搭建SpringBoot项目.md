---
title: 快速搭建SpringBoot项目
date: 2018-07-14
tags: [sprint boot]
categories: [spring boot]
---

#### Spring Boot简介

Spring Boot是Spring社区发布的一个开源项目，旨在帮助开发者快速并且更简单的构建项目。它使用习惯优于配置的理念让你的项目快速运行起来，使用Spring Boot很容易创建一个独立运行（运行jar，内置Servlet容器，Tomcat、jetty）、准生产级别的基于Spring框架的项目，使用SpringBoot你可以不用或者只需要很少的配置文件。

#### Spring Boot核心功能
- 独立运行的Spring项目：可以以jar包形式独立运行，通过java -jar xx.jar即可运行
- 内嵌Servlet容器：可以选择内嵌Tomcat、Jetty等
- 提供starter简化maven配置：一个maven项目，使用了spring-boot-starter-web时，会自动加载Spring Boot的依赖包
- 自动配置Spring：Spring Boot会根据在类路径中的jar包、类，为jar包中的类自动配置Bean
- 准生产的应用监控：提供基于http、ssh、telnet对运行时的项目进行监控
- 无代码生成和xml配置：主要通过条件注解来实现

#### Spring Boot项目搭建

1、 [spring boot官方](http://note.youdao.com/) ，填写相关的项目信息、jdk版本等，就会生成一个maven项目的压缩包，下载解压导入IDE就可以

2、IDE下直接创建，推荐使用STS(Spring Tool Suite)、IntelliJ IDEA均支持直接搭建，STS是Spring基于eclipse进行二次开发的工具。

Spring Tool Suite　：新建Spring Initializr项目,填写项目信息和选择技术，将项目设置成maven项目。

IntelliJ IDEA：新建Spring Starter project,填写项目信息和选择技术完成maven工程创建。

3、Spring Boot CLI工具，使用命令创建。

4、手工构建maven项目
- 任意IDE新建空maven项目
- 修改pom.xml添加Spring Boot的父级依赖Spring-boot-starter-parent，添加之后这个项目就是一个Spring Boot项目了


```
<parent>

    <groupId>org.springframework.boot</groupId>

    <artifactId>spring-boot-starter-parent</artifactId>

    <version>1.5.4.RELEASE</version>


</parent>
```

Spring-boot-starter-parent是一个特殊的starter，用来提供相关的maven默认依赖，使用之后，常用的包依赖可以省略version标签

- 修改pom.xml添加web支持的starter
```
<dependency>

    <groupId>org.springframework.boot</groupId>

    <artifactId>spring-boot-starter-web</artifactId>

</dependency>
```

- 添加Spring boot编译插件

```
<build>

    <plugins>

        <plugin>

            <groupId>org.springframework.boot</groupId>

            <artifactId>spring-boot-maven-plugin</artifactId>

        </plugin>

    </plugins>

</build>
```

项目生成之后，会在根包目录下生成一个入口类，添加一个测试控制器简单测试一下

```
package com.wisely.ch5_2_4;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@SpringBootApplication //Spring Boot核心注解，用于开启自动配置
public class DemoApplication {

    @RequestMapping("/")
    String index(){
      return "Hello Spring Boot";
    }
  
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

执行main方法之后，在浏览器中访问http://localhost:8080,可以得到如下结果：

![image](http://otqvaruzt.bkt.clouddn.com/10180714-4.png)