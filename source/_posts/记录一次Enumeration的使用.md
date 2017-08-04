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

## 三、枚举的各种使用实例

直接上代码，看注释
```
package com.mxl.enumtest;

/**
 * 枚举用法详解
 * 
 * @author Mao
 * 
 */
public class TestEnum {
	/**
	 * 普通枚举
	 * 
	 * @author Mao
	 * 
	 */
	public enum ColorEnum {
		red, green, yellow, blue;
	}

	/**
	 * 枚举像普通的类一样可以添加属性和方法，可以为它添加静态和非静态的属性或方法
	 * 
	 * @author Mao
	 * 
	 */
	public enum SeasonEnum {
		// 注：枚举写在最前面，否则编译出错
		spring, summer, autumn, winter;

		private final static String position = "test";

		public static SeasonEnum getSeason() {
			if ("test".equals(position))
				return spring;
			else
				return winter;
		}
	}

	/**
	 * 性别
	 * 
	 * 实现带有构造器的枚举
	 * 
	 * @author Mao
	 * 
	 */
	public enum Gender {
		// 通过括号赋值,而且必须带有一个参构造器和一个属性跟方法，否则编译出错
		// 赋值必须都赋值或都不赋值，不能一部分赋值一部分不赋值；如果不赋值则不能写构造器，赋值编译也出错
		MAN("MAN"), WOMEN("WOMEN");

		private final String value;

		// 构造器默认也只能是private, 从而保证构造函数只能在内部使用
		Gender(String value) {
			this.value = value;
		}

		public String getValue() {
			return value;
		}
	}

	/**
	 * 订单状态
	 * 
	 * 实现带有抽象方法的枚举
	 * 
	 * @author Mao
	 * 
	 */
	public enum OrderState {
		/** 已取消 */
		CANCEL {
			public String getName() {
				return "已取消";
			}
		},
		/** 待审核 */
		WAITCONFIRM {
			public String getName() {
				return "待审核";
			}
		},
		/** 等待付款 */
		WAITPAYMENT {
			public String getName() {
				return "等待付款";
			}
		},
		/** 正在配货 */
		ADMEASUREPRODUCT {
			public String getName() {
				return "正在配货";
			}
		},
		/** 等待发货 */
		WAITDELIVER {
			public String getName() {
				return "等待发货";
			}
		},
		/** 已发货 */
		DELIVERED {
			public String getName() {
				return "已发货";
			}
		},
		/** 已收货 */
		RECEIVED {
			public String getName() {
				return "已收货";
			}
		};

		public abstract String getName();
	}

	public static void main(String[] args) {
		// 枚举是一种类型，用于定义变量，以限制变量的赋值；赋值时通过“枚举名.值”取得枚举中的值
		ColorEnum colorEnum = ColorEnum.blue;
		switch (colorEnum) {
		case red:
			System.out.println("color is red");
			break;
		case green:
			System.out.println("color is green");
			break;
		case yellow:
			System.out.println("color is yellow");
			break;
		case blue:
			System.out.println("color is blue");
			break;
		}

		// 遍历枚举
		System.out.println("遍历ColorEnum枚举中的值");
		for (ColorEnum color : ColorEnum.values()) {
			System.out.println(color);
		}

		// 获取枚举的个数
		System.out.println("ColorEnum枚举中的值有" + ColorEnum.values().length + "个");

		// 获取枚举的索引位置，默认从0开始
		System.out.println(ColorEnum.red.ordinal());// 0
		System.out.println(ColorEnum.green.ordinal());// 1
		System.out.println(ColorEnum.yellow.ordinal());// 2
		System.out.println(ColorEnum.blue.ordinal());// 3

		// 枚举默认实现了java.lang.Comparable接口
		System.out.println(ColorEnum.red.compareTo(ColorEnum.green));// -1

		// --------------------------
		System.out.println("===========");
		System.out.println("季节为" + SeasonEnum.getSeason());

		// --------------
		System.out.println("===========");
		for (Gender gender : Gender.values()) {
			System.out.println(gender.value);
		}

		// --------------
		System.out.println("===========");
		for (OrderState order : OrderState.values()) {
			System.out.println(order.getName());
		}
	}

}
```

看输出

```
color is blue
遍历ColorEnum枚举中的值
red
green
yellow
blue
ColorEnum枚举中的值有4个
0
1
2
3
-1
===========
季节为spring
===========
MAN
WOMEN
===========
已取消
待审核
等待付款
正在配货
等待发货
已发货
已收货

```

## 四、总结
Enumeration是在看公司代码的过程中遇到的，由于之前没有使用过，所以去了解了下，jdk自带的一种数据结构。在jdk1.5以前，通常的做法是定义一个静态常量类，但自jdk1.5后，java引入了枚举（关键字enum，全称为 enumeration，值类型），在枚举中，我们可以把相关的常量分组到一个枚举类型里，枚举也比常量类有更多灵活的用法，使用枚举，可以有效的提高代码的整洁性、可读性、可维护性等等，这里简单总结一下常用的枚举用法。