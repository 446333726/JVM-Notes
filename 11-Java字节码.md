---
layout:

title: 11-Java字节码

date: 2017-04-30

updated: 2017-04-30

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


# 11-JVM字节码执行

## 11.1 javap

- class文件反汇编工具

```
public class Calc {
	public int calc() {
		int a = 500;
		int b = 200;
		int c = 50;
		return (a + b) / c;
	}
}

```

- javap –verbose Calc

```
public int calc();
  Code:
   Stack=2, Locals=4, Args_size=1
   0:   sipush  500
   3:   istore_1
   4:   sipush  200
   7:   istore_2
   8:   bipush  50
   10:  istore_3
   11:  iload_1
   12:  iload_2
   13:  iadd
   14:  iload_3
   15:  idiv
   16:  ireturn
}

```

## 11.2简单的字节码执行过程


- sipush ：500入栈
- istore : 500入局部变量表

![image](http://clsaaleanjvmimgbed-1252032169.cossh.myqcloud.com/course_sulianchengjin/11-2-1-1.png)

![image](http://clsaaleanjvmimgbed-1252032169.cossh.myqcloud.com/course_sulianchengjin/11-2-1-2.png)

![image](http://clsaaleanjvmimgbed-1252032169.cossh.myqcloud.com/course_sulianchengjin/11-2-1-3.png)

- iload_1 第一个局部变量压栈

![image](http://clsaaleanjvmimgbed-1252032169.cossh.myqcloud.com/course_sulianchengjin/11-2-1-4.png)

- iadd 2个数出栈 相加，和 入栈

![image](http://clsaaleanjvmimgbed-1252032169.cossh.myqcloud.com/course_sulianchengjin/11-2-1-5.png)

- idiv 2元素出栈 结果入栈
- ireturn 将栈顶的整数返回

![image](http://clsaaleanjvmimgbed-1252032169.cossh.myqcloud.com/course_sulianchengjin/11-2-1-6.png)

![image](http://clsaaleanjvmimgbed-1252032169.cossh.myqcloud.com/course_sulianchengjin/11-2-1-7.png)

- 上图黑框中的为字节码命令
- 字节码指令为一个byte整数

```
_nop                  =   0, // 0x00
_aconst_null          =   1, // 0x01
_iconst_0             =   3, // 0x03
_iconst_1             =   4, // 0x04
_dconst_1             =  15, // 0x0f
_bipush               =  16, // 0x10
_iload_0              =  26, // 0x1a
_iload_1              =  27, // 0x1b
_aload_0              =  42, // 0x2a
_istore               =  54, // 0x36
_pop                  =  87, // 0x57
_imul                 = 104, // 0x68
_idiv                 = 108, // 0x6c

```

- void setAge(int) 方法的字节码: 2A 1B B5 00 20 B1
- 2A _aload_0
    - 无参
    - 将局部变量slot0 作为引用 压入操作数栈
- 1B _iload_1
    - 无参
    - 将局部变量slot1 作为整数 压入操作数栈
- B5 _putfield
    - 设置对象中字段的值
    - 参数为2bytes (00 20) (指明了字段)
        - 指向常量池的引用
        - Constant_Fieldref
        - 此处为User.age
    - 弹出栈中2个对象objectref, value
    - 将栈中的value赋给objectref的给定字段
- B1 _return


## 11.3常用的字节码

- 常量入栈
```
aconst_null     null对象入栈
iconst_m1       int常量-1入栈
iconst_0        int常量0入栈
iconst_5
lconst_1        long常量1入栈
fconst_1        float 1.0入栈
dconst_1        double 1.0 入栈
bipush          8位带符号整数入栈
sipush          16位带符号整数入栈
ldc             常量池中的项入栈
```

- 局部变量压栈

```
xload(x为i l f d a)
    分别表示int，long，float，double，object ref
xload_n(n为0 1 2 3)
xaload(x为i l f d a b c s)
    分别表示int, long, float, double, obj ref ,byte,char,short
    从数组中取得给定索引的值，将该值压栈
    iaload
        执行前，栈：..., arrayref, index
        它取得arrayref所在数组的index的值，并将值压栈
        执行后，栈：..., value
```

- 出栈装载入局部变量

```
xstore(x为i l f d a)
    出栈，存入局部变量
xstore_n(n 0 1 2 3)
    出栈，将值存入第n个局部变量
xastore(x为i l f d a b c s)
    将值存入数组中
    iastore
        执行前，栈：...,arrayref, index, value
        执行后，栈：...
        将value存入arrayref[index]
```

- 通用栈操作（无类型）

```
nop
pop
    弹出栈顶1个字长
dup
    复制栈顶1个字长，复制内容压入栈
```

- 类型转化

```
i2l
i2f
l2i
l2f
l2d
f2i
f2d

d2i
d2l
d2f
i2b
i2c
i2s

i2l
    将int转为long
    执行前，栈：..., value
    执行后，栈：...,result.word1,result.word2
    弹出int，扩展为long，并入栈

```

- 整数运算

```
iadd
ladd
isub
lsub
idiv
ldiv
imul
lmul
iinc
```

- 浮点运算

```
fadd
dadd
fsub
dsub
fdiv
ddiv
fmul
dmul
```

- 对象操作指令

```
new
getfield
putfield
getstatic
putstatic
```

- 条件控制

```
ifeq  如果为0，则跳转
ifne  如果不为0，则跳转
iflt   如果小于0 ，则跳转
ifge  如果大于0，则跳转
if_icmpeq 如果两个int相同，则跳转

ifeq
    参数 byte1,byte2
    value出栈 ，如果栈顶value为0则跳转到(byte1<<8)|byte2
    执行前，栈：...,value
    执行后，栈：...
```

- 方法调用

```
invokespecial通常根据引用的类型选择方法，而不是对象的类来选择！即它使用静态绑定而不是动态绑定。
invokespecial对私有方法 超类方法 实例初始化方法
xreturn x为空，表示返回void

invokevirtual
invokespecial
invokestatic
invokeinterface
xreturn(x为 i l f d a 或为空)
```



## 11.4 使用ASM生成Java字节码


- ASM是Java字节码操作框架
- 可以用于修改现有类或者动态产生新类
- 用户
    - AspectJ
    - Clojure
    - Ecplise
    - spring
    - cglib(ASM的封装)
        - hibernate(生成动态代理)

### 11.4.1 ASM HelloWorld

```
ClassWriter cw = new ClassWriter(ClassWriter.COMPUTE_MAXS|ClassWriter.COMPUTE_FRAMES);  
cw.visit(V1_7, ACC_PUBLIC, "Example", null, "java/lang/Object", null);  
MethodVisitor mw = cw.visitMethod(ACC_PUBLIC, "<init>", "()V", null,  null);  
mw.visitVarInsn(ALOAD, 0);  //this 入栈
mw.visitMethodInsn(INVOKESPECIAL, "java/lang/Object", "<init>", "()V");  
mw.visitInsn(RETURN);  
mw.visitMaxs(0, 0);  
mw.visitEnd();  
mw = cw.visitMethod(ACC_PUBLIC + ACC_STATIC, "main",  "([Ljava/lang/String;)V", null, null);  
mw.visitFieldInsn(GETSTATIC, "java/lang/System", "out",  "Ljava/io/PrintStream;");  
mw.visitLdcInsn("Hello world!");  
mw.visitMethodInsn(INVOKEVIRTUAL, "java/io/PrintStream", "println",  "(Ljava/lang/String;)V");  
mw.visitInsn(RETURN);  
mw.visitMaxs(0,0);  
mw.visitEnd();  
byte[] code = cw.toByteArray();  
AsmHelloWorld loader = new AsmHelloWorld();  
Class exampleClass = loader  
    .defineClass("Example", code, 0, code.length);  
exampleClass.getMethods()[0].invoke(null, new Object[] { null }); 

```

### 11.4.2 字节码编织入

- 模拟实现AOP字节码织入
    - 在函数开始部分或者结束部分嵌入字节码
    - 可用于进行鉴权、日志等

```
在操作前加上鉴权或者日志

public class Account { 
	 public void operation() { 
		 System.out.println("operation...."); 
	 } 
}

```

```
我们要嵌入的内容
public class SecurityChecker { 
    public static boolean checkSecurity() { 
    System.out.println("SecurityChecker.checkSecurity ...");
    return true;
    } 
}

```

```
class AddSecurityCheckClassAdapter extends ClassVisitor {
    public AddSecurityCheckClassAdapter( ClassVisitor cv) {
		super(Opcodes.ASM5, cv);
	}
    // 重写 visitMethod，访问到 "operation" 方法时，
    // 给出自定义 MethodVisitor，实际改写方法内容
    public MethodVisitor visitMethod(final int access, final String name, 
        final String desc, final String signature, final String[] exceptions) { 
        MethodVisitor mv = cv.visitMethod(access, name, desc, signature,exceptions);
        MethodVisitor wrappedMv = mv; 
        if (mv != null) { 
            // 对于 "operation" 方法
            if (name.equals("operation")) { 
                // 使用自定义 MethodVisitor，实际改写方法内容
                wrappedMv = new AddSecurityCheckMethodAdapter(mv); 
            } 
        } 
        return wrappedMv; 
    } 
}

```

```
class AddSecurityCheckMethodAdapter extends MethodVisitor { 
	 public AddSecurityCheckMethodAdapter(MethodVisitor mv) { 
		 super(Opcodes.ASM5,mv); 
	 } 
	 public void visitCode() { 
		 visitMethodInsn(Opcodes.INVOKESTATIC, "geym/jvm/ch10/asm/SecurityChecker", 
			"checkSecurity", "()Z"); 
		 super.visitCode();
	 } 
}

```

```
class AddSecurityCheckMethodAdapter extends MethodVisitor { 
	 public AddSecurityCheckMethodAdapter(MethodVisitor mv) { 
		 super(Opcodes.ASM5,mv); 
	 } 
	 public void visitCode() { 
		 visitMethodInsn(Opcodes.INVOKESTATIC, "geym/jvm/ch10/asm/SecurityChecker", 
			"checkSecurity", "()Z"); 
		 super.visitCode();
	 } 
}

```

```
SecurityChecker.checkSecurity ...
operation....

public class Generator{ 
	 public static void main(String args[]) throws Exception { 
		 ClassReader cr = new ClassReader("geym.jvm.ch10.asm.Account"); 
		 ClassWriter cw = new ClassWriter(ClassWriter.COMPUTE_MAXS|ClassWriter.COMPUTE_FRAMES); 
		 AddSecurityCheckClassAdapter classAdapter = new AddSecurityCheckClassAdapter(cw); 
		 cr.accept(classAdapter, ClassReader.SKIP_DEBUG); 
		 byte[] data = cw.toByteArray(); 
		 File file = new File("bin/geym/jvm/ch10/asm/Account.class"); 
		 FileOutputStream fout = new FileOutputStream(file); 
		 fout.write(data); 
		 fout.close(); 
	 } 
}

```


## 11.5 JIT及其相关参数

- 字节码执行性能较差，所以可以对于热点代码编译成机器码再执行，在运行时的编译，叫做JIT Just-In-Time
- JIT的基本思路是，将热点代码，就是执行比较频繁的代码，编译成机器码。

![image](http://clsaaleanjvmimgbed-1252032169.cossh.myqcloud.com/course_sulianchengjin/11-5-1-1.png)

- 在方法编译成机器码的过程中,源字节码未返回函数仍然能够继续执行,这称为栈上替换.

- -XX:CompileThreshold=1000:域值的设置,调用超过这个次数则调用JIT
- -XX:+PrintCompilation:

```
-XX:CompileThreshold=1000
-XX:+PrintCompilation

public class JITTest {

    public static void met(){
        int a=0,b=0;
        b=a+b;
    }
    
    public static void main(String[] args) {
        for(int i=0;i<1000;i++){
            met();
        }
    }
}

```

```
输出
56    1             java.lang.String::hashCode (55 bytes)
56    2             java.lang.String::equals (81 bytes)
57    3             java.lang.String::indexOf (70 bytes)
60    4             java.lang.String::charAt (29 bytes)
61    5             java.lang.String::length (6 bytes)
61    6             java.lang.String::lastIndexOf (52 bytes)
61    7             java.lang.String::toLowerCase (472 bytes)
67    8             geym.jvm.ch2.jit.JITTest::met (9 bytes)

```

```
-Xint
    解释执行
-Xcomp
    全部编译执行
-Xmixed
    默认，混合
```