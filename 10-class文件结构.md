---
layout:

title: 9-锁

date: 2017-04-29

updated: 2017-04-29

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

# 10-class文件结构


## 10.1 语言无关性

![image](http://clsaaleanjvmimgbed-1252032169.cossh.myqcloud.com/course_sulianchengjin/10-1-1-1.png)

## 10.2 文件结构

![image](http://clsaaleanjvmimgbed-1252032169.cossh.myqcloud.com/course_sulianchengjin/10-2-1-1.png)

### 10.2.1 魔数

- 0xCAFEBABE
- 每个Class文件的头4个字节称为魔数（Magic Number），它的唯一作用是用于确定这个文件是否为一个能被虚拟机接受的Class文件。很多文件存储标准中都使用魔数来进行身份识别，譬如图片格式，如gif或jpeg等在文件头中都存有魔数。Class文件魔数的值为0xCAFEBABE。如果一个文件不是以0xCAFEBABE开头，那它就肯定不是Java class文件。

### 10.2.2 版本

- minor_version u2
- major_version u2
- 紧接着魔数的4个字节存储的是Class文件的版本号：第5和第6是次版本号（Minior Version），第7个和第8个字节是主版本号(Major Version)。
- Java的版本号是人45开始的，JDK1.1之后的每个JDK大版本发布主版本号向上加1，高版本的JDK能向下兼容以前版本的Class文件，但不能运行以后版本的Class文件，即使文件格式并未发生变化。JDK1.1能支持版本号为45.0~45.65535的Class文件，JDK1.2则能支持45.0~46.65535的Class文件。
- JDK1.7可生成的Class文件主版本号的最大值为51.0。 

![image](http://images.cnblogs.com/cnblogs_com/xiaoruoen/bianyiqi.jpg)

### 10.2.3 常量池

- 紧接着魔数与版本号之后的是常量池入口，常量池是Class文件结构中与其它项目关联最多的数据类型，也是占用Class文件空间最大的数据项目之一，同时它还是在文件中第一个出现的表类型数据项目。由于常量池中常量的数量是不固定的，所以在常量池的入口需要放置一项u2类型的数据，代表常量池容量计数值(constant_pool_count)。从1开始计数。第0项腾出来满足后面某些指向常量池的索引值的数据在特定情况下需要表达"不引用任何一个常量池项目"的意思，这种情况就可以把索引值置为0来表示。但尽管constant_pool列表中没有索引值为0的入口，缺失的这一入口也被constant_pool_count计数在内。例如，当constant_pool中有14项，constant_poo_count的值为15。Class文件结构中只有常量池的容量计数是从1开始的，对于其他集合类型，包括接口索引集合、字段表集合、方法表集合等的容量计数都是从0开始的。

- 常量池之中主要存放两大类常量：字面量和符号引用。字面量比较接近于Java语言层面的常量概念，如文本字符串、被声明为final的常量值等。而符号引用则属于编译原理方面的概念，包括了下面三类常量:
    - 类和接口的全限定名
    - 字段的名称和描述符
    - 方法的名称和描述符  
-  Java代码在进行Java编译的时候，并不像C和C++那样有"连接"这一步骤，而是在虚拟机加载Class文件的时候进行动态连接。也就是说，在Class文件中不会保存各个方法和字段的最终内存布局信息，因此这些字段和方法的符号引用不经过转换的话是无法被虚拟机使用的。当虚拟机运行时，需要从常量池获得对应的符号引用，再在类创建时或运行时解析并翻译到具体的内存地址之中。 
-  常量池中的每一项常量都是一个表，共有11种结构各不相同的表结构数据，这11种表都有一个共同的特点，就是表开始的第一位是一个u1类型的标志位(tag，取值为1至12，缺少标志为2的数据类型)，代有当前对象属于哪种常量类型，11常量类型所代表的具体含义如下表所示。

![image](http://images.cnblogs.com/cnblogs_com/xiaoruoen/320256/r_clc.png)

![image](http://images.cnblogs.com/cnblogs_com/xiaoruoen/clcjg.jpg)

- 第9位 16转换为十进制为22,代表常量池中有21个常量。第10位的07带表的是一个常量的tag值，可以从上表中看到，07代表CONSTANT_Class_info类型，从上表中可以看出，

- CONSTANT_Class_info类型的结构有一个u1类型的tag,有一个u2类型的name_index，数量都是1。那么可以看出接下来的第11位与第12位的值0002就是name_index的值。即指向的常量池中的第一个常量。第二项常量的标志位为0x01(看第13位)，也就是CONSTANT_Utf_info类型，CONSTANT_Utf_info类型有u2型的length与u1型的bytes。依此类推。

```
constant_pool_count u2
constant_pool  cp_info
CONSTANT_Utf8		1	UTF-8编码的Unicode字符串
CONSTANT_Integer	3	int类型的字面值
CONSTANT_Float		4	float类型的字面值
CONSTANT_Long		5	long类型的字面值
CONSTANT_Double		6	double类型的字面值
CONSTANT_Class		7	对一个类或接口的符号引用
CONSTANT_String		8	String类型字面值的引用
CONSTANT_Fieldref	9	对一个字段的符号引用
CONSTANT_Methodref	10	对一个类中方法的符号引用
CONSTANT_InterfaceMethodref	11	对一个接口中方法的符号引用
CONSTANT_NameAndType	12	对一个字段或方法的部分符号引用

```

- CONSTANT_Utf8
    - tag 1
    - length u2
    - bytes[length]

![image](http://clsaaleanjvmimgbed-1252032169.cossh.myqcloud.com/course_sulianchengjin/10-2-3-1.png)
![image](http://clsaaleanjvmimgbed-1252032169.cossh.myqcloud.com/course_sulianchengjin/10-2-3-2.png)
![image](http://clsaaleanjvmimgbed-1252032169.cossh.myqcloud.com/course_sulianchengjin/10-2-3-3.png)

- CONSTANT_Integer
    - tag 3
    - byte u4
```
public static final int sid=99;
```
![image](http://clsaaleanjvmimgbed-1252032169.cossh.myqcloud.com/course_sulianchengjin/10-2-3-4.png)

- CONSTANT_String
    - tag 8
    - string_index  u2 (指向utf8的索引)
```
public static final String sname="geym";
```
![image](http://clsaaleanjvmimgbed-1252032169.cossh.myqcloud.com/course_sulianchengjin/10-2-3-5.png)

- CONSTANT_NameAndType
    - tag 12
    - name_index u2 (名字，指向utf8)
    - descriptor_index u2 (描述符类型，指向utf8)
![image](http://clsaaleanjvmimgbed-1252032169.cossh.myqcloud.com/course_sulianchengjin/10-2-3-6.png)

- CONSTANT_Class
    - tag 7
    - name_index u2 (名字，指向utf8)
![image](http://clsaaleanjvmimgbed-1252032169.cossh.myqcloud.com/course_sulianchengjin/10-2-3-7.png)

- CONSTANT_Fieldref ,CONSTANT_Methodref ,CONSTANT_InterfaceMethodref
    - tag 9 ,10, 11
    - class_index  u2 (指向CONSTANT_Class)
    - name_and_type_index u2 (指向CONSTANT_NameAndType)
![image](http://clsaaleanjvmimgbed-1252032169.cossh.myqcloud.com/course_sulianchengjin/10-2-3-8.png)
![image](http://clsaaleanjvmimgbed-1252032169.cossh.myqcloud.com/course_sulianchengjin/10-2-3-9.png)

### 10.2.4 访问符

- 紧接常量池后的两个字节称为access_flags，它展示了文件中定义的类或接口的几段信息。例如，访问标志指明文件中定义的是类还是接口;访问标志还定义的在类或接口的声明中，使用了哪种修饰符oder和接口是抽象的，还是公共的;类的类型可以为final，而final类不可能是抽象的;接口不能为final类型的。这些标志位的定义如下表所示：

![image](http://images.cnblogs.com/cnblogs_com/xiaoruoen/fwbz2.jpg)

-  如一个TestClass类被public关键字修饰但没有被声明为final和abstract,并且它使用了JDK1.2之后的编译器进行编译，因此它的ACC_PUBLIC、ACC_SUPER标志应该为真。因此它的access_flags的值应为:0x0001|0x0020 = 0x0021。

### 10.2.5 类、超类、接口

- this_class u2
    - 指向常量池的Class
- super_class u2
    - 指向常量池的Class
- interface_count u2
    - 接口数量
- interfaces    
    - interface_count 个 interface u2
    - 每个interface是指向CONSTANT_Class的索引


- 类索引:访问标志后面接下来的两个字节是类索引(this_class)，它是一个对常量池的索引。在this_class位置的常量池入口必须为CONSTANT_Class_info表。该表由两个部分组成——tag和name_index。tag部分是代表其的标志位，name_index位置的常量池入口为一个包含了类或接口全限定名的CONSTANT_Utf8_info表。 

- 父类索引:在class文件中，紧接在this_class之后是super_class项，它是一个两个字节的常量池索引。在super_class位置的常量池入口是一个指向该类超类全限定名的CONSTANT_Class_info入口。因为Java程序中所有对象的基类都是java.lang.Object类，除了Object类以外，常量池索引super_class对于所有的类均有效。对于Object类，super_class的值为0。对于接口，在常量池入口super_class位置的项为java.lang.Object

- interfaces_count和interfaces:紧接着super_class的是interfaces_count，此项的含义为：在文件中出该类直接实现或者由接口所扩展的父接口的数量。在这个计数的后面，是名为interfaces的数组，它包含了对每个由该类或者接口直接实现的父接口的常量池索引。每个父接口都使用一个常量池中的CONSTANT_Class_info入口来描述，该CONSTANT_Class_info入口指向接口的全限定名。这个数组只容纳那些直接出现在类声明的implements子句或者接口声明的extends子句中的父接口。超类按照在implements子句和extends子句中出现的顺序在这个数组中显现。 

### 10.2.6 字段

- field_count
    - 字段数量
- fields
    - field_count个field_info
- field
    - access_flags u2 
    - name_index u2 
    - descriptor_index u2 
    - attributes_count u2 
    - attribute_info attributes[attributes_count];


- 在class文件中，紧接在interfaces后面的是对在该类或者接口中所声明的字段的描述。首先是名为fields_count的计数，它是类变量和实例变量的字段的数量总和。在这个计数后面的是不同长度的field_info表的序列(fields_count指出了序列中有多少个field_info表)。只有在文件中由类或者接口声明了的字段才能在fields列表中列出。在fields列表中，不列出从超类或者父接口继承而来的字段。另一方面，fields列表可能会包含在对应的Java源文件中没有叙述的字段，这是因为Java编译器可以会在编译时向类或者接口添加字段。
- 在Java中，描述字段的信息有：字段的作用域、是实例变量还是类变量（static）、可变性(final)、并发可见性(volatile)、可否序列化(trasient)、字段数据类型、字段名称。这些信息中，各个修饰符都是布尔值，要么有某个修饰符，要么没有，很适合使用标志位来表示。而字段叫什么名字、字段被定义为什么数据类型，这些都是无法固定的，只能引用常量池中的常量来描述。

![image](http://images.cnblogs.com/cnblogs_com/xiaoruoen/zdbjg.jpg)

- 字段修饰符放在access_flags项目中，它与类中的access_flags项目是非常相似的，都是一个u2的数据类型，其中可以设置 的标志位和含义如下表所示

![image](http://images.cnblogs.com/cnblogs_com/xiaoruoen/zdfwbz2.jpg)

- 跟随access_flags标志的是两项索引值：name_index和descriptor_index。它们都是对常量池的引用，分别代表着字段的简单名称及字段和方法的描述符。描述符的作用是用来描述字段的数据类型、方法的参数列表（包括数量、类型以及顺序）和返回值。根据描述符规则，基本数据类型(byte、char、double、float、int、long、short、boolean)及代表无返回值的void类型都用一个大写字符来表示，而对象类型则用字符L加对象的全限定名来表示。

![image](http://images.cnblogs.com/cnblogs_com/xiaoruoen/bsf.jpg)

- 对于数组类型，每一个维度将使用一个前置的"["字符来描述，如一个定义的"java.lang.String[][]"类型的二维数组，将被记录为:"[[Ljava/lang/String;",一个整型数组"int[]"将被记录为"[I"
- 用描述符来描述方法时，按照先参数列表，后返回值的顺序描述，参数列表按照参数的严格顺序放在一组小括号"()"之内。如方法void inc()的描述符为"()V"，方法java.lang.String.toString()的描述符为"()Ljava/lang/String;"。

- name_index u2
    - 常量池引用 ，表示字段的名字
- descriptor_index
    - 表示字段的类型
```
B byte
C char
D double
F float
I int
J long
S short
Z boolean
V void
L 对象
Ljava/lang/Object;
[
数组 [[Ljava/lang/String; = String[][]

```

### 10.2.7 方法

- methods_count
    - 方法数量
- methods
    - methods_count个method_info
- method_info
    - access_flags u2 
    - name_index u2 
    - descriptor_index u2 
    - attributes_count u2 
    - attribute_info attributes[attributes_count];
- name_index u2
    - 方法名字，常量池UTF-8索引
- descriptor_index u2
    - 描述符，用于表达方法的参数和返回值
- 方法描述符
    - void inc()   ()V
    - void setId(int)  (I)V
    - int indexOf(char[],int ) ([CI)I

- 紧接着field后面的是对在该类或者接口中所声明的方法的描述。其结构与fields一样，不一样的是访问标志。
![image](http://images.cnblogs.com/cnblogs_com/xiaoruoen/ffbzw2.jpg)

### 10.2.8 属性

- 在field和method中，可以有若干个attribute，类文件也有attribute，用于描述一些额外的信息
    - attribute_name_index u2 
        - 名字，指向常量池UTF-8
    - attribute_length u4 
        - 长度
    - info[attribute_length] u1
        - 内容 
- attribute本身也可以包含其他attribute
- 随着JDK的发展不断有新的attribute加入


- class文件中最后的部分是属性，它给出了在该文件类或者接口所定义的属性的基本信息。属性部分由attributes_count开始，attributes_count是指出现在后续attributes列表的attribute_info表的数量总和。每个attribute_info的第一项是指向常量池中CONSTANT_Utf8_info表的引引，该表给出了属性的名称。
- 属性有许多种。Java虚拟机规范定义了几种属性，但任何人都可以创建他们自己的属性种类，并且把它们置于class文件中，Java虚拟机实现必须忽略任何不能识别的属性。
- java虚拟机预设的9项虚拟机应当能识别的属性如下表所示。

![image](http://images.cnblogs.com/cnblogs_com/xiaoruoen/suxing.jpg)

- Deprecated
    - attribute_name_index u2
    - attribute_length u4
    - attribute_name_index
        - 指向包含Deprecated的UTF-8常量
    - attribute_length
        - 为0
- ConstantValue
    - attribute_name_index u2
    - attribute_length u4
    - constantvalue_index u2
    - attribute_name_index
        - 包含ConstantantValue字面量的UTF-8索引
    - attribute_length
        - 为2
    - constantvalue_index
        - 常量值，指向常量池，可以是UTF-8，Float, Double 等

![image](http://clsaaleanjvmimgbed-1252032169.cossh.myqcloud.com/course_sulianchengjin/10-2-8-0.png)

- Code
    - 有关exception_table 表示，在start_pc和end_pc之间，如果遇到catch_type异常或者它的子异常，则转到handler_pc处理

![image](http://clsaaleanjvmimgbed-1252032169.cossh.myqcloud.com/course_sulianchengjin/10-2-8-1.png)

- LineNumberTable - Code属性的属性

![image](http://clsaaleanjvmimgbed-1252032169.cossh.myqcloud.com/course_sulianchengjin/10-2-8-2.png)

- LocalVariableTable -  Code属性的属性

![image](http://clsaaleanjvmimgbed-1252032169.cossh.myqcloud.com/course_sulianchengjin/10-2-8-3.png)

- Exceptions属性
    - 和Code属性平级
    - 表示方法抛出的异常(不是try catch部分，而是 throws部分)
    - 结构
        - attribute_name_index u2 
        - attribute_length u4 
        - number_of_exceptions u2 
        - exception_index_table[number_of_exceptions] u2 
        - 指向Constant_Class的索引
```
public void setAge(int age)  throws IOException{
	try{
		this.age = age;
	}catch(IllegalStateException e){
		this.age = 0;
	}
}

```
![image](http://clsaaleanjvmimgbed-1252032169.cossh.myqcloud.com/course_sulianchengjin/10-2-8-4.png)

- SourceFile
    - 描述生成Class文件的源码文件名称
    - 结构
        - attribute_name_index u2
        - attribute_length u4
            - 固定为2
        - soucefile_index u2
            -   UTF-8常量索引

## 10.3 class文件结构例子

```
public class User {
	private int id;
	private String name;
	private int age;
	public int getId() {
		return id;
	}
	public void setId(int id) {
		this.id = id;
	}
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public int getAge() {
		return age;
	}
	public void setAge(int age) {
		this.age = age;
	}
	
}

```

![image](http://clsaaleanjvmimgbed-1252032169.cossh.myqcloud.com/course_sulianchengjin/10-3-1-1.jpg)

![image](http://clsaaleanjvmimgbed-1252032169.cossh.myqcloud.com/course_sulianchengjin/10-3-1-2.png)

![image](http://clsaaleanjvmimgbed-1252032169.cossh.myqcloud.com/course_sulianchengjin/10-3-1-3.png)

![image](http://clsaaleanjvmimgbed-1252032169.cossh.myqcloud.com/course_sulianchengjin/10-3-1-4.png)