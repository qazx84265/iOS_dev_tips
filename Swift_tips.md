# Swift tips

### 1.mutating关键字

Swift中protocol的功能比OC中强大很多，不仅能再class中实现，同时也适用于struct、enum。
使用 mutating 关键字修饰方法是为了能在该方法中修改 struct 或是 enum 的变量，在设计接口的时候，也要考虑到使用者程序的扩展性。所以要多考虑使用mutating来修饰方法。

首先，先定义一个protocol

```
protocol ExampleProtocol {
    var simpleDescription: String { get }
    mutating func adjust()
}
```

在上面，定义了一个ExampleProtocol，接下来我们写一个class来遵守这个协议

```
class SimpleClass: ExampleProtocol {
    var simpleDescription: String = "A very simple class"
    var anotherProperty: Int = 110
    // 在 class 中实现带有mutating方法的接口时，不用mutating进行修饰。因为对于class来说，类的成员变量和方法都是透明的，所以不必使用 mutating 来进行修饰
    func adjust() {
        simpleDescription += " Now 100% adjusted"
    }
}
// 打印结果
var a = SimpleClass()
a.adjust()
let aDescription = a.simpleDescription
```

在 struct中实现协议ExampleProtocol

```
struct SimpleStruct: ExampleProtocol {
    var simpleDescription: String = "A simple structure"
    mutating func adjust() {
        simpleDescription += "(adjusted)"
    }
}
```

在enum中实现协议ExampleProtocol

```
enum SimpleEnum: ExampleProtocol {
    case First, Second, Third
    var simpleDescription: String {
        get {
            switch self {
            case .First:
                return "first"
            case .Second:
                return "second"
            case .Third:
                return "third"
            }
        }

        set {
            simpleDescription = newValue
        }
    }

    mutating func adjust() {

    }
}
```

**错误信息**

如果将ExampleProtocol中修饰方法的mutating去掉，编译器会报错说没有实现protocol。如果将struct中的mutating去掉，则会报错不能改变结构体的成员。



&nbsp;
### 2. 隐式解包(implicitly unwrapped)

```
// explicitly unwrapped
let possibleString: String? = "An optional string."
let forcedString: String = possibleString! // requires an exclamation mark
 
// implicitly unwrapped
// Note: assumedString is also type of optional
let assumedString: String! = "An implicitly unwrapped optional string."
let implicitString: String = assumedString // no need for an exclamation mark”
```


