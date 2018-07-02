---
layout : post
title : "JDK1.8新特性"
category : JAVA
tagline: ""
date : 2016-06-10
tags : [JDK,新特性]
---

### 接口的默认方法
Java 8允许我们给接口添加一个非抽象的方法实现，只需要使用 default关键字即可，这个特征又叫做扩展方法，默认方法sqrt将在子类上可以直接使用。

### Lambda 表达式
之前集合排序需要给静态方法 Collections.sort 传入一个List对象以及一个比较器来按指定顺序排列。通常做法都是创建一个匿名的比较器对象然后将其传递给sort方法。

 	Collections.sort(list, new Comparator<TestBean>() {
			@Override
			public int compare(TestBean o1, TestBean o2) {
				return o1.getNum()- o2.getNum();
			}
		});

lambda表达式：

    Collections.sort(list, (TestBean a,TestBean b)->a.getNum().compareTo(b.getNum()));

再来一个简单例子

	public int add(int x, int y) {
        return x + y;
    }

lambda表达式：

	(int x, int y) -> x + y;
	(x, y) -> x + y;
	(x, y) -> { return x + y; } 

**在lambda表达式中访问外层作用域和老版本的匿名对象中的方式很相似。可以直接访问标记了final的外层局部变量，或者实例的字段以及静态变量。**

### 函数式接口
每一个lambda表达式都对应一个类型，通常是接口类型。而“函数式接口”是指仅仅只包含一个抽象方法的接口，每一个该类型的lambda表达式都会被匹配到这个抽象方法。因为 默认方法 不算抽象方法，所以你也可以给你的函数式接口添加默认方法。
我们可以将lambda表达式当作任意只包含一个抽象方法的接口类型，确保你的接口一定达到这个要求，你只需要给你的接口添加 @FunctionalInterface 注解，编译器如果发现你标注了这个注解的接口有多于一个抽象方法的时候会报错的。

	@FunctionalInterface
	interface Converter<F, T> {
   	 T convert(F from);
	}
	Converter<String, Integer> converter = (from) -> Integer.valueOf(from);
	Integer converted = converter.convert("123");
	System.out.println(converted);    // 123

如果没有@FunctionalInterface也是可以的,

### 方法与构造函数引用


### 访问局部变量

### 访问对象字段与静态变量

### 访问接口的默认方法