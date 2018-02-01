---
layout:

title: 3-JVM常用参数配置

date: 2017-04-24

updated: 2017-04-24

tags:
- JVM
- JVM优化

categories: JVM原理、诊断与优化

permalink:

thumbnail:

toc: true

comment: true

notag: false

top: false

---

# 5-GC参数

- 除了栈上分配和大对象,绝大多数对象直接分配在eden区
- 大对象可能直接分配在老年代


## 5.1 堆的回顾
![image](http://clsaaleanjvmimgbed-1252032169.cossh.myqcloud.com/course_sulianchengjin/5-1-1-1.png)

## 5.2 串行收集器

- 最古老，最稳定
- 效率高
- 可能会产生较长的停顿(单线程垃圾回收,没法利用多核性能)
- -XX:+UseSerialGC
    - 新生代、老年代使用串行回收
    - 新生代复制算法
    - 老年代标记-压缩
![image](http://clsaaleanjvmimgbed-1252032169.cossh.myqcloud.com/course_sulianchengjin/5-2-1-1.png)

## 5.3 并行收集器

- ParNew
    - -XX:+UseParNewGC
        - 新生代并行
        - 老年代串行
    - Serial收集器新生代的并行版本
    - 复制算法
    - 多线程，需要多核支持
    - -XX:parallelGCThreads 限制线程数量
![image](http://clsaaleanjvmimgbed-1252032169.cossh.myqcloud.com/course_sulianchengjin/5-3-1-1.png)

- Parallel收集器
    - 类似ParNew
    - 新生代复制算法
    - 老年代 标记-压缩
    - 更加关注吞吐量
    - -XX:+UseParallelGC 
        - 使用Parallel收集器+ 老年代串行
    - -XX:+UseParallelOldGC
        - 使用Parallel收集器+ 并行老年代
![image](http://clsaaleanjvmimgbed-1252032169.cossh.myqcloud.com/course_sulianchengjin/5-3-1-2.png)

- -XX:MaxGCPauseMills
    - 最大停顿时间，单位毫秒
    - GC尽力保证回收时间不超过设定值
- -XX:GCTimeRatio
    - 0-100的取值范围
    - 垃圾收集时间占总时间的比
    - 默认99，即最大允许1%时间做GC

>这两个参数是矛盾的。因为停顿时间和吞吐量不可能同时调优


## 5.4 CMS收集器

### 5.4.1 CMS基本概念
- Concurrent Mark Sweep 并发标记清除(并发的意思是与用户程序一起执行,可能导致吞吐量降低)
- 标记-清除算法
- 与标记-压缩相比
- 并发阶段会降低吞吐量
- 老年代收集器（新生代使用ParNew）
- -XX:+UseConcMarkSweepGC

### 5.4.2 CMS执行过程

>CMS运行过程比较复杂，着重实现了标记的过程，可分为

- 初始标记(用户线程阻塞)
    - 根可以直接关联到的对象
    - 速度快
- 并发标记（和用户线程一起）
    - 主要标记过程，标记全部对象
- 重新标记(用户线程阻塞)
    - 由于并发标记时，用户线程依然运行，因此在正式清理前，再做修正
- 并发清除（和用户线程一起）
    - 基于标记结果，直接清理对象

![image](http://clsaaleanjvmimgbed-1252032169.cossh.myqcloud.com/course_sulianchengjin/5-4-2-1.png)

```
1.662: [GC [1 CMS-initial-mark: 28122K(49152K)] 29959K(63936K), 0.0046877 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
1.666: [CMS-concurrent-mark-start]
1.699: [CMS-concurrent-mark: 0.033/0.033 secs] [Times: user=0.25 sys=0.00, real=0.03 secs] 
1.699: [CMS-concurrent-preclean-start]
1.700: [CMS-concurrent-preclean: 0.000/0.000 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
1.700: [GC[YG occupancy: 1837 K (14784 K)]1.700: [Rescan (parallel) , 0.0009330 secs]1.701: [weak refs processing, 0.0000180 secs] [1 CMS-remark: 28122K(49152K)] 29959K(63936K), 0.0010248 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
1.702: [CMS-concurrent-sweep-start]
1.739: [CMS-concurrent-sweep: 0.035/0.037 secs] [Times: user=0.11 sys=0.02, real=0.05 secs] 
1.739: [CMS-concurrent-reset-start]
1.741: [CMS-concurrent-reset: 0.001/0.001 secs] [Times: user=0.00 sys=0.00, real=0.00 secs]

```

### 5.4.3 CMS的特点


- 尽可能降低停顿
    - 会影响系统整体吞吐量和性能
    - 比如，在用户线程运行过程中，分一半CPU去做GC，系统性能在GC阶段，反应速度就下降一半
- 清理不彻底
    - 因为在清理阶段，用户线程还在运行，会产生新的垃圾，无法清理
- 因为和用户线程一起运行，不能在空间快满时再清理
    - -XX:CMSInitiatingOccupancyFraction设置触发GC的阈值
    - 如果不幸内存预留空间不够，就会引起concurrent mode failure

```
33.348: [Full GC 33.348: [CMS33.357: [CMS-concurrent-sweep: 0.035/0.036 secs] [Times: user=0.11 sys=0.03, real=0.03 secs] 
 (concurrent mode failure): 47066K->39901K(49152K), 0.3896802 secs] 60771K->39901K(63936K), [CMS Perm : 22529K->22529K(32768K)], 0.3897989 secs] [Times: user=0.39 sys=0.00, real=0.39 secs]

```
>使用串行收集器作为后备


## 5.5 Tomcat实例演示


