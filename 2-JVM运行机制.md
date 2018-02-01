---
layout:

title: 2-JVM运行机制

date: 2017-04-21

updated: 2017-04-21

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


## 2.1 JVM启动流程

![image](http://clsaaleanjvmimgbed-1252032169.cossh.myqcloud.com/course_sulianchengjin/2-1-1-1.png)


## 2.2 JVM基本结构

![image](http://clsaaleanjvmimgbed-1252032169.cossh.myqcloud.com/course_sulianchengjin/2-2-1-1.png)

### 2.2.1 PC寄存器

- 每个线程拥有一个PC寄存器
- 在线程创建时 创建
- 指向下一条指令的地址
- 执行本地方法时，PC的值为undefined

### 2.2.2 方法区

- 保存装载的类信息
- 类型的常量池
- 字段，方法信息
- 方法字节码
- 通常和永久区(Perm)关联在一起

>JDK6时，String等常量信息置于方法

>JDK7时，已经移动到了堆

### 2.2.3 Java堆

- 和程序开发密切相关
- 应用系统对象都保存在Java堆中
- 所有线程共享Java堆
- 对分代GC来说，堆也是分代的
- GC的主要工作区间

### 2.2.4 Java栈

- 线程私有
- 栈由一系列帧组成（因此Java栈也叫做帧栈）
- 帧保存一个方法的局部变量、操作数栈、常量池指针
- 每一次方法调用创建一个帧，并压栈

- **局部变量表 包含参数和局部变量**

```
public class StackDemo {

    public static int runStatic(int i,long l,float  f,Object o ,byte b){
        return 0;
    }
    public int runInstance(char c,short s,boolean b){
        return 0;
        }
    }

```
![image](http://clsaaleanjvmimgbed-1252032169.file.myqcloud.com/course_sulianchengjin/2-2-4-1.png)

- **函数调用组成帧栈**

![image](http://clsaaleanjvmimgbed-1252032169.file.myqcloud.com/course_sulianchengjin/2-2-4-2.png)

- **操作数栈**
    - Java没有寄存器，所有参数传递使用操作数栈
```
public static int add(int a,int b){
int c=0;
c=a+b;
return c;
}

```
```
0:   iconst_0 // 0压栈
1:   istore_2 // 弹出int，存放于局部变量2
2:   iload_0  // 把局部变量0压栈
3:   iload_1 // 局部变量1压栈
4:   iadd      //弹出2个变量，求和，结果压栈
5:   istore_2 //弹出结果，放于局部变量2
6:   iload_2  //局部变量2压栈
7:   ireturn   //返回
```

![image](http://clsaaleanjvmimgbed-1252032169.file.myqcloud.com/course_sulianchengjin/2-2-4-3.png)

- **栈上分配**
    - 小对象（一般几十个bytes），在没有逃逸的情况下，可以直接分配在栈上
    - 直接分配在栈上，可以自动回收，减轻GC压力
    - 大对象或者逃逸对象无法栈上分配


```
堆上分配
每次需要清理空间

public void method(){    
BcmBasicString* str=new BcmBasicString;    ....    delete str;
}

```

```
栈上分配
函数调用完成自动清理

public void method(){    
BcmBasicString str;  
  ....
}

```

```
public class OnStackTest {
    public static void alloc(){
        byte[] b=new byte[2];
        b[0]=1;
    }
    public static void main(String[] args) {
        long b=System.currentTimeMillis();
        for(int i=0;i<100000000;i++){
            alloc();
        }
        long e=System.currentTimeMillis();
        System.out.println(e-b);
    }
}


-server -Xmx10m -Xms10m
-XX:+DoEscapeAnalysis -XX:+PrintGC
输出结果 5


-server -Xmx10m -Xms10m  
-XX:-DoEscapeAnalysis -XX:+PrintGC
……
[GC 3550K->478K(10240K), 0.0000977 secs]
[GC 3550K->478K(10240K), 0.0001361 secs]
[GC 3550K->478K(10240K), 0.0000963 secs]
564

```


## 2.3 内存模型

### 2.3.1 堆/栈/方法区的交互

```
public class AppMain
//运行时, jvm把appmain的信息都放入方法区
{
    public static void main(String[] args) 
    //main 方法本身放入方法区。
    {
        Sample test1 = new Sample( " 测试1 " );
        //test1是引用，所以放到栈区里，Sample是自定义对象应该放到堆里面
        Sample test2 = new Sample( " 测试2");
        test1.printName();
        test2.printName();
    }
}

public class Sample
//运行时, jvm 把appmain的信息都放入方法区
{
    private name;
    //new Sample实例后， name 引用放入栈区里， name 对象放入堆里
    public Sample(String name)
    {
     this .name = name;
    }
    //print方法本身放入 方法区里。
    public void printName()
    {
    System.out.println(name);
    }
}

```
![image](http://clsaaleanjvmimgbed-1252032169.cossh.myqcloud.com/course_sulianchengjin/2-3-1-1.png)

>为了能让递归函数调用的次数更多一些，应该怎么做呢？

### 2.3.2 JVM线程内存模型

- 每一个线程有一个工作内存和主存独立
- 工作内存存放主存中变量的值的拷贝

![image](http://clsaaleanjvmimgbed-1252032169.cossh.myqcloud.com/course_sulianchengjin/2-3-1-2.png)

>当数据从主内存复制到工作存储时，必须出现两个动作：第一，由主内存执行的读（read）操作；第二，由工作内存执行的相应的load操作；当数据从工作内存拷贝到主内存时，也出现两个操作：第一个，由工作内存执行的存储（store）操作；第二，由主内存执行的相应的写（write）操作

>每一个操作都是原子的，即执行期间不会被中断

>对于普通变量，一个线程中更新的值，不能马上反应在其他变量中

>如果需要在其他线程中立即可见，需要使用 volatile 关键字


![image](http://clsaaleanjvmimgbed-1252032169.cossh.myqcloud.com/course_sulianchengjin/2-3-1-3.png)

```
volatile 不能代替锁
一般认为volatile 比锁性能好（不绝对）

选择使用volatile的条件是：
语义是否满足应用

public class VolatileStopThread extends Thread{
    private volatile boolean stop = false;
    public void stopMe(){
        stop=true;
    }
    
    public void run(){
        int i=0;
        while(!stop){
        i++;
        }
       System.out.println("Stop thread");
    }
    
    public static void main(String args[]) throws InterruptedException{
        VolatileStopThread t=new VolatileStopThread();
        t.start();
        Thread.sleep(1000);
        t.stopMe();
        Thread.sleep(1000);
    }
}
```

### 2.3.3 内存可见性

- 可见性
    - 一个线程修改了变量，其他线程可以立即知道
- 保证可见性的方法
    - volatile
    - synchronized （unlock之前，写变量值回主存）
    - final(一旦初始化完成，其他线程就可见)

### 2.3.4 有序性

- 有序性
    - 在本线程内，操作都是有序的
    - 在线程外观察，操作都是无序的。（指令重排 或 主内存同步延时）
- 指令重排
    - 线程内串行语义
    - 写后读	a = 1;b = a;	写一个变量之后，再读这个位置。
    - 写后写	a = 1;a = 2;	写一个变量之后，再写这个变量。
    - 读后写	a = b;b = 1;	读一个变量之后，再写这个变量。
    - 以上语句不可重排
    - 编译器不考虑多线程间的语义
    - 可重排： a=1;b=2;
- 指令重排 – 破坏线程间的有序性

```
线程A首先执行writer()方法
线程B线程接着执行reader()方法
线程B在int i=a+1 是不一定能看到a已经被赋值为1

因为在writer中，两句话顺序可能打乱

class OrderExample {
    int a = 0;
    boolean flag = false;
    
    public void writer() {
        a = 1;                   
        flag = true;           
    }
    
    public void reader() {
        if (flag) {                
            int i =  a +1;      
            ……
        }
    }
}

```

- 指令重排 – 保证有序性的方法


```
同步后，即使做了writer重排，因为互斥的缘故，reader 线程看writer线程也是顺序执行的。

class OrderExample {
    int a = 0;
    boolean flag = false;
    
    public synchronized void writer() {
        a = 1;                   
        flag = true;           
    }
    
    public synchronized void reader() {
        if (flag) {                
            int i =  a +1;      
            ……
        }
    }
}

```

- 指令重排原则


- 程序顺序原则：一个线程内保证语义的串行性
- volatile规则：volatile变量的写，先发生于读
- 锁规则：解锁(unlock)必然发生在随后的加锁(lock)前
- 传递性：A先于B，B先于C 那么A必然先于C
- 线程的start方法先于它的每一个动作
- 线程的所有操作先于线程的终结（Thread.join()）
- 线程的中断（interrupt()）先于被中断线程的代码
- 对象的构造函数执行结束先于finalize()方法


## 2.4 编译和解释运行的概念

- 解释运行
    - 解释执行以解释方式运行字节码
    - 解释执行的意思是：读一句执行一句
- 编译运行（JIT）
    - 将字节码编译成机器码
    - 直接执行机器码
    - 运行时编译
    - 编译后性能有数量级的提升

