# Swift编码规范

本文参考自：https://github.com/Ramshandilya/The-Swift-Style-Guide
 
## 命名
起名字的时候需要让名字简单而且有意义，并且使用驼峰式命名规则，同时要扪心自问：这个名字能不能充分的解释它的行为。对于其他开发者来说，有意义的名字是很重要的。
* **尽量** 使用自身就很有描述性的英文名字
* **必须** 使用大驼峰命名法来定义类、结构体、别名、枚举，协议
* **必须** 使用小驼峰命名法来定义变量、函数、方法
* 除非这种缩写被广泛使用，否则应该尽量 **避免** 缩写。例如 (fileURL)
* **禁止** 创建相似的名称，特别是不要创建仅以大小写区分的名字
* **禁止** 使用下划线开头的定义（这可能导致和原生库的写法冲突）
* **禁止** 创建简单字母的名字（例如：i, j, k），除非是在循环或者数组的下标
* **尽可能** 不要创建以（`Manager`、`Helper`或者`Utility`）结尾的类，因为这些名字的意义非常通用并且容易被误解，并且我们倾向于在这种名字的类里面放很多代码，当然这不是绝对的，如果能很好的组织，也还是可以使用

#### 函数和参数
对于所有函数和`init`方法，最好对所有的参数都打上标签(label)，除非当前上下文已经特别清楚了。如果外面的参数（第一个参数）加了标签之后能让方法调用可读性更高，也打上标签。
```swift
func dateFromString(dateString: String) -> NSDate
func convertPointAt(column column: Int, row: Int) -> CGPoint
func timedAction(afterDelay delay: NSTimeInterval, perform action: SKAction) -> SKAction!

// would be called like this:
dateFromString("2014-03-14")
convertPointAt(column: 42, row: 13)
timedAction(afterDelay: 1.0, perform: someOtherAction)
```

#### 类前缀
Swift使用Module当作命名空间，不需要类前缀

## 代码组织
代码文件应该类似于一下的组织结构
```swift
import Foundation

class NewsMaster {

    //MARK: Public properties
    let features: [Feature]
    private(set) var version: Float
    
    //MARK: Private properties
    private let bugs: [Bug]
    
    //MARK: Init
    init(features: [Feature], version: Float)
    
    //MARK: Public methods
    func validFeatures() -> [Feature]
    
    //MARK: Private methods
    private func fixAllBugs()
}

//MARK: SomeProtocol methods
extension NewsMaster: Testable {

	var testVersion: Float {
		return version
	}
	
    func testFeature(feature: Feature) -> Bool {
        return isValid
    }
}
```

## 注释
一般来说，代码应该尽可能的自己能说明自己，但是有一些行为他们不能够通过静态的代码来表示出来。在这种情况下，应该提示一些注释去解释_为什么_这些代码会写成这样。**修改代码的时候一定要同步修改注释，删除代码也是一样的**

## 可变-不可变
使用`let`关键字定义常亮，使用`var`关键字定义变量，如果变量的值没有发生变化，尽量使用`let`来进行描述
* Xcode7之后编译器会尽量的提示你去使用let
* 可以把所有的定义都申明成`let`， 除非编译不通过的时候再改成`var`
* `let`可以编译器针对代码做一些优化

## 类型
优先使用Swift的原生类型。Swift的所有类型都可以被扩展，有时候与其去重新搞一个类型，倒不如扩展现有的类型。

记住：Objective-C类型和原生的Swift类型不能够自动桥接，如下代码所示：`NSString`不会隐式的转换成`String`

```swift
func lowercase(string: String) -> String

let string: NSString = /* ... */

lowercase(string) // compile error
lowercase(string as String) // no error
```

如果编译阶段能够推测出来的类型，就不要在申明的时候去重复说明

**Preferred:**
```swift
let name = "NewsMaster"
let features = [.News, .Favorite, .Account]
let developers = [.News: "Suen", .Account: "Suen"]
```
**Not preferred :**
```swift
let name: String = "NewsMaster"
let features: [Feature] = [.News, .Favorite, .Account]
let developers: [Feature: String] = [.News: "Suen", .Account: "Suen"]
```

同时，冒号应该靠近类型的定义（左侧不留空格）

**Preferred :**
```swift
class EventTaskQueue: TaskQueue
let events: [NSTimeInterval: Event]
```

