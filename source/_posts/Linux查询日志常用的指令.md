---
title: Linux查询日志常用的指令
date: 2017-10-13
tags: [Linux]
categories: [Linux]
---

#### 导论
工作中经常会遇到各式各样的异常，需要借助日志来分析，所以记录一下最近使用蛮多的各种情况下的日志查询指令。

#### 指令

##### 场景一

查询关键字在文本中的所在并显示行数。

```
cat Jane.txt | grep -n 'heiress'
```

![图一](http://otqvaruzt.bkt.clouddn.com/1013_1.jpg)

或者直接使用grep也可以，当然使用管道更好，后面再说。

```
grep -n heiress --color Jane.txt
```
效果是一样的，就不放图了，注意一定要加上-n，不然不会显示行数，--color可以不加，默认会加深显示。

##### 场景二

定位到指定的行数，有时候查询日志的时候发现关键字所在的行数异常信息不完整，则可以定位到关键字附近的二三十行进一步排查。

查看6339-6341行之间的内容。
```
cat Jane.txt | tail -n +6339 | head -n 3
```
![图一](http://otqvaruzt.bkt.clouddn.com/1013_2.jpg)

也可以使用vi编辑。
```
vi Jane.txt
```
之后再使用下面的指令。
```
:行数
```
效果是一样的，我一般都是用后者，因为很灵活。

退出vi编辑器执行如下指令即可。
```
:exit
```
或者这个指令。
```
:wq
```
但一般日志文件需要root权限，所以这个指令普通用户起不来作用。

![图一](http://otqvaruzt.bkt.clouddn.com/1013_3.jpg)

##### 场景三

现在发现上个月一笔订单出现了异常，需要查看当天日志，但是发现上个月的日志有600M，太占内存了，所以服务器将其压缩打包了，这个时候能不能不解压就直接查看，当然可以。

现在将《简·爱》压缩再来试试。

执行这个指令压缩。
```
tar -czf Jane.tar.gz Jane.txt
```
![图一](http://otqvaruzt.bkt.clouddn.com/1013_4.jpg)

压缩好了，接下来直接检索压缩包中的Jane.txt的内容，执行如下指令。注意一定要加--binary-files参数，不然查不出来。
```
zcat Jane.tar.gz | grep -n --binary-files=text 'heiress'
```
![图一](http://otqvaruzt.bkt.clouddn.com/1013_5.jpg)

##### 场景四

同样压缩文件的日志信息显示的也不完整，需要定位到周围的行数，同样可以，执行下面的指令。

```
gzip -dc Jane.tar.gz | tail -n +6339 | head -n 3
```

上面的指令是定位到压缩文件内容中的6339-6341行，可以看到查询出来的内容和之前查询的内容一致。

![图一](http://otqvaruzt.bkt.clouddn.com/1013_6.jpg)

#### 总结
Linux指令太多太多，简单的讲下笔者自己常用的几个，每个人面对不同的场景都有自己的方式去查询，这里只做参考，不做引导。
