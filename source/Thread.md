# iOS 面试-多线程

- [线程与进程的区别和联系?](#线程与进程的区别和联系)
- [进程间常用通信方式](#进程间常用通信方式)
- [iOS中实现多线程的几种方案](#ios中实现多线程的几种方案)
- [线程死锁的四个条件](#线程死锁的四个条件)
- [任务、队列](#任务、队列)
- [GCD---队列](#gcd---队列)
- [OC 你了解的锁有哪些](#oc-你了解的锁有哪些)
- [Dispatch Semaphore 信号量](#dispatch-semaphore-信号量)
- [NSOperationQueue 的优点](#nsoperationqueue-的优点)
- [NSOperation 和 NSOperationQueue](#nsoperation-和-nsoperationqueue)
- [NSOperation 重写 start 和 main 方法的区别](#nsoperation-重写-start-和-main-方法的区别)
- [怎么用 GCD 实现多读单写](#怎么用-gcd-实现多读单写)

#### 线程与进程的区别和联系?
```
这个问题算是很经典了, 也经常见到
概念:
1、进程是具有一定独立功能的程序、它是系统进行资源分配和调度的一个独立单位，重点在系统调度和单独的单位，
也就是说进程是可以独立运行的一段程序。    
2、线程是进程的一个实体，是 CPU 调度和分派的基本单位，他是比进程更小的能独立运行的基本单位，
线程自己基本上不拥有系统资源。
联系:
1、一个线程只能属于一个进程，而一个进程可以有多个线程，但至少有一个线程（通常说的主线程）。    
2、资源分配给进程，同一进程的所有线程共享该进程的所有资源
3、线程在执行过程中，需要协作同步。不同进程的线程间要利用消息通信的办法实现同步。    
4、处理机分给线程，即真正在处理机上运行的是线程。   
5、线程是指进程内的一个执行单元，也是进程内的可调度实体。
```
#### 进程间常用通信方式
```
1、URL Scheme
2、Keychain
3、UIPasteboard 剪切板功能
4、UIDocumentInteractionController
5、local socket
6、AirDrop
7、UIActivityViewController
8、App Groups
9、Universal Link 通连
```
#### 多线程的理解
```
1.同一时间，CPU 只能处理 1 条线程，只有 1 条线程在执行。
多线程并发执行，其实是 CPU 快速地在多条线程之间调度(切换)。
如果 CPU 调度线程的时间足够快，就造成了多线程并发执行的假象
2.如果线程非常非常多，CPU 会在 N 多线程之间调度，
消耗大量的 CPU 资源，每条线程被调度执行的频次会降低(线程的执行效率降低)
3. 多线程的优点: 能适当提高程序的执行效率，能适当提高资源利用率(CPU、内存利用率)
4. 多线程的缺点:
开启线程需要占用一定的内存空间(默认情况下，主线程占用 1M，子线程占用 512KB)，
如果开启大量的线程，会占用大量的内存空间，降低程序的性能
线程越多，CPU 在调度线程上的开销就越大, 程序设计更加复杂: 比如线程之间的通信、多线程的数据共享
```
#### iOS中实现多线程的几种方案
```
NSThread 面向对象的，需要程序员手动创建线程，但不需要手动销毁。子线程间通信很难。

GCD（Grand Central Dispatch） 是面向底层的 C 语言的 API，充分利用了设备的多核，自动管理线程生命周期。
NSOperation 基于 GCD 封装，更加面向对象，比 GCD 多了一些功能。
进行多线程开发可以控制线程总数及线程依赖关系，可以设置自身的优先级，
还可以判断 Operation 当前的状态(暂停，继续，取消)。

异同：
1. GCD 执行效率更高，而且由于队列中执行的是由 block 构成的任务，这是一个轻量级的数据结构，写起来更方便
2. GCD 只支持 FIFO 的队列，而 NSOperationQueue 可以通过设置最大并发数，设置优先级，添加依赖关系等调整执行顺序
3. NSOperationQueue 甚至可以跨队列设置依赖关系，但是 GCD 只能通过设置串行队列，或者在队列内添加 barrier(dispatch_barrier_async)任务，才能控制执行顺序
4. NSOperationQueue 因为面向对象，所以支持 KVO，可以监测 operation 是否正在执行(isExecuted)、是否结束(isFinished)、是否取消(isCanceld)

使用：
1.实际项目开发中，很多时候只是会用到异步操作，不会有特别复杂的线程关系管理，
所以苹果推崇的且优化完善、运行快速的 GCD 是首选
2.如果考虑异步操作之间的事务性，顺序行，依赖关系，比如多线程并发下载，
 GCD 需要自己写更多的代码来实现，而 NSOperationQueue 已经内建了这些支持
3.不论是 GCD 还是 NSOperationQueue，我们接触的都是任务和队列，都没有直接接触到线程，
事实上线程管理也的确不需要我们操心，系统对于线程的创建，调度管理和释放都做得很好。
而 NSThread 需要我们自己去管理线程的生命周期，还要考虑线程同步、加锁问题，造成一些性能上的开销
```
#### 线程死锁的四个条件
```
（1）互斥条件：一个资源每次只能被一个线程使用。
（2）占有且等待（请求与保持条件）：即线程已经至少保持了一个资源，但又提出了新的资源请求，
而该资源已经被其他线程占有，此时请求线程被阻塞，但对自己已获得的资源保持不放
（3）不可强行占有（不可剥夺条件）: 即线程所获得的资源在未使用完毕之前，不能被其他线程强行夺走，
只能由获得该资源的线程自己主动释放
（4）循环等待条件: 若干线程之间形成一种头尾相接的循环等待资源关系。
```
如：
```
- (void)viewDidLoad {
    [super viewDidLoad];
    // 会出现死锁
    dispatch_sync(dispatch_get_main_queue(), ^{
        NSLog(@"block");
    });
}
在主线程中运用主队列同步，也就是把任务放到了主线程的队列中。 
同步对于任务是立刻执行的，那么当把任务放进主队列时，它就会立马执行，
只有执行完这个任务， viewDidLoad 才会继续向下执行。
而 viewDidLoad 和任务都是在主队列上的，由于队列的先进先出原则，
任务又需等待 viewDidLoad 执行完毕后才能继续执行，
viewDidLoad 和这个任务就形成了相互循环等待，就造成了死锁。 
想避免这种死锁，可以将同步改成异步 dispatch_async,
或者将 dispatch_get_main_queue 换成其他串行或并行队列，都可以解决。
```

#### 任务、队列
```
任务：就是执行操作的意思，也就是在线程中执行的那段代码。在 GCD 中是放在 block 中的。
执行任务有两种方式:同步执行(sync)和异步执行(async)

同步(Sync):同步添加任务到指定的队列中，在添加的任务执行结束之前，会一直等待，
直到队列里面的任务完成之后再继续执行，即会阻塞线程。
只能在当前线程中执行任务(是当前线程，不一定是主线程)，不具备开启新线程的能力。

异步(Async):线程会立即返回，无需等待就会继续执行下面的任务，不阻塞当前线程。
可以在新的线程中执行任务，具备开启新线程的能力(并不一定开启新线程)。
如果不是添加到主队列上，异步会在子线程中执行任务

队列(Dispatch Queue)：这里的队列指执行任务的等待队列，即用来存放任务的队列。
队列是一种特殊的线性表，采用 FIFO(先进先出)的原则，即新任务总是被插入到队列的末尾，
而读取任务的时候总是从队列的头部开始读取。每读取一个任务，则从队列中释放一个任务
在 GCD 中有两种队列:串行队列和并发队列。
两者都符合 FIFO(先进先出)的原则。两者的主要区别是: 执行顺序不同，以及开启线程数不同。

串行队列(SerialDispatchQueue): 同一时间内，队列中只能执行一个任务，
只有当前的任务执行完成之后，才能执行下一个任务。
(只开启一个线程，一个任务执行完毕后，再执行下一个任务)。
主队列是主线程上的一个串行队列,是系统自动为我们创建的。

并发队列(ConcurrentDispatchQueue): 同时允许多个任务并发执行。
(可以开启多个线程，并且同时执行任务)。并发队列的并发功能只有在异步(dispatch_async)函数下才有效

总之：同步和异步指的是任务，并发和串行指的是任务的队列
```
#### GCD---队列
```
而 GCD 共有三种队列类型:
main queue:通过 dispatch_get_main_queue()获得，这是一个与主线程相关的串行队列。

global queue:全局队列是并发队列，由整个进程共享。
存在着高、中、低三种优先级的全局队列。调用 dispath_get_global_queue 并传入优先级来访问队列。

自定义队列:通过函数 dispatch_queue_create 创建的队列
```
#### OC 你了解的锁有哪些
```
互斥锁会在访问被加锁数据时，会休眠等待，当数据解锁，互斥锁会被唤醒。
自旋锁遇到被加锁数据时，会进入死循环等待(忙等)，当数据解锁，自旋锁马上访问。
自旋锁的优点在于，因为自旋锁不会引起调用者睡眠，所以不会进行线程调度，CPU 时间片轮转等耗时操作。
所以如果能在很短的时间内获得锁，自旋锁的效率远高于互斥锁。
缺点在于，自旋锁一直占用 CPU，他在未获得锁的情况下，一直运行--自旋，所以占用着 CPU，
如果不能在很短的时间内获得锁，这无疑会使 CPU 效率降低。自旋锁不能实现递归调用。

OSSpinLock：自旋锁，不安全 iOS10 后不建议使用，可能会出现优先级反转问题
os_unfair_lock：用于取代不安全的 OSSpinLock ，从 iOS10 开始才支持，也是互斥锁
pthread_mutex：互斥锁，等待锁的线程会处于休眠状态
@synchronized：关键字加锁，是对 mutex 递归锁的封装，加锁代码少，性能差
因为里面会加入异常处理, 所以耗时。
适用线程不多，任务量不大的多线程加锁（@synchronized(nil) 不起任何作用）
NSLock：互斥锁，是对 mutex 普通锁的封装。所有锁（包括 NSLock）的接口实际上都是
通过 NSLocking 协议定义的，它定义了 lock 和 unlock 方法。
你使用这些方法来获取和释放该锁。
NSRecursiveLock：递归锁，有时候加锁代码中存在递归调用，递归开始前加锁，
递归开始调用后重复执行此方法以至于加锁代码照成死锁，使用 NSRecursiveLock，便可解决问题。
NSCondition：条件锁，是对 mutex 和 cond 的封装
NSConditionLock：是对 NSCondition 的进一步封装，可以设置具体的条件值
dispatch_semaphore：信号量实现加锁
```
[不再安全的 OSSpinLock](https://blog.ibireme.com/2016/01/16/spinlock_is_unsafe_in_ios/)
性能如下：
![锁的性能. png](./image/锁的性能.png)

#### Dispatch Semaphore 信号量
```
GCD 中的信号量是指 Dispatch Semaphore，是持有计数的信号。 
Dispatch Semaphore 提供了三个函数
1.dispatch_semaphore_create:创建一个 Semaphore 并初始化信号的总量 
2.dispatch_semaphore_signal:发送一个信号，让信号总量加 1 
3.dispatch_semaphore_wait:可以使总信号量减 1，
当信号总量小于 0 时就会一直等待(阻塞所在线程)，否则就可以正常执行。
尽量不要在主线程操作。
Dispatch Semaphore 在实际开发中主要用于:
 保持线程同步，将异步执行任务转换为同步执行任务 
 保证线程安全，为线程加锁
```
#### NSOperationQueue 的优点
```
NSOperation、NSOperationQueue 是苹果提供给我们的一套多线程解决方案。
实际上 NSOperation、 NSOperationQueue 是基于 GCD 更高一层的封装，完全面向对象。
但是比 GCD 更简单易用、代码可读 性也更高。
1、可以添加任务依赖，方便控制执行顺序 
2、可以设定操作执行的优先级 
3、任务执行状态控制:isReady,isExecuting,isFinished,isCancelled
如果只是重写 NSOperation 的 main 方法，由底层控制变更任务执行及完成状态，以及任务退出 
如果重写了 NSOperation 的 start 方法，自行控制任务状态
系统通过 KVO 的方式移除 isFinished==YES 的 NSOperation
3、可以设置最大并发量
```
#### NSOperation 和 NSOperationQueue
```
操作(Operation):
执行操作的意思，换句话说就是你在线程中执行的那段代码。
在 GCD 中是放在 block 中的。
在 NSOperation 中，使用 NSOperation 子类 NSInvocationOperation、 NSBlockOperation，
或者自定义子类来封装操作。
操作队列(OperationQueues):
这里的队列指操作队列，即用来存放操作的队列。不同于 GCD 中的调度队列 FIFO(先进先出)的原则。 
NSOperationQueue 对于添加到队列中的操作，
首先进入准备就绪的状态(就绪状态取决于操作之间的依赖关系)，
然后进入就绪状态的操作的开始执行顺序(非结束执行顺序)由操作之间相对的优先级决定(优先级是操作对象自身的属性)。
操作队列通过设置最大并发操作数(maxConcurrentOperationCount)来控制并发、串行。 
NSOperationQueue 为我们提供了两种不同类型的队列:主队列和自定义队列。主队列运行在主线程之上，
而自定义队列在后台执行。
```
#### NSOperation 重写 start 和 main 方法的区别
```
系统的 NSOperation 中的 start 方法中默认实现是会调用 main 方法，main 方法结束，就意味着NSOperation执行完毕。
如果自定义的 NSOperation，通过重写 start 方法，里面创建具体的任务，并且不要调用 super 去执行 main，因为 main 函数执行完操作就结束了。属于非并发队列。
而 start 方法就算执行完毕，它的 finish 属性也不会变，因此你可以控制这个 operation 的生命周期，在具体的异步任务完成之后手动 cancel 掉这个 operation。是并发队列。
```
#### 怎么用 GCD 实现多读单写
```
多读单写: 可以多个读者同时读取数据，而在读的时候，不能取写入数据。并且，在写的过程中，不能有其他写者去写。
即读者之间是并发的，写者与读者或其他写者是互斥的。
1. 加读写锁（pthread_rwlock）来实现
2. 可以用 dispatch_barrier_sync(栅栏函数)去实现
dispatch_barrier_sync: 提交一个栅栏函数在执行中,它会等待栅栏函数执行完
dispatch_barrier_async: 提交一个栅栏函数在异步执行中,它会立马返回,
而 dispatch_barrier_sync 和 dispatch_barrier_async 的区别也就在于会不会阻塞当前线程

简单示例：
var queue = DispatchQueue(label: "read", attributes: .concurrent)

func getValue(closure: @escaping (String?) -> Void?){
    queue.async {
        closure(self.dic["read"])
        print("read")
    }
}

func setDic(value: String) {
    queue.async(flags: .barrier) {
        self.dic["read"] = value
        print("write")
    }
}
```