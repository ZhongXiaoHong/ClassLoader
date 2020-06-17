# ClassLoader
Android ClassLoader 总结

> JVM VS  Dalvik

Android应用程序运行在Dalvik/ART虚拟机上，并且一个应用程序对应一个单独的Dalvik虚拟机实例。
Dalvik虚拟机实际上也可以算是一个Java虚拟机，只不过它执行的不是class文件，而是dex文件。
Dalvik虚拟机与java虚拟机共享差不多的特性，差别在于两者指行的指令集不一样，前者的指令集时基于寄存器，后者的指令是基于堆栈的 
另外一点class文件是一个文件一个class,dex文件则是一个文件多个class


> 什么是基于栈的虚拟机，什么是基于寄存器的虚拟机

对于基于栈的虚拟机来讲，每一个运行的线程都有自己私有的虚拟机栈，栈中记录了方法的调用历史，每一次调用方法会对应生成一个栈帧压入虚拟机栈，每一栈帧又包含局部变量表、操作数栈、动态链接、返回地址这几部分。详细可以参考![](https://github.com/ZhongXiaoHong/JVM)

寄存器是CPU的一部分，是一个有限存储容量的高速存储部件，可以用来暂存指令、数据
![](https://github.com/ZhongXiaoHong/ClassLoader/blob/master/61788888.jpg)

运行大致过程如下：
首先根据当前的程序计数器，从指令集读取指令，交给指令寄存器，转化为CPU指令，比如上图 0 号指令表示去加载地址为100的数值1到数据寄存器AX,
1 号指令表示去加载地址为104的数值1到数据寄存器BX,2 号指令将A 、B数据寄存器通过ALU算数逻辑单元计算之后将结果保存在CX，3 号指令将数据寄存器CX数值保存在108地址上


基于寄存器的虚拟机
基于寄存器的虚拟机没有操作数栈，局部变量表，但是有很多虚拟寄存器，相当于虚拟寄存器把操作数栈，局部变量表给合并了。和操作数栈相同，这些虚拟的寄存器也是存放在运行时栈中，本质上是一个数组，与JVM相似，在DalvikVM中每个线程都有自己的程序计数器、调用栈，方法调用生成的栈帧也是会进入调用栈的,虚拟寄存器实际上就是模拟了基于栈的虚拟机中的栈帧的操作数栈压栈出栈计算然后在压栈到局部变量表的过程，实际上虚拟寄存器可以看成对这个步骤的优化，做同样的事用更简单的方式

那基于栈的虚拟机栈与基于寄存器的虚拟机有什么区别呢？

```java
   public void test() {
        int a = 1;
        int b = 2;
        int c = a + b;
    }

```
基于栈的虚拟机生成的指令集如下：

![](https://github.com/ZhongXiaoHong/ClassLoader/blob/master/6179999999999999999.jpg)

基于寄存器的虚拟机

![](https://github.com/ZhongXiaoHong/ClassLoader/blob/master/617666666666666.jpg)

可以发现基于寄存器的指令数明显减少，数据移动次数也减少

**Android虚拟机Dalvik、ART都是基于寄存器的虚拟机**


> ART VS Dalvik

Dalvik虚拟机运行的是dex字节码，解释执行（首先要先翻译成机器码，然后在执行），Android2.2之后又支持JIT即时编译,在程序运行的过程中选择热点代码（经常执行的代码）进行编译成机器码，所以Dalvik是解释执行+JIT，apk安装的时候会执行一个dexopt操作，将dex文件转化成Odex文件。

ART虚拟机执行的是本地机器码，
apk安装的时候会利用dex2oat工具，将dex文件编译成本地机器码，这个过程被称为预先编译机制AOT（Ahead  of Time）


**android 5.0开始正式切换到ART,Android的5.0、6.0安装apk会比较慢，这是因为安装的时候多了编译机器码的过程（AOT）,所以在Android N (7.0)之后又做了优化**

**ART 在Android N 之后的优化：**

应用安装时不作AOT,运行过程中解释执行，对经常调用的方法进行JIT,经过JIT编译的方法会记录到profile配置文件中，在手机充电等有空闲的场景下，编译守护进程会运行，根据profile文件对常用代码进行AOT编译生成art文件，待下次运行直接使用

JIT编译的代码只有本次运行有效，程序退出就没了
art.odex是机器码，dalvik的odex是dex


> android ClassLoader的继承树

![](https://github.com/ZhongXiaoHong/ClassLoader/blob/master/6176666668888888888.jpg)

比如：
Activity的ClassLoader是BootClassLoader
AppCompatActivity的ClassLoader是PathClassLoader，因为AppCompatActivity是兼容包的类，不是Framwork中的类


> PathClassLoader是如何加载类的

PathClassLoader通过loadClass方法加载，这个方法来自父类的父类ClassLoader

```java

    
   protected Class<?> loadClass(String name, boolean resolve)
        throws ClassNotFoundException
    {
            // First, check if the class has already been loaded
            //TODO 找缓存
            Class<?> c = findLoadedClass(name);
            if (c == null) {
                try {
                    if (parent != null) {
                    //TODO parent 也是ClassLoader 到底是具体类型呢？
                    //parent 是ClassLoader对象一个属性也是ClassLoader类型
                    //这里调用loadClass前，先调用parent的loadClass,可想而知，parent里面又会调用parent的parent的loadClass
                    //这是责任链模式,
                    //对于PathClassLoader 这个parent是谁呢？实际上是BootClassLoader而不是BaseDexClassLoader,
                    //BaseDexClassLoader是在继承关系上是PathClassLoader的直接父类，这里不能被parent这个命名误导，就认为是类的父类
                    //这里直接把parent理解成一个普通变量，是ClassLoader类型，是创建ClassLoader对象是外界传进来的
                    //系统创建PathClassLoader时传进来的是BootClassLoader
                        c = parent.loadClass(name, false);
                    } else {
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                    // ClassNotFoundException thrown if class not found
                    // from the non-null parent class loader
                }

                if (c == null) {
                    // If still not found, then invoke findClass in order
                    // to find the class.
                    //TODO  parent加载不到再自己加载，这个机制叫做双亲委托机制
                    c = findClass(name);
                }
            }
            return c;
            
 
 
 protected Class<?> findClass(String name) throws ClassNotFoundException {
        List<Throwable> suppressedExceptions = new ArrayList<Throwable>();
        //TODO  从DexPathList查找,详细怎么查找见下文
        Class c = pathList.findClass(name, suppressedExceptions);
        if (c == null) {
            ClassNotFoundException cnfe = new ClassNotFoundException(
                    "Didn't find class \"" + name + "\" on path: " + pathList);
            for (Throwable t : suppressedExceptions) {
                cnfe.addSuppressed(t);
            }
            throw cnfe;
        }
        return c;
    }
    

```

> 双亲委托机制的作用

1.安全：防止核心API库被篡改，如果没有双亲委托机制，即去除  c = parent.loadClass(name, false)直接自己loadClass是不安全的，比如说开发者自己定义了一个String类包名类名和系统的String类完全相同，这个时候调用加载的时候就只会加载自己编写的String类，系统的String就不会被调用，自己编写的String类就顶替了系统的String，就相当于系统核心APi被篡改了，如果有双亲委托机制这种情况就不会出现，BootClassLoader首先加载系统的String类直接返回了，自己编写的String没机会被加载。

2.避免重复加载：当父类加载器已经加载过该类的时候，就没有必要子ClassLoader在加载一次

> 为什么PathClassLoader可以加载应用程序的类

```
  public PathClassLoader(String dexPath, ClassLoader parent) {
        super(dexPath, null, null, parent);
    }
```
可以看到在创建PathClassLoader的时候会传一个dexPath地址，这个就是应用程序的dex路径，根据这个地址就可以加载到。

> 理解DexPathList

```java
  public BaseDexClassLoader(String dexPath, File optimizedDirectory,
            String librarySearchPath, ClassLoader parent, boolean isTrusted) {
        super(parent);
        this.pathList = new DexPathList(this, dexPath, librarySearchPath, null, isTrusted);

        if (reporter != null) {
            reportClassLoaderChain();
        }
    }
```
DexPathList在BaseDexClassLoader构造方法中被创建

```java
     DexPathList(ClassLoader definingContext, String dexPath,
            String librarySearchPath, File optimizedDirectory, boolean isTrusted) {
        if (definingContext == null) {
            throw new NullPointerException("definingContext == null");
        }

        if (dexPath == null) {
            throw new NullPointerException("dexPath == null");
        }

        if (optimizedDirectory != null) {
            if (!optimizedDirectory.exists())  {
                throw new IllegalArgumentException(
                        "optimizedDirectory doesn't exist: "
                        + optimizedDirectory);
            }

            if (!(optimizedDirectory.canRead()
                            && optimizedDirectory.canWrite())) {
                throw new IllegalArgumentException(
                        "optimizedDirectory not readable/writable: "
                        + optimizedDirectory);
            }
        }

        this.definingContext = definingContext;

        ArrayList<IOException> suppressedExceptions = new ArrayList<IOException>();
        // save dexPath for BaseDexClassLoader
        //TODO spliDexPath回对dexPath分离，比如说当要加载a、b两个dex的时候，dexPath的值
        //TODO 就是这样传值 /a/a.dex:/a/b.dex,所以通过spliDexPath可以分离出每个要加载的dex的路径
        //TODO 调用makeDexElement生成Element数组，一个dex生成一个Element
        //TODO 所以这里就是将dex包装成一个Element
        this.dexElements = makeDexElements(splitDexPath(dexPath), optimizedDirectory,
                                           suppressedExceptions, definingContext, isTrusted);

        // Native libraries may exist in both the system and
        // application library paths, and we use this search order:
        //
        //   1. This class loader's library path for application libraries (librarySearchPath):
        //   1.1. Native library directories
        //   1.2. Path to libraries in apk-files
        //   2. The VM's library path from the system property for system libraries
        //      also known as java.library.path
        //
        // This order was reversed prior to Gingerbread; see http://b/2933456.
        this.nativeLibraryDirectories = splitPaths(librarySearchPath, false);
        this.systemNativeLibraryDirectories =
                splitPaths(System.getProperty("java.library.path"), true);
        List<File> allNativeLibraryDirectories = new ArrayList<>(nativeLibraryDirectories);
        allNativeLibraryDirectories.addAll(systemNativeLibraryDirectories);

        this.nativeLibraryPathElements = makePathElements(allNativeLibraryDirectories);

        if (suppressedExceptions.size() > 0) {
            this.dexElementsSuppressedExceptions =
                suppressedExceptions.toArray(new IOException[suppressedExceptions.size()]);
        } else {
            dexElementsSuppressedExceptions = null;
        }
    
    
```

> ClassLoader自己加载类是如何加载的

```java

 protected Class<?> findClass(String name) throws ClassNotFoundException {
        List<Throwable> suppressedExceptions = new ArrayList<Throwable>();
        //TODO  从DexPathList查找,
        Class c = pathList.findClass(name, suppressedExceptions);
        if (c == null) {
            ClassNotFoundException cnfe = new ClassNotFoundException(
                    "Didn't find class \"" + name + "\" on path: " + pathList);
            for (Throwable t : suppressedExceptions) {
                cnfe.addSuppressed(t);
            }
            throw cnfe;
        }
        return c;
    }
    
    
      public Class<?> findClass(String name, List<Throwable> suppressed) {
        for (Element element : dexElements) {
        //TODO 遍历每一个Element查找要加载的类
            Class<?> clazz = element.findClass(name, definingContext, suppressed);
            if (clazz != null) {
                return clazz;
            }
        }

        if (dexElementsSuppressedExceptions != null) {
            suppressed.addAll(Arrays.asList(dexElementsSuppressedExceptions));
        }
        return null;
    }
    
```

2-----17：36















