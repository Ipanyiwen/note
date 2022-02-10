1. 切片和数组和字符串

   1.1 切片数据结构. 

   ```go
   type SliceHeader struct {
   	Data uintptr
   	Len  int
   	Cap  int
   }
   ```

   1.2 切片扩容

   1. 如果期望容量大于当前容量的两倍就会使用期望容量；
   2. 如果当前切片的长度小于 1024 就会将容量翻倍；
   3. 如果当前切片的长度大于 1024 就会每次增加 25% 的容量，直到新容量大于期望容量；

   1.3 字符串没有cap，不会扩容

2. map

   2.1 数据结构

   ​	实现： 拉链法，桶+链表的形式

   ​	数据结构： 

   ```go
   type hmap struct {
   	count     int	//当前哈希表中的元素数量；
   
   	flags     uint8   // hash当前的状态
   	B         uint8   // 2^B个桶的容量
   	noverflow uint16  // overflow 计数器，溢出桶的大小，每次创建一个新的溢出桶，计数器+1
   	hash0     uint32  // hash种子
   
   	buckets    unsafe.Pointer // 指向桶的指针
   	oldbuckets unsafe.Pointer // 扩容的时候旧桶的指针,大小是当前buckets的一半
   	nevacuate  uintptr	// 计数器，用于计算旧的桶分流的大小
   
   	extra *mapextra  // 溢出桶，当某个桶满了，就会获取nextOverflow存放新的数据
   }
   
   type mapextra struct {
   	overflow    *[]*bmap   // 溢出桶数组
   	oldoverflow *[]*bmap   // 旧的溢出桶数组
   	nextOverflow *bmap    // 下一个溢出桶的位置
   }
    
   type bmap struct {     // bmap 是bucket 指向的数据，对应拉链法中的桶
       topbits  [8]uint8     // key的高八位，用于快速判断key是否在当前bucket中
       keys     [8]keytype    // 当前bucket存储的key
       values   [8]valuetype  // 当前bucket存储的value
       pad      uintptr       
       overflow uintptr	   // 溢出桶位置
   }
   ```

    2.2 扩容

   1. 装载因子已经超过 6.5； 翻倍扩容。 LoadFactor（装载因子）= hash表中已存储的键值对的总数量/hash桶的个数（即hmap结构中buckets数组的个数）LoadFactor = count / 2^B 
   2. 哈希使用了太多溢出桶；等量扩容，buckets的大小不会发生变化。 溢出桶太多判断： 当桶总数 < 2 ^ 15 时，如果溢出桶总数 >= 桶总数，则认为溢出桶过多。当桶总数 >= 2 ^ 15 时，直接与 2 ^ 15 比较，当溢出桶总数 >= 2 ^ 15 时，即认为溢出桶太多了。

   2.3 正常桶与溢出桶的关系

   1. 当桶的数量小于 2^4 时，由于数据较少、使用溢出桶的可能性较低，会省略创建的过程以减少额外开销；
   2. 当桶的数量多于 2^4 时，会额外创建 2^(B−4) 个溢出桶；
   3. 正常桶和溢出桶都是一段连续的地址空间（数组），不过正常桶会有指向溢出桶的指针

   2.4 并发安全的map: sync.Map

