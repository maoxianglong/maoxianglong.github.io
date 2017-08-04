---
title: 工具类之spring beanUtils和apache beanUtils
date: 2017-08-02
tags: [java, spring, 工具类]
categories: [jdk,spring,Apache]
---

## 一、beanUtils
开发过程中会遇到对象之间的属性copy，避免了使用get/set重复操作，BeanUtils还是挺好用的。在开发过程中，使用BeanUtils确实会节省很多代码，让代码清晰简洁，但是性能却不一定会高过get/set，这里不考虑性能，就从便利性方面记录一下最近用过的工具类，再比较一下两种工具类的区别。BeanUtils中有很多好用的方法，用起来确实是很爽，但是所有的东西有利必有弊，怎样用好这些工具类大杀器，还是得看实际业务。下面以copyProperties(sourceDate, targetData)方法为例记录使用过程。

## 二、实例

源数据对象
```
package com.mxl.utils;

import java.util.Date;

public class SourceData {

	private String name;
	private String email;
	private Date createDate;
	
	public SourceData(){};
	
	public SourceData(String name , String email , Date createDate) {
		this.name = name;
		this.email = email;
		this.createDate = createDate;
	}
	
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public String getEmail() {
		return email;
	}
	public void setEmail(String email) {
		this.email = email;
	}
	public Date getCreateDate() {
		return createDate;
	}
	public void setCreateDate(Date createDate) {
		this.createDate = createDate;
	}

	@Override
	public String toString() {
		return "SourceData [name=" + name + ", email=" + email
				+ ", createDate=" + createDate + "]";
	}
}
```

目标数据对象
```
package com.mxl.utils;

import java.util.Date;

public class TargetData {

	private String name;
	private String email;
	private Date createDate;
	
	public TargetData(){};
	
	public TargetData(String name , String email , Date createDate) {
		this.name = name;
		this.email = email;
		this.createDate = createDate;
	}
	
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public String getEmail() {
		return email;
	}
	public void setEmail(String email) {
		this.email = email;
	}
	public Date getCreateDate() {
		return createDate;
	}
	public void setCreateDate(Date createDate) {
		this.createDate = createDate;
	}

	@Override
	public String toString() {
		return "TargetData [name=" + name + ", email=" + email
				+ ", createDate=" + createDate + "]";
	}
	
}

```

客户端代码

```
package com.mxl.utils;

import java.lang.reflect.InvocationTargetException;
import java.util.Date;

public class TestBeanUtils {

	public static void main(String[] args) throws IllegalAccessException, InvocationTargetException {
		
		SourceData sourceDate = new SourceData("曹操" , "caocao@163.com" , new Date(0));
		
		TargetData targetData = new TargetData();
		
		//spring中的BeanUtils
		org.springframework.beans.BeanUtils.copyProperties(sourceDate, targetData);
		
		System.out.println("spring BeanUtils : " + targetData.toString());
		
		//apache中的BeanUtils
		org.apache.commons.beanutils.BeanUtils.copyProperties(targetData, sourceDate);
		
		System.out.println("apache BeanUtils : " + targetData.toString());
	}
	
}

```
输出
```
spring BeanUtils : TargetData [name=曹操, email=caocao@163.com, createDate=Thu Jan 01 08:00:00 CST 1970]
apache BeanUtils : TargetData [name=曹操, email=caocao@163.com, createDate=Thu Jan 01 08:00:00 CST 1970]
```

两个工具类都能将源数据copy到目标对象，但是这些工具都不是万能的，往下看。

源数据对象将createDate改为字符串类型，目标数据对象不变。
```
package com.mxl.utils;

import java.util.Date;

public class SourceData {

	private String name;
	private String email;
//	private Date createDate;
	private String createDate;
	
	public SourceData(){};
	
	public SourceData(String name , String email , String createDate) {
		this.name = name;
		this.email = email;
		this.createDate = createDate;
	}
	
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public String getEmail() {
		return email;
	}
	public void setEmail(String email) {
		this.email = email;
	}
	public String getCreateDate() {
		return createDate;
	}
	public void setCreateDate(String createDate) {
		this.createDate = createDate;
	}

	@Override
	public String toString() {
		return "SourceData [name=" + name + ", email=" + email
				+ ", createDate=" + createDate + "]";
	}
}

```

客户端代码

