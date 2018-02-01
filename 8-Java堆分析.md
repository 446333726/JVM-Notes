---
layout:

title: 8-Java堆分析

date: 2017-04-27

updated: 2017-04-27

tags:
- JVM
- JVM诊断

categories: JVM原理、诊断与优化

permalink:

thumbnail:

toc: true

comment: true

notag: false

top: false

---

[TOC]

# 8-Java堆分析

## 8.1 内存溢出(OOM)的原因

- 堆/永久区/线程栈/直接内存只要其中一个超过内存容量就会抛出OOM

### 8.1.1 堆内存溢出

- 占用大量堆空间，直接溢出


```
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
at geym.jvm.ch8.oom.SimpleHeapOOM.main(SimpleHeapOOM.java:14)


public static void main(String args[]){
    ArrayList<byte[]> list=new ArrayList<byte[]>();
    for(int i=0;i<1024;i++){
        list.add(new byte[1024*1024]);
    }
}

```
- 解决方法：增大堆空间，及时释放内存

### 8.1.2 永久区溢出

```
生成大量的类
public static void main(String[] args) {
    for(int i=0;i<100000;i++){
        CglibBean bean = new CglibBean("geym.jvm.ch3.perm.bean"+i,new HashMap());
    }
}

```

```
Caused by: java.lang.OutOfMemoryError: PermGen space
[Full GC[Tenured: 2523K->2523K(10944K), 0.0125610 secs] 2523K->2523K(15936K), 
[Perm : 4095K->4095K(4096K)], 0.0125868 secs] [Times: user=0.02 sys=0.00, real=0.01 secs] 
Heap
 def new generation   total 4992K, used 89K [0x28280000, 0x287e0000, 0x2d7d0000)
  eden space 4480K,   2% used [0x28280000, 0x282966d0, 0x286e0000)
  from space 512K,   0% used [0x286e0000, 0x286e0000, 0x28760000)
  to   space 512K,   0% used [0x28760000, 0x28760000, 0x287e0000)
 tenured generation   total 10944K, used 2523K [0x2d7d0000, 0x2e280000, 0x38280000)
   the space 10944K,  23% used [0x2d7d0000, 0x2da46cf0, 0x2da46e00, 0x2e280000)
 compacting perm gen  total 4096K, used 4095K [0x38280000, 0x38680000, 0x38680000)
   the space 4096K,  99% used [0x38280000, 0x3867fff0, 0x38680000, 0x38680000)
    ro space 10240K,  44% used [0x38680000, 0x38af73f0, 0x38af7400, 0x39080000)
    rw space 12288K,  52% used [0x39080000, 0x396cdd28, 0x396cde00, 0x39c80000)

```

- 解决方法：增大Perm区,允许Class回收

### 8.1.3 栈溢出

- 这里的栈溢出指，在创建线程的时候，需要为线程分配栈空间，这个栈空间是向操作系统请求的，如果操作系统无法给出足够的空间，就会抛出OOM

```
-Xmx1g -Xss1m

public static class SleepThread implements Runnable{
    public void run(){
        try {
            Thread.sleep(10000000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

public static void main(String args[]){
    for(int i=0;i<1000;i++){
        new Thread(new SleepThread(),"Thread"+i).start();
        System.out.println("Thread"+i+" created");
    }
}

```

```
Exception in thread "main" java.lang.OutOfMemoryError: 
unable to create new native thread

```

- 解决方法：减少堆内存,减少线程栈大小

### 8.1.4 直接内存溢出

- ByteBuffer.allocateDirect()无法从操作系统获得足够的空间
- 直接内存需要GC回收，但是直接内存无法引起GC。直接内存使用满时，无法触发GC。
- 如果堆空间很富余，无法触发GC，直接内存可能就会溢出。如果堆空间触发GC，直接内存可以回收

```
-Xmx1g -XX:+PrintGCDetails

for(int i=0;i<1024;i++){
    ByteBuffer.allocateDirect(1024*1024);
    System.out.println(i);
      System.gc();
}

```
- 解决方法：减少堆内存,有意触发GC


## 8.2 MAT使用基础

下载:http://www.eclipse.org/mat/downloads.php

- 柱状图显示，显示每个类的使用情况，
比如类的数量，所占空间等