**Not preferred :**
```swift
class EventTaskQueue: TaskQueue
let events : [NSTimeInterval : Event]
```

## 可选
如果变量或者函数的返回值可以接受空值`nil`, 则使用`?`把变量申明成可选值。

#### 避免使用强制拆包
如果你定义了一个类型为`User?`的变量`friend`, 请不要使用强制拆包(`friend!`)去获取`friend`的值，强制拆包很容易就导致运行时崩溃。

代替方案， 使用 *Optional Binding* 来安全的拆包:
```swift
if let friend = friend {
    // Use unwrapped `friend` value in here
} else {
    // If appropriate, handle the case where the optional is nil
}
```
如果同时拆多个包时，请不要嵌套`if-let`语句，这样会导致代码嵌套太深(`Pyramid of doom`)。Swift可以在一个`if-let`中同时对多个可选的变量进行拆包。
```swift
let name: String?
let age: Int?

if let name = name, age = age where age >= 13 {
    /* ... */
}
```
或者可以使用 `guard` 语句
```swift
guard let modelURL = NSBundle.mainBundle().URLForResource(modelName, withExtension:"momd") else {
    fatalError("Error loading model from bundle")
}
        
guard let managedObjectModel = NSManagedObjectModel(contentsOfURL: modelURL) else {
    fatalError("Error initializing mom from: \(modelURL)")
}
        
let coordinator = NSPersistentStoreCoordinator(managedObjectModel: managedObjectModel)
```
如下类似的情况，你可以使用Swift的`Optional Chaining`:
```swift
// 如果`friend`有值，则直接获取`friend`的name，否则则直接返回nil，后面的被短路
if let name = friend?.name {
    // print(name)
}
```
#### 避免隐式的强制拆包
尽可能的使用 `let friend: User?` 来代替 `let friend: User!`.
仅仅针对一些在使用之前肯定会被创建的实例变量，允许使用隐式的强制拆包。
比如以下这种情况，你可以使用!
``` swift
class User {

	private(set) let uid: String!
	private(set) let name: String!

	init?(dictionary: [String: String])	{
		guard let 
			uid = dictionary["uid"] as? String,
			name = dictionary["name"] as? String else {
			return nil
		}
		self.uid = uid
		self.name = name
	}
}

```

## 闭包

如果函数/方法的最后一个参数是闭包，请使用尾随闭包(Trailing closure)， 如果闭包是这个函数/方法的唯一参数, 则可以省略括号。没有使用的闭包参数应该替换成`_`(如果没有参数用到的话，则使用一个`_`把所有参数都省略), 闭包中参数的类型不需要申明，让编译器去推测

```swift
func executeRequest(request: Request, completion: (Value?, Error?) -> Void)

executeRequest(someRequest) { (result, _) in /* ... */ }
```

如果闭包的结构比较清晰的话，使用隐式`return`

```swift
let numbers = [1, 2, 3, 4, 5]
let even = filter(numbers) { $0 % 2 == 0 }
```

## 类和结构体
记住：structs是值类型。结构体主要针对那些没有用来标识id的东西。一个数组包含[a, b, c] 和另一个数组也包含 [a, b, c] 是一样的,并且这两个数组完全是可互换的。你使用第一个数组，还是使用第二个数组根本不影响，因为他们根本就是一个东西，这也是为什么arrays是结构体的原因。

类是引用类型，类主要针对有identify的或者有特定的生命周期的对象。你应该把Person定义成一个类，两个person对象就是两个不同的人，因为两就算个人有相同的名字，相同的生日，这不代表他们就是同一个人。但是Persion的生日可以用结构体来表示，因为你定义的1991年3月4号和我定义的1991年3月4号肯定是同一天。

> _值类型_ 用于呈现APP内的 **数据**
> _引用类型_ 用于体现APP内的 **行为**

说明一下，继承绝逼不是一个很好的使用类的理由，因为多态可以通过协议来实现，方法的重用可以通过组合来实现。

