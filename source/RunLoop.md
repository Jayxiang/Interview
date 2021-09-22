# iOS 面试 - RunLoop

- [什么是 RunLoop](#什么是-runloop)
- [RunLoop 的数据结构](#runloop-的数据结构)
- [RunLoop 的 Mode](#runloop-的-mode)
- [RunLoop 的实现机制和内部逻辑](#runloop-的实现机制和内部逻辑)
- [RunLoop 和线程](#runloop-和线程)
- [PerformSelector 的实现原理？](#performselector-的实现原理？)
- [autoreleasePool 在何时被释放？](#autoreleasepool-在何时被释放？)
- [子线程里面，需要加 autoreleasepool 吗](#子线程里面，需要加-autoreleasepool-吗)
- [事件响应的过程？](#事件响应的过程？)
- [手势识别的过程？](#手势识别的过程？)
- [解释一下 NSTimer](#解释一下-nstimer)
- [利用 runloop 解释一下页面的渲染的过程](#利用-runloop-解释一下页面的渲染的过程)
- [什么是异步绘制](#什么是异步绘制)

#### 什么是 RunLoop
```
RunLoop 是通过内部维护的事件循环 (Event Loop) 来对事件 / 消息进行管理的一个对象。
1、没有消息处理时，休眠已避免资源占用，由用户态切换到内核态(CPU - 内核态和用户态)
2、有消息需要处理时，立刻被唤醒，由内核态切换到用户态
主要作用：1. 保证程序不退出。2. 监听事件(如触摸事件、时钟事件、网络事件)。
还可以：
1. 监听主线程 Runloop 的状态，在主线程空闲的时候或者合适的时候去执行一些任务。
比如，把一个大任务拆分成一个个小任务，当主线程处于休眠状态时 (kCFRunLoopBeforeWaiting) 
依次提交到 Runloop 里执行，合理利用 CPU，提高 CPU 使用效率。
2. 根据主线程 Runloop 的状态(kCFRunLoopBeforeSources 和 kCFRunLoopAfterWaiting，
即正在处理任务的状态)，做实时监控主线程卡顿。
3. 可以自己创建一个后台线程，并且开启 Runloop 来等待任务或者用来专门处理某类任务。
比如，AFNetworking 中创建一个后台线程来专门接收 Delegate 回调。
4. 根据 Runloop 的 Mode 来提交不同的任务，然后根据状态在 Mode 之间切换。
比如 TableView 中将加载图片的任务放到 NSDefaultRunLoopMode 模式下去处理，
保证 UITrackingRunLoopMode 模式下界面滑动的流畅性。
```
#### RunLoop 的数据结构
```
NSRunLoop(Foundation)是 CFRunLoop(CoreFoundation)的封装，提供了面向对象的 API
RunLoop 相关的主要涉及五个类：
CFRunLoopRef：RunLoop 对象
CFRunLoopModeRef：运行模式
CFRunLoopSourceRef：输入源 / 事件源
CFRunLoopTimerRef：定时源
CFRunLoopObserverRef：观察者
一个 RunLoop 包含若干个 Mode，每个 Mode 又包含若干个 Source/Timer/Observer。
每次调用 RunLoop 的主函数时，只能指定其中一个 Mode，这个 Mode 被称作 CurrentMode。
如果需要切换 Mode，只能退出 Loop，再重新指定一个 Mode 进入。
这样做主要是为了分隔开不同组的 Source/Timer/Observer，让其互不影响。

CFRunLoopSource 分为 source0 和 source1 两种
source0: 即非基于 port 的，也就是用户触发的事件。需要手动唤醒线程，将当前线程从内核态切换到用户态
source1: 基于 port 的，包含一个 mach_port 和一个回调，可监听系统端口和通过内核和其他线程发送的消息，
能主动唤醒 RunLoop，接收分发系统事件。具备唤醒线程的能力。

CFRunLoopTimer: 基于时间的触发器，它和 NSTimer 是 toll-free bridged 的，可以混用。
其包含一个时间长度和一个回调（函数指针）。当其加入到 RunLoop 时，RunLoop 会注册对应的时间点，
当时间点到时，RunLoop 会被唤醒以执行那个回调。

CFRunLoopObserver: 观察者，每个 Observer 都包含了一个回调（函数指针），
当 RunLoop 的状态发生变化时，观察者就能通过回调接受到这个变化。
typedef CF_OPTIONS(CFOptionFlags, CFRunLoopActivity) {
    kCFRunLoopEntry         = (1UL << 0), // 即将进入 Loop
    kCFRunLoopBeforeTimers  = (1UL << 1), // 即将处理 Timer
    kCFRunLoopBeforeSources = (1UL << 2), // 即将处理 Source
    kCFRunLoopBeforeWaiting = (1UL << 5), // 即将进入休眠
    kCFRunLoopAfterWaiting  = (1UL << 6), // 刚从休眠中唤醒
    kCFRunLoopExit          = (1UL << 7), // 即将退出 Loop
    kCFRunLoopAllActivities = 0x0FFFFFFFU // 所有状态
};
```
#### RunLoop 的 Mode
```
一个 RunLoop 对象中可能包含多个 Mode，且每次调用 RunLoop 的主函数时，
只能指定其中一个 Mode(CurrentMode)。
切换 Mode，需要重新指定一个 Mode 。
主要是为了分隔开不同的 Source、Timer、Observer，让它们之间互不影响。
总共是有五种 CFRunLoopMode:
kCFRunLoopDefaultMode：默认模式，主线程是在这个运行模式下运行
UITrackingRunLoopMode：跟踪用户交互事件（用于 ScrollView 追踪触摸滑动，
保证界面滑动时不受其他 Mode 影响）
UIInitializationRunLoopMode：在刚启动 App 时第进入的第一个 Mode，启动完成后就不再使用
GSEventReceiveRunLoopMode：接受系统内部事件，通常用不到
kCFRunLoopCommonModes：伪模式，不是一种真正的运行模式，
是同步 Source/Timer/Observer 到多个 Mode 中的一种解决方案
```
#### RunLoop 的实现机制和内部逻辑
```
对于 RunLoop 而言最核心的事情就是保证线程在没有消息的时候休眠，在有消息时唤醒，以提高程序性能。
这个机制是依靠系统内核来完成的(苹果操作系统核心组件 Darwin 中的 Mach)。
RunLoop 通过 mach_msg()函数接收、发送消息。
它的本质是调用函数 mach_msg_trap()，相当于是一个系统调用，会触发内核状态切换。
在用户态调用时会切换到内核态; 内核态中内核实现的 mach_msg()函数会完成实际的工作。
即基于 port 的 source1，监听端口，端口有消息就会触发回调; 
而 source0，要手动标记为待处理和手动唤 醒 RunLoop
大致逻辑如下：
1. 通知 Observer 已经进入了 RunLoop
2. 通知 Observer 即将处理 Timer
3. 通知 Observer 即将处理非基于端口的输入源（即将处理 Source0）
4. 处理那些准备好的非基于端口的输入源（处理 Source0）
5. 如果基于端口的输入源准备就绪并等待处理，请立刻处理该事件。转到第 9 步（处理 Source1）
6. 通知 Observer 线程即将休眠
7. 将线程置于休眠状态，直到发生以下事件之一:
(1)事件到达基于端口的输入源（port-based input sources）(也就是 Source0)
(2)Timer 到时间执行
(3)外部手动唤醒
(4)为 RunLoop 设定的时间超时
8. 通知 Observer 线程刚被唤醒（还没处理事件）
9. 处理待处理事件：
(1)如果是 Timer 事件，处理 Timer 并重新启动循环，跳到第 2 步
(2)如果输入源被触发，处理该事件（文档上是 deliver the event）
(3)如果 RunLoop 被手动唤醒但尚未超时，重新启动循环，跳到第 2 步
10. 通知观察者 RunLoop 结束。
```
![runloop.png](./image/runloop.png)

#### RunLoop 和线程
```
线程和 RunLoop 是一一对应的, 其映射关系是保存在一个全局的 Dictionary 里，
线程是 key ，Runloop 是 value。
自己创建的线程默认是没有开启 RunLoop 的
创建一个常住线程：
1、为当前线程开启一个 RunLoop（第一次调用 [NSRunLoop currentRunLoop]
方法时实际是会先去创建一个 RunLoop）
2、向当前 RunLoop 中添加一个 Port/Source 等维持 RunLoop 的事件循环
（如果 RunLoop 的 mode 中一个 item 都没有，RunLoop 会退出）
3、启动该 RunLoop
```
#### PerformSelector 的实现原理？
```
当调用 NSObject 的 performSelecter:afterDelay: 后，
实际上其内部会创建一个 Timer 并添加到当前线程的 RunLoop 中。
所以如果当前线程没有 RunLoop，则这个方法会失效。

当调用 performSelector:onThread: 时，实际上其会创建一个 Timer 加到对应的线程去，
同样的，如果对应线程没有 RunLoop 该方法也会失效。
```
#### autoreleasePool 在何时被释放？
```
1.App 启动后，苹果在主线程 RunLoop 里注册了两个 Observer，
其回调都是 _wrapRunLoopWithAutoreleasePoolHandler()。
2. 第一个 Observer 监视的事件是 Entry(即将进入 Loop)，
其回调内会调用 _objc_autoreleasePoolPush() 创建自动释放池。
其 order 是 -2147483647，优先级最高，保证创建释放池发生在其他所有回调之前。
3. 第二个 Observer 监视了两个事件： BeforeWaiting(准备进入休眠) 
时调用_objc_autoreleasePoolPop() 和 _objc_autoreleasePoolPush() 释放旧的池并创建新池；
Exit(即将退出 Loop) 时调用 _objc_autoreleasePoolPop() 来释放自动释放池。
这个 Observer 的 order 是 2147483647，优先级最低，保证其释放池子发生在其他所有回调之后。
4. 在主线程执行的代码，通常是写在诸如事件回调、Timer 回调内的。
这些回调会被 RunLoop 创建好的 AutoreleasePool 环绕着，
所以不会出现内存泄漏，开发者也不必显示创建 Pool 了
```
#### 子线程里面，需要加 autoreleasepool 吗
```
子线程默认不开启 runloop, 当产生 autorelease 对象时候，如果发现没有autoreleasepool，会自动创建一个,
一个被自动创建出来的 autoreleasepool 会绑定到当前 pthread，
使用这个线程的本地存储来保存 pool，并在 thread 退出/销毁时释放 pool。

自定义的 NSOperation 和 NSThread 需要手动创建自动释放池。
比如： 自定义的 NSOperation 类中的 main 方法里就必须添加自动释放池。
否则出了作用域后，自动释放对象会因为没有自动释放池去处理它，而造成内存泄露。
但对于 blockOperation 和 invocationOperation 这种默认的Operation，
系统已经帮我们封装好了，不需要手动创建自动释放池。

GCD 开辟子线程不需要手动创建 autoreleasepool，因为 GCD 的每个队列都会自行创建 autoreleasepool。
```
#### 事件响应的过程？
```
1. 苹果注册了一个 Source1 (基于 mach port 的) 用来接收系统事件，
其回调函数为 __IOHIDEventSystemClientQueueCallback()。
2. 当一个硬件事件 (触摸 / 锁屏 / 摇晃等) 发生后，
首先由 IOKit.framework 生成一个 IOHIDEvent 事件并由 SpringBoard 接收。
SpringBoard 只接收按键(锁屏 / 静音等)，触摸，加速，接近传感器等几种 Event，
随后用 mach port 转发给需要的 App 进程。
随后苹果注册的那个 Source1 就会触发回调，并调用 _UIApplicationHandleEventQueue() 进行应用内部的分发。
3._UIApplicationHandleEventQueue() 会把 IOHIDEvent 处理并包装成 UIEvent 进行处理或分发，
其中包括识别 UIGesture / 处理屏幕旋转 / 发送给 UIWindow 等。
通常事件比如 UIButton 点击、touchesBegin/Move/End/Cancel 事件都是在这个回调中完成的。
4.UIWindow 对 UIEvent 中的每个 UITouch 实例调用 hitTest 方法寻找其 hitTestView，
并在 hitTest 递归调用过程中，记录最终的 hitTestView 及其各父视图上挂载的 gestureRecognizer（记录为 gestureRecognizers 属性）
同级的多个 view 之间 hitTest 调用顺序为逆序（后 addSubView 的先调用 hitTest）
默认根据 pointInside 方法的返回值来决定是否递归查找其子 view
5.将每个 UITouch 对象发给其对应的 gestureRecognizers 对象以及 hitTestView（即调用它们的 touchesBegin 方法）
识别成功的 gestureRecognizer 将独占相关的 touch，所有其他 gestureRecognizer 和 hitTestView 都将收到 touchsCancelled 回调，并且以后不会再收到此 touch 的事件

一个特例是：系统默认的 UIControl（UISlider, UISwitch 等）的控件的父 view 上的 gestureRecognizer 优先级低于 UIControl 本身

也需要配合相关 gestureRecognizer 冲突解决相关方法（如 canBePreventedByGestureRecognizer:、canPreventGestureRecognizer:等）的具体实现使用

如果 hitTestView 实例不响应 touchesBegin 等方法，则顺着 responder chain 继续找 nextResponder 来调用

若实现了 touchesBegin 等方法，则在其中调用 [super touchesBegin] 方法会将 touch 事件沿着 responder chain 向上传递
```
#### 手势识别的过程？
```
1. 当 _UIApplicationHandleEventQueue() 识别了一个手势时，
其首先会调用 Cancel 将当前的 touchesBegin/Move/End 系列回调打断。
随后系统将对应的 UIGestureRecognizer 标记为待处理。
2. 苹果注册了一个 Observer 监测 BeforeWaiting (Loop 即将进入休眠) 事件，
这个 Observer 的回调函数是 _UIGestureRecognizerUpdateObserver()，
其内部会获取所有刚被标记为待处理的 GestureRecognizer，并执行 GestureRecognizer 的回调。
3. 当有 UIGestureRecognizer 的变化 (创建 / 销毁 / 状态改变) 时，这个回调都会进行相应处理。
```
#### 解释一下 NSTimer
```
NSTimer 其实就是 CFRunLoopTimerRef，他们之间是 toll-free bridged 的。
一个 NSTimer 注册到 RunLoop 后，RunLoop 会为其重复的时间点注册好事件。
例如 10:00, 10:10, 10:20 这几个时间点。RunLoop 为了节省资源，
并不会在非常准确的时间点回调这个 Timer。Timer 有个属性叫做 
Tolerance(宽容度)，标示了当时间点到后，容许有多少最大误差。
如果某个时间点被错过了，例如执行了一个很长的任务，则那个时间点的回调也会跳过去，不会延后执行。
就比如等公交，如果 10:10 时我忙着玩手机错过了那个点的公交，那我只能等 10:20 这一趟了。

CADisplayLink 是一个和屏幕刷新率一致的定时器(但实际实现原理更复杂，和 NSTimer 并不一样，
其内部实际是操作了一个 Source)。如果在两次屏幕刷新之间执行了一个长任务，
那其中就会有一帧被跳过去(和 NSTimer 相似)，造成界面卡顿的感觉。
在快速滑动 TableView 时，即使一帧的卡顿也会让用户有所察觉。 
开源的 AsyncDisplayLink 就是为了解决界面卡顿的问题，其内部也用到了 RunLoop
```
#### 利用 runloop 解释一下页面的渲染的过程
```
当我们调用 [UIView setNeedsDisplay] 时，这时会调用当前 View.layer 的 
[view.layer setNeedsDisplay]方法。
这等于给当前的 layer 打上了一个脏标记，而此时并没有直接进行绘制工作。
而是会到当前的 Runloop 即将休眠，也就是 beforeWaiting 时才会进行绘制工作。
紧接着会调用 [CALayer display]，进入到真正绘制的工作。
CALayer 层会判断自己的 delegate 有没有实现异步绘制的代理方法 displayer:，
这个代理方法是异步绘制的入口，如果没有实现这个方法，那么会继续进行系统绘制的流程，然后绘制结束。
CALayer 内部会创建一个 Backing Store，用来获取图形上下文。接下来会判断这个 layer 是否有 delegate。
如果有的话，会调用 [layer.delegate drawLayer:inContext:]，
并且会返回给我们 [UIView DrawRect:] 的回调，让我们在系统绘制的基础之上再做一些事情。
如果没有 delegate，那么会调用 [CALayer drawInContext:]。 
以上两个分支，最终 CALayer 都会将位图提交到 Backing Store，最后提交给 GPU。 
至此绘制的过程结束
```
#### 什么是异步绘制
```
异步绘制，就是可以在子线程把需要绘制的图形，提前在子线程处理好。
将准备好的图像数据直接返给主线程使用，这样可以降低主线程的压力。
异步绘制的过程:
要通过系统的 [view.delegate displayLayer:] 这个入口来实现异步绘制。
 -代理负责生成对应的 Bitmap
 -设置该 Bitmap 为 layer.contents 属性的值。
```

经典文章：[深入理解 RunLoop](https://blog.ibireme.com/2015/05/18/runloop/)
