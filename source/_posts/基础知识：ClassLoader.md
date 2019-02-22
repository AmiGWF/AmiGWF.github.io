---
title: '基础知识: ClassLoader'
copyright: true
date: 2019-02-15 18:11:50
tags:
- classload
categories:
- Java
---

## 1. ClassLoader的作用

我们编写好的Java程序都是由若干个.class文件组织而成，在启动程序运行的时候并不会一次性加载所有的class文件，而是根据程序的需要，通过Java的类加载机制(ClassLoader)动态的加载某些class文件到内存中，从而被其他class使用，所以**ClassLoader就是用来动态加载class文件到内存中的.**

<!-- more -->

---

## 2. Java默认提供的三个ClassLoader

**(1). Bootstrap ClassLoader**
**启动类加载器**，是Java最顶层的类加载器。主要负责加载JDK中的核心类库，如**jdk_xxx/jre/lib/rt.jar，jdk_xxx/jre/lib/resources.jar，jdk_xxx/jre/lib/charsets.jar和class**(还可以通过启动jvm时指定-Xbootclasspath和路径来改变Bootstrap ClassLoader的加载目录，比如java -Xbootclasspath/a:path被指定的文件追加到默认的bootstrap路径中).

```java
//1.从测试类中可以看到本机的加载路径
URL[] urls = sun.misc.Launcher.getBootstrapClassPath().getURLs();  
for (int i = 0; i < urls.length; i++) {  
    System.out.println(urls[i].toExternalForm());  
} 

//打印日志
file:/E:/java_jdk_1.8/Java/jre.8.0_92/lib/resources.jar
file:/E:/java_jdk_1.8/Java/jre.8.0_92/lib/rt.jar
file:/E:/java_jdk_1.8/Java/jre.8.0_92/lib/sunrsasign.jar
file:/E:/java_jdk_1.8/Java/jre.8.0_92/lib/jsse.jar
file:/E:/java_jdk_1.8/Java/jre.8.0_92/lib/jce.jar
file:/E:/java_jdk_1.8/Java/jre.8.0_92/lib/charsets.jar
file:/E:/java_jdk_1.8/Java/jre.8.0_92/lib/jfr.jar
file:/E:/java_jdk_1.8/Java/jre.8.0_92/classes

//上述的打印结果通过查询系统属性 sun.boot.class.path 也可以得知
System.getProperty("sun.boot.class.path")
```

**(2). ExtClassLoader**
**扩展类加载器**，负责加载Java的扩展类库，默认加载**jdk_xxx/jre/lib/ext**目录下的jar和class(还可以加载-D java.ext.dirs选项指定的目录).

**(3). AppClassLoader**
**系统类加载器**，负责加载应用程序classpath目录下所有的jar和class.

**(4). 不同加载器之间的区别**

1. 除了Java默认提供的三个ClassLoader之外，用户还可以根据自身需要自定义ClassLoader，但是必须继承自`java.lang.ClassLoader，上面提到的ExtClassLoader和AppClassLoader也都是继承自java.lang.ClassLoader`;

2. `Bootstrap ClassLoader并不是继承自java.lang.ClassLoader`，因为它并不是一个普通的Java类，而是有C++编写的底层类，并嵌入到JVM中，成为JVM的一部分，当JVM启动后，Bootstrap ClassLoader也随着启动，负责加载完核心类库后，并构造Extension ClassLoader和App ClassLoader类加载器;

3. 以上三个ClassLoader的加载顺序为：`Bootstrap ClassLoader --> ExtClassLoader --> AppClassLoader`.

**(5). 源码解析**

- **1. sun.misc.Launcher : Java虚拟机的入口**

  ```java
  private static Launcher launcher = new Launcher();
  private static String bootClassPath = System.getProperty("sun.boot.class.path");
  //Launcher的构造方法(精简之后的)
  public Launcher() {
    ExtClassLoader localExtClassLoader;
    try {
        localExtClassLoader = ExtClassLoader.getExtClassLoader();
    } catch (IOException localIOException1) {
        throw new InternalError("Could not create extension class loader",
                localIOException1);
    }
    try {
        this.loader = AppClassLoader.getAppClassLoader(localExtClassLoader);
    } catch (IOException localIOException2) {
        throw new InternalError(
                "Could not create application class loader",
                localIOException2);
    }
    Thread.currentThread().setContextClassLoader(this.loader);
  }
  ```

- 源码中可看到Launcher中初始化了`内部类ExtClassLoader`和`AppClassLoader`;

- 内部类ExtClassLoader和AppClassLoader的有相同的继承关系：`ExtClassLoader -- > URLClassLoader --> SecureClassLoader --> ClassLoader`;

- 源码中并没有初始化Bootstrap ClassLoader，只是通过`System.getProperty("sun.boot.class.path")`加载路径，这个就是Bootstrap ClassLoader加载jar包的路径.