For example, this class hierarchy:
```swift
class Vehicle {
    let name: String
    let numberOfWheels: Int
    
    init(numberOfWheels: Int) {
        self.numberOfWheels = numberOfWheels
    }

    func description() -> String {
        return "The vehicle is a \(name) which runs on \(numberOfWheels) wheels."
    }
}

class Bicycle: Vehicle {
    init() {
        super.init(numberOfWheels: 2)
    }
}

class Car: Vehicle {
    init() {
        super.init(numberOfWheels: 4)
    }
}
```
could be refactored into these definitions:
```swift
protocol Vehicle {
    var numberOfWheels: Int { get }
    var name: String { get} 
    func description() -> String
}

struct Bicycle: Vehicle {
    let name = "Bicycle"
    let numberOfWheels = 2
}
struct Car: Vehicle {
    let name = "Car"
    let numberOfWheels = 4
}

extension Vehicle {
    func description() -> String {
        return "The vehicle is a \(name) which runs on \(numberOfWheels) wheels."
    }
}
```
#### self 的使用
Swift不要求在访问属性和方法的时候加`self`, 为了代码的简洁，仅当以下情况的时候使用`self`:
* 方法中为了区分参数和当前类属性名称的时候
* 闭包中访问`self`的属性和方法（编译器要求）
* 访问父类/父类的Category继承下来的属性

以下情况均不使用`self`：
* 访问当前类的属性
* 访问任何方法

```swift
class User {
	private let uid: String
	private let name: String

    init(uid: String, name: String) {
        self.uid = uid
	    self.name = name

        let closure = {
		    print(self.uid)
        }
    }
}
```
#### 协议实现
当一个类要实现某个协议的时候，最好为当前类添加一个分开的扩展来实现这个协议，这样就把和这个协议相关的方法聚合在了一起，同时也可以简化给类实现协议的过程。

同时，不要忘了添加一个`// MARK: `标记来很好的组织代码

**Preferred :**
```swift
class MyViewcontroller: UIViewController {
  // class stuff here
}

// MARK: - UITableViewDataSource
extension MyViewcontroller: UITableViewDataSource {
  // table view data source methods
}

// MARK: - UIScrollViewDelegate
extension MyViewcontroller: UIScrollViewDelegate {
  // scroll view delegate methods
}
```
**Not Preferred :**
```swift
class MyViewcontroller: UIViewController, UITableViewDataSource, UIScrollViewDelegate {
  // all methods
}
```

#### 结构体的初始化
使用Swift原生提供的构造器而不是OC遗留的构造器

**Preferred:**
```swift
let bounds = CGRect(x: 40, y: 20, width: 120, height: 80)
let centerPoint = CGPoint(x: 96, y: 42)
```
**Not Preferred:**
```swift
let bounds = CGRectMake(40, 20, 120, 80)
let centerPoint = CGPointMake(96, 42)
```
优先使用struct作用域内的常量`CGRect.infinite`, `CGRect.null`, 代替 `CGRectInfinite`, `CGRectNull`, etc. For existing variables, you can use the shorter `.zero`.

## 单例
把 `init` 方法标记为 `private`, 这样可以阻止其他人使用()去创建其他的实例
```swift
final class Tracker {
    static let sharedInstance = Tracker()
    private init() {} 
}
```
同时, 也不需要使用  `dispatch_once` 来确保线程了， 摘自 [苹果的博客](https://developer.apple.com/swift/blog/?id=7) -
>The lazy initializer for a global variable (also for static members of structs and enums) is run the first time that global is accessed, and is launched as `dispatch_once` to make sure that the initialization is atomic. This enables a cool way to use `dispatch_once` in your code: just declare a global variable with an initializer and mark it private.

另外： 不滥用原则

## 分号
Swift不要求每条执行语句后面都加分号，只有在很多语句放在同一行的时候才要求。
但是请不要把多条执行语句放在一行，除了使用`for-conditional-increment`语句的时候。然而：尽量使用`for-in`来代替`for-conditional-increment`。

## 其他
* 所有的方法/函数应该只有一个唯一的出口。除了你在使用`guard`语句
* 不要直接使用数字值或者字符串值（比如Magic numbers）;
* 申明变量的时候尽量让变量的所处的作用于最小，并且在申明的同时进行初始化
* 请不要提交永远不会执行的代码，直接删除之。适用于：
    * 函数/方法永远不会被调用
    * 被注释掉的代码
* 请不要提交任何没有提供实际功能的代码。适用于：
    * Xcode 自动生成的代码（什么都没做，除了调用super）
    
##Credits
iOS官方的编码规范 [苹果官方编码规范](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/CodingGuidelines/CodingGuidelines.html).

