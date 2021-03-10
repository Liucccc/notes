## 类加载运行全过程
当我们用java命令运行某个类的main函数启动程序时，首先需要通过类加载器把主类加载到 JVM。
```java
package com.liucccc.jvm;

public class Math {
    public static final int a = 1;
    public static User user = new User();

    // 一个方法对应一块栈桢内存区域
    public int test() {
        int j = 1;
        int k = 2;
        int m = (j + k) * 10;
        return m;
    }

    public static void main(String[] args) {
        Math math = new Math();
        math.test();
    }
}

```
通过Java命令执行代码的大体流程如下：

![](http://ww1.sinaimg.cn/large/005uJn97gy1go6figm87mj30sj0iztb1.jpg)

其中loadClass的类加载过程有如下几步：

**加载 >> 验证 >> 准备 >> 解析 >> 初始化** >> 使用 >> 卸载

1. **加载**：在硬盘上查找并通过IO读入字节码文件，使用到类时才会加载，例如调用类的 main()方法，new对象等等，在加载阶段会在内存中生成一个**代表这个类的 java.lang.Class对象**，作为方法区这个类的各种数据的访问入口
2. **验证**：校验字节码文件的正确性
3. **准备**：给类的静态变量分配内存，并赋予默认值
4. **解析**：将**符号引用**替换为直接引用，该阶段会把一些静态方法(符号引用，比如 main()方法)替换为指向数据所存内存的指针或句柄等(直接引用)，这是所谓的**静态链接**过程(类加载期间完成)，**动态链接**是在程序运行期间完成的将符号引用替换为直接引用
5. **初始化**：对类的静态变量初始化为指定的值，执行静态代码块

![](http://ww1.sinaimg.cn/large/005uJn97gy1go6fqfjpvoj30mf0bjq3s.jpg)

类被加载到方法区中后主要包含 **运行时常量池、类型信息、字段信息、方法信息、类加载器的引用、对应class实例的引用**等信息。

**类加载器的引用**：这个类到类加载器实例的引用

**对应class实例的引用**：类加载器在加载类信息放到方法区中后，会创建一个对应的Class 类型的对象实例放到堆(Heap)中, 作为开发人员访问方法区中类定义的入口和切入点。

**注意**，主类在运行过程中如果使用到其它类，会逐步加载这些类。 jar包或war包里的类不是一次性全部加载的，是使用到时才加载。

```java
package com.liucccc.jvm;

public class TestDynamicLoad {
    static {
        System.out.println("*************load TestDynamicLoad************");
    }

    public static void main(String[] args) {
        new A();
        System.out.println("*************load test************");
        B b = null;
        System.out.println("*************load test b Dynamic************");
        new B();
    }
}

class A {
    static {
        System.out.println("*************load A************");
    }

    public A() {
        System.out.println("*************initial A************");
    }
}

class B {
    static {
        System.out.println("*************load B************");
    }

    public B() {
        System.out.println("*************initial B************");
    }
}

```

运行结果

```
*************load TestDynamicLoad************
*************load A************
*************initial A************
*************load test************
*************load test b Dynamic************
*************load B************
*************initial B************
```

## 类加载器和双亲委派机制

上面的类加载过程主要是通过类加载器来实现的，Java里有如下几种类加载器

1. 引导类加载器()：负责加载支撑JVM运行的位于JRE的lib目录下的核心类库，比如rt.jar、charsets.jar等
2. 扩展类加载器(ExtClassLoader)：负责加载支撑JVM运行的位于JRE的lib目录下的ext扩展目录中的JAR类包
3. 应用程序类加载器(AppClassLoader)：负责加载ClassPath路径下的类包，主要就是加载你自己写的那些类
4. 自定义加载器：负责加载用户自定义路径下的类包

看一个类加载器示例：

```java
package com.liucccc.jvm;

import sun.misc.Launcher;

import java.net.URL;

public class TestJDKClassLoader {
    public static void main(String[] args) {
        System.out.println(String.class.getClassLoader());
        System.out.println(com.sun.crypto.provider.DESKeyFactory.class.getClassLoader());
        System.out.println(TestJDKClassLoader.class.getClassLoader());

        System.out.println();
        ClassLoader appClassLoader = ClassLoader.getSystemClassLoader();
        ClassLoader extClassloader = appClassLoader.getParent();
        ClassLoader bootstrapClassLoader = extClassloader.getParent();
        System.out.println("bootstrapClassLoader："+bootstrapClassLoader);
        System.out.println("extClassloader："+extClassloader);
        System.out.println("appClassLoader："+appClassLoader);

        System.out.println();
        System.out.println("bootstrapLoader加载以下文件：");
        URL[] urLs = Launcher.getBootstrapClassPath().getURLs();
        for (URL urL : urLs) {
            System.out.println(urL);
        }

        System.out.println();
        System.out.println("extClassloader加载以下文件");
        System.out.println(System.getProperty("java.ext.dirs"));

        System.out.println();
        System.out.println("appClassLoader加载以下文件：");
        System.out.println(System.getProperty("java.class.path"));

    }
}

```

运行结果

```
null
sun.misc.Launcher$ExtClassLoader@8efb846
sun.misc.Launcher$AppClassLoader@18b4aac2

bootstrapClassLoader：null
extClassloader：sun.misc.Launcher$ExtClassLoader@8efb846
appClassLoader：sun.misc.Launcher$AppClassLoader@18b4aac2

bootstrapLoader加载以下文件：
file:/Library/Java/JavaVirtualMachines/jdk1.8.0_231.jdk/Contents/Home/jre/lib/resources.jar
file:/Library/Java/JavaVirtualMachines/jdk1.8.0_231.jdk/Contents/Home/jre/lib/rt.jar
file:/Library/Java/JavaVirtualMachines/jdk1.8.0_231.jdk/Contents/Home/jre/lib/sunrsasign.jar
file:/Library/Java/JavaVirtualMachines/jdk1.8.0_231.jdk/Contents/Home/jre/lib/jsse.jar
file:/Library/Java/JavaVirtualMachines/jdk1.8.0_231.jdk/Contents/Home/jre/lib/jce.jar
file:/Library/Java/JavaVirtualMachines/jdk1.8.0_231.jdk/Contents/Home/jre/lib/charsets.jar
file:/Library/Java/JavaVirtualMachines/jdk1.8.0_231.jdk/Contents/Home/jre/lib/jfr.jar
file:/Library/Java/JavaVirtualMachines/jdk1.8.0_231.jdk/Contents/Home/jre/classes

extClassloader加载以下文件
/Users/liucccc/Library/Java/Extensions:/Library/Java/JavaVirtualMachines/jdk1.8.0_231.jdk/Contents/Home/jre/lib/ext:/Library/Java/Extensions:/Network/Library/Java/Extensions:/System/Library/Java/Extensions:/usr/lib/java

appClassLoader加载以下文件：
/Library/Java/JavaVirtualMachines/jdk1.8.0_231.jdk/Contents/Home/jre/lib/charsets.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_231.jdk/Contents/Home/jre/lib/deploy.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_231.jdk/Contents/Home/jre/lib/ext/cldrdata.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_231.jdk/Contents/Home/jre/lib/ext/dnsns.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_231.jdk/Contents/Home/jre/lib/ext/jaccess.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_231.jdk/Contents/Home/jre/lib/ext/jfxrt.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_231.jdk/Contents/Home/jre/lib/ext/localedata.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_231.jdk/Contents/Home/jre/lib/ext/nashorn.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_231.jdk/Contents/Home/jre/lib/ext/sunec.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_231.jdk/Contents/Home/jre/lib/ext/sunjce_provider.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_231.jdk/Contents/Home/jre/lib/ext/sunpkcs11.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_231.jdk/Contents/Home/jre/lib/ext/zipfs.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_231.jdk/Contents/Home/jre/lib/javaws.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_231.jdk/Contents/Home/jre/lib/jce.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_231.jdk/Contents/Home/jre/lib/jfr.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_231.jdk/Contents/Home/jre/lib/jfxswt.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_231.jdk/Contents/Home/jre/lib/jsse.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_231.jdk/Contents/Home/jre/lib/management-agent.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_231.jdk/Contents/Home/jre/lib/plugin.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_231.jdk/Contents/Home/jre/lib/resources.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_231.jdk/Contents/Home/jre/lib/rt.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_231.jdk/Contents/Home/lib/ant-javafx.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_231.jdk/Contents/Home/lib/dt.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_231.jdk/Contents/Home/lib/javafx-mx.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_231.jdk/Contents/Home/lib/jconsole.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_231.jdk/Contents/Home/lib/packager.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_231.jdk/Contents/Home/lib/sa-jdi.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_231.jdk/Contents/Home/lib/tools.jar:/Users/liucccc/Java/java-learning/jvm/target/classes:/Users/liucccc/.m2/repository/org/springframework/boot/spring-boot-starter/2.4.3/spring-boot-starter-2.4.3.jar:/Users/liucccc/.m2/repository/org/springframework/boot/spring-boot/2.4.3/spring-boot-2.4.3.jar:/Users/liucccc/.m2/repository/org/springframework/spring-context/5.3.4/spring-context-5.3.4.jar:/Users/liucccc/.m2/repository/org/springframework/spring-aop/5.3.4/spring-aop-5.3.4.jar:/Users/liucccc/.m2/repository/org/springframework/spring-beans/5.3.4/spring-beans-5.3.4.jar:/Users/liucccc/.m2/repository/org/springframework/spring-expression/5.3.4/spring-expression-5.3.4.jar:/Users/liucccc/.m2/repository/org/springframework/boot/spring-boot-autoconfigure/2.4.3/spring-boot-autoconfigure-2.4.3.jar:/Users/liucccc/.m2/repository/org/springframework/boot/spring-boot-starter-logging/2.4.3/spring-boot-starter-logging-2.4.3.jar:/Users/liucccc/.m2/repository/ch/qos/logback/logback-classic/1.2.3/logback-classic-1.2.3.jar:/Users/liucccc/.m2/repository/ch/qos/logback/logback-core/1.2.3/logback-core-1.2.3.jar:/Users/liucccc/.m2/repository/org/apache/logging/log4j/log4j-to-slf4j/2.13.3/log4j-to-slf4j-2.13.3.jar:/Users/liucccc/.m2/repository/org/apache/logging/log4j/log4j-api/2.13.3/log4j-api-2.13.3.jar:/Users/liucccc/.m2/repository/org/slf4j/jul-to-slf4j/1.7.30/jul-to-slf4j-1.7.30.jar:/Users/liucccc/.m2/repository/jakarta/annotation/jakarta.annotation-api/1.3.5/jakarta.annotation-api-1.3.5.jar:/Users/liucccc/.m2/repository/org/springframework/spring-core/5.3.4/spring-core-5.3.4.jar:/Users/liucccc/.m2/repository/org/springframework/spring-jcl/5.3.4/spring-jcl-5.3.4.jar:/Users/liucccc/.m2/repository/org/yaml/snakeyaml/1.27/snakeyaml-1.27.jar:/Users/liucccc/.m2/repository/org/slf4j/slf4j-api/1.7.30/slf4j-api-1.7.30.jar:/Applications/IntelliJ IDEA.app/Contents/lib/idea_rt.jar

```

## 类加载器初始化过程

参见类运行加载全过程图可知其中会创建JVM启动器实例sun.misc.Launcher

sun.misc.Launcher初始化使用了单例模式设计，保证一个JVM虚拟机内只有一个 sun.misc.Launcher实例。

在Launcher构造方法内部，其创建了两个类加载器，分别是sun.misc.Launcher.ExtClassLoader(扩展类加载器)和sun.misc.Launcher.AppClassLoader(应用类加载器)。

JVM默认使用Launcher的getClassLoader()方法返回的类加载器AppClassLoader的实例加载我们的应用程序

```java
public Launcher() {
  Launcher.ExtClassLoader var1;
  try {
    //构造扩展类加载器，在构造的过程中将其父加载器设置为null
    var1 = Launcher.ExtClassLoader.getExtClassLoader();
  } catch (IOException var10) {
    throw new InternalError("Could not create extension class loader", var10);
  }

  try {
    //构造应用类加载器，在构造的过程中将其父加载器设置为ExtClassLoader，
    //Launcher的loader属性值是AppClassLoader，我们一般都是用这个类加载器来加载我们自己写的应用程序
    this.loader = Launcher.AppClassLoader.getAppClassLoader(var1);
  } catch (IOException var9) {
    throw new InternalError("Could not create application class loader", var9);
  }

  Thread.currentThread().setContextClassLoader(this.loader);
  String var2 = System.getProperty("java.security.manager");
  // 省略代码
}
```

## 双亲委派机制

> 机制

![](http://ww1.sinaimg.cn/large/005uJn97gy1go6ghsjz51j30f70fmdgj.jpg)

这里类加载其实就有一个**双亲委派机制**，加载某个类时会先委托父加载器寻找目标类，找不到再委托上层父加载器加载，如果所有父加载器在自己的加载类路径下都找不到目标类，则在自己的类加载路径中查找并载入目标类。

 比如我们的Math类，最先会找应用程序类加载器加载，应用程序类加载器会先委托扩展类加载器加载，扩展类加载器再委托引导类加载器，顶层引导类加载器在自己的类加载路径里找了半天没找到Math类，则向下退回加载Math类的请求，扩展类加载器收到回复就自己加载，在自己的类加载路径里找了半天也没找到Math类，又向下退回Math类的加载请求给应用程序类加载器， 应用程序类加载器于是在自己的类加载路径里找Math类，结果找到了就自己加载了。 **双亲委派机制说简单点就是，先找父亲加载，不行再由儿子自己加载**

> 双亲委派核心代码

```java
protected Class<?> loadClass(String name, boolean resolve)
        throws ClassNotFoundException
    {
        synchronized (getClassLoadingLock(name)) {
            // First, check if the class has already been loaded
          	// 先检查该类是否已经加载过了
            Class<?> c = findLoadedClass(name);
            if (c == null) {
                long t0 = System.nanoTime();
                try {
                    // 如果当前类加载器有父类加载器
                    if (parent != null) {
                        // 委托父类加载器加载，其实还会调用当前方法
                        c = parent.loadClass(name, false);
                    } else {
                        // 如果当前类加载器没有父，委托引导类加载器加载，引导类加载器是C写的
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                    // ClassNotFoundException thrown if class not found
                    // from the non-null parent class loader
                }

                if (c == null) {
                    // If still not found, then invoke findClass in order
                    // to find the class.
                    // 如果仍然没加载到该类，该类加载器自己尝试加载
                    long t1 = System.nanoTime();
                    //都会调用URLClassLoader的findClass方法在加载器的类路径里查找并加载该类
                    c = findClass(name);

                    // this is the defining class loader; record the stats
                    sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                    sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                    sun.misc.PerfCounter.getFindClasses().increment();
                }
            }
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }
    }
```

> 双亲委派机制有什么优势

1. **沙箱安全机制**：自己写的java.lang.String.class类不会被加载，这样便可以防止核心 API库被随意篡改
2. **避免类的重复加载**：当父亲已经加载了该类时，就没有必要子ClassLoader再加载一 次，**保证被加载类的唯一性**

举个例子

```java
package java.lang;

/**
 * String
 *
 * @author liuchao
 * @created date 2021/3/3 21:40
 */
public class String {
    public static void main(String[] args) {
        System.out.println("自定义String");
    }
}
```

运行输出

```
错误: 在类 java.lang.String 中找不到 main 方法, 请将 main 方法定义为:
   public static void main(String[] args)
否则 JavaFX 应用程序类必须扩展javafx.application.Application
```

## 全盘负责委托机制

“全盘负责”是指当一个ClassLoder装载一个类时，除非显示的使用另外一个ClassLoder，该类所依赖及引用的类也由这个ClassLoder载入。

## 自定义类加载器

自定义类加载器只需要继承 java.lang.ClassLoader 类，该类有两个核心方法，一个是 loadClass(String, boolean)，实现了双亲委派机制，还有一个方法是findClass，默认实现是空方法，所以自定义类加载器主要是重写findClass方法。

```java
package com.liucccc.jvm;

import java.io.FileInputStream;
import java.lang.reflect.Method;

public class MyClassLoaderTest {
    static class MyClassLoader extends ClassLoader {
        private String classPath;

        public MyClassLoader(String classPath) {
            this.classPath = classPath;
        }

        private byte[] loadByte(String name) throws Exception {
            name = name.replaceAll("\\.", "/");
            FileInputStream fis = new FileInputStream(classPath + "/" + name + ".class");
            int len = fis.available();
            byte[] data = new byte[len];
            fis.read(data);
            fis.close();
            return data;
        }

        @Override
        protected Class<?> findClass(String name) throws ClassNotFoundException {
            try {
                byte[] data = this.loadByte(name);
                //defineClass将一个字节数组转为Class对象，这个字节数组是class文件读取后最终的字节数组。
                return defineClass(name, data, 0, data.length);
            } catch (Exception e) {
                e.printStackTrace();
                throw new ClassNotFoundException();
            }
        }
    }

    public static void main(String[] args) throws Exception {
        //初始化自定义类加载器，会先初始化父类ClassLoader，其中会把自定义类加载器的父加载器设置为应用程序类加载器AppClassLoader
        MyClassLoader myClassLoader = new MyClassLoader("/Users/liucccc/Java/temp");
        Class<?> clazz = myClassLoader.loadClass("com.liucccc.jvm.User1");
        Object obj = clazz.newInstance();
        Method method = clazz.getDeclaredMethod("method1", null);
        method.invoke(obj, null);
        System.out.println(clazz.getClassLoader());
    }
}
```

User1.class

```java
package com.liucccc.jvm;

public class User1 {
    private String username;
    private String password;

    public User1() {

    }

    public User1(String username, String password) {
        this.username = username;
        this.password = password;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public void method1() {
        System.out.println("自定义类加载器加载类调用方法");
    }

}
```

输出结果

```
自定义类加载器加载类调用方法
com.liucccc.jvm.MyClassLoaderTest$MyClassLoader@5ca881b5
```

## 打破双亲委派机制

打破双亲委派，自定义类加载器加载自己实现的java.lang.String.class

```java
package com.liucccc.jvm;

import java.io.FileInputStream;
import java.lang.reflect.Method;

public class MyClassLoaderTest2 {
    static class MyClassLoader extends ClassLoader {
        private String classPath;

        public MyClassLoader(String classPath) {
            this.classPath = classPath;
        }

        private byte[] loadByte(String name) throws Exception {
            name = name.replaceAll("\\.", "/");
            FileInputStream fis = new FileInputStream(classPath + "/" + name + ".class");
            int len = fis.available();
            byte[] data = new byte[len];
            fis.read(data);
            fis.close();
            return data;
        }

        @Override
        protected Class<?> findClass(String name) throws ClassNotFoundException {
            try {
                byte[] data = this.loadByte(name);
                //defineClass将一个字节数组转为Class对象，这个字节数组是class文件读取后最终的字节数组。
                return defineClass(name, data, 0, data.length);
            } catch (Exception e) {
                e.printStackTrace();
                throw new ClassNotFoundException();
            }
        }

        /**
         * 重写类加载方法，实现自己的加载逻辑，不委派给父加载
         *
         * @param name    name
         * @param resolve resolve
         */
        @Override
        protected Class<?> loadClass(String name, boolean resolve) throws ClassNotFoundException {
            synchronized (getClassLoadingLock(name)) {
                // First, check if the class has already been loaded
                Class<?> c = findLoadedClass(name);
                if (c == null) {
                    // If still not found, then invoke findClass in order
                    // to find the class.
                    long t1 = System.nanoTime();
                    c = this.findClass(name);

                    // this is the defining class loader; record the stats
                    //sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                    sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                    sun.misc.PerfCounter.getFindClasses().increment();
                }
                if (resolve) {
                    resolveClass(c);
                }
                return c;
            }
        }
    }

    public static void main(String[] args) throws Exception {
        //初始化自定义类加载器，会先初始化父类ClassLoader，其中会把自定义类加载器的父加载器设置为应用程序类加载器AppClassLoader
        MyClassLoader myClassLoader = new MyClassLoader("/Users/liucccc/Java/temp");
        //尝试用自己改写类加载机制去加载自己写的java.lang.String.classjava
        Class<?> clazz = myClassLoader.loadClass("java.lang.String");
        Object obj = clazz.newInstance();
        Method method = clazz.getDeclaredMethod("method1", null);
        method.invoke(obj, null);
        System.out.println(clazz.getClassLoader());
    }
}
```

输出结果

```
java.lang.SecurityException: Prohibited package name: java.lang
	at java.lang.ClassLoader.preDefineClass(ClassLoader.java:655)
	at java.lang.ClassLoader.defineClass(ClassLoader.java:754)
	at java.lang.ClassLoader.defineClass(ClassLoader.java:635)
	at com.liucccc.jvm.MyClassLoaderTest2$MyClassLoader.findClass(MyClassLoaderTest2.java:35)
	at com.liucccc.jvm.MyClassLoaderTest2$MyClassLoader.loadClass(MyClassLoaderTest2.java:57)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:351)
	at com.liucccc.jvm.MyClassLoaderTest2.main(MyClassLoaderTest2.java:76)
Exception in thread "main" java.lang.ClassNotFoundException
	at com.liucccc.jvm.MyClassLoaderTest2$MyClassLoader.findClass(MyClassLoaderTest2.java:38)
	at com.liucccc.jvm.MyClassLoaderTest2$MyClassLoader.loadClass(MyClassLoaderTest2.java:57)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:351)
	at com.liucccc.jvm.MyClassLoaderTest2.main(MyClassLoaderTest2.java:76)
```
> Hotspot源码JVM启动执行main方法流程

![Hotspot源码JVM启动执行main方法流程.jpg](http://ww1.sinaimg.cn/large/005uJn97gy1go7khkn91lj31k3128gpm.jpg)

