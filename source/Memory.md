# iOS 面试 - 内存管理

- [iOS 内存管理](#ios-内存管理)
- [什么是 ARC（ARC 是为了解决什么问题诞生的）](#什么是-arc（arc-是为了解决什么问题诞生的）)
- [iOS 内存分配](#ios-内存分配)
- [内存管理方案](#内存管理方案)
- [什么是悬垂指针？什么是 野指针?](#什么是悬垂指针？什么是-野指针)
- [深拷贝与浅拷贝分别是什么？](#深拷贝与浅拷贝分别是什么？)
- [循环引用](#循环引用)
- [MRC（手动引用计数）和 ARC(自动引用计数)](#mrc（手动引用计数）和-arc自动引用计数)
- [ARC 的 retainCount 怎么存储的？](#arc-的-retaincount-怎么存储的？)
- [一个 NSObject 对象占用多少个字节](#一个-nsobject-对象占用多少个字节)
- [内存泄漏的几种原因](#内存泄漏的几种原因)
- [常见的循环引用，及解决办法](#常见的循环引用，及解决办法)
- [简述一下自动释放池(@autoreleasePool)底层怎么实现？](#简述一下自动释放池autoreleasepool底层怎么实现？)
- [Dealloc 的实现机制](#dealloc-的实现机制)
- [64bit 和 32bit 下 long 和 char 所占字节](#64bit-和-32bit-下-long-和-char-所占字节)
- [retain、release 的实现机制?](#retain、release-的实现机制)

#### iOS 内存管理
```
在 Objective-C 的内存管理中，其实就是引用计数 (reference count) 的管理。且需要遵循黄金法则.
iOS 内存管理方法有两种：手动引用计数 (Manual Reference Counting) 和
自动引用计数(Automatic Reference Counting)。
从 OS X Lion 和 iOS 5 开始，不再需要程序员手动调用 retain 和 release 方法
来管理 Objective-C 对象的内存，而是引入一种新的内存管理机制 Automatic Reference Counting(ARC)，
简单来说，它让编译器来代替程序员来自动加入 retain 和 release 方法来持有和放弃对象的所有权。
只有 oc 对象需要进行内存管理, 因为创建对象时，指向对象的指针放在栈中，由系统维护，
而指针指向的对象，则是放在堆中，需要开发人员维护
非 oc 对象类型比如基本数据类型不需要进行内存管理, 会放到栈中.
对于内存管理可能产生的问题有:
野指针错误：访问了一块坏的内存（已经被回收的，不可用的内存），会有 EXC_BAD_ACCESS 错误
僵尸对象：所占内存已经被回收的对象，僵尸对象不能再被使用（打开僵尸对象检测）。
空指针：没有指向任何东西的指针（存储的东西是 0,null,nil），给空指针发送消息不会报错。
```
#### 什么是 ARC（ARC 是为了解决什么问题诞生的）
```
ARC 是 Auto Reference Counting 的缩写，即自动引用计数，由编译器在代码合适的位置中
自动添加 retain/Release/Autorelease/dealloc 方法从而进行内存管理.
ARC 几个要点：
在对象被创建时 retain count +1，在对象被 release 时 retain count -1. 
当 retain count 为 0 时，销毁对象。
程序中加入 autoreleasepool 的对象会由系统自动加上 autorelease 方法，如果该对象引用计数为 0，则销毁。
那么 ARC 是为了解决什么问题诞生的呢？这个得追溯到 MRC 手动内存管理时代说起。
MRC 下内存管理的缺点：
当我们要释放一个堆内存时，首先要确定指向这个堆空间的指针都被 release 了。（避免提前释放）
释放指针指向的堆空间，首先要确定哪些指针指向同一个堆，这些指针只能释放一次。
（MRC 下即谁创建，谁释放，避免重复释放）
模块化操作时，对象可能被多个模块创建和使用，不能确定最后由谁去释放。
多线程操作时，不确定哪个线程最后使用完毕
```
#### iOS 内存分配
```
内核区: 一些系统内核方法
栈(stack): 方法调用，局部变量等，是连续的，高地址往低地址扩展
堆(heap): 通过 alloc 等分配的对象，是离散的，低地址往高地址扩展，需要我们手动控制
未初始化数据(bss): 未初始化的全局变量等
已初始化数据(data): 已初始化的全局变量等
代码段(text): 程序代码
注意：
1. 系统使用一个链表来维护所有已经分配的内存空间（系统仅仅记录，并不管理具体的内容）
2. 变量使用结束后，需要释放内存，OC 中是判断引用计数是否为 0，
如果是就说明没有任何变量使用该空间，那么系统将其回收
3. 当一个 app 启动后，代码区、常量区、全局区大小就已经固定，
因此指向这些区的指针不会产生崩溃性的错误。
而堆区和栈区是时时刻刻变化的（堆的创建销毁，栈的弹入弹出），
所以当使用一个指针指向这个区里面的内存时，一定要注意内存是否已经被释放，
否则会产生程序崩溃（也即是野指针报错）
```
![内存分配. png](./image/内存分配.png)

#### 内存管理方案
```
Tagged Pointer:
1. 专门用来存储小的对象，例如 NSNumber 和 NSDate
2. TaggedPointer 指针值不再是地址，而是真正的值。
3. 节省内存，提高执行效率。
判断对象是否在使用 TaggedPointer，是看标志位是否为 1

NONPOINTER_ISA:
非指针型 isa : 值的部分代表 class 地址
指针型 isa：值代表 class 地址
64 bit 存储一个内存地址显然是种浪费。于是可以优化存储方案，用一部分额外的存储空间存储其他内容。
第一位的 0 或 1 代表是纯地址型 isa 指针，还是 NONPOINTER_ISA 指针。 
第二位，代表是否有关联对象
第三位代表是否有 C++ 代码。
接下来33位代表指向的内存地址
接下来有 弱引用 的标记 接下来有是否 delloc 的标记....等等

散列表: 复杂的数据结构，包括了引用计数表和弱引用表
通过 SideTables() 结构来实现的，SideTables()结构下，有很多 SideTable 的数据结构。
而 SideTable 当中包含了自旋锁，引用计数表，弱引用表。
SideTables() 实际上是一个哈希表，通过对象的地址来计算该对象的引用计数在哪个 SideTable 中。
自旋锁原理：如果锁被其他线程获取，当前线程会不断去探测锁是否被释放。
若有释放，会第一时间去获取这个锁。普通的锁线程没有获取到会从用户态切换为内核态进行休眠，
而使用自旋锁的线程不会休眠，只会忙等。

引用计数表：实际是用 hash 表实现的。应用计数会存在多张 sideTable 中。修改引用计数，需要经过两次 hash 算法，
第一次是从 sideTables 中找到具体的 sideTable。第二次是从 sideTable 中找到对应的引用计数。
之所以设计成多张 sideTable 而不是一张 sideTable，是因为每次操作都需要加锁，减锁操作。
多张可以分离锁，加快操作速度。

弱引用计数表：苹果使用 sideTables 保存所有的 weak 引用。key 就是对象，weak_entry_t 作为值。
weak_entry_t 中保存了所有指向该对象的弱引用。
```
#### 什么是悬垂指针？什么是 野指针?
```
悬垂指针: 指针指向的内存已经被释放了，但是指针还存在，这就是一个悬垂指针或者说迷途指针
野指针: 没有进行初始化的指针，其实都是野指针
```
#### 深拷贝与浅拷贝分别是什么？
```
浅拷贝：只复制指向对象的指针，而不复制引用对象本身。只是新创建了类的空间，然后将属性的值复制一遍；
  对于属性所指向的内存空间并没有重新创建；因此通过浅拷贝的新旧两个对象的属性
  其实还是指向一块相同的内存空间 
深拷贝：复制引用和对象本身。不仅仅新创建了类的空间，还新创建了每一个属性对应的空间，
  所以深拷贝也称为完全拷贝；通过深拷贝得来的新对象和旧对象，两个对象的属性都是
  指向各自的内存空间，不再共享空间
另外可变对象复制（copy，mutableCopy）的都是深拷贝; 不可变对象 copy 是浅拷贝，
mutableCopy 是深拷贝。
但是注意 copy 返回的都是不可变对象，如果对 copy 返回值去调用可变对象的接口就会 crash. 
另外：
1. 对不可变的数组、字典、集合等，copy 是指针拷贝，mutablecopy 是内容拷贝
2. 对于可变的数组、字典、集合等，copy，mutablecopy 都是内容拷贝
但是，对于集合对象的内容复制仅仅是对对象本身，但是对象的里面的元素还是指针复制。要想复制整个
集合对象，就要用集合深复制的方法，有两种:
(1)使用 initWithArray:copyItems: 方法，将第二个参数设置为 YES 即可
(2)将集合对象进行归档 (archive) 然后解归档 (unarchive):
```
#### 循环引用
```
循环引用的实质：多个对象相互之间有强引用，不能释放让系统回收。
如何解决循环引用？
1、避免产生循环引用，通常是将 strong 引用改为 weak 引用。 
比如在修饰属性时用 weak 在 block 内调用对象方法时，使用其弱引用
在 MRC 下，__block 不会增加其引用计数，避免了循环引用
在 ARC 下，__block 修饰对象会被强引用，无法避免循环引用，需要手动解除。
2、代理 (delegate) 循环引用属于相互循环引用
delegate 是 iOS 中开发中比较常遇到的循环引用，一般在声明 delegate 的时候都要
使用弱引用 weak, 或者 assign ,
当然怎么选择使用 assign 还是 weak，MRC 的话只能用 assign，在 ARC 的情况下最好使用 weak，
因为 weak 修饰的变量在释放后自动指向 nil，防止野指针存在
3、NSTimer 循环引用属于相互循环使用
如果是不重复定时器，在回调方法里将定时器 invalidate 并置为 nil 即可。 如果是重复定时器，
在合适的位置将其 invalidate 并置为 nil 即可，还可以定义中间层让他弱引用。
4、block 循环引用 
由于 block 会对 block 中的对象进行持有操作, 就相当于持有了其中的对象，
而如果此时 block 中的对象又持有了该 block，则会造成循环引用。
解决方案就是使用 __weak 修饰 self 即可。还应该在 block 中对 对象使用 __strong 修饰，
使得在 block 期间对 对象持有，block 执行结束后，解除其持有。
```
#### MRC（手动引用计数）和 ARC(自动引用计数)
```
MRC：alloc，retain，release，retainCount,autorelease,dealloc
ARC：
1.ARC 是 LLVM 和 Runtime 协作的结果
2.ARC 禁止手动调用 retain，release，retainCount,autorelease 关键字
3.ARC 新增 weak，strong 关键字

引用计数管理：
alloc: 经过一系列函数调用，最终调用了 calloc 函数，这里并没有设置引用计数为 1
retain: 经过两次哈希查找，找到其对应引用计数值，然后将引用计数加 1(实际是加偏移量)
release：和 retain 相反，经过两次哈希查找，找到其对应引用计数值，然后将引用计数减 1
```
#### ARC 的 retainCount 怎么存储的？
```
存在 64 张哈希表中，根据哈希算法去查找所在的位置，无需遍历，十分快捷

散列表（引用计数表、weak 表）
- SideTables 表在非嵌入式的 64 位系统中，有 64 张 SideTable 表
- 每一张 SideTable 主要是由三部分组成。自旋锁、引用计数表、弱引用表。
- 全局的引用计数之所以不存在同一张表中，是为了避免资源竞争，解决效率的问题。
- 引用计数表中引入了分离锁的概念，将一张表分拆成多个部分，对他们分别加锁，可以实现并发操作，提升执行效率

引用计数表（哈希表）
通过指针的地址，查找到引用计数的地址，大大提升查找效率
通过 DisguisedPtr(objc_object) 函数存储，同时也通过这个函数查找，这样就避免了循环遍历。
```
#### 一个 NSObject 对象占用多少个字节
```
系统分配了 16 个字节给 NSObject 对象（通过 malloc_size 函数获取）
但是 NSObject 对象内部只使用了 8 个字节内存的空间（64bit 环境下，可通过 class_getInstanceSize 函数获得）
```
#### 内存泄漏的几种原因
```
1、代理强引用导致循环引用：使用 weak 修饰即可
2、NSTimer 不正确使用导致，可使用 block 创建定时器，但也要 invalidate
3、block 导致循环引用
使用 __weak typeof(self) weakself = self;
4、通知造成的内存泄漏
记得在 dealloc 移除通知即可
5、CoreGraphics 框架里申请的内存忘记释放
6、CoreFoundation 框架里申请的内存忘记释放   
7、WKWebView addScriptMessageHandler 导致内存泄漏可以用另一个类代替解决，也需要在 dealloc 移除  
8、地图使用导致内存泄漏，记得在 viewDidDisappear 销毁即可 
```
#### 常见的循环引用，及解决办法
```
1. 父类与子类
如：在使用 UITableView 的时候，将 UITableView 给 Cell 使用，cell 中的 strong 引用会造成循环引用。
解决：strong 改为 weak
2.block
self 将 block 作为自己的属性变量，而在 block 的方法体里面又引用了 self 本身，
此时就很简单的形成了一个循环引用。
解决：__weak typeof(self) weakSelf = self;
     __strong typeof (weakSelf) strongSelf = weakSelf;
3.Delegate
@property (nonatomic, weak) id <TestDelegate> delegate;
如果将 weak 改为 strong，则会造成循环引用
4.NSTimer
NSTimer 的 target 对传入的参数都是强引用（即使是 weak 对象）
解决方法：1. 自己封装成 block timer
2. 使用 iOS10 后系统的 block timer
3. 使用 GCD 封装计时器
```
#### 简述一下自动释放池(@autoreleasePool)底层怎么实现？
```
说到内存管理就要说一下自动释放池了, 当你创建一个新的自动释放池时，
它会创建一个 AutoreleasePoolPage，当一个对象收到发送 autorelease 消息时，
他被添加到当前线程的处于栈顶的自动释放池中，当自动释放池被回收时，他们从栈中被删除，
并且会给池子里所有的对象都会做一次 release 操作。
简单说 AutoreleasePoolPage 是双向链表，每张链表头尾相接，有 parent、child 指针，
每创建一个池子，会在首部创建一个哨兵对象,作为标记
最外层池子的顶端会有一个 next 指针。当链表容量满了，就会在链表的顶端，并指向下一张表。

单个自动释放池的执行过程就是 objc_autoreleasePoolPush() —> [object autorelease] —> objc_autoreleasePoolPop(void *)。

objc_autoreleasePoolPush:
把当前 next 位置置为 nil，即哨兵对象,然后 next 指针指向下一个可入栈位置，
AutoreleasePool 的多层嵌套，即每次 objc_autoreleasePoolPush，实际上是不断地向栈中插入哨兵对象。
objc_autoreleasePoolPop: 根据传入的哨兵对象找到对应位置。
给上次 push 操作之后添加的对象依次发送 release 消息。 回退 next 指针到正确的位置。

经典例子: 这段代码有什么问题? 如何修改?
for (int i = 0; i < 1000000; i++) {
    NSString *str = @"Abc";
    str = [str lowercaseString];
    str = [str stringByAppendingString:@"xyz"];
    NSLog(@"%@",str);
 }
带来的问题就是在每执行一次循环，就会有一个 str 加到当前 NSRunloop 中的自动释放池中，
只有当自动释放池被 release 的时候，
自动释放池中的标示了 autorelease 的这些数据所占用的内存空间才能被释放掉。由于循环次数过大,
导致内存空间将被耗尽而没有被释放掉，所以就会出现内存忽然增高又忽然降低的情况, 
严重会导致内存溢出的现象。
解决添加一个局部的自动释放池，那么每执行一次循环就会释放一次，则不会造成内存泄露:
for (int i = 0; i < 1000000; i++) {
        @autoreleasepool {
            NSString *str = @"Abc";
            str = [str lowercaseString];
            str = [str stringByAppendingString:@"xyz"];
            NSLog(@"%@",str);
        }
}
新增的自动释放池可以减少内存用量，因为系统会在块的末尾把这些对象回收掉。
而上述这些临时对象，正在回收之列。

在 MRC 模式下，只要给对象发送 autorelease 消息，这个对象就会被添加到自动释放池。
在 ARC 模式下，一般来说，使用类方法实例化的对象才是自动释放的对象，才能被添加到自动释放池。
自动释放的对象如果没有被添加到 pool 中，就会产生内存泄露。
```
#### Dealloc 的实现机制
```
Dealloc 调用流程:
1.首先调用 _objc_rootDealloc()
2.接下来调用 rootDealloc() 
3.这时候会判断是否可以被释放，判断的依据主要有5个，判断是否有以上五种情况
 NONPointer_ISA
 weakly_reference  has_assoc
 has_cxx_dtor
 has_sidetable_rc
4-1.如果有以上五中任意一种，将会调用 object_dispose()方法，做下一步的处理。 
4-2.如果没有之前五种情况的任意一种，则可以执行释放操作，C 函数的free()。 

object_dispose() 调用流程：
1.直接调用 objc_destructInstance()
2.之后调用 C 函数的 free()

objc_destructInstance() 调用流程：
1.先判断 hasCxxDtor，如果有 C++ 的相关内容，要调用 object_cxxDestruct() ，销毁 C++ 相关的内容。
2.再判断 hasAssocitatedObjects，如果有的话，
要调用 object_remove_associations()，销毁关联对象的一系列操作。
3.然后调用 clearDeallocating()。

clearDeallocating() 调用流程：
1.先执行 sideTable_clearDellocating()。
2.再执行 weak_clear_no_lock,在这一步骤中，会将指向该对象的弱引用指针置为 nil。
3.接下来执行 table.refcnts.eraser()，从引用计数表中擦除该对象的引用计数。
4.至此为止，Dealloc 的执行流程结束。
```
#### 64bit 和 32bit 下 long 和 char 所占字节
```
char:1字节(ASCII2=256个字符)
char*(即指针变量):4个字节(32位的寻址空间是2,即32个bit，也就是4个字节。同理
64 位编译器为 8 个字节)
short int : 2 个字节 范围 -2~> 2 即 -32768~>32767
int: 4 个字节 范围 -2147483648~>2147483647
unsignedint:4个字节
long:4个字节 范围 和int一样 64位下8个字节，范围
-9223372036854775808~9223372036854775807
long long: 8 个字节 范围-9223372036854775808~9223372036854775807
unsigned long long: 8 个字节 最大值:1844674407370955161
float:4个字节
double:8个字节
```
#### retain、release 的实现机制?
```
二者的实现机制类似，概括讲就是通过第一层 hash 算法，找到 指针变量 所对应的 sideTable。
然后再通过一层 hash 算法，找到存储引用计数的 size_t，然后对其进行增减操作。
retainCount 不是固定的 1， SIZE_TABLE_RC_ONE 是一个宏定义，实际上是一个值为 4 的偏移量。
```