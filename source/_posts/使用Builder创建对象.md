---
title: 使用Builder创建对象
date: 2017-08-21
tags: [java,jdk,对象创建]
categories: [jdk笔记]
---

## 导论

昨天又刷了一遍《Effective java》创建对象相关的内容，感觉还是有必要记录归档下，方便记忆和以后复习。

对于一个有着大量可选参数的类来说，我们在构建的时候一般有一下几种方式：

## 方式一：静态工厂方法
静态工厂方法相比于普通的构造器有如下优点：
1. 静态工厂方法有名称，便于阅读。
2. 调用静态工厂方法无需创建对象。
3. 静态工厂方法可以返回返回类型的任何子类
4. 在创建参数化实例对象时，代码会变得更加的简介

直接看enumSet中的源代码就可以了，典型的静态工厂方法
```
    public static <E extends Enum<E>> EnumSet<E> noneOf(Class<E> elementType) {
        Enum[] universe = getUniverse(elementType);
        if (universe == null)
            throw new ClassCastException(elementType + " not an enum");

        if (universe.length <= 64)
            return new RegularEnumSet<>(elementType, universe);
        else
            return new JumboEnumSet<>(elementType, universe);
    }
```

## 方式二：JavaBean模式
核心代码：
```
public class User {
    private long uid;
    private String nick;
    private int gender;
    private int age;
    private String email;

    public long getUid() {
        return uid;
    }

    public void setUid(long uid) {
        this.uid = uid;
    }

    public String getNick() {
        return nick;
    }

    public void setNick(String nick) {
        this.nick = nick;
    }

    public int getGender() {
       return gender;
    }

    public void setGender(int gender) {
       this.gender = gender;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String getEmail() {
           return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }
}
```
客户端代码
```
User user = new User();
user.setAge(23);
user.setGender(1);
user.setNick("superman");
user.setEmail("abc@xx.com");
user.setUid(10001);
```
这种模式应该是平时用的最多的，但是遇到大量参数的对象时候这种模式就会让代码显得很冗余。

## 方式三：Builder构建模式
这种模式解决了上面的大量参数，设置参数线程不安全的问题，由于Builder采用的是内部类，所以不存在线程安全的问题。
代码
```
package com.mxl.builder.model;

/**
 * 
 * @author Mao
 *
 */
public class NutritionFacts {

	private final int servingSize;
	private final int servings;
	private final int calories;
	private final int fat;
	private final int sodium;
	private final int carbohydrate;

	public static class Builder {

		private final int servingSize;
		private final int servings;

		private int calories = 0;
		private int fat = 0;
		private int sodium = 0;
		private int carbohydrate = 0;

		public Builder(int servingSize, int servings) {
			this.servingSize = servingSize;
			this.servings = servings;
		}

		public Builder calories(int val) {
			this.calories = val;
			return this;
		}

		public Builder fat(int val) {
			this.fat = val;
			return this;
		}

		public Builder sodium(int val) {
			this.sodium = val;
			return this;
		}

		public Builder carbohydrate(int val) {
			this.carbohydrate = val;
			return this;
		}

		public NutritionFacts build() {
			return new NutritionFacts(this);
		}
	}

	//私有构造方法将builder参数copy到NutritionFacts中
	private NutritionFacts(Builder build) {
		servingSize = build.servingSize;
		servings = build.servings;
		sodium = build.sodium;
		fat = build.fat;
		calories = build.calories;
		carbohydrate = build.carbohydrate;
	}

	@Override
	public String toString() {
		return "NutritionFacts [servingSize=" + servingSize + ", servings="
				+ servings + ", calories=" + calories + ", fat=" + fat
				+ ", sodium=" + sodium + ", carbohydrate=" + carbohydrate + "]";
	}

	//客户端创建对象
	public static void main(String[] args) {
		NutritionFacts demo = new NutritionFacts.Builder(10, 20).calories(15)
				.fat(24).carbohydrate(23).carbohydrate(27).build();
		System.out.println(demo);
	}
}

```
代码很简洁，容易阅读。但是它自身也还是有一些缺陷的，在创建一个对象的时候必须先构建一个他的Builder对象，还是增加了一些开销。


