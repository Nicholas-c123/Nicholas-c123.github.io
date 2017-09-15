---
layout: post
title:  "Java characters 终结篇"
excerpt:
date:   2017-09-15
categories: JVM
author: 张祝浩
headings:
  - Java char unicode 等问题终结篇
  - 代码验证
tag:
  - JVM
---
# Java char unicode 等问题终结篇
在面试Java的时候，考官总是喜欢问些char能不能存下中文啦之类的无聊的问题。有人说Java内存里是用Unicode存储char的所以可以存储中文。
**错！**
我们追根问底一下。
首先Unicode是一个编码字符集(coded character set)，也是ISO国际标准，是为了解决各国人民在美国人第一版计算机只有ascii字符而没有本国文字而自创了各式各样互不兼容的字符集出现的（GB2312 Big-5 日本的 韩国的。。。。），而且目前还一直在发展中，Unicode规定的仅仅是世界上所有的字符和其对应的某个数字——0x????（Unicode code points）存在的映射关系。起初的设计是Unicode只占两个字节16位（Byte），也就是最多只能支持65536种字符。但是咱们博大精深的汉字人家美国佬起初设计的Unicode怎么可能包括我中华民族的所有汉字。JVM平台诞生之初匹配当时的Unicode字符，规定char只能占2个字节，但是随着Unicode的发展，字符编码超出`0xFFFF`的字符（被称为Supplementary character——补充字符集）就不能存储在一个char变量中了，于是JSR小组决定适配一下新的Unicode字符集。
这里又牵扯到了character encoding scheme的概念。utf-8 utf-16 utf-32是常见的基于Unicode的字符编码方案，我来分别解释下。
1. utf-32使用4个字节来编码每个Unicode 字符，也就是最多表示42亿的字符，这远大于目前Unicode的所有字符，所以所有Unicode char都统一用4个字节表示，但是对于ascii字符，每个字符我们就用了4倍的存储空间，这在网络传输中明显是不划算的，低效率的。
2. utf-16使用一个或者两个16位的数（2或4字节）来编码Unicode 字符。用标志位来区分对应的原始Unicode字符应当占4个或者2个字节。但是他做的依旧不彻底，对于Ascii字符来说还是多占用了一个字节。但是这家伙诞生之初可是和第一版Unicode一一对应的啊，因此微软开始将其采用的UTF-16直接称为Unicode。JVM平台最后采用的也是utf-16作为其对0xFFFF以上的平面[^1]的改造
3. utf-8使用1到4个单字节数来编码Unicode。也是用每个字节开始的标志位区分原始字符应当占用几个字节。具体转换方式就不详述了。
综上所述，JSR小组最后决定直接采用utf-16作为JVM内部的encoding[^2]，这样最大限度保留了向前兼容性，所有在0x0000-0xFFFF(Basic Multilingual Plane ,BMP， 第一版Unicode 基本平面 包括了常用汉字)而对于超出BMP的字符，JVM会自动将其转为utf-16存储在char[]中。
所以，回到一开始的问题，Java统一使用Unicode字符集 采用utf-16编码。由于汉字博大精深，Unicode里不在BMP里的汉字都不能用一个char变量存储。下面是我随意从Wikipedia上找的一个神奇的汉字，大家可以试一试。
## 代码验证

```java
public class Main {
	public static void main(String[] args) {
		//char c = '𠒑'; won't compile!
		String c = "𠒑";
		int codePoint = c.codePointAt(0);
		int lowwer = c.charAt(1);
		System.out.printf("%x\n",codePoint);
		System.out.printf("%x\n",lowwer);
	}
}
```

***
[^1]: [wiki:Unicode Plane](https://en.wikipedia.org/wiki/Plane_(Unicode)#Supplementary_Ideographic_Plane)
[^2]: [Oracle- supplementary character](http://www.oracle.com/us/technologies/java/supplementary-142654.html)