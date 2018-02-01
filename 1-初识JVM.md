---
layout:

title: 1-初识JVM

date: 2017-04-20

updated: 2017-04-20

tags:
- JVM
- JVM原理

categories: JVM原理、诊断与优化

permalink:

thumbnail:

toc: true

comment: true

notag: false

top: false

---

[TOC]
# 1-初识JVM

## 1.1 JVM的概念

- JVM是Java Virtual Machine的简称。意为Java虚拟机
- 虚拟机指通过软件模拟的具有完整硬件系统功能的、运行在一个完全隔离环境中的完整计算机系统
- 有哪些虚拟机
    - VMWare(硬件虚拟,虚拟出来的东西能找到对应的硬件)
    - Visual Box(硬件虚拟,虚拟出来的东西能找到对应的硬件)
    - JVM(软件虚拟,虚拟出来的东西在现实中并不存在)
- VMWare或者Visual Box都是使用软件模拟物理CPU的指令集
- JVM使用软件模拟Java 字节码的指令集

## 1.2 JVM发展历史

- 1996年 SUN JDK 1.0 Classic VM
    - 纯解释运行，使用外挂进行JIT
- 1997年 JDK1.1 发布
    - AWT、内部类、JDBC、RMI、反射
- 1998年 JDK1.2 Solaris Exact VM
    - JIT 解释器混合 	
    - Accurate Memory Management精确内存管理，数据类型敏感
    - 提升的GC性能
    - JDK1.2开始 称为Java 2 J2SE J2EE J2ME 的出现加入Swing Collections
- 2000年 JDK 1.3 Hotspot 作为默认虚拟机发布
    - 加入JavaSound
- 2002年 JDK 1.4 Classic VM退出历史舞台
    - Assert 正则表达式  NIO  IPV6 日志API  加密类库
- 2004年发布 JDK1.5 即 JDK5 、J2SE 5 、Java 5
    - 泛型
    - 注解
    - 装箱
    - 枚举
    - 可变长的参数
    - Foreach循环
- JDK1.6 JDK6
    - 脚本语言支持
    - JDBC 4.0
    - Java编译器 API
- 2011年 JDK7发布
    - 延误项目推出到JDK8
    - G1
    - 动态语言增强
    - 64位系统中的压缩指针
    - NIO 2.0
- 2014年 JDK8发布
    - Lambda表达式
    - 语法增强  Java类型注解
- 2016年JDK9
    - 模块化

- 使用最为广泛的JVM为HotSpot
    - HotSpot 为Longview Technologies开发 被SUN收购
- 2006年 Java开源 并建立OpenJDK
    - HotSpot  成为Sun JDK和OpenJDK中所带的虚拟机
- 2008 年 Oracle收购BEA
    - 得到JRockit VM
- 2010年Oracle 收购 Sun	
    - 得到Hotspot
- Oracle宣布在JDK8时整合JRockit和Hotspot，优势互补
    - 在Hotspot基础上，移植JRockit优秀特性

## 1.3 JVM种类
- KVM
    - SUN发布
    - IOS Android前，广泛用于手机系统
- CDC/CLDC HotSpot
    - 手机、电子书、PDA等设备上建立统一的Java编程接口
    - J2ME的重要组成部分
- JRockit
    - BEA 
- IBM J9 VM
    - IBM内部
- Apache Harmony
    - 兼容于JDK 1.5和JDK 1.6的Java程序运行平台
    - 与Oracle关系恶劣 退出JCP ，Java社区的分裂
    - OpenJDK出现后，受到挑战 2011年 退役没有大规模商用经历
    - 对Android的发展有积极作用

## 1.4 Java语言规范

### 1.4.1 语法

- 语法结构

```
IfThenStatement:    if (Expression)Statement

if(true){do sth;}

ArgumentList:
	Argument
	ArgumentList , Argument
add(a,b,c,d);

```

