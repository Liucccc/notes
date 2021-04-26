## 对象内存分配

> 对象内存分配流程图

![undefined](http://ww1.sinaimg.cn/large/005uJn97gy1gocu8q14rpj31n60o20ur.jpg)

> 对象栈上分配

​		为了减少临时对象在堆内分配的数量，JVM通过**逃逸分析**确定该对象不会被外部访问。如果不会逃逸可以将该对象在**栈上分配**内存，这样该对象所占用的内存空间就可以随栈帧出栈而销毁，就减轻了垃圾回收的压力

对象逃逸分析：就是分析对象动态作用域，当一个对象在方法中被定义后，它可能被外部方法所引用，例如作为调用参数传递到其他地方中

```java
public class EscapeAnalysis {
    /**
     * 对象发生了逃逸，user对象的作用域逃出了方法
     */
    public User test1() {
        User user = new User();
        user.setUsername("123");
        // ...省略中间代码
        return user;
    }

    /**
     * 对象没有发生逃逸，user对象的作用域还在方法内
     * 这样的对象其实可以将其分配在栈内存中，
     * 方法结束时跟随栈内存一起被回收掉
     */
    public void test2() {
        User user = new User();
        user.setUsername("123");
        // ...省略中间代码
    }
}
```

​		JVM对于这种情况可以通过开启逃逸分析参数(-XX:+DoEscapeAnalysis)来优化对象内存分配位置，使其通过**标量替换**优先分配在栈上(**栈上分配**)，**JDK7之后默认开启逃逸分析**，如果要关闭使用参数(-XX:-DoEscapeAnalysis)

**标量替换：**通过逃逸分析确定该对象不会被外部访问，并且对象可以被进一步分解时，**JVM不会创建该对象**，而是将该对象成员变量分解若干个被这个方法使用的成员变量所代替，这些代替的成员变量在栈帧或寄存器上分配空间，这样就不会因为没有一大块连续空间导致对象内存不够分配。开启标量替换参数(-XX:+EliminateAllocations)，**JDK7之后默认开启**

**标量与聚合量：**标量即不可被进一步分解的量，而JAVA的基本数据类型就是标量（如：int，long等基本数据类型以及reference类型等），标量的对立就是可以被进一步分解的量，而这种量称之为聚合量。而在JAVA中对象就是可以被进一步分解的聚合量

**栈上分配依赖于逃逸分析和标量替换同时使用：-XX:+DoEscapeAnalysis -XX:+EliminateAllocations**

## 对象在Eden区分配

大多数情况下，对象在新生代中 Eden 区分配。当 Eden区没有足够空间进行分配时，虚拟机将发起一次Minor GC

1. **Minor GC/Young GC**：指发生新生代的的垃圾收集动作，Minor GC非常频繁，回收速度一般也比较快
2. **Major GC/Full GC**：一般会回收老年代 ，年轻代，方法区的垃圾，Major GC的速度一般会比Minor GC的慢10倍以上

**Eden与Survivor区默认8:1:1，JVM默认有这个参数-XX:+UseAdaptiveSizePolicy(默认开启)，会导致这个8:1:1比例自动变化，如果不想这个比例有变化可以设置参数-XX:-UseAdaptiveSizePolicy**

## 大对象直接进入老年代

​		大对象就是需要大量连续内存空间的对象（比如：字符串、数组）。JVM参数 **-XX:PretenureSizeThreshold** 可以设置大对象的大小，如果对象超过设置大小会直接进入老年代，不会进入年轻代，**这个参数只在 Serial 和ParNew两个收集器下有效**。

比如设置JVM参数：-XX:PretenureSizeThreshold=1000000 (单位是字节)  -XX:+UseSerialGC  ，再执行下上面的第一个程序会发现大对象直接进了老年代

**为什么要这样呢？**

​		为了避免为大对象分配内存时的复制操作而降低效率。

## 长期存活的对象将进入老年代

​		虚拟机采用了**分代收集**的思想来管理内存，那么内存回收时就必须能识别哪些对象应放在新生代，哪些对象应放在老年代中。为了做到这一点，虚拟机给每个对象一个对象年龄（Age）计数器

​		如果对象在 Eden 出生并经过第一次 Minor GC 后仍然能够存活，并且能被 Survivor 容纳的话，将被移动到 Survivor 空间中，并将对象年龄设为1。对象在 Survivor 中每熬过一次 MinorGC，年龄就增加1岁，当它的年龄增加到一定程度（默认为15岁，CMS收集器默认6岁，不同的垃圾收集器会略微有点不同），就会被晋升到老年代中。对象晋升到老年代的年龄阈值，可以通过参数 **-XX:MaxTenuringThreshold** 来设置

## 对象动态年龄判断

​		当前放对象的Survivor区域里(其中一块区域，放对象的那块s区)，一批对象的总大小大于这块Survivor区域内存大小的50%(-XX:TargetSurvivorRatio可以指定)，那么此时**大于等于**这批对象年龄最大值的对象，就可以直接进入老年代了，例如Survivor区域里现在有一批对象，年龄1+年龄2+年龄n的多个年龄对象总和超过了Survivor区域的50%，此时就会把年龄n(含)以上的对象都放入老年代。这个规则其实是希望那些可能是长期存活的对象，尽早进入老年代。**对象动态年龄判断机制一般是在minor gc之后触发的**

## 老年代空间分配担保机制

年轻代每次**minor gc**之前JVM都会计算下老年代**剩余可用空间**
如果这个可用空间小于年轻代里现有的所有对象大小之和(**包括垃圾对象**)
就会看一个“-XX:-HandlePromotionFailure”(jdk1.8默认就设置了)的参数是否设置了
如果有这个参数，就会看看老年代的可用内存大小，是否大于之前每一次minor gc后进入老年代的对象的**平均大小**。
如果上一步结果是小于或者之前说的参数没有设置，那么就会触发一次Full gc，对老年代和年轻代一起回收一次垃圾，如果回收完还是没有足够空间存放新的对象就会发生"OOM"
当然，如果minor gc之后剩余存活的需要挪动到老年代的对象大小还是大于老年代可用空间，那么也会触发full gc，full gc完之后如果还是没有空间放minor gc之后的存活对象，则也会发生“OOM”

![undefined](http://ww1.sinaimg.cn/large/005uJn97gy1gocuqfxyrvj30to0k40vi.jpg)

## 对象内存回收

1. **引用计数法**

   给对象中添加一个引用计数器，每当有一个地方引用它，计数器就加1；当引用失效，计数器就减1；任何时候计数器为0的对象就是不可能再被使用的

   **这个方法实现简单，效率高，但是目前主流的虚拟机中并没有选择这个算法来管理内存，其最主要的原因是它很难解决对象之间相互循环引用的问题**，如下例子：

   ```java
   /**
    * 引用计数法，很难解决对象之间相互循环引用的问题的例子
    */
   public class ReferenceCountingGc {
       Object instance = null;
   
       public static void main(String[] args) {
           ReferenceCountingGc objA = new ReferenceCountingGc();
           ReferenceCountingGc objB = new ReferenceCountingGc();
           objA.instance = objB;
           objB.instance = objA;
           objA = null;
           objB = null;
       }
   }
   ```

2. **可达性分析算法**

   将**“GC Roots”** 对象作为起点，从这些节点开始向下搜索引用的对象，找到的对象都标记为**非垃圾对象**，其余未标记的对象都是垃圾对象

   **GC Roots**根节点：线程栈的本地变量、静态变量、本地方法栈的变量等等

![undefined](http://ww1.sinaimg.cn/large/005uJn97gy1gocuuho035j30cx08zwer.jpg)

## 常见引用类型

**强引用**、**软引用**、弱引用、虚引用

```java
/**
 * 常见引用类型
 */
public class ReferenceType {
    /**
     * 强引用：普通的变量引用
     */
    public static User user1 = new User();
    /**
     * 软引用：将对象用SoftReference软引用类型的对象包裹，正常情况不会被回收，
     * 但是GC做完后发现释放不出空间存放新的对象，则会把这些软引用的对象回收掉。
     * 软引用可用来实现内存敏感的高速缓存
     */
    public static SoftReference<User> user2 = new SoftReference<User>(new User());

    /**
     * 弱引用：将对象用WeakReference软引用类型的对象包裹，
     * 弱引用跟没引用差不多，GC会直接回收掉，很少用
     */
    public static WeakReference<User> user3 = new WeakReference<User>(new User());

    /**
     * 虚引用：虚引用也称为幽灵引用或者幻影引用，它是最弱的一种引用关系，几乎不用
     */
}
```

## finalize()方法最终判定对象是否存活。不推荐使用

​		即使在可达性分析算法中不可达的对象，也并非是“非死不可”的，这时候它们暂时处于“缓刑”阶段，要真正宣告一个对象死亡，至少要经历再次标记过程。

**标记的前提是对象在进行可达性分析后发现没有与GC Roots相连接的引用链。**

**1. 第一次标记并进行一次筛选。**

筛选的条件是此对象是否有必要执行finalize()方法。

当对象没有覆盖finalize方法，对象将直接被回收。

**2. 第二次标记**

如果这个对象覆盖了finalize方法，finalize方法是对象脱逃死亡命运的最后一次机会，如果对象要在finalize()中成功拯救自己，只要重新与引用链上的任何的一个对象建立关联即可，譬如把自己赋值给某个类变量或对象的成员变量，那在第二次标记时它将移除出“即将回收”的集合。如果对象这时候还没逃脱，那基本上它就真的被回收了。

注意：一个对象的finalize()方法只会被执行一次，也就是说通过调用finalize方法自我救命的机会就一次。

finalize()方法的运行代价高昂， 不确定性大， 无法保证各个对象的调用顺序， 如今已被官方明确声明为不推荐使用的语法

## 如何判断一个类是无用的类

方法区主要回收的是无用的类，那么如何判断一个类是无用的类呢？

类需要同时满足下面3个条件才能算是 **“无用的类”**

1. 该类所有的对象实例都已经被回收，也就是 Java 堆中不存在该类的任何实例
2. 加载该类的 ClassLoader 已经被回收（此条件很苛刻，很难被回收，除非自定义类加载）
3. 该类对应的 java.lang.Class 对象没有在任何地方被引用，无法在任何地方通过反射访问该类的方法