- **2. 路径加载的区别**

  ```java

  //Bootstrap ClassLoader -- Launcher中的静态变量加载

  private static String bootClassPath = System.getProperty("sun.boot.class.path");

//ExtClassLoader -- Launcher中的内部类加载
String str = System.getProperty("java.ext.dirs");

//AppClassLoader -- Launcher中的内部类加载
String str = System.getProperty("java.class.path");

```
- 从上面这三个不同的加载路径，可以得到如下的结果:
```java
(1). Bootstrap ClassLoader
E:\java_jdk_1.8\Java\jre.8.0_92\lib\resources.jar;
E:\java_jdk_1.8\Java\jre.8.0_92\lib\rt.jar;E:\java_jdk_1.8\Java\jre.8.0_92\lib\sunrsasign.jar;
E:\java_jdk_1.8\Java\jre.8.0_92\lib\jsse.jar;
E:\java_jdk_1.8\Java\jre.8.0_92\lib\jce.jar;
E:\java_jdk_1.8\Java\jre.8.0_92\lib\charsets.jar;
E:\java_jdk_1.8\Java\jre.8.0_92\lib\jfr.jar;
E:\java_jdk_1.8\Java\jre.8.0_92\classes

(2). ExtClassLoader
E:\java_jdk_1.8\Java\jre.8.0_92\lib\ext;C:\WINDOWS\Sun\Java\lib\ext

(3). AppClassLoader 
当前应用的bin目录，下面存放着class文件
E:\woorkspace_jni\MClassLoader\bin;
```

---

## 3. ClassLoader加载类的原理

**(1). 双亲委托模型说明**

> (1). ClassLoader是使用双亲委托模型来加载类的，每一个ClassLoader的实例都有一个父加载器的引用(并不是父类),虚拟机内置的类加载器(Bootstrap ClassLoader)没有父加载器，但是它可以作为其他ClassLoader实例的父加载器。
> 
> (2). 当某一个ClassLoader的实例需要加载某一个类时，会先去委托其父加载器加载这类，直到自顶层的Bootstrap ClassLoader，如果Bootstrap ClassLoader完成了加载就直接返回，否则就转交给ExtClassLoader去加载，还没有加载就继续转交给AppClassLoader去加载，任然没有加载就返回给发起者，让它指定到文件系统或者网络的URL中加载该类，最终没有找到就会抛出异常ClassNotFoundException。

**(2). 委托流程图**

