## JDK体系结构

![clipboard.png](http://ww1.sinaimg.cn/large/005uJn97gy1go7kmtr2p9j30r30fik98.jpg)

## Java语言的跨平台特性

![clipboard (1).png](http://ww1.sinaimg.cn/large/005uJn97gy1go7knz6xgej30dg0fqdh5.jpg)

## JVM整体结构及内存模型

![JVM内存模型.png](http://ww1.sinaimg.cn/large/005uJn97gy1go7kpcng8oj30vo0l8mzx.jpg)

## JVM内存参数设置

![clipboard (2).png](http://ww1.sinaimg.cn/large/005uJn97gy1go7kua2czmj30g70afglv.jpg)

> Spring Boot程序的JVM参数设置格式(Tomcat启动直接加在bin目录下catalina.sh文件里)：

```java
java -Xms2048M -Xmx2048M -Xmn1024M -Xss512K -XX:MetaspaceSize=256M -XX:MaxMetaspaceSize=256M -jar microservice-eureka-server.jar
```

1. **-Xss**：每个线程的栈大小(最小160k)

2. **-Xms**：初始堆大小，默认物理内存的1/64

3. **-Xmx**：最大堆大小，默认物理内存的1/4

4. **-Xmn**：新生代大小

5. **-XX:NewSize**：设置新生代初始大小

6. **-XX:NewRatio**：默认2表示新生代占年老代的1/2，占整个堆内存的1/3。

7. **-XX:SurvivorRatio**：默认8表示一个survivor区占用1/8的Eden内存，即1/10的新生代内存。

   关于元空间的JVM参数有两个：-XX:MetaspaceSize=N和 -XX:MaxMetaspaceSize=N
   
8. **-XX:MaxMetaspaceSize**： 设置元空间最大值， 默认是-1， 即不限制， 或者说只受限于本地内存大小。
   
9. **-XX:MetaspaceSize**： 指定元空间触发Fullgc的初始阈值(元空间无固定初始大小)， 以字节为单位，默认是21M左右，达到该值就会触发full gc进行类型卸载， 同时收集器会对该值进行调整： 如果释放了大量的空间， 就适当降低该值； 如果释放了很少的空间， 那么在不超过-XX：MaxMetaspaceSize（如果设置了的话） 的情况下， 适当提高该值。这个跟早期jdk版本的**-XX:PermSize** 参数意思不一样，**-XX:PermSize** 代表永久代的初始容量。

  由于调整元空间的大小需要Full GC，这是非常昂贵的操作，如果应用在启动的时候发生大量Full GC，通常都是由于永久代或元空间发生了大小调整，基于这种情况，一般建议在JVM参数中将MetaspaceSize和MaxMetaspaceSize设置成一样的值，并设置得比初始值要大，对于8G物理内存的机器来说，一般我会将这两个值都设置为256M。

> StackOverflowError、-Xss160K

```java
package com.liucccc.jvm;

public class StackOverflowTest {
    static int count = 0;

    static void redo() {
        count++;
        redo();
    }

    public static void main(String[] args) {
        try {
            redo();
        } catch (Throwable t) {
            t.printStackTrace();
            System.out.println(count);
        }
    }
}

```

输出结果

```
java.lang.StackOverflowError
	at com.liucccc.jvm.StackOverflowTest.redo(StackOverflowTest.java:14)
	at com.liucccc.jvm.StackOverflowTest.redo(StackOverflowTest.java:14)
	at com.liucccc.jvm.StackOverflowTest.redo(StackOverflowTest.java:14)
	at com.liucccc.jvm.StackOverflowTest.redo(StackOverflowTest.java:14)
	at com.liucccc.jvm.StackOverflowTest.redo(StackOverflowTest.java:14)
	at com.liucccc.jvm.StackOverflowTest.redo(StackOverflowTest.java:14)
	at com.liucccc.jvm.StackOverflowTest.redo(StackOverflowTest.java:14)
	at com.liucccc.jvm.StackOverflowTest.redo(StackOverflowTest.java:14)
	at com.liucccc.jvm.StackOverflowTest.redo(StackOverflowTest.java:14)
	at com.liucccc.jvm.StackOverflowTest.redo(StackOverflowTest.java:14)
...
850
```

**结论：**

-Xss设置越小count值越小，说明一个线程栈里能分配的栈帧就越少，但是对JVM整体来说能开启的线程数会更多

## **日均百万级订单交易系统如何设置JVM参数**

![亿级流量电商系统JVM参数设置优化.png](http://ww1.sinaimg.cn/large/005uJn97gy1go7llolsgrj30rr147ju8.jpg)



**结论：** 

**通过上面这些内容介绍，大家应该对JVM优化有些概念了，就是尽可能让对象都在新生代里分配和回收，尽量别让太多对象频繁进入老年代，避免频繁对老年代进行垃圾回收，同时给系统充足的内存大小，避免新生代频繁的进行垃圾回收**


