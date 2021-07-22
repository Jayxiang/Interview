# iOS 面试 - Runtime

- [什么是 runtime？](#什么是-runtime？)
- [一个 NSObject 对象占用多少内存空间](#一个-nsobject-对象占用多少内存空间)
- [说一下对 isa 指针的理解](#说一下对-isa-指针的理解)
- [class_rw_t 和 class_ro_t](#classrwt-和-classrot)
- [Runtime 的方法缓存?存储的形式、数据结构以及查找的过程?](#runtime-的方法缓存存储的形式、数据结构以及查找的过程)
- [什么是 method swizzling?](#什么是-method-swizzling)
- [使用 method swizzling 需要注意什么](#使用-method-swizzling-需要注意什么)
- [KVC 实现原理](#kvc-实现原理)
- [KVO 实现原理](#kvo-实现原理)
- [如何手动关闭 KVO？](#如何手动关闭-kvo？)
- [消息传递和转发机制](#####传递和转发机制)
- [类和元类](#类和元类)
- [类对象的数据结构](#类对象的数据结构)
- [为什么要设计 metaclass](#为什么要设计-metaclass)
- [分类（Category）实现原理](#分类（category）实现原理)
- [使用 runtime Associate 方法关联的对象，需要在主对象 dealloc 的时候释放么](#使用-runtime-associate-方法关联的对象，需要在主对象-dealloc-的时候释放么)
- [Category 在编译过后，是在什么时机与原有的类合并到一起的？](#category-在编译过后，是在什么时机与原有的类合并到一起的？)
- [分类（Category）可以添加 weak 属性吗](#分类（category）可以添加-weak-属性吗)
- [category 和 extension 的区别](#category-和-extension-的区别)
- [weak 原理](#weak-原理)
- [objc 中向一个 nil 对象发送消息将会发生什么](#objc-中向一个-nil-对象发送消息将会发生什么)
- [class_copyPropertyList 与 class_copyIvarList 区别](#classcopypropertylist-与-classcopyivarlist-区别)
- [class、objc_getClass、object_getclass 方法有什么区别?](#class、objcgetclass、objectgetclass-方法有什么区别)
- [[self class] 与 [super class]](#self-class-与-super-class)

#### 什么是 runtime？
```
Objective-C 是基于 C 的，它为 C 添加了面向对象的特性，
它将很多静态语言在编译和链接时期做的事放到了 runtime 运行时来处理。
Runtime（简称运行时）是一套纯 C（C 和汇编编写的）API，而 OC 就是运行时机制，
也就是在运行时的一些机制，其中最主要的就是消息机制。

应用：在程序运行中，动态创建一个类
程序运行中，动态修改一个类的属性方法
动态获取类的属性和方法
动态交换方法，俗称：Method Swizzling
动态添加属性：关联对象（objective-C Associated objects）给分类增加属性
json 转 model

使用：导入的头文件 <objc/message.h> <objc/runtime.h>
```
#### 一个 NSObject 对象占用多少内存空间
```
受限于内存分配的机制，一个 NSObject 对象都会分配 16byte 的内存空间。
但是实际上在 64 位 下，只使用了 8byte; 在 32 位下，只使用了 4byte
一个 NSObject 实例对象成员变量所占的大小，实际上是 8 字节
获取 Obj-C 指针所指向的内存的大小，实际上是 16 字节
对象在分配内存空间时，会进行内存对齐，所以在 iOS 中，分配内存空间都是 16 字节的倍数。
```
#### 说一下对 isa 指针的理解
```
isa 等价于 is kind of
实例对象 isa 指向类对象
类对象指 isa 向元类对象
元类对象的 isa 指向元类的基类
isa 有两种类型: 纯指针，指向内存地址; 
NON_POINTER_ISA，除了内存地址，还存有一些其他信息
在 Runtime 源码查看 isa_t 是共用体:
// 简化后
union isa_t {

    Class cls;
    // 相当于是unsigned long bits;
    uintptr_t bits;
    struct {
      uintptr_t nonpointer        : 1; // 0代表普通指针，1代表优化过可存储更多信息
      uintptr_t has_assoc         : 1; // 是否设置过关联对象
      uintptr_t has_cxx_dtor      : 1; // 有没有 c++ 析构函数
      uintptr_t shiftcls          : 33; /*MACH_VM_MAX_ADDRESS 0x1000000000 内存地址值*/ 
      uintptr_t magic             : 6; // 用于在调试时分辨对象是否未完成初始化
      uintptr_t weakly_referenced : 1; // 是否被弱引用指向过
      uintptr_t deallocating      : 1; // 是否正在释放
      uintptr_t has_sidetable_rc  : 1; // 引用计数器是否过大无法存储 isa 中，
      如果为1则存放在 SideTable
      uintptr_t extra_rc          : 19 // 里面存储的值引用计数减1
    };
};
```
#### class_rw_t 和 class_ro_t
```
class_rw_t：
rw 代表可读可写。
// 类的方法、属性、协议等信息都保存在class_rw_t结构体中
struct class_rw_t {
    // Be warned that Symbolication knows the layout of this structure.
    uint32_t flags;
    uint32_t version;
    // 指向只读结构体，存放类初始信息
    const class_ro_t *ro;
    
    /**
    这三个都是二维数组，可读可写，包含类的初始内容，分类内容。
    methods 存储 method_list_t -----> method_t
    还有一部分数据是 class_ro_t 合并过来的
    */
    // 方法信息
    method_array_t methods;
    // 属性信息
    property_array_t properties;
    // 协议信息
    protocol_array_t protocols;

    // ...
};
class_ro_t：
存储了当前类在编译期就已经确定的属性、方法以及遵循的协议。
因为在编译期就已经确定了，所以是ro(readonly)的，不可修改
struct class_ro_t {
    uint32_t flags;
    uint32_t instanceStart;
    uint32_t instanceSize;
#ifdef __LP64__
    uint32_t reserved;
#endif

    const uint8_t * ivarLayout;
    
    const char * name;
    // 方法列表
    method_list_t * baseMethodList;
    // 协议列表
    protocol_list_t * baseProtocols;
    // 变量列表
    const ivar_list_t * ivars;

    const uint8_t * weakIvarLayout;
    // 属性列表
    property_list_t *baseProperties;

    method_list_t *baseMethods() const {
        return baseMethodList;
    }
};
```
#### Runtime 的方法缓存?存储的形式、数据结构以及查找的过程?
```
cache_t 增量扩展的哈希表结构。哈希表内部存储的 bucket_t。
bucket_t 中存储的是 SEL 和 IMP 的键值对。
如果是有序方法列表，采用二分查找
如果是无序方法列表，直接遍历查找
查询散列表函数，其中 cache_hash(k, m)是静态内联方法，
将传入的 key 和 mask 进行&操作返回 uint32_t 索引值。
do-while 循环查找过程，当发生冲突 cache_next 方法将索引值减 1。
```
#### 什么是 method swizzling?
```
概括来说就是方法交换。
Method Swizzling 是改变一个 selector 的实际实现的技术。通过这一技术，
我们可以在运行时通过修改类的分发表中 selector 对应的函数，来修改方法的实现。
每一个类都有一个方法列表，存放着方法的名字实现的映射关系，selector 的本质就是方法名，
IMP 有点类似函数指针，指向具体的 method 实现，通过 selector 就可以找到对应的 IMP。
交换方法的几种实现方式：
（1）利用 method_exchangeImplementations 交换两个方法的实现
（2）利用 class_replaceMethod 替换方法的实现。
（3）利用 method_setImplementation 来直接设置某个方法的 IMP。
应用：替换 ViewController 生命周期方法进行无痕埋点
解决获取索引、添加、删除元素越界崩溃问题
防止按钮重复暴力点击
```
#### 使用 method swizzling 需要注意什么
```
1. 避免交换父类方法
如果当前类未实现被交换的方法而父类实现了的情况下，此时父类的实现会被交换，
若此父类的多个继承者都在交换时会导致方法被交换多次而混乱，同时当调用父类的方法时会因为找不到而发生崩溃。
2. 交换方法应在 +load 方法
方法交换应当在调用前完成交换，+load 方法发生在运行时初始化过程中类被加载的时候调用，
且每个类被加载的时候只会调用一次 load 方法，调用的顺序是父类、类、分类，
且他们之间是相互独立的，不会存在覆盖的关系，
所以放在 +load 方法中可以确保在使用时已经完成交换。
3. 交换方法应该放到 dispatch_once 中执行
在第 2 点已经写到，+load 方法在类被加载的时候调用，且只会调用一次，那为什么还需要 dispatch_once 呢？
这是为了防止手动调用 +load 方法而导致反复的被交换，因为这是存在可能的。
4. 交换的分类方法应该添加自定义前缀，避免冲突
因为分类的方法会覆盖类中同名的方法，这样会导致无法预估的后果
5. 交换的分类方法应调用原实现
很多情况我们不清楚被交换的的方法具体做了什么内部逻辑，而且很多被交换的方法都是系统封装的方法，
所以为了保证其逻辑性都应该在分类的交换方法中去调用原被交换方法，
注意：调用时方法交换已经完成，在分类方法中应该调用分类方法本身才正确。
```
#### KVC 实现原理
```
setValue 机制：
1. 程序优先调用 set<Key>: 或 _set<Key > 属性值方法，代码通过 setter 方法完成设置。
注意，这里的 <key> 是指成员变量名，首字母大小写要符合 KVC 的命名规则
2. 如果没有找到 setName：或 _setName 方法，KVC 机制
会检查 + (BOOL)accessInstanceVariablesDirectly 方法有没有返回 YES，默认该方法会返回 YES，
如果你重写了该方法让其返回 NO 的话，那么在这一步 KVC 会执行 setValue：forUndefinedKey：方法，
不过一般开发者不会这么做。
所以 KVC 机制会搜索该类里面有没有名为 < key > 的成员变量，
无论该变量是在类接口处定义，还是在类实现处定义，
也无论用了什么样的访问修饰符，只在存在以 < key > 命名的变量，KVC 都可以对该成员变量赋值。
3. 如果该类即没有 set<key>：方法，也没有_<key > 成员变量，KVC 机制会搜索_is<Key > 的成员变量
4. 和上面一样，如果该类即没有 set<Key>：方法，也没有_<key > 和_is<Key > 成员变量，
KVC 机制再会继续搜索 < key > 和 is<Key > 的成员变量。再给它们赋值。
5. 如果上面列出的方法或者成员变量都不存在，系统将会执行
该对象的 setValue：forUndefinedKey：方法，默认是抛出异常。

总之：如果没有找到 set<Key> 或 _set<Key> 方法的话，
会按照_key，_iskey，key，iskey 的顺序搜索成员并进行赋值操作。

valueForKey 机制：
1. 首先按 get<Key>,<key>,is<Key>,_<Key > 的顺序方法查找 getter 方法，找到的话会直接调用。
如果是 BOOL 或者 Int 等值类型， 会将其包装成一个 NSNumber 对象。
2. 如果上面的 getter 没有找到，KVC 则会查找 countOf<Key>,objectIn<Key>AtIndex 
或 < Key>AtIndexes 格式的方法。
如果 countOf<Key > 方法和另外两个方法中的一个被找到，那么就会返回一个
可以响应 NSArray 所有方法的代理集合(它是 NSKeyValueArray，是 NSArray 的子类)，
调用这个代理集合的方法，或者说给这个代理集合发送属于 NSArray 的方法，
就会以 countOf<Key>,objectIn<Key>AtIndex 或 < Key>AtIndexes 这几个方法组合的形式调用。
还有一个可选的 get<Key>:range: 方法。所以你想重新定义 KVC 的一些功能，
你可以添加这些方法，需要注意的是你的方法名要符合 KVC 的标准命名方法，包括方法签名。
3. 如果上面的方法没有找到，那么会同时查找 countOf<Key>，enumeratorOf<Key>,
memberOf<Key > 格式的方法。
如果这三个方法都找到，那么就返回一个可以响应 NSSet 所的方法的代理集合，
和上面一样，给这个代理集合发 NSSet 的消息，
就会以 countOf<Key>，enumeratorOf<Key>,memberOf<Key > 组合的形式调用。
4. 如果没有发现简单 getter 方法，或集合存取方法组，
以及接收类方法 accessInstanceVariablesDirectly 是返回 YES 的。
搜索一个名为_<key>、_is<Key>、<key>、is<Key > 的实例，根据他们的顺序。
如果发现对应的实例，则立刻获得实例可用的值
总结：他会按照 _<key>,_is<Key>,<key>,is<Key > 的顺序搜索成员变量名，
如果(BOOL)accessInstanceVariablesDirectly 返回 NO 的话，
那么会直接调用 valueForUndefinedKey: 方法，默认是抛出异常。
```
#### KVO 实现原理
```
KVO 的实现依赖于 Objective-C 强大的 Runtime，当观察某对象 A 时，
KVO 机制动态创建一个对象 A 当前类的子类，
并为这个新的子类重写了被观察属性 keyPath 的 setter 方法。setter 方法随后
负责通知观察对象属性的改变状况。
Apple 使用了 isa-swizzling 来实现 KVO 。当观察对象 A 时，
KVO 机制动态创建一个新的名为：NSKVONotifying_A 的新类，该类继承自对象 A 的本类，
且 KVO 为 NSKVONotifying_A 重写观察属性的 setter 方法，
setter 方法会负责在调用原 setter 方法之前和之后，通知所有观察对象属性值的更改情况。
KVO 的键值观察通知依赖于 NSObject 的两个方法: willChangeValueForKey: 和 
didChangeValueForKey: ，在存取数值的前后分别调用 2 个方法：
被观察属性发生改变之前，willChangeValueForKey: 被调用，通知系统该 keyPath 的属性值即将变更；
当改变发生后，didChangeValueForKey: 被调用，通知系统该 keyPath 的属性值已经变更；
之后， observeValueForKey:ofObject:change:context: 也会被调用。
且重写观察属性的 setter 方法这种继承方式的注入是在运行时而不是编译时实现的。
```
#### 如何手动关闭 KVO？
```
重写被观察对象的 automaticallyNotifiesObserversForKey 方法，返回 NO
重写 automaticallyNotifiesObserversOf<key>，返回 NO
注意：关闭 KVO 后，需要手动在赋值前后添加 willChangeValueForKey 和 didChangeValueForKey，
才可以收到观察通知。
```
#### 消息传递和转发机制
```
消息机制原理：对象根据方法编号 SEL 去映射表查找对应的方法实现 IMP。
一个对象的方法 [obj foo]，编译器转为消息发送 objc_msgSend(obj,foo), Runtime 时执行的流程如下：
（1）通过 obj 的 isa 指针找到它的 class
（2）先在缓存 objc_cache 中查找，再在 class 的 method list 找 foo
（3）如果 class 中没找到 foo，继续往他的 superclass rootclass 中找
（4）一旦找到 foo 这个函数，就去执行它的实现 IMP，没有找到则进入消息转发机制
消息转发：
1. 所属类的动态方法解析
首先，如果沿着继承树没有搜索到相关的方法时，则就会向接受者所属的类进行一次请求，
看是否可以动态的添加一个方法，如下：
+(BOOL)resolveInstanceMethod:(SEL)name
如果第一步不能处理，会进行到第二步。
2. 调用 forwardingTargetForSelector 把任务转发给另一个对象。
如果第二步骤也不能处理的时候，会交给第三步骤。
3. 调用 forwardInvocation 转发给其他，在调用 forwardInvocation: 
之前会调用 methodSignatureForSelector:(SEL)aSelector 方法来获取这个选择器的方法签名，
然后在 -(void)forwardInvocation:(NSInvocation *)anInvocation 方法中
你就可以通过 anInvocation 拿到相应信息做处理
4. 那么最后消息未能处理的时候，还会调用到
- (void)doesNotRecognizeSelector:(SEL)aSelector 这个方法
```
#### 类和元类
```
类对象(objc_class)
Objective-C 类是由 Class 类型来表示的，它实际上指的是一个指向 objc_class 结构体的指针。
struct objc_class 结构体定义了很多变量，通过命名不难发现：
结构体保存了指向父类的指针，类的名字，版本，实例大小，方法列表，缓存以及遵守的协议列表等。
类对象就是一个结构体，这个结构体存放的数据称为元数据。
从 isa 指针指向的结构体创建，类对象 isa 指针指向的类我们称为元类（metaclass）
SEL（objc_selector）：objc_msgSend 函数第二个参数类型为 SEL，
它是 selector 在 Objective-C 中的表示类型，
selector 是方法选择器，可以理解为区分方法的 ID，这个 ID 的数据结构为 SEL，也称为方法名称 / 编号；
IMP：就是指向最终实现程序的内存地址的指针。
总结：类对象中包含了类的实例变量，实例方法的定义，而元类对象中包括了
类的类方法 (也就是 C++ 中的静态方法) 的定义。
```
![类. png](./image/类.png)
#### 类对象的数据结构
```
类对象就是 objc_class
它的结构相对丰富一些。继承自 objc_object 结构体，所以包含 isa 指针
 isa:指向元类
 superClass: 指向父类
 Cache: 方法的缓存列表
 data: 数据。是一个被封装好的 class_rw_t 。
```
#### 为什么要设计 metaclass
```
实例对象就干存储属性值的事，类对象存储实例方法列表，元类对象存储类方法列表，
完美的符合 6 大设计原则中的单一职责，
而且忽略了对对象类型的判断和方法类型的判断可以大大的提升消息发送的效率，
并且在不同种类的方法走的都是同一套流程，在之后的维护上也大大节约了成本。
```
#### 分类（Category）实现原理
```
Category 实际上是 Category_t 的结构体，在运行时，新添加的方法，都被以倒序插入到原有方法列表的最前面，
所以不同的 Category，添加了同一个方法，执行的实际上是最后一个。
Category 在刚刚编译完的时候，和原来的类是分开的，
只有在程序运行起来后，通过 Runtime ，Category 和原来的类才会合并到一起。
```
#### 使用 runtime Associate 方法关联的对象，需要在主对象 dealloc 的时候释放么
```
无论在 MRC 下还是 ARC 下均不需要，被关联的对象在生命周期内要比对象本身释放的晚很多，
它们会在 dealloc 调用的 object_dispose()方法中释放。
```
#### Category 在编译过后，是在什么时机与原有的类合并到一起的？
```
1. 程序启动后，通过编译之后，Runtime 会进行初始化，调用 _objc_init。
2. 然后会 map_images。
3. 接下来调用 map_images_nolock。
4. 再然后就是 read_images，这个方法会读取所有的类的相关信息。
5. 最后是调用 reMethodizeClass:，这个方法是重新方法化的意思。
6. 在 reMethodizeClass: 方法内部会调用 attachCategories: ，这个方法会传入 Class 和 Category ，
会将方法列表，协议列表等与原有的类合并。最后加入到 class_rw_t 结构体中。
```
#### 分类（Category）可以添加 weak 属性吗
```
默认 runtime 只提供如下几种修饰实现：
OBJC_ASSOCIATION_ASSIGN
OBJC_ASSOCIATION_RETAIN_NONATOMIC
OBJC_ASSOCIATION_COPY_NONATOMIC
OBJC_ASSOCIATION_RETAIN
OBJC_ASSOCIATION_COPY
1. 使用 OBJC_ASSOCIATION_COPY 关联策略将 block copy 到堆上，利用 block 把持有的 weak 对象返回，
如果对象不存在了，返回的便是空值。
2. 实际上使用支持弱引用的容器如 NSHashTable、NSMapTable、NSPointerArray 都是可以实现的。
原理很简单，使用容器持有关联的对象，当该对象不存在时，
容器自身便有自动移除已销毁对象的特性，这样就实现了 weak 属性。
```
[参考：给分类添加 weak 属性的几种方法](https://blog.chenyalun.com/2019/01/20/Weak%20Associated%20Object/)
#### category 和 extension 的区别
```
extension 可以添加实例变量，而 category 是无法添加实例变量的（因为在运行期，对象的内存布局已经确定，
如果添加实例变量就会破坏类的内部布局）。
1、extension 在编译期决议，它就是类的一部分，但是 category 则完全不一样，它是在运行期决议的。
extension 在编译期和头文件里的 @interface 以及实现文件里的 @implementation 一起形成一个完整的类，
extension 伴随类的产生而产生，亦随之一起消亡
2、extension 一般用来隐藏类的私有信息，你必须有一个类的源码才能为一个类添加 extension，
所以你无法为系统的类比如 NSString 添加 extension，除非创建子类再添加 extension。
而 category 不需要有类的源码，我们可以给系统提供的类添加 category
3、extension 可以添加实例变量，而 category 不可以。
4、extension 和 category 都可以添加属性，但是 category 的属性
不能生成成员变量和 getter、setter 方法的实现。
```
#### weak 原理
```
Runtime 维护了一个 Weak 表，用于存储指向某个对象的所有 Weak 指针。
Weak 表其实是一个哈希表，Key 是所指对象的地址，Value 是 Weak 指针的
地址（这个地址的值是所指对象的地址）的数组。
一个对象 A, 里面有一个 weak 属性 B, 首先会有一个 hash 表, 键为 A 的地址, 值为一个数组, 
这个数组里包含了 B 指针的地址, 当销毁的时候回根据 B 指针的地址获取到 B 的指针, 然后置为 nil
在对象被回收的时候，经过层层调用，会最终触发下面的方法将所有 Weak 指针的值设为 nil。
简单来说，这个方法首先根据对象地址获取所有 Weak 指针地址的数组，
然后遍历这个数组把其中的数据设为 nil，最后把这个 entry 从 Weak 表中删除。
1. 初始化时：runtime 会调用 objc_initWeak 函数，初始化一个新的 weak 指针指向对象的地址。
2. 添加引用时：objc_initWeak 函数会调用 storeWeak() 函数， 
storeWeak() 的作用是更新指针指向，创建对应的弱引用表。
3. 释放时, 调用 clearDeallocating 函数。会经过两次 hash 运算，
第一次 hash(obj)得到 sideTables 中具体的 sideTable
第二次 hash(obj) 从 sideTable 中的 weak_table 获取具体的 weak_entry_t。
然后 clearDeallocating 函数根据对象地址获取所有 weak 指针地址的数组，
然后遍历这个数组把其中的数据设为 nil，最后把这个 entry 从 weak 表中删除，最后清理对象的记录。
```
#### objc 中向一个 nil 对象发送消息将会发生什么
```
如果向一个 nil 对象发送消息，首先在寻找对象的 isa 指针时就是 0 地址返回了，所以不会出现任何错误。也不会崩溃。
详解:
如果一个方法返回值是一个对象，那么发送给 nil 的消息将返回 0(nil);
如果方法返回值为指针类型，其指针大小为小于或者等于 sizeof(void*) ，float，double，long double 
或者 long long 的整型标量，发送给 nil 的消息将返回 0;
如果方法返回值为结构体,发送给 nil 的消息将返回 0。结构体中各个字段的值将都是 0; 
如果方法的返回值不是上述提到的几种情况，那么发送给 nil 的消息的返回值将是未定义的。
```
#### class_copyPropertyList 与 class_copyIvarList 区别
```
class_copyPropertyList: 只获取 @property 声明的属性，获取后不带下划线
class_copyIvarList: 不但获取了 @property 属性，获取后带下划线，
而且还获取了 @interface 大括号中声明的变量
```
#### class、objc_getClass、object_getclass 方法有什么区别?
```
[obj class]：则分两种情况：一是当 obj 为实例对象时，
[obj class]中 class 是实例方法：- (Class)class，返回的 obj 对象中的 isa 指针；
二是当 obj 为类对象（包括元类和根类以及根元类）时，调用的是类方法：+ (Class)class，返回的结果为其本身。
objc_getClass：参数是类名的字符串，返回的就是这个类的类对象；
object_getClass：参数是 id 类型，它返回的是这个 id 的 isa 指针所指向的 Class，
如果传参是 Class，则返回该 Class 的 metaClass
```
#### [self class] 与 [super class]
```
self 是类的隐藏参数，指向当前调用方法的这个类的实例;
super 本质是一个编译器标示符，和 self 是指向的同一个消息接受者。
不同点在于:super 会告诉编译器，当调用方法时，去调用父类的方法，而不是本类中的方法。
当使用 self 调用方法时，会从当前类的方法列表中开始找，如果没有，就从父类中再找;
而当使用 super 时，则从父类的方法列表中开始找。然后调用父类的这个方法。
在调用[super class]的时候，runtime 会去调用 objc_msgSendSuper 方法，而不是 objc_msgSend;
在 objc_msgSendSuper 方法中，第一个参数是一个 objc_super 的结构体，这个结构体里面有两个变量，
一个是接收消息的 receiver，一个是当前类的父类 super_class。
objc_msgSendSuper 的工作原理应该是这样的:
从 objc_super 结构体指向的 superClass 父类的方法列表开始查找 selector，
找到后以 objc->receiver 去调用父类的这个 selector。
注意，最后的调用者是 objc->receiver，而不是  super_class.
```