- 词法结构
    - \u + 4个16进制数字 表示UTF-16
    - 行终结符：CR, or LF, or CR LF.
    - 空白符:空格 tab \t 换页 \f  行终结符
    - 注释
    - 提示符
    - 关键字

### 1.4.2 变量

- Int
    - 0 2 0372 0xDada_Cafe 1996 0x00_FF__00_FF
- Long
    - 0l 0777L 0x100000000L 2_147_483_648L 0xC0B0L
- Float
    - 1e1f 2.f .3f 0f 3.14f 6.022137e+23f
- Double
    - 1e1 2. .3 0.0 3.14 1e-9d 1e137
- 操作
```
+=
-=
*=
/=
&=
|=
^=
%=
<<=
>>=
>>>=
```

### 1.4.3 类型

- 元类型 
    - byte short int long float char boolean 
- 变量初始值
    - boolean false
    - char \u0000
- 泛型
- 按值与按引用传递
```
class Value { int val; }

class Test {
    public static void main(String[] args) {
        int i1 = 3;
        int i2 = i1;
        i2 = 4;
        System.out.print("i1==" + i1);
        System.out.println(" but i2==" + i2);
        Value v1 = new Value();
        v1.val = 5;
        Value v2 = v1;
        v2.val = 6;
        System.out.print("v1.val==" + v1.val);
        System.out.println(" and v2.val==" + v2.val);
    }
}

```
- 输出
    - i1 i2为不同的变量
    - v1 v2为引用同一个实例

```
i1==3 but i2==4
v1.val==6 and v2.val==6

```

## 1.5 JVM规范

- Java语言规范定义了什么是Java语言
- Java语言和JVM相对独立
    - Groovy
    - Clojure
    - Scala
- JVM主要定义二进制class文件和JVM指令集等

### 1.5.1 Class 文件格式

### 1.5.2 数字的内部表示和存储

- 原码：第一位为符号位（0为正数，1为负数）
- 反码：符号位不动，原码取反
- 负数补码：符号位不动，反码加1
- 正数补码：和原码相同
- 打印整数的二进制表示

```
int a=-6;
for(int i=0;i<32;i++){
	int t=(a & 0x80000000>>>i)>>>(31-i);
	System.out.print(t);
}
```

```
-6
原码： 10000110
反码： 11111001
补码： 11111010


-1
原码： 10000001
反码： 11111110
补码： 11111111

```

- 为什么要用补码
    - 统一零的表示
    - 将减法化为加法运算

```
0
正数：00000000
负数：10000000

0
负数：10000000
反码：11111111
补码：00000000

-6+5
11111010
+ 00000101
= 11111111

-4+5
11111100
+ 00000101
= 00000001

-3+5
11111101
+ 00000101
= 00000010

```

- Float的表示与定义
    - 支持 IEEE 754


### 1.5.3 returnAddress数据类型定义

### 1.5.4 定义PC

### 1.5.5 堆

### 1.5.6 栈

### 1.5.7 方法区

### 1.5.8 JVM 指令集

- 类型转化
    - l2i  
- 出栈入栈操作
    - aload  astore
- 运算
    - iadd  isub
- 流程控制
    - ifeq ifne
- 函数调用
    - invokevirtual invokeinterface  invokespecial  invokestatic 

```
void spin() {
  int i; 
  for (i = 0; i < 100; i++) { ;
     // Loop body is empty
   }
 } 
 
0   iconst_0       // Push int constant 0
1   istore_1       // Store into local variable 1 (i=0)
2   goto 8         // First time through don't increment
5   iinc 1 1       // Increment local variable 1 by 1 (i++)
8   iload_1        // Push local variable 1 (i)
9   bipush 100     // Push int constant 100
11  if_icmplt 5    // Compare and loop if less than (i < 100)
14  return         // Return void when done

```


### 1.5.9 JVM对Java Library 提供支持

- 反射 java.lang.reflect
- ClassLoader
- 初始化class和interface
- 安全相关 java.security
- 多线程
- 弱引用
