---
title: 关于maven引用依赖报错
date: 2017-08-21
tags: [maven,异常]
categories: [问题解决]
---

## 一、导论
今天遇到一个比较坑的问题，后来才发现是自己犯傻了，记录一下，避免以后重复犯错。

## 二、问题详情
异常代码
```
The type com.snowball.tsm.datagram.Command cannot be resolved. It is indirectly referenced from required .class files
```

## 三、问题原因
```
List<com.snowball.tsm.datagram.Command> commands = shiftInResponse.getCommands();
```
问题就出现在这里，这里引用了两个相同名称的类Command，但是包路径不一样，导致引用失败，检查了半天发现pom文件里面的jar的该有的都有，不应该啊。后来才发现其中一个Command是在tsm中，然而我并没有发现，一直在hub中再找，路径都不对哥你当然找不到啦。

## 四、解决办法
在maven配置文件中加上tsm依赖，问题解决
```
<dependency>
	<groupId>com.snowball.tsm</groupId>
	<artifactId>tsm-protocol</artifactId>
	<version>1.0.4</version>
</dependency>
```

## 五、反省
检查代码不够细心，时间浪费在弱智问题上。