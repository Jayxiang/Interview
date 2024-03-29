# iOS 面试-Block

- [block 的实质是什么？一共有几种 block？都是什么情况下生成的？](#block-的实质是什么？一共有几种-block？都是什么情况下生成的？)
- [block 变量截获](#block-变量截获)
- [__block 修饰符作用](#__block-修饰符作用)
- [block 和函数指针的理解](#block-和函数指针的理解)
- [代理、通知、block 和单例使用场景](#代理、通知、block-和单例使用场景)
- [block 循环引用](#block-循环引用)

#### block 的实质是什么？一共有几种 block？都是什么情况下生成的？
```
block 是将函数及其执行上下文封装起来的对象。
block 对象是一个 c 语言级别的语法和运行机制。它与标准 c 函数类似，不同之处在于，
它除了有可执行的代码之外，还包含了与堆、栈内存绑定的变量。
作为一个回调，block 特别的有用，因为 block 既包含了回调期间的代码，又包含了执行期间需要的数据。
iOS 中有三种:
NSStackBlock    存储于栈区
NSGlobalBlock   存储于程序数据区
NSMallocBlock   存储于堆区
NSGlobalBlock:
  block 内部没有引用外部变量的 Block 类型都是 NSGlobalBlock 类型，存储于全局数据区，
  由系统管理其内存，retain、copy、release 操作都无效。
NSStackBlock:
  block 内部引用外部变量，retain、release 操作无效，存储于栈区，变量作用域结束时，
  其被系统自动释放销毁. 但在 ARC 下 block 变量在赋值的时候系统自动将其拷贝到堆区了
NSMallocBlock:
  [block retain]，[blockA copy]操作后变量类型变为 NSMallocBlock，支持 retain、release.
即如果对栈 Block 进行 copy，将会 copy 到堆区，对堆 Block 进行 copy，
将会增加引用计数，对全局 Block 进行 copy，因为是已经初始化的，所以什么也不做。
另外，__block 变量在 copy 时，由于__forwarding 的存在，
栈上的指针会指向堆上的__forwarding 变量，而堆上的__forwarding 指针指向其自身，
所以，如果对 __block 的修改，实际上是在修改堆上的__block 变量。 
即__forwarding 指针存在的意义就是，无论在任何内存位置，都可以顺利地访问同一个__block 变量。 
```
```
// 大致结构
struct __block_impl {
  void *isa; // 指向所属类的指针
  int Flags; // 标志变量，在实现block的内部操作时会用到
  int Reserved; // 保留变量
  void *FuncPtr; // block执行时调用的函数指针
};
```
#### block 变量截获
```
1、局部变量截获是值截获。
2、局部静态变量截获是指针截获。
3、全局变量，静态全局变量截获: 不截获, 直接访问/修改。
所以一般情况下，如果我们要对 block 截获的局部变量进行赋值操作需添加__block 修饰符，
而对全局变量，静态变量是不需要添加__block 修饰符的。 
另外，block 里访问 self 或成员变量都会去截获 self。
因为__block 修饰的变量也是以指针形式截获的，并且生成了一个新的结构体对象:
```
#### __block 修饰符作用
```
__block 可以用于解决 block 内部无法修改 auto 变量值的问题
__block 不能修饰全局变量、静态变量（static）
编译器会将 __block 变量包装成一个对象
__block 修改变量：age->__forwarding->age
__Block_byref_age_0 结构体内部地址和外部变量 age 是同一地址

如：__block int age = 10
编译器会将 __block 变量包装成一个对象
struct __Block_byref_age_0 {
 void *__isa;
 __Block_byref_age_0 *__forwarding;// age 的地址
 int __flags;
 int __size;
 int age;// age 的值
};
```
#### block 和函数指针的理解
```
相同点：
1、函数指针和 Block 都可以实现回调的操作，声明上也很相似，实现上都可以看成是一个代码片段。
2、函数指针类型和 Block 类型都可以作为变量和函数参数的类型。（typedef 定义别名之后，这个别名就是一个类型）
不同点：
1、函数指针只能指向预先定义好的函数代码块（可以是其他文件里面定义，通过函数参数动态传入的），
函数地址是在编译链接时就已经确定好的。
2、Block 本质是 Objective-C 对象，是 NSObject 的子类，可以接收消息。
3、函数里面只能访问全局变量，而 Block 代码块不光能访问全局变量，
还拥有当前栈内存和堆内存变量的可读性（当然通过__block 访问指示符修饰的
局部变量还可以在 block 代码块里面进行修改）。
4、从内存的角度看，函数指针只不过是指向代码区的一段可执行代码，
而 block 实际上是程序运行过程中在栈内存动态创建的对象，
可以向其发送 copy 消息将 block 对象拷贝到堆内存，以延长其生命周期。
```
#### 代理、通知、block 和单例使用场景
```
代理: 这是很常用的方式，特点是一对一的形式，而且逻辑结构非常清晰。
需要定义协议方法并且实现协议方法，会使代码结构变复杂.
通知: 通知中心实际上是在程序内部提供了消息广播的一种机制。
通知中心是基于观察者模式的，它允许注册、删除观察者。它是多对多关系.
block: 是一段特殊的代码块。使用起来有点像函数. 特点也是一对一的, 代码结构更加紧凑，不需要额外定义方法.
这几个在开发中经常用于数据传递, 如果只是单个参数往往使用 block, 
如果参数内容较多则使用代理传值. 而如果是跨界面传值往往使用通知传值.
当然传值的方式还有单例, 数据存储来做传值.
```
#### block 循环引用
```
由于 block 捕获的修饰的变量会去持有变量，那么如果用 __block 修饰 self，
且 self 持有 block，并且 block 内部使用到 __block 修饰的 self 时，就会造成多循环引用，
即 self 持有 block，block 持有__block 变量，而 __block 变量持有 self，造成内存泄漏。

如果要解决这种循环引用，可以主动断开__block 变量对 self 的持有，
即在 block 内部使用完 self 后，将其置为 nil，但这种方式有个问题，
如果 block 一直不被调用，那么循环引用将一直存在。 所以，我们最好还是用__weak 来修饰 self
```