# SwiftNotes
My repo of notes on Swift

# Enumerations

- Enum
- Enum: CaseIterable
- Enum { case x(Int)} (Associated value)
- Enum: String (e.rawValue)
- Indirect (recursive enumerations)

```Swift
// Declare
enum E1 { case a, b, c}
enum E2 {
  case a
  case b
  case c
}

// Use
e = .b
switch e {
  case .a: print("a")
  case .b: print("b")
  default: print("c")
}

// Use iterable protocol
enum E1 : CaseIterable { case a, b, c}
for x in E1.allCases { print(x) }

// Associated value
enum Barcode {
  case upc(Int, Int, Int, Int) // UPC: system-manufactuer-product-check
  case qrCode(String) // string max 2953 ISO 8859-1 characters
}
var productBarcode = Barcode.upc(8, 85909, 51226, 3)
productBarcode = .qrCode("ABCDEFGHIJKLMNOP")
switch productBarcode {
case .upc(let numberSystem, let manufacturer, let product, let check):
    print("UPC: \(numberSystem), \(manufacturer), \(product), \(check).")
case .qrCode(let productCode):
    print("QR code: \(productCode).")
}
//or
switch productBarcode {
case let .upc(numberSystem, manufacturer, product, check):
    print("UPC : \(numberSystem), \(manufacturer), \(product), \(check).")
case let .qrCode(productCode):
    print("QR code: \(productCode).")
}

// Use a raw type (string, char, number types, each enum must use a unique value)
// Implicitly the value of a string will be the enum name, the value of a number will start
// at 0 or whatever you set the first element of the enum. 
enum E1 : String { case a = "a", b = "BB", c = "C"}
print(E1.b.rawValue)
let eb = E1("BB") // set eb to E1.b

// Recursive enumerations: an enum that can use an enum as an element.
enum ArithmeticExpression {
    case number(Int)
    indirect case addition(ArithmeticExpression, ArithmeticExpression)
    indirect case multiplication(ArithmeticExpression, ArithmeticExpression)
}
indirect enum ArithmeticExpression {
    case number(Int)
    case addition(ArithmeticExpression, ArithmeticExpression)
    case multiplication(ArithmeticExpression, ArithmeticExpression)
}
// Combine expressions
let five = ArithmeticExpression.number(5)
let four = ArithmeticExpression.number(4)
let sum = ArithmeticExpression.addition(five, four)
let product = ArithmeticExpression.multiplication(sum, ArithmeticExpression.number(2))
// An evaluate the enum expression
func evaluate(_ expression: ArithmeticExpression) -> Int {
    switch expression {
    case let .number(value):
        return value
    case let .addition(left, right):
        return evaluate(left) + evaluate(right)
    case let .multiplication(left, right):
        return evaluate(left) * evaluate(right)
    }
}
print(evaluate(product))
```

# Closures

Closures are blocks of code similar to functions, nested functions, blocks or lambdas in other languages. **A closure can close over variables and constants** aka access them from the context it is defined in, and swift handles the memory management aspects.
* Global (named) functions don't capture values
* Nested (named) functions capture values from the enclosing function
* (unamed) Expressions capture values from the surrounding context
Closure syntax optimizations:
* **Infer parameter and return types** (value types) from context
* **Implicit return** from a single expression closure
* **$0 Shorthand** for argument names
* **f(){} Trailing closure** syntax

A method can have a closure as a parameter:
```func sorted(by: (_ s1: String, _ s2:String) -> Bool { if by(x,y) ...}```

```swift
let a ["x","a",b"]
// Passing a function
func backward(_ s1: String, _ s2: String) -> Bool { return s1 > s2 }
var ra1 = a.sorted(by: backward)
// Passing a closure (params, return declared before "in" keyword indicating the begining of the closure. They can't be given a default value
var ra2 = a.sorted(by: { (s1: String, s2: String) -> Bool in return s1 > s2} )

// Swift can infer the type from the caller's context so it can be ommited
var ra3 = a.sorted(by: {s1, s2 in return s1 > s2} )
// Since it's a single expression, Swift can infer the return statement
var ra4 = a.sorted(by: {s1, s2 in s1 > s2} )
// Shorthand $0... can be used to avoid naming inputs, and "in" keyword
var ra5 = a.sorted(by: {$0 > $1} )
// If the types of the variables define an operator we can infer the variables
var ra6 = a.sorted( by: > )

// Trailing closure
var ra7 = a.sorted() {$0 > $1}
var ra8 = a.sorted {$0 > $1}

```

# Struct and Class

**Value types**: Basic types, enums, structs are copied on assignment. Collections like arrays, dictionaries ad strings do a lazy copy when modified.

Struct have built in initializers you can use.

**Reference type**: Classes are passed by pointer. To tell if two instances of a class are the same use === !==, vs == which check they have the same value

Classes don't have an implicit initializer. 

Swift provides pointers and buffer types only if you need to do manual memory management.




