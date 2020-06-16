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




