# SwiftNotes
My repo of notes on Swift

# Enumerations

Enums can be more than a list of elements in Swift.

- **Enum** declarations (it is a copy by value type, not by reference)
- Enum: CaseIterable (for x in E.**allCases**{})
- Enum Associated value  Enum E {case **x(Int)**}
- Enum: BaseType (e.**rawValue**)
- Recursive enumerations **indirect case** 

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

# Functions

A function is a closure with a name. Its type is defined by its parameter and return types.
* _ No parameter name
* name: and argument labels for parameters
* = default value
* A function can be a parameter of a function or returned by a function
* Optional return Type?
* 
```Swift
// Declare a function f with one integer input parameter v, returning a string
func f(v: Int) -> String { return "x"}
// Call the function
print(f(v: 3))
// A function with no parameter still needs to be called with parentheses
// Note it is the same as func f2() -> Void {...}
func f2() {...}
f2()
// Or with multiple parameters:
func f3(vI: Int, vS: String) {...}
f3(vI: 1, vS: "x")
// Ignore return value
let _ = f(1)

// Return multiple values in a tuple
func f4(a: [Int]) -> (min: Int, max: Int) {...
  return (vMin, vMax) // don't need to name explicitly the tuple elements
}
let r = f4([1,2,3])
print ("Values from \(r.min) to \(r.max)")

// Handling optional ? return value
func f5() -> String? {... return }
let x = f5() // x is nil because
if let x = f5() {print(x)} // optional binding prevents using a nil value
guard let x = f5() else { print("no x!") } // optional binding guard handles nil value, rest of the code can assume it is set

// Implicit return, for single expression functions returning by value, we can ommit "return"
func msg() -> String {"The message"}
// Function parameters have an argument name (for the call), which don't need to be unique as parameters are passed in order, and a parameter name which are unique (used within the function). Use _ to not specify an argument name.
func f6(n: Int) -> Int {return n}
let x = f6(n: 33)
func f7(l n: Int) -> Int {return n}
let x = f7(l: 33)
 func f8(_ n: Int) -> Int {return n}
let x = f8(33)
// You can set a default value with =
 func f9(_ n: Int = 33) -> Int {return n}
let x = f9()
```
* **... Variadic** parameters accepts 0 or more values of the specified type, seen as an array in the function
* Passing multiple variadic parameters requires all but first parameters to have explicit labels
```Swift
func f(_ nums:Double...) -> Double {
  var t: Double = 0
  for n in nums { t += n }
  return t / Double(nums.count)
}
```
* Function parameters are read only unless they have the **inout** keyword (the parameter must be a var, and you mark it with an &).
```Swift
func f(_ a: inout Int) { a++ }
var x = 1
f(&x)
```
* A variable can have a function type and be assigned a function
* Similarly it can be another function's parameter, and can be returned by a function
```Swift
func f1() {...}
var r: () -> Void = f1
r()
func f2() {...}
r = f2
func f3(fn: () -> Void) {...}
f3(fn: r)
func f4() -> () -> Void {return f1}
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

```Swift
let a ["x","a","b"]
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
// For example it's used for map: for arrays to apply an action to all elements
let x = ary.map { (value -> String in ... return digitNames[value]! }
// With multiple closures, label all but the first one
func loadPicture(from server: Server, completion: (Picture) -> Void, onFailure: () -> Void) {...}
loadPicture(from: s) {pic in ...} onFailure: {...}
```
* **Capturing variables by reference** from context: Swift will do a lazy capture and manage memory as needed
* **Closures and Functions are reference types** (var f = someFunc)

```Swift
func makeInc(for amount: Int) -> () -> Int {
  // Acts as a hidden global
  var total
  func inc() -> Int {
    total += amount // Capture a REFERENCE to total and amount
    return total
  }
  // Return the closure function
  return inc
}
let inc10 = makeInc(for: 10) // Creates an instance of total
inc10() // returns 10
inc10() // returns 20
let inc3 = makeInc(for: 3) // Creates a new instance of total
inc3() // returns 3
inc10() // returns 30
inc3() // returns 6
```

* **@escaping** Declares escaping closures, aka closures passed as arguments to a function, which get used after the function returns. For example the function assigns it to a global.
* Escaping closures capturing variables from their parent class must explicitly use **self** (non escaping closures can ommit self and do implicit references)  
* Structures and Enumerations don't allow escaping closures to self because they are value types and not reference types.
* 
```Swift
var handlers: [() -> Void] = []

class C {
  var x = 0
  func f(h: @escaping () -> Void) {
    self.x += 1
    handlers.append(h)
  }
}
// Note you can capture all of self in the capture list to allow implicit references
  func f(h: @escaping () -> Void) { [self] in
    x += 1
    handlers.append(h)
  }

```
* **Autoclosures** are closures automatically created to wrap an expression passed as an argument to a function. It lazily returns the value of the expression only when it's called.
```Swift
// Let's look at lazy evaluation of expressions:
var a = ["a", "b", "c"]
let rem = {a.remove(at: 0) } // declares closure, so not yet evaluated
print(a.count) // 3
print("Deleting \(rem())") // Calls closure & prints "Deleting 3"
print(a.count) // 2 - lazy deletion

// This works the same with a function call
func do(entry func: () -> String) {
  print("do \(func())")
}
do(entry: {a.remove(at: 0)} ) // Prints "do b"

// An @autclosure syntax makes this behavior implicit
func do(entry func: @autoclosure () -> String) {...}
do(entry: a.remove(at: 0) ) // Prints "do b"
// Avoid declaring @autoclosures except when it's clearly useful, like debug handlers.
// You can have @autoclosure @escaping

var customersInLine = ["Barry", "Daniella"]
var customerProviders: [() -> String] = []
func collectCustomerProviders(_ customerProvider: @autoclosure @escaping () -> String) {
    customerProviders.append(customerProvider)
}
// Appends two customer removals in customerProviders, but remove is not evaluated
collectCustomerProviders(customersInLine.remove(at: 0))
collectCustomerProviders(customersInLine.remove(at: 0))

print("Collected \(customerProviders.count) closures.") // Prints "Collected 2 closures."

for customerProvider in customerProviders {
    print("Now serving \(customerProvider())!")
}
// Prints "Now serving Barry!", "Barry" is removed from position 0
// Prints "Now serving Daniella!", customersInLine is now empty
```


# Struct and Class

**Value types**: Basic types, enums, structs are copied on assignment. Collections like arrays, dictionaries ad strings do a lazy copy when modified.

Struct have built in initializers you can use.

**Reference type**: Classes are passed by pointer. To tell if two instances of a class are the same use === !==, vs == which check they have the same value

Classes don't have an implicit initializer. 

Swift provides pointers and buffer types only if you need to do manual memory management.




