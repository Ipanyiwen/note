New/delete malloc/free的区别。 - https://www.jianshu.com/p/090dcac5aae3

源码编译成二进制程序的流程。 - https://blog.csdn.net/fyyyr/article/details/79229183

堆和栈区别。除了堆和栈还有哪些储存区。 - https://blog.csdn.net/u012398613/article/details/23215253

为什么要使用虚析构函数，什么时候会使用虚构造函数。 

-    直接的讲，C++中基类采用virtual虚析构函数是为了防止内存泄漏。具体地说，如果派生类中申请了内存空间，并在其析构函数中对这些内存空间进行释放。假设基类中采用的是非虚析构函数，当删除基类指针指向的派生类对象时就不会触发动态绑定，因而只会调用基类的析构函数，而不会调用派生类的析构函数。那么在这种情况下，派生类中申请的空间就得不到释放从而产生内存泄漏。所以，为了防止这种情况的发生，C++中基类的析构函数应采用virtual虚析构函数。

c++中重写/重载/隐藏的区别。 - https://blog.csdn.net/zx3517288/article/details/48976097#:~:text=%E9%9A%90%E8%97%8F%EF%BC%9A%E6%98%AF%E6%8C%87%E6%B4%BE%E7%94%9F%E7%B1%BB,%E7%B1%BB%E5%87%BD%E6%95%B0%E9%83%BD%E4%BC%9A%E8%A2%AB%E9%9A%90%E8%97%8F%E3%80%82&text=%E9%87%8D%E5%86%99(%E8%A6%86%E7%9B%96)%EF%BC%9A%E6%98%AF,%E8%B0%83%E7%94%A8%E8%A2%AB%E9%87%8D%E5%86%99%E5%87%BD%E6%95%B0%E3%80%82

c++11, 14, 17, 20的了解。- https://blog.csdn.net/weixin_43246024/article/details/111311636

左值和右值的区别、move语义, 右值引用。 - https://zhuanlan.zhihu.com/p/335994370

编译、链接过程 - https://blog.csdn.net/czc1997/article/details/81175399

 c++ volatile的用法 - https://blog.csdn.net/lxiao428/article/details/83830983

STL中map和hashmap的区别，使用场合 - https://blog.csdn.net/cws1214/article/details/9842679

