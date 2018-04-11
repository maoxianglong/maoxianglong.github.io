---
title: IntelliJ IDEA配置maven
date: 2017-10-25
tags: [maven]
categories: [maven]
---

#####  导论

从eclipse换成传说中高大上的IntelliJ IDEA，使用了一段时间，快捷键算是熟悉过来了，但还是感觉不太舒服，记录一下自己最近配置maven的一些步骤图解，没什么干货，主要是怕忘记，所以还是有必要记录一下

#####  正文

######  使用IntelliJ IDEA配置maven全局文件

![image](http://otqvaruzt.bkt.clouddn.com/%E5%9B%BE%E4%B8%80.png)

A处为maven本地的全局配置文件，可以将里面的默认本地地址改为自定义的本地地址，因为一般默认都是放在C盘，不介意的可以不用改，本地的maven全局仓库也需要对应修改。

B处对应的是maven本地仓库的地址，具体步骤往下看。

######  配置全局maven本地仓库地址

打开settings.xml，修改如下配置，我是改为E:/maven路径，以后你所有项目的jar都会下载到该路径下，前提是你必须设置了全局的配置文件。

```
<localRepository>E:/maven</localRepository>
```


也可将公司的私服配置到settings.xml里面，这样以后不需要在每个工程pom.xml里面配置私服了，相当方便。加上如下代码即可。

```
<profile>  
	<id>release</id>  
	<repositories>  
		<repository>  
		<id>nexus-releases</id>  
		<url>公司对应的maven私服URL</url>  
		<releases>  
			 <enabled>true</enabled>  
		</releases>  
		<snapshots>  
			 <enabled>false</enabled>  
		</snapshots>  
		</repository>  
	</repositories>  
</profile>
```

######  maven编译

打开maven的配置页面，怎么添加maven配置就不多讲了，直接讲下配置信息。
![image](http://otqvaruzt.bkt.clouddn.com/%E5%9B%BE%E4%BA%8C.png)

和eclipse一样
- Working directory：工程路径。
- Command line：执行指令
- Profile：环境配置文件

点击General选项，如下图
![image](http://otqvaruzt.bkt.clouddn.com/%E5%9B%BE%E4%B8%89.png)

标记的地方需要和之前配置的全局环境一致。

#####  总结
IntelliJ IDEA是同事一直推荐我使用的，这段时间用IntelliJ IDEA确实感觉比eclipse强大了很多，就是快捷键别扭，不过可以修改，我比较懒，没有去修改默认的快捷键，用了这么多年的eclipse突然被换掉想想还是挺怀念的。