![image](http://clsaaleanjvmimgbed-1252032169.cossh.myqcloud.com/course_sulianchengjin/8-2-1-1.png)


- 显示线程信息

![image](http://clsaaleanjvmimgbed-1252032169.cossh.myqcloud.com/course_sulianchengjin/8-2-1-4.png)

- 显示堆总体信息，比如消耗最大的一些对象等

![image](http://clsaaleanjvmimgbed-1252032169.cossh.myqcloud.com/course_sulianchengjin/8-2-1-5.png)






### 8.2.1 浅堆(Shallow Heap)与深堆(Retained Heap)

- 浅堆 
    - 一个对象结构所占用的内存大小
    - 3个int类型以及一个引用类型合计占用内存3*4+4=16个字节。再加上对象头的8个字节，因此String对象占用的空间，即浅堆的大小是16+8=24字节
    - 对象大小按照8字节对齐
    - 浅堆大小和对象的内容无关，只和对象的结构有关

- 深堆
    - 一个对象被GC回收后，可以真实释放的内存大小
    - 只能通过对象访问到的（直接或者间接）所有对象的浅堆之和 （支配树）

### 8.2.2 显示入引用（incoming）和出引用(outgoing)

- 显示一个对象引用的对象,显示引用这个对象的对象

![image](http://clsaaleanjvmimgbed-1252032169.cossh.myqcloud.com/course_sulianchengjin/8-2-1-6.png)


### 8.2.3 支配树

- 显示支配树

![image](http://clsaaleanjvmimgbed-1252032169.cossh.myqcloud.com/course_sulianchengjin/8-2-1-2.png)

![image](http://clsaaleanjvmimgbed-1252032169.cossh.myqcloud.com/course_sulianchengjin/8-2-1-3.png)

### 8.2.4 分析实例

![image](http://clsaaleanjvmimgbed-1252032169.cossh.myqcloud.com/course_sulianchengjin/8-2-3-1.png)

![image](http://clsaaleanjvmimgbed-1252032169.cossh.myqcloud.com/course_sulianchengjin/8-2-3-2.png)

![image](http://clsaaleanjvmimgbed-1252032169.cossh.myqcloud.com/course_sulianchengjin/8-2-3-3.png)

>可以看到，所有的Point实例浅堆和深堆的大小都是16字节。而dLine对象，浅堆为16字节，深堆也是16字节，这是因为dLine对象内的两个点f和g没有被设置为null，因此，即使dLine被回收，f和g也不会被释放。对象cLine内的引用对象d和e由于仅在cLine内还存在引用，因此只要cLine被释放，d和e必然也作为垃圾被回收，即d和e在cLine的保留集内，因此cLine的深堆为16*2+16=48字节。
对于aLine和bLine对象，由于两者均持有对方的一个点，因此，当aLine被回收时，公共点a在bLine中依然有引用存在，故不会被回收，点a不在aLine对象的保留集中，因此aLine的深堆大小为16+16=32字节。对象bLine与aLine完全一致。


## 8.4 使用Visual VM分析堆

- java自带的多功能分析工具，可以用来分析堆Dump


## 8.5 Tomcat OOM分析案例

- Tomcat OOM
    - Tomcat 在接收大量请求时发生OOM，获取堆Dump文件，进行分析。
- 使用MAT打开堆
- 分析目的：
    - 找出OOM的原因
    - 推测系统OOM时的状态
    - 给出解决这个OOM的方法

![image](http://clsaaleanjvmimgbed-1252032169.cossh.myqcloud.com/course_sulianchengjin/8-5-1-1.png)

![image](http://clsaaleanjvmimgbed-1252032169.cossh.myqcloud.com/course_sulianchengjin/8-5-1-2.png)

![image](http://clsaaleanjvmimgbed-1252032169.cossh.myqcloud.com/course_sulianchengjin/8-5-1-3.png)

![image](http://clsaaleanjvmimgbed-1252032169.cossh.myqcloud.com/course_sulianchengjin/8-5-1-4.png)

![image](http://clsaaleanjvmimgbed-1252032169.cossh.myqcloud.com/course_sulianchengjin/8-5-1-5.png)
![image](http://clsaaleanjvmimgbed-1252032169.cossh.myqcloud.com/course_sulianchengjin/8-5-1-6.png)
![image](http://clsaaleanjvmimgbed-1252032169.cossh.myqcloud.com/course_sulianchengjin/8-5-1-7.png)
![image](http://clsaaleanjvmimgbed-1252032169.cossh.myqcloud.com/course_sulianchengjin/8-5-1-8.png)

- 解决方法：
    1. OOM由于保存session过多引起，可以考虑增加堆大小
    2. 如果应用允许，缩短session的过期时间，使得session可以及时过期，并回收
