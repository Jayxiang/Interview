# iOS 面试 - UIKit

- [简述 APP 生命周期](# 简述 APP 生命周期)
- [APP 启动过程及简单优化](# APP 启动过程及简单优化)
- [UIViewController 的生命周期](# UIViewController 的生命周期)
- [loadView 什么作用](# loadView 什么作用)
- [View 中的加载顺序](# View 中的加载顺序)
- [layoutSubviews 何时会被调用](# layoutSubviews 何时会被调用)
- [什么是 KVC 和 KVO？](# 什么是 KVC 和 KVO？)
- [开发中，保存数据有哪几种方式？](# 开发中，保存数据有哪几种方式？)
- [关键字 const/static/extern、UIKIT_EXTERN 区别和用法以及与宏的区别](# 关键字 const/static/extern、UIKIT_EXTERN 区别和用法以及与宏的区别)
- [关键字组合 static inline](# 关键字组合 static inline)
- [block 的实质是什么？一共有几种 block？都是什么情况下生成的？](# block 的实质是什么？一共有几种 block？都是什么情况下生成的？)
- [代理、通知、block 和单例使用场景](# 代理、通知、block 和单例使用场景)
- [frame 和 bounds 有什么不同？](# frame 和 bounds 有什么不同？)
- [UIView、CALayer 和 UIWindow 是什么关系?](# UIView、CALayer 和 UIWindow 是什么关系?)
- [isMemberOfClass 、 isKindOfClass 和 isSubclassOfClass 联系与区别](# isMemberOfClass 、 isKindOfClass 和 isSubclassOfClass 联系与区别)
- [将一个函数在主线程执行的几种方法](# 将一个函数在主线程执行的几种方法)
- [+initialize 与 +load 有什么用处，区别](# +initialize 与 +load 有什么用处，区别)
- [layoutIfNeeded 和 setNeedsLayout 的区别](# layoutIfNeeded 和 setNeedsLayout 的区别)
- [优化你是从哪几方面着手](# 优化你是从哪几方面着手)
- [常见的循环引用，及解决办法](# 常见的循环引用，及解决办法)
- [isEqual,isEqualToString 和 == 区别](# isEqual,isEqualToString 和 == 区别)
- [pod update 和 pod install 的区别](# pod update 和 pod install 的区别)
- [CoreGraphics, CoreAnimation 区别](# CoreGraphics, CoreAnimation 区别)

#### 简述 APP 生命周期

一张图即可反映 APP 生命周期:

![APP](http://upload-images.jianshu.io/upload_images/969362-b29a3c3255875ca7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#### APP 启动过程及简单优化
```
1. 解析 Info.plist
加载相关信息，如闪屏
沙箱建立，权限检查
2. Mach-O 加载
如果是胖二进制，寻找合适 CUP 类别的部分
加载所有依赖 Mach-O 文件
定位内部，外部指针引用，如字符串，函数
执行声明函数
加载扩展类
C++ 静态对象加载，调用 Objc 的 +(load)函数
3. 程序执行
main 函数
执行 UIApplicationMain 函数
UIApplicationDelegate 对象开始处理监听事件

如果 APP 启动缓慢，可以想到的因素
main() 函数内有耗时操作;
动态库加载太多;
rootViewControlle 以及 childViewController 的加载，view 和 subViews 的加载耗时;
优化:
移除不需要用到的动态库
移除不需要用到的类
合并功能类似的类和扩展（Category）
压缩图片资源
优化 applicationWillFinishLaunching
优化 rootViewController 加载
```
#### UIViewController 的生命周期
```
0.loadView: 加载视图
1.loadViewIfNeeded(iOS9 后): 重新加载视图包括 viewDidLoad
2.viewDidLoad: 视图控制器中的视图加载完成，viewController 自带的 view 加载完成
3.viewWillAppear: 视图将要出现
4.viewWillLayoutSubviews: 即将布局其 Subviews
5.viewDidLayoutSubviews: 已经布局其 Subviews
6.viewDidAppear: 视图已经出现
7.viewWillDisappear: 视图将要消失
8.viewDidDisappear: 视图已经消失
这个面试点在实际开发中还是比较重要的, 毕竟写个视图都会有自己的生命周期, 从而更好的优化视图加载.
```
一张老图:![生命周期](https://upload-images.jianshu.io/upload_images/969362-c504f39458411cbd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#### loadView 什么作用
```
loadView 在 View 为 nil 时调用，早于 ViewDidLoad，通常用于代码实现控件，
收到内存警告时会再次调用。
loadView 默认做的事情是: 如果此 Viewcontroller 存在一个对应的 nib 文件，那么就加载这个 nib。
否则，就创建一个 UIView 对象。
如果你用 Interface Builder 来创建界面，那么不应该重载这个方法。

如果你想自己创建 View 对象，那么可以重载这个方法，此时你需要自己给 View 属性赋值。
你自定义的方法不应该调用 super。如果你需要对 View 做一些其他定制操作，在 ViewDidload 中去做

根据上面说明可以知道，有两种情况：

1、如果你用了 nib 文件，重载这个方法就没有太大意义。因为 loadView 的作用就是加载 nib。
如果你重载了这个方法不调用 super，那么 nib 文件就不会被加载。
如果调用了 super，那么 view 已经加载完了，你需要做的其他事情在 viewDidLoad 里面做更合适。

2、如果你没有用 nib，这个方法默认就是创建一个空的 view 对象。如果你想自己控制 view 对象的创建，
例如创建一个特殊尺寸的 view，那么可以重载这个方法，自己创建一个 UIView 对象，然后指定 self.view = myView;
但这种情况也没有必要调用 super，因为反正你也不需要在 super 方法里面创建的 view 对象。
如果调用了 super，那么就是浪费了一些资源而已
```

#### View 中的加载顺序
```
1.initWithCoder（如果没有 storyboard 就会调用 initWithFrame）
2.awakeFromNib: 作为第一个方法的助手，方便处理一些额外的设置。
3.layoutSubviews: 一般设置子控件的 frame。
4.drawRect:UI 控件都是画上去的，在这一步就是把所有的东西画上去。
layoutSubviews 方便数据计算，drawRect 方便视图重绘。
drawRect 在以下情况下会被调用：
1、如果在 UIView 初始化时没有设置 rect 大小，将直接导致 drawRect 不被自动调用。
drawRect 掉用是在 Controller->loadView,Controller->viewDidLoad 两方法之后掉用的.
  所以不用担心在 控制器中, 这些 View 的 drawRect 就开始画了. 
  这样可以在控制器中设置一些值给 View(如果这些 View?draw 的时候需要用到某些变量 值).
2、该方法在调用 sizeToFit 后被调用，所以可以先调用 sizeToFit 计算出 size。
然后系统自动调用 drawRect: 方法。
3、通过设置 contentMode 属性值为 UIViewContentModeRedraw。
那么将在每次设置或更改 frame 的时候自动调用 drawRect:。
4、直接调用 setNeedsDisplay，或者 setNeedsDisplayInRect: 触发 drawRect:，
但是有个前提条件是 rect 不能为 0。
在实际开发中 layoutSubviews 个人用的比较多, 用来重新设置控件大小.
drawRect 一般用来做绘图个人用的不多.
```
#### layoutSubviews 何时会被调用
```
当要调整 subViews 时候，需要重写 layoutSubviews 方法。 
1: 初始化 init 方法时候不会触发。
2: 滚动 UIScrollView 时会触发
3: 旋转 UIScreen 时会触发
4: 当改变 view 的值时候会触发，前提是 frame 前后值发生了变化 
5: 当改变 UIview 的大小时候会触发
```
#### 什么是 KVC 和 KVO？
```
KVC 也就是 key-value-coding , 即键值编码，通常是用来给某一个对象的属性进行赋值.
开发中我们可以对私有属性进行赋值的, 修改一些控件的内部属性, 还可以用于字典转模型.
KVO，即 key-value-observing, 利用一个 key 来找到某个属性并监听其值得改变。
其实这也是一种典型的观察者模式。
大致用法:
1. 添加观察者
2. 在观察者中实现监听方法，observeValueForKeyPath: ofObject: change: context:
3. 移除观察者
简单实现原理:
当一个类的属性被观察的时候，系统会通过 runtime 动态的创建一个该类的派生类，
并且会在这个类中重写基类被观察的属性的 setter 方法，
而且系统将这个类的 isa 指针指向了派生类，从而实现了给监听的属性赋值时调用的是派生类的 setter 方法。
重写的 setter 方法会在调用原 setter 方法前后，通知观察对象值得改变。
KVC/KVO 实现的根本是 Objective-C 的动态性和 runtime
```
#### 开发中，保存数据有哪几种方式？
```
所谓的持久化，就是将数据保存到磁盘中，使得在应用程序重启后可以继续访问之前保存的数据.
iOS 本地数据保存有多种方式, 比如 NSUserDefaults、Plist 文件保存、
归档 (NSKeyedArchiver)、SQLite、CoreData、KeyChain(钥匙串) 等多种方式。
NSUserDefaults:
  NSUserDefaults 是一个单例对象, 在整个应用程序的生命周期中都只有一个实例。
  NSUserDefaults 保存的数据类型有：NSNumber, 基本数据类型(int，NSInter,float,double,CGFlat..),
  NSString, NSData, NSArray, NSDictionary, NSURL。
  NSUserDefaults 一般保存配置信息，比如用户名、密码、是否保存用户名和密码、是否离线下载等一些配置条件信息。
plist 文件保存:
  plist 文件是将某些特定的类，通过 XML 文件的方式保存在目录中。
  plist 主要保存的数据类型为 NSString、NSNumber、NSData、NSArray、NSDictionary。
可以看出 NSUserDefaults 和 plist 有一定局限性.
归档: 在 iOS 中是另一种形式的序列化，只要遵循了 NSCoding 协议的对象都可以通过它实现序列化。
  需要注意的: 必须遵循并实现 NSCoding 协议; 保存文件的扩展名可以任意指定; 
  继承时必须先调用父类的归档解档方法
上面的几个存储方法，都是覆盖存储。如果想要增加一条数据就必须把整个文件读出来，
然后修改数据后再把整个内容覆盖写入文件。所以它们都不适合存储大量的内容。
因此保存大量数据可以优先考虑用数据库，sql 语句对查询操作有优化作用，所以从查询速度或者插入效率都是很高的。
SQLite: 在 iOS 中要使用 SQLite3, 需要添加库文件：libsqlite3.dylib 并导入主头文件，
这是一个 C 语言的库，所以直接使用 SQLite3 还是比较麻烦的。
  不过在一般开发过程中，使用的都是第三方开源库 FMDB，封装了这些基本的 c 语言方法，
  使得我们在使用时更加容易理解，提高开发效率。
CoreData:
  CoreData 框架提供了对象 - 关系映射 (ORM) 的功能, 即能够将 OC 对象转化成数据, 
  保存在 SQLite3 数据库文件中, 也能将保存在数据库中的数据还原成 OC 对象. 在次数据操作期间, 不需要编写任何 SQL 语句.
上面几种都是保存到沙盒中.
KeyChain: 钥匙串是苹果公司 Mac OS 中的密码管理系统。一个钥匙串可以包含多种类型的数据：
密码（包括网站，FTP 服务器，SSH 帐户，网络共享，无线网络，加密磁盘镜像等），私钥，电子证书和加密笔记等。
  当应用程序被删除后，保存到 KeyChain 里面的数据不会被删除，
  所以 KeyChain 是保存到沙盒范围以外的地方。安全性也比较高.
  KeyChain 还有一个用途，就是替代 UDID。UDID 已经被废除了，所以只能用 UUID 代替，
  所以我们可以把 UUID 用 KeyChain 保存。
```
#### 关键字 const/static/extern、UIKIT_EXTERN 区别和用法以及与宏的区别
```
const:
  1.const 用来修饰右边的基本变量或指针变量
  2. 被修饰的变量只读，不能被修改
  int  const  *p   //  *p 只读 ;p 变量
  int  *const  p  // *p 变量 ; p 只读
  const  int   *const p //p 和 * p 都只读
  int  const  *const  p   //p 和 * p 都只读
  开发者经常定义只读变量:
  const CGFloat kWidth = 10.0;
static:
  1. 修饰局部变量, 保证局部变量永远只初始化一次，在程序的运行过程中永远只有一份内存，
  生命周期类似全局变量了，但是作用域不变。
  2. 修饰全局变量使全局变量的作用域仅限于当前文件内部，即当前文件内部才能访问该全局变量。
  3. 修饰函数时，被修饰的函数被称为静态函数，使得外部文件无法访问这个函数，仅本文件可以访问
   另外在开发中经常在单例中使用.
extern: 它的作用是声明外部全局变量。这里需要特别注意 extern 只能声明，不能用于实现，
而且定义和分配内存都在原来类中。
  UIKIT_EXTERN: 可以解决重复定义的问题, 可以参照苹果的做法, 比如系统预置的通知:
  UIKIT_EXTERN NSString *const   UIKeyboardWillShowNotification;
  UIKIT_EXTERN NSString *const  UIKeyboardDidShowNotification;
宏: 1. 宏在编译开始之前就会被替换，而 const 只是变量进行修饰; 
  2. 宏可以定义一些函数方法，const 不能 
  3. 宏编译时只替换不做检查不报错，也就是说有重复定义问题。而 const 会编译检查，会报错  
定义不对外公开的常量的时候，我们应该尽量先考虑使用 static 方式声名 const 来替代使用宏定义。
const 不能满足的情况再考虑使用宏定义。
例如:
  static const CGFloat kWidth = 10.0;
  可以代替 #define WIDTH 10.0
```
#### 关键字组合 static inline
```
inline 函数, 即内联函数, 他可以向编译器申请, 将使用 inline 修饰的函数内容, 内联到函数调用的位置
内联函数的作用类似于 #define, 但是他比 #define 有一些优点
相对于函数直接调用: inline 修饰的函数, 不会再调用这个函数的时候, 调用 call 方法, 
就不会将函数压栈, 产生内存消耗。
这样就减少了调用的开销, 提高效率. 所以执行速度确比一般函数的执行速度要快.
相对于宏的优点:
1. 宏需要预编译, 而内联函数是一个函数, 不许要预编译
2. 编译器调用内联函数的时候, 会检查函数的传参是否正确, 但是宏就不会提醒参数
3. 可以使用所在类的保护成员及私有成员。
但是内联函数的使用也有限制：
1. 内联函数只是我们向编译器提供的申请, 编译器不一定采取 inline 形式调用函数.
2. 内联函数只能对一些小型的函数起作用, 如果函数中消耗的内存很大, 比如 for 循环, 则内联函数就会默认失效
3. 内联函数的定义须在调用之前. 另外如果调用次数多的话，会使可执行文件变大。
使用 static 的原因：
static 只是为了表明该函数只在该文件中可见！也就是说，在同一个工程中，
就算在其他文件中也出现同名、同参数的函数也不会引起函数重复定义的错误！
另外和普通函数的区别：
1. 普通函数调用需要开辟栈帧和回收栈帧，内联函数不开辟和回收栈帧，在调用出展开代码
2. 普通函数会在编译完生成函数名对应的符号，链接的时候在符号表上可以找到，内联函数不生成符号
```

#### block 的实质是什么？一共有几种 block？都是什么情况下生成的？
```
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
#### frame 和 bounds 有什么不同？
```
frame 指的是：该 view 在父 view 坐标系统中的位置和大小。（参照点是父亲的坐标系统）
bounds 指的是：该 view 在本身坐标系统中 的位置和大小。（参照点是本身坐标系统）
理解这两个概念在实际开发中还是比较重要的.
```
#### UIView、CALayer 和 UIWindow 是什么关系?
```
UIView 是 iOS 系统中界面元素的基础, 所有的界面元素都继承自它, 
UIView 本身完全是由 CoreAnimation 来实现. 真正的绘图部分, 是由一个 CALayer 类来管理.
最大的区别是 UIView 继承自 UIResponder, 能接收并响应事件, 负责显示内容的管理, 
而 CALayer 继承自 NSObject, 不能响应事件, 负责显示内容的绘制. 
UIView 是基于 CALayer 的高层封装。而 CALayer 不支持自动布局.
另外 UIWindow 是一种特殊的 UIView, 通常在一个程序中只会有一个 UIWindow，
但可以手动创建多个 UIWindow，同时加到程序里面。
UIWindow 在程序中主要起到三个作用：
1、作为容器，包含 app 所要显示的所有视图
2、传递触摸消息到程序中 view 和其他对象
3、与 UIViewController 协同工作，方便完成设备方向旋转的支持
```
#### isMemberOfClass 、 isKindOfClass 和 isSubclassOfClass 联系与区别
```
联系：都能检测一个对象是否是某个类的成员
区别：
isKindOfClass: 确定一个对象是否是一个类的成员, 或者是派生自该类的成员.
isSubclassOfClass 和 isKindOfClass 的作用基本上是一致的，只不过一个是类方法，一个是对象方法。
isMemberOfClass: 确定一个对象是否是当前类的成员. 他的筛选条件更为苛刻，
只有当类型完全匹配的时候才会返回 YES。
```
#### 将一个函数在主线程执行的几种方法
```
//GCD 方法，通过向主线程队列发送一个 block 块，使 block 里的方法可以在主线程中执行。
dispatch_async(dispatch_get_main_queue(), ^{
    // 需要执行的方法
});
//NSOperation 方法
NSOperationQueue *mainQueue = [NSOperationQueue mainQueue];
NSBlockOperation *operation = [NSBlockOperation blockOperationWithBlock:^{
    // 需要执行的方法
}];
[mainQueue addOperation:operation];
//NSThread 方法
[self performSelector:@selector(method) onThread:[NSThread mainThread] withObject:nil waitUntilDone:YES modes:nil];
[self performSelectorOnMainThread:@selector(method) withObject:nil waitUntilDone:YES];
[[NSThread mainThread] performSelector:@selector(method) withObject:nil];
//RunLoop 方法
[[NSRunLoop mainRunLoop] performSelector:@selector(method) withObject:nil];
```
#### +initialize 与 +load 有什么用处，区别
```
通常情况下，我们在开发过程中可能不必关注这两个方法。如果有需要定制，
我们可以在自定义的 NSObject 子类中给出这两个方法的实现，
这样在类的加载和初始化过程中，自定义的方法可以得到调用。
+load: 通过函数地址直接调用，是在 runtime 加载类分类的时候调用，只会调用一次
1. 当父类和子类都实现 load 函数时, 父类的 load 方法执行顺序要优先于子类
2. 当子类未实现 load 方法时, 不会调用父类 load 方法
3. 类中的 load 方法执行顺序要优先于类别(Category)
4. 当有多个类别 (Category) 都实现了 load 方法, 这几个 load 方法都会执行, 
但执行顺序不确定(其执行顺序与类别在 Compile Sources 中出现的顺序一致)
5. 当然当有多个不同的类的时候, 每个类 load  执行顺序与其在 Compile Sources 出现的顺序一致

+ initialize：会在第一次接收到消息时调用，会通过 objc_msgSend 调用，
每个类只会 initialize 一次（父类可能多次，但不代表初始化多次）。
1. 父类的 initialize 方法会比子类先执行
2. 当子类未实现 initialize 方法时, 会调用父类 initialize 方法( 可能会多次调用), 
子类实现 initialize 方法时, 会覆盖父类 initialize 方法.
3. 当有多个 Category 都实现了 initialize 方法, 会覆盖类中的方法, 
只执行一个(会执行 Compile Sources 列表中最后一个 Category 的 initialize 方法)

```
|   | +load | + initialize |
| --- | --- | --- |
| 调用时机 | 被添加到 runtime 时（main 前） | 到第一条消息前，可能永远不调用 |
| 调用顺序 | 父类 ->子类 ->分类 | 父类 ->本类(如果有分类，则调用分类) |
| 若自身未实现，是否沿用父类的方法 | 否 | 是 |
| 类别中的定义 | 全都执行，但后于本类的方法（顺序与 Compile Sources 出现的顺序一致） | 覆盖本类的方法，只执行一个（执行 Compile Sources 列表中最后一个 Category 的 initialize 方法） |
[更多请看：iOS 类方法 load 和 initialize 详解](https://www.jianshu.com/p/c52d0b6ee5e9)

#### layoutIfNeeded 和 setNeedsLayout 的区别
```
首先我们布局总会重新触发 layoutSubviews 方法
setNeedsLayout：
标记为需要重新布局，异步调用 layoutIfNeeded 刷新布局，不立即刷新，在下一轮 runloop 结束前刷新，
对于这一轮 runloop 之内的所有布局和 UI 上的更新只会刷新一次，layoutSubviews 一定会被调用。

layoutIfNeeded：
如果有需要刷新的标记，立即调用 layoutSubviews 进行布局（如果没有标记，不会调用 layoutSubviews）。

如果需要立即刷新则调用
[self setNeedsLayout];
[self layoutIfNeeded];

是开发中如果需要在 Masonry 添加约束后如果想获取 frame 可以直接调用
layoutIfNeeded 即可立即获取 frame
```
#### 优化你是从哪几方面着手
```
1. 不要阻塞主线程
2. 尽量 view 设置为完全不透明，减少 GPU 渲染的消耗
3. 懒加载和延迟加载。
4. 处理内存警告
5. 重用开销大的对象，例如 NSDateFormatter，NSNumberFormatter 等等，写成单例。
6.App 启动时间优化

UITableView 优化：
1. 正确使用 reuseIdentifier 来重用 cell
2. 尽量 view 设置为完全不透明
3. 缓存行高
4. 如果 cell 内现实的内容来自 web，使用异步加载，缓存请求结果
5. 异步绘制，遇到复杂界面，遇到性能瓶颈时，可能就是突破口；
6. 尽量不要动态的 add 或者 remove 子控件。最好在初始化时就添加完，然后通过 hidden 来控制是否显示。
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
#### isEqual,isEqualToString 和 == 区别
```
isEqual：默认情况下是比较两个对象的内存地址；isEqual：就是提供了一个可以自定义相等标准的方法。
系统自带的类 (比如 Foundation 中 的 NSString, NSArray 等) 重写了这个方法，改变了这个方法的判断规则,
一般改为比较两个对象的内容，不是内存地址.

isEqualToString: 字符串比较，只比较字符串本身的内容是否一致，不比较内存地址.

==：如果两个对象的内存地址是一样，返回 true，如果内存地址不一样，返回 false.
```
#### pod update 和 pod install 的区别
```
pod install 用于工程第一次安装 pod 库和修改 podfile(添加，更新，移除 pod 库)的时候。
pod update 用于更新特定的 pod(或所有的 pod)版本时。
```
#### CoreGraphics, CoreAnimation 区别
```
CoreGraphics(核心图形): 它是 iOS 的核心图形库，包含 Quartz2D 绘图 API 接口, 
常用的是 point，size，rect 等这些图形，都定义在这个框架中，
类名以 CG 开头的都属于 CoreGraphics 框架，它提供的都是 C 语言函数接口
CoreAnimation(核心动画)：
1. CoreAnimation 是跨平台的，既可以支持 IOS，也支持 MAC OS。
2. CoreAnimation 执行动画是在后台，不会阻塞主线程。
3. CoreAnimation 作用在 CALayer，不是 UIView。
4. CoreGraphics 和 CoreAnimation 的关系：它们都是跨 iOS 和 Mac OS 使用的，
这点区别于 UIKit，并且 CoreAnimation 中大量使用到 CoreGraphics 中的类，因为实现动画要用到图形库中的东西。
5. CoreGraphics 是底层绘制框架，我们实际会用到的也就是 CG 开头的一些底层绘制函数和变量，这是一个纯 C 语言框架。
```