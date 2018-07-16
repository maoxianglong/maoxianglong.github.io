---
title: eclipse中设置java文件的版本校验
date: 2018-07-11
tags: [错误]
categories: []
---


解决eclipse工程运行正常但是工程有红叉的问题，红叉看着不爽，就想有的人看到代码由warn就不爽一样。
今天eclipse从服务器上D下来一个工程,工程明明编译、运行都没问题，但是工程上却总会出现一个红叉：

![image](https://img-blog.csdn.net/20170430192531039)    

首先在eclipse里面显示problems选项卡，找出问题所在：

![image](https://img-blog.csdn.net/20170430192547539)

看到如上的错误提示：Java compiler level does not match the version of the installed Java project，这个问题是说Java编译器的级别和Java Project Facet中设置的Java级别不一致导致的。

因为我工程里面使用的jre版本是1.8，而验证java文件却是使用的1.7，所以报这个异常。
![image](http://otqvaruzt.bkt.clouddn.com/jdkban.png)

知道了问题在哪，解决起来就简单了，右键工程，点击“Properties”，然后点击左侧的“Project Facets”，在右边将Java的级别设置为跟Java编译器级别一致即可。
![image](https://img-blog.csdn.net/20170430192602393)