3. mutex， rwmutex

   3.1 mutex加锁过程：

   - 如果互斥锁处于初始化状态，会通过置位 `mutexLocked` 加锁；
   - 如果互斥锁处于 `mutexLocked` 状态并且在普通模式下工作，会进入自旋，执行 30 次 `PAUSE` 指令消耗 CPU 时间等待锁的释放；
   - 如果当前 Goroutine 等待锁的时间超过了 1ms，互斥锁就会切换到饥饿模式；
   - 互斥锁在正常情况下会通过 [`runtime.sync_runtime_SemacquireMutex`](https://draveness.me/golang/tree/runtime.sync_runtime_SemacquireMutex) 将尝试获取锁的 Goroutine 切换至休眠状态，等待锁的持有者唤醒；
   - 如果当前 Goroutine 是互斥锁上的最后一个等待的协程或者等待的时间小于 1ms，那么它会将互斥锁切换回正常模式；

- 3.2 mutex解锁过程：

  - 当互斥锁已经被解锁时，调用 [`sync.Mutex.Unlock`](https://draveness.me/golang/tree/sync.Mutex.Unlock) 会直接抛出异常；
  - 当互斥锁处于饥饿模式时，将锁的所有权交给队列中的下一个等待者，等待者会负责设置 `mutexLocked` 标志位；
  - 当互斥锁处于普通模式时，如果没有 Goroutine 等待锁的释放或者已经有被唤醒的 Goroutine 获得了锁，会直接返回；在其他情况下会通过 [`sync.runtime_Semrelease`](https://draveness.me/golang/tree/sync.runtime_Semrelease) 唤醒对应的 Goroutine；

  3.3 读写锁：

  - 调用`sync.RWMutex.Lock`尝试获取写锁时；
    - 每次 [`sync.RWMutex.RUnlock`](https://draveness.me/golang/tree/sync.RWMutex.RUnlock) 都会将 `readerCount` 其减一，当它归零时该 Goroutine 会获得写锁；
    - 将 `readerCount` 减少 `rwmutexMaxReaders` 个数以阻塞后续的读操作；
  - 调用 [`sync.RWMutex.Unlock`](https://draveness.me/golang/tree/sync.RWMutex.Unlock) 释放写锁时，会先通知所有的读操作，然后才会释放持有的互斥锁；

4. context 用途

   在不同 Goroutine 之间同步请求特定数据、取消信号以及处理请求的截止时间， context.Context会从最顶层的 Goroutine 一层一层传递到最下层。context.Context可以在上层 Goroutine 执行出现错误时，将信号及时同步给下层。

   ```go
   type Context interface {
       Deadline() (deadline time.Time, ok bool)
       Done() <-chan struct{}
       Err() error
       Value(key interface{}) interface{}
   }
   ```

   - context.Background 和 context.TODO 返回预先初始化好了的私有变量 `background` 和 `todo`。这两个私有变量都是通过 `new(emptyCtx)` 语句初始化的，context.emptyCtx实现的都是空方法，没有任何功能
   - context.WithCancel(parent Context) (ctx Context, cancel CancelFunc) 函数： 一旦我们执行返回的取消函数或parent的上下文的Done channel被关闭时，当前上下文以及它的子上下文都会被取消。
   - context.WithDeadline(parent Context, d time.Time) (ctx Context, cancel CancelFunc):  当截止日期超过或取消函数被调用时，该 context 将被取消
   - context.WithTimeout(parent Context, timeout time.Duration) (ctx Context, cancel CancelFunc): 此函数类似于 context.WithDeadline。不同之处在于它将持续时间作为参数输入而不是时间对象
   - context.WithValue(parent Context, key, val interface{}) (ctx Context, cancel CancelFunc): 值 val 与 key 关联，子context可以获取获取到所有父上下文的值，比较常见的使用场景是传递请求对应用户的认证令牌以及用于进行分布式追踪的请求 ID。


5. for 和 range

   for range 循环会在底层转换为for循环

   ```go
   for k, v := range arr {}
   -----------> 
   a1 := arr 
   size := len(arr)
   it := 0
   k := it 
   v := nil
   for ; it < size; it++ {
   	k = it 
   	v = a1[it]
   }
   ```

   range遍历hash表会引入随机数，随机从某个bucket开始

   range遍历string会将一个或者多个byte字节转换为rune，所以和for循环情况不一致

6. select 

   `select` 能够让 Goroutine 同时等待多个 Channel 可读或者可写。

   编译器会对select语句优化： 

   1. 空的 `select` 语句会被转换成调用`runtime.block`直接挂起当前 Goroutine；

   2. 如果select语句中只包含一个case，编译器会将其转换成`if ch == nil { block }; n;`表达式；首先判断操作的 Channel 是不是空的，然后执行 `case` 结构中的内容；
   3. 如果 `select` 语句中只包含两个 `case` 并且其中一个是 `default`，那么会使用 `runtime.selectnbrecv` 和 `runtime.selectnbsend`非阻塞地执行收发操作；

   4. 在默认情况下会通过 `runtime.selectgo` 获取执行 `case` 的索引，并通过多个 `if` 语句执行对应 `case` 中的代码；

   在编译器已经对 `select` 语句进行优化之后，Go 语言会在运行时执行编译期间展开的 runtime.selectgo函数，该函数会按照以下的流程执行：

   1. 随机生成一个遍历的轮询顺序 `pollOrder` 并根据 Channel 地址生成锁定顺序 `lockOrder`；

   2. 根据`pollOrder`遍历所有的`case`查看是否有可以立刻处理的 Channel；

      1. 如果存在，直接获取 `case` 对应的索引并返回；
      2. 如果不存在，创建 `runtime.sudog`结构体，将当前 Goroutine 加入到所有相关 Channel 的收发队列，并调用 `runtime.gopark`挂起当前 Goroutine 等待调度器的唤醒；

   3. 当调度器唤醒当前 Goroutine 时，会再次按照 `lockOrder` 遍历所有的 `case`，从中查找需要被处理的 `runtime.sudog` 对应的索引；

7. channel 

   ```go
   type hchan struct {
   	qcount   uint           // 元素个数；
   	dataqsiz uint           // 循环队列的长度
   	buf      unsafe.Pointer // 缓冲区数据指针
   	elemsize uint16         // 元素大小
   	closed   uint32       
   	elemtype *_type         // 元素类型 
   	sendx    uint           // 发送操作当前队列的位置
   	recvx    uint           // 接收操作当前队列的位置
   	recvq    waitq          // 缓冲区不足阻塞的goroutine
   	sendq    waitq          // 缓冲区不足阻塞的goroutine
   
   	lock mutex              // 互斥锁
   }
   
   type waitq struct { // 双向链表
       first *sudog
       last  *sudog
   }
   ```
   
   channel 是一个用于同步和通信的有锁队列，使用互斥锁解决程序中可能存在的线程竞争问题

   发送流程： 

   1. 如果当前 Channel 的 `recvq` 上存在已经被阻塞的 Goroutine，那么会直接将数据发送给当前 Goroutine 并将其设置成下一个运行的 Goroutine；
   2. 如果 Channel 存在缓冲区并且其中还有空闲的容量，我们会直接将数据存储到缓冲区 `sendx` 所在的位置上；
   3. 如果不满足上面的两种情况，会创建一个 [`runtime.sudog`](https://draveness.me/golang/tree/runtime.sudog) 结构并将其加入 Channel 的 `sendq` 队列中，当前 Goroutine 也会陷入阻塞等待其他的协程从 Channel 接收数据；
   
   接收流程： 

   1. 如果 Channel 为空，那么会直接调用 [`runtime.gopark`](https://draveness.me/golang/tree/runtime.gopark) 挂起当前 Goroutine；
   2. 如果 Channel 已经关闭并且缓冲区没有任何数据，[`runtime.chanrecv`](https://draveness.me/golang/tree/runtime.chanrecv) 会直接返回；
   3. 如果 Channel 的 `sendq` 队列中存在挂起的 Goroutine，会将 `recvx` 索引所在的数据拷贝到接收变量所在的内存空间上并将 `sendq` 队列中 Goroutine 的数据拷贝到缓冲区；
   4. 如果 Channel 的缓冲区中包含数据，那么直接读取 `recvx` 索引对应的数据；
   5. 在默认情况下会挂起当前的 Goroutine，将 [`runtime.sudog`](https://draveness.me/golang/tree/runtime.sudog) 结构加入 `recvq` 队列并陷入休眠等待调度器的唤醒；
   
8. 调度器

      用户态线程绑定内核态线程，1:1 的关系则不需要调度器，全局交给CPU来调度，但是线程资源昂贵，由于操作系统只知道内核态线程，不如多个用户态线程绑定一个内核态线程，即N:1, 那么N个用户态线程我们一般称之为协程，但是N个协程切换、调度不能再交给cpu调度，所以需要调度器来帮助调度，但是这种还是有问题，那就是多个协程序只能在一个线程，浪费了多核机器的能力，所以最优解是M:N，但是调度器的实现则变得比较复杂。

   golang的M:N调度器设计： 

   - G-M模型(已经被废弃)： 调度器从全局队列取一个Goroutine（协程）,然后交给某个线程M执行

   - G-M-P模型：加入了执行器P， P有着大小为256的本地Goroutine队列，所有的 P 都在程序启动时创建，并保存在数组中，最多有 `GOMAXPROCS`(可配置) 个。从G创建新的G‘时，会先入G所在的P的本地队列，当本地队列满的时候，才会入全局队列。每个M会和一个P绑定，M会从P的本地队列中取出G来执行，当P的本地队列为空时，会从全局队列拿一部分G或者从其他P的本地队列中偷一半G放到本地队列。

     https://learnku.com/articles/41728

9. 内存分配器

   空闲链表分配: 隔离适应（Segregated-Fit）— 将内存分割成多个链表，每个链表中的内存块大小相同，申请内存时先找到满足条件的链表，再从链表中选择合适的内存块；

   Go 语言的内存分配器会根据申请分配的内存大小选择不同的处理逻辑，运行时根据对象的大小将对象分成微对象、小对象和大对象三种：
   | 类别 | 大小 | 分配策略 |
   | ------ | ---------- | ---------- |
   | 微对象 | (0, 16B) | 先使用微型分配器，再依次尝试线程缓存、中心缓存和堆分配内存 |
   | 小对象 | [16B, 32KB] | 依次尝试使用线程缓存、中心缓存和堆分配内存 |
   | 大对象 | (32KB, +∞) | 直接在堆上分配内存 |

   **多级缓存机制：** 

   线程缓存： 属于每一个独立的线程，它能够满足线程上绝大多数的内存分配需求，因为不涉及多线程，所以也不需要使用互斥锁来保护内存，这能够减少锁竞争带来的性能损耗

   中心缓存： 当线程缓存不能满足需求时，运行时会使用中心缓存作为补充解决小对象的内存分配，

   页堆缓存： 在遇到 32KB 以上的对象时，内存分配器会选择页堆直接分配大内存。

10. 函数传参： 值传递

11. golang 接口

12. golang 反射机制

13. golang垃圾回收 - https://segmentfault.com/a/1190000022030353
    - 三色标记算法： 
      三色对象：
         - 白色对象 — 潜在的垃圾，其内存可能会被垃圾收集器回收；
         - 黑色对象 — 活跃的对象，包括不存在任何引用外部指针的对象以及从根对象可达的对象；
         - 灰色对象 — 活跃的对象，因为存在指向白色对象的外部指针，垃圾收集器会扫描这些对象的子对象；

      工作原理：

      1. 只要是新创建的对象,默认的颜色都是标记为“白色”.
      2. 从根节点开始遍历所有对象，把遍历到的对象从白色集合放入“灰色”集合。
      3. 遍历灰色集合，将灰色对象引用的对象从白色集合放入灰色集合，之后将此灰色对象放入黑色集合
      4. 重复**第三步**, 直到灰色中无任何对象.
      5. 回收所有的白色标记表的对象. 也就是回收垃圾.

    - 屏障机制

      三色标记算法在以下两种情况下会出现对象丢失： 

      - 条件1: 一个白色对象被黑色对象引用**(白色被挂在黑色下)**
      - 条件2: 灰色对象与它之间的可达关系的白色对象遭到破坏**(灰色同时丢了该白色)**

      要解决对象丢失的问题最简单的方式就是STW，直接禁止掉其他用户程序对对象引用关系的干扰，但是STW资源浪费严重，为了提高性能，需要一个机制来破坏上述两个条件-屏障机制，有两种方式来实现： 

      1. 强三色不变式： 不存在黑色对象引用到白色对象的指针。
      2. 弱三色不变式： 所有被黑色对象引用的白色对象都处于灰色保护状态.即有个灰色的可达链。

      ##### 插入写屏障： 

      ​	在A对象引用B对象的时候，B对象被标记为灰色，满足: **强三色不变式**. (不存在黑色对象引用白色对象的情况了， 因为白色会强制变成灰色)

      ##### 删除写屏障：

      ​	被删除的对象，如果自身为灰色或者白色，那么被标记为灰色。满足: **弱三色不变式**. (保护灰色对象到白色对象的路径不会断

      ##### 混合写屏障：

      golang栈出于性能考虑,不能在栈上加入写屏障，所以插入写屏障和删除写屏障在golang上的问题：

      - 插入写屏障：结束时需要STW来重新扫描栈，标记栈上引用的白色对象的存活；
      - 删除写屏障：回收精度低，GC开始时STW扫描堆栈来记录初始快照，这个过程会保护开始时刻的所有存活对象。

      **混合写屏障规则:**（gc过程中利用规则保持各个goroutine栈中可达对象是黑色，则无需stw）

      1、GC开始将栈上的对象全部扫描并标记为黑色(之后不再进行第二次重复扫描，无需STW)，

      2、GC期间，任何在栈上创建的新对象，均为黑色。

      3、被删除的对象标记为灰色。

      4、被添加的对象标记为灰色。

      `满足`: 变形的**弱三色不变式**.