![](https://i.imgur.com/Oev9fED.png)

**(3). 为何使用双亲委托模型**

> 因为可以避免重复加载，当父类加载了某一个类，字类的ClassLoader没必要再去重新加载一次。从安全角度考虑，假如不使用双亲委托模型，那我们便可以使用自定义的String类来替代Java中的核心api，这样会存在很大的安全隐患，如果使用双亲加载模型，当父加载器加载了之后，子加载器就不会再去加载，这样就无法使用自定义的String类替代Java中的，除非重新定义JDK中ClassLoader的加载算法。

---

## 4. 自定义ClassLoader

**(1). 重要方法：loadClass及其一般实现步骤**

- 执行`findLoadedClass(String)`去检测这个class是不是已经加载过;

- 执行父加载器的loadClass方法,如果父加载器为null，则jvm内置的加载器去替代，也就是Bootstrap ClassLoader。这也解释了ExtClassLoader的parent为null,但仍然说Bootstrap ClassLoader是它的父加载器。

- 如果向上委托父加载器没有加载成功，则通过`findClass(String)`查找。

- 如果class在上面的步骤中找到了，参数resolve又是true的话，那么loadClass()又会调用resolveClass(Class)这个方法来生成最终的Class对象。

**(2). 源码中看如何自定义**

- 图片解释(图片来源忘记了...)

  ![](https://i.imgur.com/cqZwevc.png)

- 源码解释

  ```java 

  //源码一：jdk1.8/jre8 路径下的java.lang.ClassLoader中的源码

  protected Class<?> loadClass(String className, boolean resolve) throws ClassNotFoundException {

        Class<?> clazz = findLoadedClass(className);
      
        if (clazz == null) {
            try {
                clazz = parent.loadClass(className, false);
            } catch (ClassNotFoundException e) {
                // Don't want to see this.
            }
      
            if (clazz == null) {
                clazz = findClass(className);
            }
        }
      
        return clazz;

    }

//源码二：jdk1.7/jre7 路径下的java.lang.ClassLoader中的源码
protected Class<?> loadClass(String name, boolean resolve)
        throws ClassNotFoundException
    {
        synchronized (getClassLoadingLock(name)) {
            // 首先，检测是否已经加载
            Class<?> c = findLoadedClass(name);
            if (c == null) {
                long t0 = System.nanoTime();
                try {
                    if (parent != null) {
                        //父加载器不为空则调用父加载器的loadClass
                        c = parent.loadClass(name, false);
                    } else {
                        //父加载器为空则调用Bootstrap Classloader
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                    // ClassNotFoundException thrown if class not found
                    // from the non-null parent class loader
                }

                if (c == null) {
                    // If still not found, then invoke findClass in order
                    // to find the class.
                    long t1 = System.nanoTime();
                    //父加载器没有找到，则调用findclass
                    c = findClass(name);
    
                    // this is the defining class loader; record the stats
                    sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                    sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                    sun.misc.PerfCounter.getFindClasses().increment();
                }
            }
            if (resolve) {
                //调用resolveClass()
                resolveClass(c);
            }
            return c;
        }
    }

```
- **由源码及注释可以看出，我们自定义ClassLoader时步骤如下：**
> 1.继承java.lang.ClassLoader;
> 2.重写findClass方法.

- **为什么只需要重写findClass？**
> 因为JDK已经在loadClass方法中帮我们实现了ClassLoader搜索类的算法，当在loadClass方法中搜索不到类时，loadClass方法就会调用findClass方法来搜索类，所以我们只需重写该方法即可。如没有特殊的要求，一般不建议重写loadClass搜索类的算法。

- **注意点**
> **1. defineClass()方法**
这个方法在编写自定义classloader的时候非常重要，它能将class二进制内容转换成Class对象，如果不符合要求的会抛出各种异常。
> **2. 一个ClassLoader创建时如果没有指定parent，那么它的parent默认就是AppClassLoader**，比如自定义的ClassLoader，父加载器默认是AppClassLoader，因为这样就能够保证它能访问系统内置加载器加载成功的class文件。

**(3). 开始自定义ClassLoader**
- **1. 编译生成.class文件**
创建MTestLoader.java类，编译生成MTestLoader.class文件，然后可以在项目路径下找到相关.class文件。
```java
public class MTestLoader {
    public static void mLoad() {
        System.out.println("this is a customize ClassLoader");
    }
}
```

- **2. 将上面的.class文件拷贝到备用目录,例如 E:\MLoaderTest**

- **3. 创建自定义ClassLoader文件**

  假定创建的文件名是：`DiskClassLoader`,继承自ClassLoader，然后重写`findClass(String className)`方法 ，该方法中的参数className是在创建ClassLoader对象后，调用loadClass时传入的，例如：`loader.loadClass("om.wd.loader.MTestLoader");`

  ```java

  public class DiskClassLoader extends ClassLoader {

    // class文件加载的路径

    private String loadPath;

    // 获取要加载的class名，假设fileName不为空

    private String getFileName(String fileName) {

        int index = fileName.lastIndexOf('.');
        if (index == -1) {
            return fileName + ".class";
        } else {
            return fileName.substring(index) + ".class";
        }

    }

    public DiskClassLoader(String path) {

        this.loadPath = path;

    }

    protected Class findClass(String name) throws ClassNotFoundException {

        String fileName = getFileName(name);
        File file = new File(loadPath, fileName);
        try {
            FileInputStream inputStream = new FileInputStream(file);
      
            ByteArrayOutputStream bOutputStream = new ByteArrayOutputStream();
            int len = 0;
            while ((len = inputStream.read()) != -1) {
                bOutputStream.write(len);
            }
      
            byte[] data = bOutputStream.toByteArray();
      
            bOutputStream.close();
            inputStream.close();
      
            return defineClass(name, data, 0, data.length);
      
        } catch (Exception e) {
            e.printStackTrace();
        }
        return super.findClass(name);

    }

}

```

- **4. 调用并输出结果**
```java
private static void MTest() {
    try {
        //之前拷贝到备用目录保存的路径
        DiskClassLoader loader = new DiskClassLoader("E:\\MLoaderTest");
        //获取指定的class
        Class c = loader.loadClass"om.wd.loader.MTestLoader");
        Object object = c.newInstance();
        //指定的方法
        Method m = c.getDeclaredMethod("mLoad", null);
        m.invoke(object, null);
    } catch (Exception e) {
        e.printStackTrace();
    }
}

//输出结果
this is a customize ClassLoader
```

**从输出结果可以看到，我们打印出了定义在MTestLoader.mLoad中的语句，由此证明我们自定义的ClassLoader成功加载了MTestLoader.class文件，并执行了里面的mLoad方法。**

---

## 5.contextClassLoader

**1. 简单了解**
contextClassLoader不是实际存在的ClassLoader，它只是Thread中的一个成员变量，包含setContextClassLoader和getContextClassLoader方法，我们可以在使用classloader的时候，动态的通过Thread的setContextClassLoader方法设置contextClassLoaer，从而绕过了双亲加载模型。

```java
public class Thread implements Runnable {
    /**
     * Holds the class loader for this Thread, in case there is one.
     */
    private ClassLoader contextClassLoader;

    /**
     * Set the context ClassLoader for the receiver.
     *
     * @param cl The context ClassLoader
     * @see #getContextClassLoader()
     */
    public void setContextClassLoader(ClassLoader cl) {
        contextClassLoader = cl;
    }

    /**
     * Returns the context ClassLoader for this Thread.
     *
     * @return ClassLoader The context ClassLoader
     * @see java.lang.ClassLoader
     * @see #getContextClassLoader()
     */
    public ClassLoader getContextClassLoader() {
        return contextClassLoader;
    }
}
```

---

**以上就是有关ClassLoader的资料整理**
