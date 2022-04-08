

# JVM

## 类加载机制

类加载： java虚拟机类加载过程是把Class类文件加载到内存，并对Class文件中的数据进行校验、转换、解析和初始化，最终形成可以被虚拟机直接使用的java类型的过程

![](https://img-blog.csdnimg.cn/img_convert/881c5b0064256b4adb4abeccd3ad9a01.png)

```xml
装载：
1.ClassFile --> 字节流 --> 类加载器
2.将字节流代表的静态存储结构转换成方法区的运行时数据结构
3.入口，在我们的java堆中生成一个代表这个类的java.lang.class对象，作为我们的方法区中对应的数据访问入口
```

```xml
链接：
1. 验证 你这个文件不能有问题
2. 准备 为类的静态变量开辟内存，并赋予当前类型的默认值
private static int a = 1;    a = 0;先赋值int的默认值0，
3. 解析 解析是从运行时常量池中的符号引用动态确定具体值的过程。
== 符号引用 转变为 直接引用

4. 初始化  classInit方法   a = 1; 初始化静态代码块  初始化它的父类 ....



```



```java
类加载器  Classloder

java.lang.String        源码.rt.jar
java.lang.String        classpath 业务代码自己定义的


问题： 想办法


```



java1.2前只有一个类加载器





```java
操作数 栈

存储操作数



```



```java
动态链接将这些符号方法引用转换为具体的方法引用
符号引用转变为直接引用

void a(){
    b();
}

void b(){
    c();
}

void c(){
   xxx;
}


   native方法，程序计数器不记录方法
```







## 双亲委派机制

## 沙盒哼安全机制





## java堆




