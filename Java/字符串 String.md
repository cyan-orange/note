

# 概述


1. String声明为final的，不可被继承。
2. String实现了Serializable接口：表示字符串是支持序列化的。实现了Comparable接口：表示String可以比较大小。
3. String内部定义了final char[] value用于存储字符串数据。
4. 通过字面量的方式（区别于new给一个字符串赋值，此时的字符串值声明在字符串常量池中)。
5. 字符串常量池中是不会存储相同内容的字符串。



# String的不可变性


1. 当对字符串重新赋值时，需要重写指定内存区域赋值，不能使用原有的value进行赋值。
2. 当对现的字符串进行连接操作时，也需要重新指定内存区域赋值，不能使用原有的value进行赋值。
3. 当调用String的replace()方法修改指定字符或字符串时，也需要重新指定内存区域赋值，不能使用原有的value进行赋值
4. String内部定义了一个常量数组，因此每次对字符串的操作实际上都会另外分配分配一个新的常量数组空间（**这片空间位于jvm的静态方法区**）



# 字符串拼接方式赋值的对比


1. 常量与常量的拼接结果在常量池。且常量池中不会存在相同内容的常量。
1. 只要其中一个是变量，结果就在堆中。
1. 如果拼接的结果调用intern()方法，返回值就在常量池中



举例 :
```java
String s1 = "javaEE";
String s2 = "hadoop";

String s3 = "javaEEhadoop";
String s4 = "javaEE" + "hadoop";
String s5 = s1 + "hadoop";
String s6 = "javaEE" + s2;
String s7 = s1 + s2;

System.out.println(s3 == s4);//true
System.out.println(s3 == s5);//false
System.out.println(s3 == s6);//false
System.out.println(s3 == s7);//false
System.out.println(s5 == s6);//false
System.out.println(s5 == s7);//false
System.out.println(s6 == s7);//false

String s8 = s6.intern();//返回值得到的s8使用的常量值中已经存在的“javaEEhadoop”
System.out.println(s3 == s8);//true
//****************************
String s1 = "javaEEhadoop";
String s2 = "javaEE";
String s3 = s2 + "hadoop";
System.out.println(s1 == s3);//false

final String s4 = "javaEE";//s4:常量
String s5 = s4 + "hadoop";
System.out.println(s1 == s5);//true
```


# String与其它结构的转换


##  与基本数据类型、包装类之间的转换


- String --> 基本数据类型、包装类：调用包装类的静态方法：parseXxx(str)
- 基本数据类型、包装类 --> String:调用String重载的valueOf(xxx)



## 与字符数组之间的转换


- String --> char[]:调用String的toCharArray()
- char[] --> String:调用String的构造器



## 与字节数组之间的转换


- 编码：String --> byte[]:调用String的getBytes()
- 解码：byte[] --> String:调用String的构造器



# JVM中字符串常量池存放位置说明：


jdk 1.6 (jdk 6.0 ,java 6.0):字符串常量池存储在方法区（永久区）
jdk 1.7:字符串常量池存储在堆空间
jdk 1.8:字符串常量池存储在方法区（元空间）


# String、StringBuffer、StringBuilder三者的对比


**String :** 不可变的字符序列；底层使用char[]存储
**StringBuffer :** 可变的字符序列；线程安全的，效率低；底层使用char[]存储
**StringBuilder :** 可变的字符序列；jdk5.0新增的，线程不安全的，效率高；底层使用char[]存储


# StringBuffer、StringBuilder 扩容机制


StringBuilder的构造方法，默认就是创建16的字符数组，也就是16个字节的大小 . 如果要添加的数据底层数组盛不下了，那就需要扩容底层的数组。默认情况下，扩容为原来容量的2倍 + 2，同时将原数组中的元素复制到新的数组中。
