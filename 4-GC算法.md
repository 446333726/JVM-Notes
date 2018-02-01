---
layout:

title: 3-JVM常用参数配置

date: 2017-04-23

updated: 2017-04-23

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


# 4-GC算法

## 4.1 GC的概念

- Garbage Collection 垃圾收集(不需要程序员自己进行内存回收)
- 1960年 List 使用了GC
- Java中，GC的对象是堆空间和永久区


## 4.2 GC算法

### 4.2.1 引用计数法

- 老牌垃圾回收算法
- 通过引用计算来回收垃圾
- 使用者
    - COM
    - ActionScript3
    - Python
- 引用计数器的实现很简单，对于一个对象A，只要有任何一个对象引用了A，则A的引用计数器就加1，当引用失效时，引用计数器就减1。只要对象A的引用计数器的值为0，则对象A就不可能再被使用。

![image](http://clsaaleanjvmimgbed-1252032169.cossh.myqcloud.com/course_sulianchengjin/4-2-1-1.png)

- 引用计数法的问题
    - 引用和去引用伴随加法和减法，影响性能
    - 很难处理循环引用
![image](http://clsaaleanjvmimgbed-1252032169.cossh.myqcloud.com/course_sulianchengjin/4-2-1-2.png)

### 4.2.2 标记清除

- 标记-清除算法是现代垃圾回收算法的思想基础。标记-清除算法将垃圾回收分为两个阶段：标记阶段和清除阶段。一种可行的实现是，在标记阶段，首先通过根节点，标记所有从根节点开始的可达对象。因此，未被标记的对象就是未被引用的垃圾对象。然后，在清除阶段，清除所有未被标记的对象。

![image](http://clsaaleanjvmimgbed-1252032169.cossh.myqcloud.com/course_sulianchengjin/4-2-2-1.png)


### 4.2.3 标记压缩

- 标记-压缩算法适合用于存活对象较多的场合，如老年代。它在标记-清除算法的基础上做了一些优化。和标记-清除算法一样，标记-压缩算法也首先需要从根节点开始，对所有可达对象做一次标记。但之后，它并不简单的清理未标记的对象，而是将所有的存活对象压缩到内存的一端。之后，清理边界外所有的空间。

![image](http://clsaaleanjvmimgbed-1252032169.cossh.myqcloud.com/course_sulianchengjin/4-2-3-1.png)

>标记压缩对标记清除而言，有什么优势呢？

### 4.2.4 复制算法

- 与标记-清除算法相比，复制算法是一种相对高效的回收方法
- 不适用于存活对象较多的场合 如老年代
- 将原有的内存空间分为两块，每次只使用其中一块，在垃圾回收时，将正在使用的内存中的存活对象复制到未使用的内存块中，之后，清除正在使用的内存块中的所有对象，交换两个内存的角色，完成垃圾回收

![image](http://clsaaleanjvmimgbed-1252032169.cossh.myqcloud.com/course_sulianchengjin/4-2-4-1.png)

- 复制算法的最大问题是：空间浪费 整合标记清理思想
- 解决方法
    - 大对象直接进入老年代担保空间(因为复制空间很小),大对象如果放到了复制空间,小对象可能被排挤到老年代
    - 老年对象进入老年代
    - 剩余对象做复制

![image](http://clsaaleanjvmimgbed-1252032169.cossh.myqcloud.com/course_sulianchengjin/4-2-4-2.png)

- 我们可以看出之前PrintGCDetail中打印的total和实际地址空间相减所得内存空间不同的原因是每次的To区域的内存空间都浪费掉了

![image](http://clsaaleanjvmimgbed-1252032169.cossh.myqcloud.com/course_sulianchengjin/4-2-4-3.png)

### 4.2.5 分代思想

- 依据对象的存活周期进行分类，短命对象归为新生代，长命对象归为老年代。
- 根据不同代的特点，选取合适的收集算法
    - 少量对象存活，适合复制算法
    - 大量对象存活，适合标记清理或者标记压缩


## 4.3 可触及性

>所有的算法，需要能够识别一个垃圾对象，因此需要给出一个可触及性的定义

- 可触及的
    - 从根节点可以触及到这个对象
- 可复活的
    - 一旦所有引用被释放，就是可复活状态
    - 因为在finalize()中可能复活该对象
- 不可触及的
    - 在finalize()后，可能会进入不可触及状态
    - 不可触及的对象不可能复活
    - 可以回收

```
public class CanReliveObj {
	public static CanReliveObj obj;
	@Override
	protected void finalize() throws Throwable {
	    super.finalize();
	    System.out.println("CanReliveObj finalize called");
	    obj=this;
	}
	@Override
	public String toString(){
	    return "I am CanReliveObj";
	}
}
```

```
CanReliveObj finalize called
obj 可用
第二次gc
obj 是 null

public static void main(String[] args) throws
     InterruptedException{
obj=new CanReliveObj();
    obj=null;   //可复活
    System.gc();
    Thread.sleep(1000);
    if(obj==null){
        System.out.println("obj 是 null");
    }else{
        System.out.println("obj 可用");
    }
        System.out.println("第二次gc");
        obj=null;    //不可复活
        System.gc();
        Thread.sleep(1000);
        if(obj==null){
            System.out.println("obj 是 null");
        }else{
            System.out.println("obj 可用");
        }
    }
}

```

- 经验：避免使用finalize()，操作不慎可能导致错误。
- 优先级低，何时被调用， 不确定
    - 何时发生GC不确定
- 可以使用try-catch-finally来替代它

- 根
    - 栈中引用的对象
    - 方法区中静态成员或者常量引用的对象（全局对象）
    - JNI方法栈中引用对象


## 4.4 Stop-The-World

### 4.4.1 概念解释

- Java中一种全局暂停的现象
- 全局停顿，所有Java代码停止，native代码可以执行，但不能和JVM交互
- 多半由于GC引起
    - Dump线程
    - 死锁检查
    - 堆Dump
- GC时为什么会有全局停顿？
    - 类比在聚会时打扫房间，聚会时很乱，又有新的垃圾产生，房间永远打扫不干净，只有让大家停止活动了，才能将房间打扫干净。
- 危害
    - 长时间服务停止，没有响应
    - 遇到HA系统，可能引起主备切换，严重危害生产环境。

### 4.4.2 示例

```
public static class PrintThread extends Thread{
	public static final long starttime=System.currentTimeMillis();
	@Override
	public void run(){
		try{
			while(true){
				long t=System.currentTimeMillis()-starttime;
				System.out.println("time:"+t);
				Thread.sleep(100);
			}
		}catch(Exception e){
			
		}
	}
}

```

```
大于450M时，清理内存
工作线程，消耗内存

-Xmx512M -Xms512M -XX:+UseSerialGC -Xloggc:gc.log -XX:+PrintGCDetails  -Xmn1m -XX:PretenureSizeThreshold=50 -XX:MaxTenuringThreshold=1

public static class MyThread extends Thread{
	HashMap<Long,byte[]> map=new HashMap<Long,byte[]>();
	@Override
	public void run(){
		try{
			while(true){
				if(map.size()*512/1024/1024>=450){
					System.out.println(“=====准备清理=====:"+map.size());
					map.clear();
				}
				
				for(int i=0;i<1024;i++){
					map.put(System.nanoTime(), new byte[512]);
				}
				Thread.sleep(1);
			}
		}catch(Exception e){
			e.printStackTrace();
		}
	}
}

```

- 输出

```
time:2018
time:2121
time:2221
time:2325
time:2425
time:2527
time:2631
time:2731
time:2834
time:2935
time:3035
time:3153
time:3504
time:4218
======before clean map=======:921765
time:4349
time:4450
time:4551

```

```
3.292: [GC3.292: [DefNew: 959K->63K(960K), 0.0024260 secs] 523578K->523298K(524224K), 0.0024879 secs] [Times: user=0.02 sys=0.00, real=0.00 secs] 
3.296: [GC3.296: [DefNew: 959K->959K(960K), 0.0000123 secs]3.296: [Tenured: 523235K->523263K(523264K), 0.2820915 secs] 524195K->523870K(524224K), [Perm : 147K->147K(12288K)], 0.2821730 secs] [Times: user=0.26 sys=0.00, real=0.28 secs] 
3.579: [Full GC3.579: [Tenured: 523263K->523263K(523264K), 0.2846036 secs] 524159K->524042K(524224K), [Perm : 147K->147K(12288K)], 0.2846745 secs] [Times: user=0.28 sys=0.00, real=0.28 secs] 
3.863: [Full GC3.863: [Tenured: 523263K->515818K(523264K), 0.4282780 secs] 524042K->515818K(524224K), [Perm : 147K->147K(12288K)], 0.4283353 secs] [Times: user=0.42 sys=0.00, real=0.43 secs] 
4.293: [GC4.293: [DefNew: 896K->64K(960K), 0.0017584 secs] 516716K->516554K(524224K), 0.0018346 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
……省略若干…..
4.345: [GC4.345: [DefNew: 960K->960K(960K), 0.0000156 secs]4.345: [Tenured: 522929K->12436K(523264K), 0.0781624 secs] 523889K->12436K(524224K), [Perm : 147K->147K(12288K)], 0.0782611 secs] [Times: user=0.08 sys=0.00, real=0.08 secs] 

```