```
package com.mxl.utils;

import java.lang.reflect.InvocationTargetException;
import java.util.Date;

public class TestBeanUtils {

	public static void main(String[] args) throws IllegalAccessException, InvocationTargetException {
		
		SourceData sourceDate = new SourceData("曹操" , "caocao@163.com" , new Date(0).toString());
		
		TargetData targetData = new TargetData();
		
		//spring中的BeanUtils
//		org.springframework.beans.BeanUtils.copyProperties(sourceDate, targetData);
//		
//		System.out.println("spring BeanUtils : " + targetData.toString());
		
		//apache中的BeanUtils
		org.apache.commons.beanutils.BeanUtils.copyProperties(targetData, sourceDate);
		
		System.out.println("apache BeanUtils : " + targetData.toString());
	}
	
}

```
将spring的BeanUtils注释掉，先看Apache的BeanUtils能否正常工作，看输出。
```
2017-8-2 15:32:29 org.apache.commons.beanutils.converters.DateConverter toDate
警告:     DateConverter does not support default String to 'Date' conversion.
2017-8-2 15:32:29 org.apache.commons.beanutils.converters.DateConverter toDate
警告:     (N.B. Re-configure Converter or use alternative implementation)
Exception in thread "main" org.apache.commons.beanutils.ConversionException: DateConverter does not support default String to 'Date' conversion.
	at org.apache.commons.beanutils.converters.DateTimeConverter.toDate(DateTimeConverter.java:468)
	at org.apache.commons.beanutils.converters.DateTimeConverter.convertToType(DateTimeConverter.java:343)
	at org.apache.commons.beanutils.converters.AbstractConverter.convert(AbstractConverter.java:156)
	at org.apache.commons.beanutils.converters.ConverterFacade.convert(ConverterFacade.java:60)
	at org.apache.commons.beanutils.BeanUtilsBean.convert(BeanUtilsBean.java:1078)
	at org.apache.commons.beanutils.BeanUtilsBean.copyProperty(BeanUtilsBean.java:437)
	at org.apache.commons.beanutils.BeanUtilsBean.copyProperties(BeanUtilsBean.java:286)
	at org.apache.commons.beanutils.BeanUtils.copyProperties(BeanUtils.java:137)
	at com.mxl.utils.TestBeanUtils.main(TestBeanUtils.java:20)
```
报错了，需要自定义转换器，现在没时间去自定义这个转化器，类似于struct2中的自定义转换器，定义好了之后肯定是能用的，就不细说了。再将spring的工具类注释放开，将Apache的工具类注释掉，看能否正常执行。

```
package com.mxl.utils;

import java.lang.reflect.InvocationTargetException;
import java.util.Date;

public class TestBeanUtils {

	public static void main(String[] args) throws IllegalAccessException, InvocationTargetException {
		
		SourceData sourceDate = new SourceData("曹操" , "caocao@163.com" , new Date(0).toString());
		
		TargetData targetData = new TargetData();
		
		//spring中的BeanUtils
		org.springframework.beans.BeanUtils.copyProperties(sourceDate, targetData);
		
		System.out.println("spring BeanUtils : " + targetData.toString());
		
		//apache中的BeanUtils
//		org.apache.commons.beanutils.BeanUtils.copyProperties(targetData, sourceDate);
//		
//		System.out.println("apache BeanUtils : " + targetData.toString());
	}
	
}

```
看输出。

```
Exception in thread "main" org.springframework.beans.FatalBeanException: Could not copy properties from source to target; nested exception is java.lang.IllegalArgumentException: argument type mismatch
	at org.springframework.beans.BeanUtils.copyProperties(BeanUtils.java:621)
	at org.springframework.beans.BeanUtils.copyProperties(BeanUtils.java:531)
	at com.mxl.utils.TestBeanUtils.main(TestBeanUtils.java:15)
Caused by: java.lang.IllegalArgumentException: argument type mismatch
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:39)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:25)
	at java.lang.reflect.Method.invoke(Method.java:597)
	at org.springframework.beans.BeanUtils.copyProperties(BeanUtils.java:618)
	... 2 more

```
springframework的BeanUtils完全就不支持String转Date类型，比起Apache的BeanUtils还是有稍显逊色，网络上说法很多，各不一致，无赖花二十分钟亲自试了下，总结一下：

## 三、总结

Apache的BeanUtils能支持大部分的类型属性copy，包括字符串转日期类型等等，但是需要自定义转换器，而springframework完全就不支持跨类型的转换，也没有自定义转换器这么一说。其次与网络上结果不一致的原因也可能是工具类版本不一样，笔者使用的spring版本是3.2.2，common版本是1.8.3。






















