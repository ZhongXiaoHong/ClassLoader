# ClassLoader
Android ClassLoader 总结

## JVM VS  Dalvik

Android应用程序运行在Dalvik/ART虚拟机上，并且一个应用程序对应一个单独的Dalvik虚拟机实例。
Dalvik虚拟机实际上也可以算是一个Java虚拟机，只不过它执行的不是class文件，而是dex文件。
Dalvik虚拟机与java虚拟机共享差不多的特性，差别在于两者指行的指令集不一样，前者的指令集时基于寄存器，后者的指令是基于堆栈的 
另外一点class文件是一个文件一个class,dex文件则是一个文件多个class


> 基于栈的虚拟机，基于寄存器的虚拟机
