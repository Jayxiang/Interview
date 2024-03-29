# iOS 面试 - Swift 相关

- [Swift 和 OC 的区别](#swift-和-oc-的区别)
- [如何理解面向协议编程](#如何理解面向协议编程)
- [函数式编程的理解](#函数式编程的理解)
- [结构体和类的区别](#结构体和类的区别)
- [值类型与引用类型的区别](#值类型与引用类型的区别)
- [Swift 和 OC 如何相互调用](#swift-和-oc-如何相互调用)
- [访问控制关键字 open public internal fileprivate private 的区别](#访问控制关键字-open-public-internal-fileprivate-private-的区别)
- [关键字 Strong Weak Unowned 区别](#关键字-strong-weak-unowned-区别)
- [如何理解 copy-on-write](#如何理解-copy-on-write)
- [什么是属性观察](#什么是属性观察)
- [如何将 Swift 中的协议 protocol 中的部分方法设计为可选 optional ](#如何将-swift-中的协议-protocol-中的部分方法设计为可选-optional)
- [比较 Swift 和 OC 中的初始化方法 (init) 有什么不同](#比较-swift-和-oc-中的初始化方法-init-有什么不同)
- [比较 Swift 和 OC 中的 protocol 有什么不同](#比较-swift-和-oc-中的-protocol-有什么不同)
- [什么是函数重载 Swift 支不支持函数重载](#什么是函数重载-swift-支不支持函数重载)
- [Swift 中的枚举的关联值和原始值的区分](#swift-中的枚举的关联值和原始值的区分)
- [Swift 中的闭包结构](#swift-中的闭包结构)
- [什么是尾随闭包](#什么是尾随闭包)
- [什么是逃逸闭包](#什么是逃逸闭包)
- [什么是自动闭包](#什么是自动闭包)
- [Swift 中存储属性和计算属性的区别](#swift-中存储属性和计算属性的区别)
- [什么是延迟存储属性](#什么是延迟存储属性)
- [Swift 中什么类型属性](#swift-中什么类型属性)
- [Swift 中如何使用单例模式](#swift-中如何使用单例模式)
- [Swift 中的下标是什么](#swift-中的下标是什么)
- [简要说明 Swift 中的初始化器](#简要说明-swift-中的初始化器)
- [什么是可选链](#什么是可选链)
- [什么是运算符重载](#什么是运算符重载)
- [map、filter、reduce 用法区别](#map、filter、reduce-用法区别)
- [Any、AnyObject、AnyClass 区别](#any、anyobject、anyclass-区别)
- [.self/.type/.Self 区别](#selftypeself-区别)
- [Swift 方法派发机制](#swift-方法派发机制)

#### Swift 和 OC 的区别
```
1.OC 一个类由.h和.m两个文件组成，而 swift 只有 .swift 一个文件，所以整体的文件数量比 OC 有一定减少
2.不像 C 语言和 OC 语言一样都必须有一个主函数 main() 作为程序的入口，
swift 程序从第一句（程序中是 @UIApplicationMain）开始向下顺序执行，一直到最后。
3.Swift 容易阅读，语法和文件结构简易化；Swift 更易于维护，文件分离后结构更清晰；
Swift 更加安全，它是类型安全的语言；Swift速度更快，运算性能更高；
Swift代码更少，简洁的语法，可以省去大量冗余代码。
4.Swift 可以面向协议编程、函数式编程、面向对象编程。
```
#### 如何理解面向协议编程
```
Swift 的核心是面向协议的编程。
面向协议的编程的核心是抽象（abstraction）和简化（simplicity）。
所以 swift 的核心就是抽象和简化。更好的增加复用和减少耦合。
重要的是 swift 多了一个协议扩展的特性，但却带来了编程范式的进化。
```
#### 函数式编程的理解
```
基本理解：函数式编程是是一种编程范式，强调通过数学式的函数来计算，
函数具有永恒不可变性及表达式语法，尽可能少地使用参数和状态位的特性。
避免使用程序状态和可变对象，是降低程序复杂度的有效方式之一，而这也是函数式编程的精髓。
函数式编程强调的是执行的结果，而非执行的过程。
先构建一系列简单却有一定功能的小函数，然后再将这些函数进行组装以实现完整的逻辑和复杂的运算。
使用核心：函数在 Swift 中是一等值（first-class-values），
也就是说函数可以作为参数传递给其他函数，也可以作为其他函数的返回值。
如数组的 map，filter，reduce 都是函数式编程的体现
```
[参考文章:面向协议编程](https://onevcat.com/2016/11/pop-cocoa-1/)
#### 结构体和类的区别
```
swift 中结构体比较强大，类的大部分功能结构体基本都有。主要区别有：
1.类是引用类型，而结构体是值类型
值类型：它们的实例，以及实例中所包含的任何值类型属性，在代码中传递的时候都会被复制（值拷贝）。
引用类型：引用类型在被赋予到一个变量，常量或者被传递到一个函数时，操作的并不是其拷贝。
因此，引用的是已存在的实例本身而不是其拷贝。
2.类可以继承，而结构体不能继承
3.类有析构器（deinit），而结构体没有
4.类可以使用关键字 static class 修饰方法,但是结构体只能使用关键字 static 修饰
5.结构体默认值类型的对象方法不能修改属性值，需要在函数前面加 mutating 关键字
6.创建相同属性的结构体比类更加节省内存，速度也更快，但不适合大量的运算操作
7.类需要自己定义构造器，而结构体不需要(结构体默认生成的构造器必须包括所有成员参数，
只有当所有参数都为可选型时，可直接不用传入参数直接简单构造) 
另外：class 即使没有继承 NSObject 也会生成 ro_data_t，里面包含了 ivars 属性信息。
如果属性/方法申明为 @objc 还会生成对应的方法列表。
struct 无法代替 class 的一些场景：1.需要使用继承调用 super。2.需要使用引用类型。
3.需要使用 deinit。4.需要在运行时动态转换一个实例的类型。
不是所有 struct 都会保存在栈上，部分数据大的 struct 也会保存在堆上。
当 class 只包含静态方法/属性时，考虑使用 enum 代替 class，因为 class 会生成更多的二进制代码。
为什么用 enum 而不是 struct，因为 struct 会额外生成 init 方法。
```
#### 值类型与引用类型的区别
```
值类型（Struct Enum 等）：
在栈上创建，创建速度快
内存占用小。整体占用的内存就是内部属性内存对齐后的大小
内存回收快，用栈帧控制入栈出栈即可，没有处理堆内存的开销
不需要引用计数 (结构体中使用引用类型作为属性除外)
一般是静态派发，运行速度快，也方便编译器优化，如内联等
赋值时深拷贝。系统通过 Copy-On-Write，避免不必要的 copy，减少拷贝开销
没有隐式数据共享，具有独立性不可变性
可通过 mutating 去修改结构体中的属性。这样在保证值类型的独立性的同时，也能支持对部分属性的修改。
线程安全，一般来说没有竞态条件和死锁(要注意确定值在各个子线程中是被 copy 过的)
不支持继承，避免 OOP 子类过于耦合父类的问题。
可通过协议和泛型实现抽象。但实现协议的结构体内存大小不同，因此无法直接放入数组中，
为了存储的一致性，传参赋值时系统会引入中间层 Existential Container。
此处如果结构体属性较多会复杂一点，但苹果也会有优化(Indirect Storage With Copy-On-Write)，较少开销。
总体来说，值类型的多态是有成本的，系统会尽量优化。
开发者要考虑的是：减少动态多态把协议直接当做类来使用，需要更多考虑静态多态，多结合泛型约束来使用。

引用类型(Class Function Closure 等等):
引用类型在内存使用上没有值类型高效，在堆上创建并需要有栈指针指向该区域，增加了堆内存分配和回收的开销
赋值消耗小，一般是浅拷贝复制指针。但有引用计数成本
多个指针可指向同一内存，独立性差，容易误操作
非线程安全，要考虑原子性，多线程需要线程锁配合
需要引用计数来控制内存释放，使用不当会有野指针、内存泄漏和循环引用的风险
允许继承，但继承的 Side effect 就是子类与父类的紧耦合。
比如系统的 UIStackView 主要目的只是用来布局使用，但却不得不继承 UIView 的所有属性和方法。
```

#### Swift 和 OC 如何相互调用
```
Swift 调用 OC 代码:
需要创建一个 Target-BriBridging-Header.h 的桥文件, 在桥接文件导入需要调用的 OC 代码头文件即可
OC 调用 Swift 代码:
直接导入 Target-Swift.h 文件即可, Swift 如果需要被 OC 调用,需要使用 @objc 对方法或者属性进行修饰
默认系统会提示创建
```
#### 访问控制关键字 open, public, internal, fileprivate, private 的区别
```
Swift 中有个5个级别的访问控制权限,从高到低依次是 open, public, internal, fileprivate, private
它们遵循的基本规则: 实体不能定义在具有更低访问级别（更严格）的实体中。
一个 public 的变量，其类型的访问级别不能是 internal，fileprivate 或是 private。
因为无法保证变量的类型在使用变量的地方也具有访问权限。
函数的访问级别不能高于它的参数类型和返回类型的访问级别。
因为这样就会出现函数可以在任何地方被访问，但是它的参数类型和返回类型却不可以的情况。
open: 具备最高访问权限,其修饰的类可以和方法,可以在任意模块中被访问和重写.
public: 权限仅次于 open，和 open 唯一的区别是: 不允许其他模块进行继承、重写
internal: 默认权限, 只允许在当前的模块中访问，可以继承和重写,不允许在其他模块中访问
fileprivate: 修饰的对象只允许在当前的文件中访问;
private: 最低级别访问权限,只允许在定义的作用域内访问
```
#### 关键字 Strong Weak Unowned 区别
```
Swift 的内存管理机制同 OC 一致,都是 ARC 管理机制; 
Strong, 和 Weak 用法同 OC 一样
Unowned(无主引用), 不会产生强引用，实例销毁后仍然存储着实例的内存地址
(类似于 OC 中的 unsafe_unretained), 试图在实例销毁后访问无主引用，会产生运行时错误(野指针)
```
#### 如何理解 copy-on-write
```
值类型(比如: struct), 在复制时, 复制对象与原对象实际上在内存中指向同一个对象,
当且仅当修改复制的对象时, 才会在内存中创建一个新的对象
为了提升性能，集合类型 Array、Dictionary、Set 采取了 Copy On Write 的技术
比如仅当有“写”操作时，才会真正执行拷贝操作
对于标准库值类型的赋值操作，Swift 能确保最佳性能，所有没必要为了保证最佳性能来避免赋值
```
#### 什么是属性观察
```
属性观察是指在当前类型内对特性属性进行监测,并作出响应,
属性观察是 swift 中的特性,具有2种, willset 和 didset
var title: String {
    willSet {
        print("willSet", newValue)
    }
    didSet {
        print("didSet", oldValue, title)
    }
}
willSet 会传递新值，默认叫 newValue
didSet 会传递旧值，默认叫 oldValue
在初始化器中设置属性值不会触发 willSet 和 didSet
```
#### 如何将 Swift 中的协议 protocol 中的部分方法设计为可选 optional
```
1.在协议和方法前面添加 @objc,然后在方法前面添加 optional 关键字,该方式实际上是将协议转为了 OC 的方式
@objc protocol someProtocol {
  @objc  optional func test()
}
2.使用扩展(extension),来规定可选方法,在 swift 中,协议扩展可以定义部分方法的默认实现
protocol someProtocol {
    func test()
}

extension someProtocol{
    func test() {
        print("test")
    }
}
```
#### 比较 Swift 和 OC 中的初始化方法 (init) 有什么不同
```
swift 的初始化方法,更加严格和准确, swift 初始化方法需要保证所有的非 optional 的成员变量都完成初始化, 
同时 swfit 新增了 convenience 和 required 两个修饰初始化器的关键字
convenience:只提供一种方便的初始化器,必须通过一个指定初始化器来完成初始化
required:是强制子类重写父类中所修饰的初始化方法
```
#### 比较 Swift 和 OC 中的 protocol 有什么不同
```
相同点: 两者都可以被用作代理;
不同点: Swift 中的 protocol 还可以对接口进行抽象,可以实现面向协议,从而大大提高编程效率,
Swift 中的 protocol 可以用于结构体, 枚举;
```
#### 什么是函数重载 Swift 支不支持函数重载
```
函数重载是指: 函数名称相同,函数的参数个数不同, 或者参数类型不同,
或参数标签不同, 返回值类型与函数重载无关
swift 支持函数重载
```
#### Swift 中的枚举的关联值和原始值的区分
```
关联值--有时会将枚举的成员值跟其他类型的变量关联存储在一起，会非常有用:
// 关联值
enum Date {
    case digit(year: Int, month: Int, day: Int)
    case string(String)
}
原始值--枚举成员可以使用相同类型的默认值预先关联，这个默认值叫做:原始值
// 原始值
enum Grade: String {
  case perfect = "A"
  case great = "B"
  case good = "C"
  case bad = "D"
}
```
#### Swift 中的闭包结构
```
{
    (参数列表) -> 返回值类型 in 函数体代码
}
```
#### 什么是尾随闭包
```
将一个很长的闭包表达式作为函数的最后一个实参
使用尾随闭包可以增强函数的可读性
尾随闭包是一个被书写在函数调用括号外面(后面)的闭包表达式
// fn 就是一个尾随闭包参数
func add(v1: Int, v2: Int, fn: (Int, Int) -> Int) {
    print(fn(v1, v2))
}

// 调用
add(v1: 10, v2: 20) {
    $0 + $1
}
```
#### 什么是逃逸闭包
```
当闭包作为一个实际参数传递给一个函数或者变量的时候，我们就说这个闭包逃逸了，
可以在形式参数前写 @escaping 来明确闭包是允许逃逸的。
非逃逸闭包、逃逸闭包，一般都是当做参数传递给函数
非逃逸闭包:闭包调用发生在函数结束前，闭包调用在函数作用域内
逃逸闭包:闭包有可能在函数结束后调用，闭包调用逃离了函数的作用域，需要通过 @escaping 声明
如果不标记函数的形式参数为 @escaping ，你就会遇到编译时错误。
// 定义一个数组用于存储闭包类型
var completionHandlers: [() -> Void] = []

//  在方法中将闭包当做实际参数,存储到外部变量中
func someFunctionWithEscapingClosure(completionHandler: @escaping () -> Void) {
    completionHandlers.append(completionHandler)
}
```
#### 什么是自动闭包
```
自动闭包是一种自动创建的用来把作为实际参数传递给函数的表达式打包的闭包。
它不接受任何实际参数，并且当它被调用时，它会返回内部打包的表达式的值。
这个语法的好处在于通过写普通表达式代替显式闭包而使你省略闭包函数形式参数的括号
func getFirstPositive(_ v1: Int, _ v2: @autoclosure () -> Int) -> Int? {
    return v1 > 0 ? v1 : v2()
}
getFirstPositive(10, 20)
为了避免与期望冲突，使用了 @autoclosure 的地方最好明确注释清楚:这个值会被推迟执行
@autoclosure 会自动将 20 封装成闭包 { 20 }
@autoclosure 只支持 () -> T 格式的参数
@autoclosure 并非只支持最后1个参数
有 @autoclosure、无 @autoclosure，构成了函数重载
如果你想要自动闭包允许逃逸，就同时使用 @autoclosure 和 @escaping 标志。
```
#### Swift 中存储属性和计算属性的区别
```
存储属性(Stored Property):
类似于成员变量这个概念
存储在实例对象的内存中
结构体、类可以定义存储属性
枚举不可以定义存储属性
计算属性(Computed Property):
本质就是方法(函数)
不占用实例对象的内存
枚举、结构体、类都可以定义计算属性
struct Circle {
    // 存储属性
    var radius: Double
    // 计算属性
    var diameter: Double {
        set {
            radius = newValue / 2
        }
        get {
            return radius * 2
        }
    }
}
```
#### 什么是延迟存储属性
```
使用 lazy 可以定义一个延迟存储属性(Lazy Stored Property)，在第一次用到属性的时候
才会进行初始化(类似 OC 中的懒加载)
lazy 属性必须是 var，不能是 let:
let 必须在实例对象的初始化方法完成之前就拥有值
如果多条线程同时第一次访问 lazy 属性:
无法保证属性只被初始化 1 次
class Test {
    // 延迟存储属性
    lazy var image: Image = {
        let url = ""
        let data = Data(url: url)
        return Image(data: data)
    }()
    // 或者
    lazy var dataArr = [String]()
}
```
#### Swift 中什么类型属性
```
严格来说，属性可以分为:
实例属性(Instance Property): 只能通过实例对象去访问:
存储实例属性(Stored Instance Property):存储在实例对象的内存中，每个实例对象都有1份
计算实例属性(Computed Instance Property)

类型属性(Type Property):只能通过类型去访问:
存储类型属性(Stored Type Property):整个程序运行过程中，就只有1份内存(类似于全局变量)
计算类型属性(Computed Type Property)

可以通过 static 定义类型属性 p 如果是类，也可以用关键字 class
不同于存储实例属性，你必须给存储类型属性设定初始值:
因为类型没有像实例对象那样的 init 初始化器来初始化存储属性

存储类型属性默认就是 lazy，会在第一次使用的时候才初始化
就算被多个线程同时访问，保证只会初始化一次
存储类型属性可以是 let

枚举类型也可以定义类型属性(存储类型属性、计算类型属性)
```
#### Swift 中如何使用单例模式
```
 public class FileManager {
    public static let shared = FileManager()
    private init() { }
}
```
#### Swift 中的下标是什么
```
使用 subscript 可以给任意类型(枚举、结构体、类)增加下标功能，有些地方也翻译为:下标脚本
subscript 的语法类似于实例方法、计算属性，本质就是方法(函数)
class Point {
    var x = 0.0, y = 0.0
    subscript(index: Int) -> Double {
        set {
            if index == 0 {
                x = newValue
            } else if index == 1 {
                y = newValue }
        }
        get {
            if index == 0 {
                return x
            } else if index == 1 {
                return y
            }
            return 0
        }
    }
}

var p = Point()
// 下标赋值
p[0] = 11.1
p[1] = 22.2
// 下标访问
print(p.x) // 11.1
print(p.y) // 22.2
```
#### 简要说明 Swift 中的初始化器
```
类、结构体、枚举都可以定义初始化器
类有2种初始化器: 指定初始化器(designated initializer)、便捷初始化器(convenience initializer)
 // 指定初始化器
init(parameters) {
    statements
}
// 便捷初始化器
convenience init(parameters) {
    statements
}
规则:
每个类至少有一个指定初始化器，指定初始化器是类的主要初始化器
默认初始化器总是类的指定初始化器
类偏向于少量指定初始化器，一个类通常只有一个指定初始化器

初始化器的相互调用规则:
指定初始化器必须从它的直系父类调用指定初始化器
便捷初始化器必须从相同的类里调用另一个初始化器
便捷初始化器最终必须调用一个指定初始化器
```
#### 什么是可选链
```
可选类型其根源是一个枚举型，里面有 None 和 Some 两种类型。
其中 nil 就是 Optional.None, 非 nil 就是 Optional.Some, 然后会通过 Some(T) 包装（wrap）原始值，
这也是为什么在使用 Optional 的时候要拆包（从 enum 里取出来原始值）的原因

public enum Optional<Wrapped> : ExpressibleByNilLiteral {
    case none
    case some(Wrapped)
    /// Creates an instance that stores the given value.
    public init(_ some: Wrapped)
}

可选链是一个调用和查询可选属性、方法和下标的过程，它可能为 nil 。
如果可选项包含值，属性、方法或者下标的调用成功；如果可选项是 nil ，属性、方法或者下标的调用会返回 nil 。
多个查询可以链接在一起，如果链中任何一个节点是 nil ，那么整个链就会得体地失败。
多个?可以链接在一起
如果链中任何一个节点是 nil，那么整个链就会调用失败
```
#### 什么是运算符重载
```
类、结构体、枚举可以为现有的运算符提供自定义的实现，这个操作叫做:运算符重载(Operator Overload)
struct Point {
    var x: Int
    var y: Int

    // 重载运算符
    static func + (p1: Point, p2: Point) -> Point   {
        return Point(x: p1.x + p2.x, y: p1.y + p2.y)
    }
}

var p1 = Point(x: 10, y: 10)
var p2 = Point(x: 20, y: 20)
var p3 = p1 + p2
```
#### map、filter、reduce 用法区别
```
map: 可以对数组中的每一个元素做一次处理

let arr = [1, 2]
let arr1 = arr.map{ $0+1 }
print(arr1) //[2, 3]

flatMap: 把多维数组变成一维数组; 
把两个不同的数组合并成一个数组，这个合并的数组元素个数是前面两个数组元素个数的乘积

let array = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
  
let arr1 = array.map{ $0 }
arr1 // [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
  
let arr2 = array.flatMap{ $0 }
arr2 // [1, 2, 3, 4, 5, 6, 7, 8, 9]

compactMap: 返回后的数组中不存在 nil， 同时也会把 Optional 解包

let possibleNumbers = ["1", "2", "three", "///4///", "5"]
// [1, 2, nil, nil, 5]
let mapped: [Int?] = possibleNumbers.map { str in Int(str) }

// [1, 2, 5]
let compactMapped: [Int] = possibleNumbers.compactMap { str in Int(str) }

filer：过滤，可以对数组中的元素按照某种规则进行一次过滤

let arr = [1, 2, 3]
let arr1 = arr.filter({ $0 < 2 }) // [1]

reduce：计算，可以对数组的元素进行计算
let arr = [1, 2, 3]
let arr1 = arr.reduce(0, { $0 + $1 }) // 6

```
#### Any、AnyObject、AnyClass 区别
```
AnyObject：是一个成员为空的协议，任何对象都实现了这个协议。代表任何 class 类型的对象实例。
Any：是一个空协议集合的别名，可以代表任何类型，包括基本数据类型, enum, struct, func(方法)等等
AnyClass: 是 AnyObject.Type 的别名。
表示任意类的元类型，任意类的类型都隐式遵守这个协议，一般我们使用不多。
typealias AnyClass = AnyObject.Type
```
#### .self/.Type/.Self 区别
```
X.self: 是一个元类型的指针，是 X.Type 类型。如果用在类型后面取得类型本身，用在实例后面取得实例本身
X.Self: 表示特定类型，并且只能用在协议中或者作为某个类的方法的返回值类型
X.Type: 元类型(metaType)是指任意类型的类型，包括类类型、结构体类型、枚举类型和协议类型。

type(of:) 表达式来获取该实例动态的、在运行阶段的类型
.self 取到的是静态的元类型，声明的时候是什么类型就是什么类型。
```
#### Swift 方法派发机制
```
根据函数调用能否在编译时或运行时确定，可以将派发机制分成两种类型:
静态派发：在编译期的时候，编译器就知道要为某个方法调用某种实现, 甚至在可能的情况下，可以将某些代码转换成 inline 函数
动态派发：一定量的运行时开销为代价，提高了语言的灵活性

编译型语言有通常有三种基本的函数派发方式:
直接派发（Direct Dispatch）:
编译后就确定了方法的调用地址（也叫静态派发），汇编代码中，直接跳到方法的地址执行，生成的汇编指令最少，速度最快
例如 C 语言，C++ 默认也是直接派发
由于缺乏动态性，无法实现多态

函数表派发（Table Dispatch）:
在运行时通过一个函数表查找需要执行的方法，多一次查表的过程，速度比直接派发慢
C++ 的虚函数（Virtual Table），维护一个虚函数表，对象创建的时候会保存虚表的指针，
调用方法之前，从对象中取出虚表地址，根据编译时的方法偏移量从虚表取出方法的地址，跳到方法的地址执行

消息派发（Message Dispatch）:
Objective-C: 方法调用包装成消息，发给运行时（相当于中间人），运行时会找到类对象，类对象会保存类的数据信息，
其中就包含方法列表（类方法在元类对象存储），或通过父类查找，直到命中执行，如果没找到方法，抛出异常，
运行时提供了很多动态的方法用于改变消息派发的行为，相比函数表派发有很强的动态性，
由于运行时支持的功能很多，方法查找的过程比较长，性能比较低

函数表派发和消息派发属于动态派发
Swift 支持上面三种函数派发方式，Swift 编译器会根据不同的情况选择不同的派发方式，基于性能考虑优先选择性能高的派发方式
```

[Swift教程](https://swiftgg.gitbook.io/swift/)