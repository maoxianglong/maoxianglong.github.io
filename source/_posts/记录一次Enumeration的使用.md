---
title: 记录一次Enumeration的使用
date: 2017-07-31
tags: [java, jdk笔记]
categories: [jdk,数据结构]
---

## 一、Enumeration接口 
Enumeration接口本身不是一个数据结构。但是，对其他数据结构非常重要。 Enumeration接口定义了从一个数据结构得到连续数据的手段。例如，Enumeration定义了一个名为nextElement的方法，可以用来从含有多个元素的数据结构中得到的下一个元素。 
先看看Enumeration中的源码
```
public interface Enumeration<E> {
    /**
     * Tests if this enumeration contains more elements.
     *
     * @return  <code>true</code> if and only if this enumeration object
     *           contains at least one more element to provide;
     *          <code>false</code> otherwise.
     */
    boolean hasMoreElements();

    /**
     * Returns the next element of this enumeration if this enumeration
     * object has at least one more element to provide.
     *
     * @return     the next element of this enumeration.
     * @exception  NoSuchElementException  if no more elements exist.
     */
    E nextElement();
}
```
它里面只声明了两个方法，用来遍历数据
## 二、实际使用
先看看简单的测试
```
package com.mxl.enumtest;

import java.util.Enumeration;

public class EnumTest {

	class MyData {

		public String[] data;

		// 构造器
		MyData() {
			data = new String[4];
			data[0] = "zero";
			data[1] = "one";
			data[2] = "two";
			data[3] = "three";
		}

		// 返回一个enumeration对象给使用程序
		Enumeration getEnum() {
			return new MyEnumeration(0, data.length, data);
		}

	}

	class MyEnumeration implements Enumeration {
		int count; // 计数器
		int length; // 存储的数组的长度
		Object[] dataArray; // 存储数据数组的引用

		// 构造器
		public MyEnumeration(int count, int length, Object[] dataArray) {
			this.count = count;
			this.length = length;
			this.dataArray = dataArray;
		}

		public boolean hasMoreElements() {
			return (count < length);
		}

		public Object nextElement() {
			return dataArray[count++];
		}
	}

	public static void main(String[] args) {

		EnumTest enumTest = new EnumTest();

		//内部类的实例化
		Enumeration enumeration = enumTest.new MyData().getEnum();

		//Enumeration的遍历
		while (enumeration.hasMoreElements()) {
			String s = (String) enumeration.nextElement();
			System.out.println(s);
		}

	}
}

```
启动之后客户端输入如下
```
zero
one
two
three

```
注意这里使用的事内部类，内部类的实例化有很多种方式，不一定非要这用这一种

## 三、总结
Enumeration是在看公司代码的过程中遇到的，由于之前没有使用过，所以去了解了下，jdk自带的一种数据结构。在jdk1.5以前，通常的做法是定义一个静态常量类，但自jdk1.5后，java引入了枚举（关键字enum，全称为 enumeration，值类型），在枚举中，我们可以把相关的常量分组到一个枚举类型里，枚举也比常量类有更多灵活的用法，使用枚举，可以有效的提高代码的整洁性、可读性、可维护性等等，这里简单总结一下常用的枚举用法。