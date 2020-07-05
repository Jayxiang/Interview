# iOS 面试 - 基本概念

- [简要叙述 OC 语言的特点](# 简要叙述 OC 语言的特点)
- [类别的作用？继承、类别和扩展在实现中有何区别](# 类别的作用？继承、类别和扩展在实现中有何区别)
- [在 OC 中类变量的 @protected,@private,@public,@package 区别](# 在 OC 中类变量的 @protected,@private,@public,@package 区别)
- [\#import、#include、@class、#import<>和 #import""的区别](#\#import、#include、@class、#import<>和 #import"" 的区别)
- [@property 的本质是什么？ivar、getter、setter 是如何生成并添加到这个类中的](#@property 的本质是什么？ivar、getter、setter 是如何生成并添加到这个类中的)
- [@property 常用属性及如何使用](#@property 常用属性及如何使用)
- [用 @property 声明的 NSString / NSArray / NSDictionary 经常使用 copy 关键字，为什么？如果改用 strong 关键字，可能造成什么问题？](# 用 @property 声明的 NSString / NSArray / NSDictionary 经常使用 copy 关键字，为什么？如果改用 strong 关键字，可能造成什么问题？)
- [为什么 IBOutlet 属性是 weak 的？](# 为什么 IBOutlet 属性是 weak 的？)
- [\__weak，\__block 的区别](#__weak，__block 的区别)
- [id 声明的对象有什么特性？](#id 声明的对象有什么特性？)
- [id 和 nil 代表什么（nil 和 NULL 的区别）](#id 和 nil 代表什么（nil 和 NULL 的区别）)
- [BOOL/bool/Boolean 的区别](#BOOL/bool/Boolean 的区别)
- [OC 的反射机制](#OC 的反射机制)
- [简述一下自动释放池底层怎么实现？](# 简述一下自动释放池底层怎么实现？)
- [堆和栈的区别？和队列](# 堆和栈的区别？和队列)
- [沙盒机制的理解和使用](# 沙盒机制的理解和使用)
- [事件响应者链的概念](# 事件响应者链的概念)
- [@synthesize 和 @dynamic 分别有什么作用？](#@synthesize 和 @dynamic 分别有什么作用？)
- [类方法和实例方法有什么区别？](# 类方法和实例方法有什么区别？)
- [浅拷贝和深拷贝的区别？](# 浅拷贝和深拷贝的区别？)
- [NSCache NSDictionary 区别](#NSCache NSDictionary 区别)

#### 简要叙述 OC 语言的特点

是根据 C 语言所衍生出来的语言，继承了 C 语言的特性，是扩充 C 的面向对象编程语言,
所以具有面向对象的语言特性，如：封装、多态、继承。
- 封装：是对象和类概念的主要特性。它是隐藏内部实现，提供外部接口。
- 继承：它可以使用现有类的所有功能，并在无需重新编写原来的类的情况下对这些功能进行扩展。
- 多态：不同对象以自己的方式响应相同的消息的能力叫做多态。
总的来说面向对象是一种思想, 目的可以让代码重用, 接口重用, 避免重复代码, 逻辑更加清晰,
更容易维护, 提高编程效率.

另外 OC 具有动态特性: 之所以叫做动态，是因为必须到运行时（runtime）才会做一些事情。
（动态特性的三个方面：动态类型、动态绑定、动态加载）  
（1）动态类型:
    动态类型，（id 类型）在编译器编译的时候不能被识别出，在运行时（runtime），程序运行的时候才会根据语境来识别。  
    静态类型，与动态类型相对。在编译的时候就能识别出来，明确的基本类型都属于静态类型。
    （int、NSString 等）  
（2）动态绑定：
    （关键词 @selector）跳过编译，在运行时动态添加函数调用，
      运行时才决定调用什么方法，传递什么参数。  
（3）动态加载：
    根据需求动态地加载资源。

#### 类别的作用？继承、类别和扩展在实现中有何区别
```
类别的作用:(1)将类的实现分散到多个不同文件或多个不同框架中。
(2)创建对私有方法的前向引用。
(3)向对象添加非正式协议。
扩展:
只有头文件没有实现文件。只能扩展方法，不能添加成员变量。扩展的方法只能在原类中实现
类扩展只能针对自定义的类，不能给系统类增加类扩展；
由于局限性比较多, 个人在开发中没有用过.
继承主要作用：
1. 重写父类方法
2. 在父类基础上增加属性，方法，协议
category 可以在不获悉，不改变原来代码的情况下往里面添加新的方法，只能添加，不能删除修改。
并且如果类别和原来类中的方法产生名称冲突，则类别将覆盖原来的方法，因为类别具有更高的优先级。
继承可以增加，修改方法，并且可以增加属性。
```
#### 在 OC 中类变量的 @protected,@private,@public,@package 区别
```
@protected (默认)该类和所有子类中的方法可以直接访问这样的变量。
@private 该类中的方法可以访问，子类不可以访问。
@public   可以被所有的类访问
@package 本包内使用，跨包不可以
实际开发中基本都是默认, 没有使用过其他的
```
#### \#import、#include、@class、#import<>和 #import"" 的区别
```
#include 与 #import 都是导入头文件的关键字, 完整地包含某个文件的内容,
后者会自动导入一次，不会重复导入, 不会引发交叉编译.
@class 仅仅是声明一个类名，并不会包含类的完整声明, 编译效率高.
可避免循环依赖, 且使用后带来的编译错误.
#import<>: 用于对系统自带的头文件的引用，编译器会在系统文件目录下查找该文件。
#import"": 用户自定义的文件用双引号引用，引用时编译器首先会在用户目录下查找，
然后去安装目录中查找，最后在系统文件目录中查找。
另外：iOS7 之后的新特性，可以使用 @import 关键词来代理 #import 引入系统类库。
     使用 @import 引入系统类库，不需要到 build phases 中先添加添加系统库到项目中。
```

#### @property 的本质是什么？ivar、getter、setter 是如何生成并添加到这个类中的
```
1.@property 的本质 = ivar (实例变量) + getter (取方法) + setter （存方法）
“属性”（property）有两大概念：实例变量（ivar）、存取方法(getter + setter)
2.ivar、 getter 、setter 是如何生成并添加到这个类中的
这是编译器自动合成的，通过 @synthesize 关键字指定，若不指定，
默认为 @synthesize  propertyName = _propertyName
若手动实现了 getter/setter 方法，则不会自动合成。
现在编译器已经默认为我们添加了 @synthesize  propertyName = _propertyName;
因此不再手动添加了，除非你真的要改变成员变量名字。
生成 getter 方法时，会判断当前属性名是否有“_”, 比如声明属性为 @property（nonatomic,copy）NSString *_name;
那么所生成的成员变量名就会变成 “_name”, 如果我们要手动生成 getter 方法，就要判断是否以“_” 开头了。
```
#### @property 常用属性及如何使用
```
读写属性: (readwrite/readonly/setter = /getter = )
引用计数:(assign/retain/copy/strong)
原⼦性: (atomic/nonatomic)
ARC 其中属性默认是：readwrite，strong, atomic
atomic 是默认的属性, 表示对象的操作属于原子操作
(原子性指事务的一个完整操作。操作成功则提交，失败则回滚),
主要是在多线程的环境下, 提供多线程访问的安全。
  我们知道在多线程的下对对象的访问都需要先上锁访问后再解锁, 保证不会同时有⼏个操作针对同⼀个对象。
  如果编程中不涉及到多线程, 不建议使用, 因为使用 atomic 比 nonatomic 更耗费系统资源。
  注意: atomic 的作用只是给 getter 和 setter 加了个锁，
  atomic 只能保证代码进入 getter 或者 setter 函数内部时是安全的，并不能保证整个对象是线程安全的。
  个人在实际开发中还真的没用过。
nonatomic 表示访问器的访问不是原⼦操作, 不支持多线程访问安全, 但 是访问性能⾼。

readwrite（默认）：readwrite 是默认值，表示该属性同时拥有 setter 和 getter。
readonly： readonly 表示只有 getter 没有 setter。

retain 表⽰对 NSObject 和及其⼦子类对象 release 旧值, 再 retain 新值, 使对象的应⽤计数增加 1。
  该属性只能使⽤用于 obejective-c 类型对象, 不能用于 Core Foundation 对象。
assign 是基本数据类型的默认属性, setter 方法将传入的参数赋值给实例变量,
可以对基本数据类型 (如 CGFloat, NSInteger,Bool,int, 代理, id 对象) 等使⽤。可以用非 OC 对象
  该方式会对象直接赋值而不会进行 retain 操作。
weak 是修饰的对象在释放之后，指针地址会被置为 nil。
  所以现在一般弱引用就是用 weak。常用于 delegate,weak 必须用于 OC 对象
strong 是在 iOS 引入 ARC 的时候引入的关键字，也是普通 OC 对象的默认属性，是 retain 的一个可选的替代。
  表示实例变量对传入的对象要有所有权关系，即强引用。
  strong 跟 retain 的意思相同并产生相同的代码，但是语意上更好更能体现对象的关系。
copy 表示重新建立一个新的计数为 1 的对象, 然后释放掉旧的值。常用于 NSString,block

另外还有一些如 Xcode 6.3 新出的关键字 nonnull,nullable, 主要是为了与 swift 混编更加方便, 
注意只能修饰对象, 不能修饰基本数据类型
nullable 表示对象可以是 NULL 或 nil
nonnull  表示对象不应该为空
null_resettable: get: 不能返回空, set 可以为空
（注意：如果使用 null_resettable, 必须 重写 get 方法或者 set 方法, 处理传递的值为空的情况
null_unspecified: 不确定是否为空

class：类属性，类使用类方法。
```
#### 用 @property 声明的 NSString / NSArray / NSDictionary 经常使用 copy 关键字，为什么？如果改用 strong 关键字，可能造成什么问题？
```
用 @property 声明 NSString、NSArray、NSDictionary 经常使用 copy 关键字，
是因为他们有对应的可变类型：NSMutableString、NSMutableArray、NSMutableDictionary，
他们之间可能进行赋值操作（就是把可变的赋值给不可变的），为确保对象中的字符串值不会无意间变动，
应该在设置新属性值时拷贝一份。
1. 因为父类指针可以指向子类对象, 使用 copy 的目的是为了让本对象的属性不受外界影响, 
使用 copy 无论给我传入是一个可变对象还是不可对象, 我本身持有的就是一个不可变的副本。
2. 如果我们使用是 strong , 那么这个属性就有可能指向一个可变对象, 如果这个可变对象在外部被修改了, 
那么会影响该属性。
3. 使用 copy 的目的是，防止把可变类型的对象赋值给不可变类型的对象时，
可变类型对象的值发送变化会无意间篡改不可变类型对象原来的值。
```
#### 为什么 IBOutlet 属性是 weak 的？
```
因为既然有外链那么视图在 xib 或者 storyboard 中肯定存在，视图已经对它有一个强引用了。 
IBoutlet 连线到控制器中作为视图的属性时用 weak 修饰就可以了, (用 strong 修饰也可以但是没有必要)
```
#### __weak，__block 的区别
```
__weak 与 weak 基本相同。前者用于修饰变量（variable），后者用于修饰属性（property）。
__weak 主要用于防止 block 中的循环引用。
__block 也用于修饰变量。它是引用修饰，所以其修饰的值是动态变化的，即可以被重新赋值的。
__block 用于修饰某些 block 内部将要修改的外部变量。
```
#### id 声明的对象有什么特性？
```
 id 声明的对象具有运行时的特性，在程序运行时才确定对象的类型。
可以指向任意类型的 OC 的对象，与 C 中的 void * 万能指针相似。
运行效率低，不可以使用点语法。
```
#### id 和 nil 代表什么（nil 和 NULL 的区别）
```
id 类型：是一个独特的数据类型，可以转换为任何数据类型，id 类型的变量可以存放任何数据类型的对象，
  在内部处理上，这种类型被定义为指向对象的指针，实际上是一个指向这种对象的实例变量的指针
NULL 是宏，是对于 C 语言指针而使用的，表示空指针
nil 是宏，是对于 Objective-C 中的对象而使用的，表示对象为空. 
当向 nil 发送消息时，不会有异常，程序将继续执行下去；
Nil 是宏，是对于 Objective-C 中的类而使用的，表示类指向空
NSNull 是类类型，是用于表示空的占位对象，与 JS 或者服务端的 null 类似的含意
```
#### BOOL/bool/Boolean 的区别
```
BOOL:
typedef signed char BOOL;
#define YES (BOOL)1
#define NO  (BOOL)0
bool:
C99 标准定义了一个新的关键字_Bool，提供了布尔类型
#define bool _Bool
#define true 1  
#define false 0
Boolean:
typedef unsigned char Boolean;
enum DYLD_BOOL { FALSE, TRUE };
```
如表所示:

| Name         |    Typedef    |      Header      |   True Value   |     False Value |
|--------------|:-------------:|:----------------:|:--------------:|----------------:|
| BOOL         |  signed char  |      objc.h      |      YES       |              NO |
| bool         |  _Bool (int)  |    stdbool.h     |      true      |           false |
| Boolean      | unsigned char |    MacTypes.h    |      TRUE      |           FALSE |
| NSNumber     | __NSCFBoolean |   Foundation.h   |     @(YES)     |           @(NO) |
| CFBooleanRef |    struct     | CoreFoundation.h | kCFBooleanTrue | kCFBooleanFalse |
#### OC 的反射机制

> Objective-C 语言中的 OC 对象，都继承自 NSObject 类。这个类为我们提供了一些基础的方法和协议，我们可以直接调用从这个类继承过来方法。大部分的动态反射支持来自 NSObject 类。NSObject 是所有类（除了一些很少见的例外）的根类。所以基本常用到的类应该都可以支持反射。

```
//class 反射: 通过类名的字符串形式实例化对象
Class class = NSClassFromString(@"user"); 
User *user = [[class alloc] init];
// 将类名变为字符串
Class class =[User class];
NSString *className = NSStringFromClass(class);

//SEL 反射: 通过方法的字符串形式实例化方法
SEL selector = NSSelectorFromString(@"setName");  
[stu performSelector:selector withObject:@"Song"];
// 将方法变成字符串
NSStringFromSelector(@selector*(setName:));
```
#### 堆和栈的区别？和队列
```
堆栈是两种数据结构, 堆，先进先出;  栈，先进后出
按管理和内存方式来看
  对于栈来讲，是由系统编译器自动管理，不需要程序员手动管理
  对于堆来讲，释放工作由程序员手动管理，不及时回收容易产生内存泄露
按分配方式分
  堆是动态分配和回收内存的，没有静态分配的堆
  栈有两种分配方式：静态分配和动态分配
  静态分配是系统编译器完成的，比如局部变量的分配
  动态分配是有 alloc 函数进行分配的，但是栈的动态分配和堆是不同  的，
  它的动态分配也由系统编译器进行释放，不需要程序员手动管理。

队列是只允许在一端进行插入操作、而在另一端进行删除操作的线性表。
允许插入的一端称为队尾，允许删除的一端称为队头。
它是一种特殊的线性表，特殊之处在于它只允许在表的前端进行删除操作，而在表的后端进行插入操作，
和栈一样，队列是一种操作受限制的线性表。
队列是一种先进先出的数据结构，又称为先进先出的线性表，简称 FIFO（First In First Out）结构。
```
#### 沙盒机制的理解和使用
```
处于安全考虑，iOS 系统的沙盒机制规定每个应用都只能访问当前沙盒目录下面的文件
（也有例外，比如在用户授权情况下访问通讯录，相册等）
特点:1. 每个应用程序的活动范围都限定在自己的沙盒里
    2. 不能随意跨越自己的沙盒去访问别的应用程序沙盒中的内容（iOS8 已经部分开放访问 extension）
    3. 在访问别人沙盒内的数据时需要访问权限。
在开发中常常需要数据存储的功能，比如存取文件，归档解档等。
沙盒根目录结构：Documents、Library、tmp 以及新加入的 SystemData。
AppName.app 目录：该目录 iOS8 之前是和 Documents 那几个在同一目录下, 
但 iOS8 之后移动到 Container—>bundle—>Application 中, 
它包含了应用程序本身的数据，包括资源文件和可执行文件等。
  程序启动以后，会根据需要从该目录中动态加载代码或资源到内存，这里用到了 lazy loading 的思想。
  由于应用程序必须经过签名，所以您在运行时不能对这个目录中的内容进行修改，否则可能会使应用程序无法启动。
Documents：这个目录存放用户数据。存放用户可以管理的文件；iTunes 备份和恢复的时候会包括此目录。
  APP 的数据库表; 必要的一些图标本地缓存; 重要的 plist 文件, 如当前登录人的信息等可存放到此目录下.
Library 目录：这个目录下有两个子目录：
  Preferences 目录：NSUserDefaults 的数据存放于此目录下。
  Caches 目录：用于存放应用程序专用的支持文件，保存应用程序再次启动过程中需要的信息。
  比如网络请求的数据。但是它缓存数据在设备低存储空间时可能会被删除,
Library 可创建子文件夹。可以用来放置您希望被备份但不希望被用户看到的数据。该路径下的文件夹，
除 Caches 以外，都会被 iTunes 备份。
tmp 目录：这个目录用于存放临时文件，保存应用程序再次启动过程中不需要的信息。
该路径下的文件不会被 iTunes 备份。该目录下的东西随时有可能被系统清理掉
SystemData 目录: 最近查看目录才发现的，新加入的一个文件夹, 存放系统的一些东西. 具体没太研究.
```
#### 事件响应者链的概念
```
响应者链表示一系列的响应者对象. 事件被交给由第一响应者对象处理, 如果第一响应者不处理, 
事件被沿着响应者链向上传递, 交给下一响应者(next responder).
一般来说, 第一响应者是视图对象或者其子类对象, 当其被触摸后事件被交由它处理,
如果它不处理, 事件就会被传递给它的视图控制器对象 (如果存在), 
然后就是它的父视图(superView) 对象(如果存在), 以此类推, 直到顶层视图.
接下来会沿着顶层视图 (top view) 到窗口 (UIWindow 对象) 再到程序(UIApplication 对象). 
如果整个过程都没有响应这个事件, 该事件就会被丢弃.
一般情况下, 在响应者中只要由对象处理事件, 事件就停止传递. 但有时候可以在视图响应方法中
根据一些条件判断来决定是否需要继续传递事件.
对于事件响应链在实际开发偶尔会用到, 当遇到某个视图不能响应时首先就要考虑到响应链的传递问题, 
然后根据代码调整.
比较重要的函数: hitTest:withEvent: 方法和 pointInside 方法
简单来说就是: 事件的传递是从上到下（父控件到子控件），
事件的响应是从下到上（顺着响应者链条向上传递：子控件到父控件。
```
[可参考文章](https://www.jianshu.com/p/2e074db792ba)

#### @synthesize 和 @dynamic 分别有什么作用？
```
1.@property 有两个对应的词，一个是 @synthesize，一个是 @dynamic。
  如果 @synthesize 和 @dynamic 都没写，那么默认的就是 @synthesize var = _var;
2.@synthesize 的语义是如果你没有手动实现 setter 方法和 getter 方法，
  那么编译器会自动为你加上这两个方法。
3.@dynamic 告诉编译器：属性的 setter 与 getter 方法由用户自己实现，不自动生成。
（当然对于 readonly 的属性只需提供 getter 即可）。假如一个属性被声明为 @dynamic var，
  然后你没有提供 setter 方法和 getter 方法，编译的时候没问题，
  但是当程序运行到 instance.var = someVar，由于缺 setter 方法会导致程序崩溃；
  或者当运行到 someVar = var 时，由于缺 getter 方法同样会导致崩溃。
  编译时没问题，运行时才执行相应的方法，这就是所谓的动态绑定。
实际开发中我们常会用到重写 getter 方法来做懒加载, 或者用 setter 方法来完成调用. 很少会将两个同时重写.
```
#### 类方法和实例方法有什么区别？
```
类方法: 在 OC 类定义方法时以 + 开头的方法，又称为静态方法。
  它不用实例就可以直接调用的方法，一般是有返回值的，返回对应的实例（数组、字符串等），
  还有可能就是本身类的实例对象。
实例方法: 在 OC 定义中以 - 开头的方法, 原理是向某个对象发送一条消息, 
如果对象中有相应的消息就会做出回应, 
OC 用的就是这种消息模式.
类方法:
类方法属于类对象
类方法只能通过类对象调用
类方法中的 self 是类对象
类方法可以调用其他类方法
类方法中不能访问成员变量
类方法不能直接调用对象方法
实例方法:
实例方法是属于实例对象的
实例方法只能呢通过实例对象调用
实例方法中的 self 是实例对象
实例方法中可以访问成员变量
实例方法中直接调用实例方法
实例方法中也可以调用类方法（通过类名）
```
#### 浅拷贝和深拷贝的区别？
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
```
#### NSCache NSDictionary 区别
```
1.NSCache 是线程安全的，NSMutableDictionary 线程不安全
2. 当内存不足时 NSCache 会自动释放内存(所以从缓存中取数据的时候总要判断是否为空)
3.NSCache 可以指定缓存的限额，当缓存超出限额自动释放内存
4.NSCache 并不会 “拷贝” 键，而是会 “保留” 它。
```
