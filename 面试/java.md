1. 类加载：

   1.1 类加载过程

   1. 加载 加载字节码文件。
   2. 链接
   3. 验证 验证字节码文件的正确性。
   4. 准备 为静态变量分配内存。
   5. 解析 将符号引用（如类的全限定名）解析为直接引用（类在实际内存中的地址）。
   6. 初始化 为静态变量赋初值。

   

   1.2 类加载器：双亲委派模式

   ​	当一个类需要加载时，判断当前类是否被加载过。已经被加载的类会直接返回，否则才会尝试加载。加载的时候，首先会把该请求委派该父类加载器的 loadClass() 处理，因此所有的请求最终都应该传送到顶层的启动类加载器 BootstrapClassLoader 中。当父类加载器无法处理时，才由自己来处理。当父类加载器为 null 时，会使用启动类加载器 BootstrapClassLoader 作为父类加载器。

   1.3 知道有哪些是违背双亲委派模式的类加载器吗？为什么？ tomcat

   1.4 springboot 类加载器加载过程：**LaunchedURLClassLoader** - https://www.jianshu.com/p/9c07ced8de14

   1.5 spi了解吗？

2. 多线程: 

   2.1 Thread 

   2.2 Runnable 

   2.3 Callable

   2.4 线程池

   1. FixThreadPool 固定数量的线程池，适用于对线程管理，高负载的系统

   2. SingleThreadPool 只有一个线程的线程池，适用于保证任务顺序执行

   3. CacheThreadPool 创建一个不限制线程数量的线程池，适用于执行短期异步任务的小程序，低负载系统

   4. ScheduledThreadPool 定时任务使用的线程池，适用于定时任务
      
      

3. map实现，线程安全的map：concurrentHashMap

4. 锁： synchronized关键字、ReentrantLock： https://blog.csdn.net/GitChat/article/details/104871532

5. java 内存模型： 

    - 程序计数器
    - 虚拟机栈
    - 本地方法栈
    - 堆
    - 方法区

6. jvm 有哪些垃圾回收器？简单介绍一下

7. java io， BIO，NIO，AIO的区别

8. java8 了解吗？有什么改动

9. 了解spring 吗？简单介绍一下

10. spring由哪些模块组成？介绍一下IOC， AOP

11. springboot 和 ssm的区别

