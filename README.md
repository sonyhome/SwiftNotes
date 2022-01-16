# SwiftNotes
My notes on Swift 5.5, IDE is XCode 13 extract from [Swift Language Guide](https://docs.swift.org/swift-book/LanguageGuide/TheBasics.html) just enough for a C programmer to understand the syntax and concepts.

Swift
* reduces programming errors: variables are initialized before use, indices checked for out of boundsm ints for overflow, optionals handle nil explicitly, memory is managed, error handling help recover.
* performs: syntax designed to make natural code compile best. Type inferance help clear concise code.

# General
Using numbers
```swift
// Comment
/* Multi line
   Comment */
let a = "a"; print(a) // ; is not required except putting multiple statements on a line
UInt8.min; UInt8.max // Int types have a min and a max
Int // uses the platform native size
UInt // Non signed
Float // 32 bit
Double // 64 bit
17 0b0101 0o44 0x3F // Literals in decimal, binary, octal and hex notation
125.0 1.25e2 // Floating point notation with exponent 125.0
0xFp2 // Floating point hex 15*2^2 = 60.0
001_234.560 // Literals can be padded with zeros and _ for readability
let x = 3
let y = Double(x) // Int->Float convertions must be explicit
```
**typealias**
```Swift
typealias myInt = Int
var x : myInt = 3
```
**tuples** are often used to return multiple values in functions.
```Swift
let http404 = (404, "Not Found")
let (_, msg) = http404
print("\(http404.0) is \(msg)")
let http200 = (code: 200, msg: "OK")
print("\(http200.code) is \(http200.msg)")
```
Optional values (**?, !**)
```Swift
let x = "123"; let y = "fails"; let z : Int? // z is an optional of value nil
var xInt = Int(x) // xInt is type Int?, value 123
let yInt = Int(y) // yInt is type Int?, value nil
if xInt != nil {print("\(xInt!)")} // ! forces unwrapping optional
if let xx = xInt {print("\(xx)")} // let does optional binding, if runs if xx is non nil
xInt = nil // works because it is an optional variable
if let xx = xInt, let yy =yInt {...} // If statement only called if all conditions are met
```
Error handling
* Use **throws** and **do try catch**.
* Debug with Assertions **assert(_:_:file:line:)**
* Debug with by enforcing preconditions **preconditionFailure(_:file:line:)** (check is disabled when compiling -Ounchecked)
* Give up on error **fatalError(_:file:line:)**
```Swift
enum fu: Error { case e1 }
func f() throws { ... throw fu.e1 }  // Function can throw an error
do { try f() } catch fu.e1 {...} catch {...} // Catch the errors
let er = try? f() // Catch error in an optional

assert(isTrue(x), "message")
if isFalse(x) {
  assertionFailure("message")
}

// x could be false but should never be at runtime
precondition(isTrue(x), "message")

guard let x = x else { fatal("x can't be nil") }
```


# Operators
For tuples (a, b) coparisons checks: same number of values, then values from left to right until comparison is resolved.
```Swift
=
+ - * / %
-x
+= -= *= /= %=
== != > < >= <= // note: Bool can't use < > operators
x ? y : z
a ?? b // coalescing returns b is a is nil (a is optional)
a! // unwraps optional a, runtime error if nil
a...b // Range operator, used in "for" control flow
a..<b // Range excludes b
a... // One sided range, goes to the last element
...b
..<b
! && || // Left associative logical operators for boolean true/false logic ! must not have a space

=== Identity operators to make sure 2 references point to the same element, for reference types like Class
!== ( == and != compare the content)
```

# Strings and Characters
Swift strings are bridged to Foundation NSString class, and Foundation extends String to expose NSString methods (import Foundation to access NSString methods with String without a cast).
Strings are value types, but the copy is lazy for performance.
```Swift
let s1 = ""
let s1 = String()
let s1 = "Message!"
let s2 = "long \
text"
let s3 = """
  Tabulation matching closing quotes are not shown
    Extra tabs shown but ignoring starting tab
  using "quotes" is ok
  mutli lines doesn't print triple quote lines
  a long line can \
  be split in the code
  """
let specialChars = "\0 \\ \t \n \r \" \' \u{12345678}" // \u is a hex unicode
let s4 = #"xxx\nyyy\nzzz"# // extended delimiter avoids escaping characters
let s4 = #"xxx\nyyy\#nzzz"# // \# is the new escape character so \#n is a newline

if s1.isEmpty {}
s1 += " more" // append to s1
s1.count // number of characters
if s3.hasPrefix("Tabulation") {...}
if s3.hasSuffix("the code") {...}
```
Characters
```Swift
let c1 : Character = "x"
for c in "string" {...} // implicit
let a = [Character] = ["C", "a", "t"]
let s = String(a) // You can build a string from chars
s.append("!")
```
String interpolation with **\()**
```Swift
let i : Int = 3
let s = "number \(2 * i)"
```
**Grapheme Clusters**: some letters can be built as a combination of unicode characters. Note that that counts as ONE character even in strings (see s1.count), and prevents strings characters to be accessed by an array index. Two strings are equal (==) if they have the same meaning and representation (for example there's multiple unicode ways to represent the same e with an accent, similarly there are different "A" letters with different linguistic meaning).
```Swift
let eWithAccent : Character = "\u{65}\u{301}"
// Grapheme impact how you index characters in strings
s1[s1.startIndex] // M
s1[s1.index(after: s1.startIndex)] // e
s1[s1.index(before: s1.endIndex)] // !
s1[s1.endIndex] // Runtime error
for i in s1.indices {...}
s1.insert(contentsOf:"Hey", at: s1.endIndex)
s1.insert("!", at: s1.endIndex)
s1.remove(at: s1.index(before: s1.endIndex))
let range = s1.index(s1.endIndex, offsetBy: -6) ..< s1.endIndex
s1.removeSubrange(range)
```
Substrings are a separate type, meant for short term manipulations, which can share the memory of the strings they spliced for performance optimization.
```Swift
let s = "foobar"
let i = s.firstIndex(of: "b") ?? s.endIndex
let ss = s[..<i] // Substring holds "foo"
```
Strings can be saved to file with different encodings (UTF-8, UTF-16, UTF-32...)
```Swift
let dog = "Dog‼🐶" // ‼ and 🐶 are special UTF characters encoded on 3 and 4 bytes
for code in dog.utf8 {
  print("\(code)", terminator: "") // Prints "68 111 103 226 128 188 240 159 144 182 "
}
for code in dog.utf16 {
  print("\(code)", terminator: "") // Prints "68 111 103 8252 55357 56374 "
}
for code in dog.unicodeScalars { // use 21-bit Uint32 values
  print("\(code)", terminator: "") // Prints "68 111 103 8252 128054 "
}
```

# Generic Collection Types
Swift defines **arrays** (indexed), **sets** (named elements) and **dictionaries** (key values). These are bridged to Foundation NSArray, NSSet, NSDictionary.

If you assign them with **let**, the collection is immutable, a **var** can be appended/removed.

```Swift
// Array declaration
var a1: Array<Int>()  // Definition with explicit initializer syntax
var a2: [Int] = [] // Shorthand definition and initialization
var a3 = Array(repeating: 0.0, count: 3) // Initialize an array of 3 zeros.
var a4 = a3 + a3 // + appends arrays
var a4: [String] = ["a", "b"]
a4.append("c")
a4 += "d"
a4[0] = "A"
print(a4[0])
a4[1..2] = ["B","C"]
a4.insert("bc", at: 1) // inserts an element position 1.
a4.remove(at: 1) // removes element at position 1.
a4.removeLast() //removes lst element
for c in a4 {print(c)} // for iteration
for (i,c) in a4.enumerated {print(c)} // access both index starting at 0, and value
```
Sets hold distinct values of the same type, with no order. The type must follow the **Hashable** protocol (aka a unique Int ID to allow *a==b* comparisons, implemented by the **hash(into:)** method) like numbers, bools, strings, enums. Sets implement set operations and comparisons.
```Swift
var s1 : Set<Character>()  // Definition with explicit initializer syntax
var s2 : Set<Character> = ["c", "d"] // Create a set with an array literal
var s3 : Set = ["b", "c"] // You must declare Set but type can be infered by Swift
s3.insert("a")
if !s3.isEmpty { print(s3.count)} // number of elements in set
if s3.contains("c") { let c = s3.remove("c")} // removes "c" from set and returns it in c
for c in s3 { ... }
for c in s3.sorted() { ... } // uses the < operator to sort the elements
// Set operations can be done efficiently
s3.union(s2)
s3.intersection(s2)
s3.substracting(s2)
s3.symmetricDifference(s2) // What is not in the intersection of s3 and s2
// Compare sets
if s2 == s3 // contain the same elements
if s2.isSubset(of: s3)
if s2.isStrictSubset(of: s3) // s2 != s3 and s2 is a subset
if s2.isSuperset(of: s3)
if s2.isStrictSuperset(of: s3) // s2 != s3 and s2 is a subset
if s2.isDisjoint(with: s3)
```
Dictionaries store key-value associations, both must have a unique type. Keys are unique,value are not. There is no order. Keys must be **Hashable**.
```Swift
var d1: Dictionary<Int, String>()
var d2: [String: Int] = [:] // Initializer syntax shorthand
d2["three"] = 3 // Insert an element
var d3 = ["one": 1, "two": 2] // Swift can infer the type of a dictionary
d3.count
d3.isEmpty
d3.updateValue(2, forKey:"two")
d3["one"] = nil // delete an entry
let one = d3.removeValue(forKey: "one")
for (k, v) in d3 {...} 
for k in d3.keys {...} 
for k in d3.keys.sorted() {...} // If you need an iteration order using operator <
for v in d3.values {...} 
let keys = [String](d3.keys)
```

# Control Flow

* For loops
```Swift
// Loop on arrays
let a = [1,2,3]
for v in a {...}
// Loop on dictionaries (note: it's unordered)
let legs = ["spider": 8, "ant": 6, "cat": 4]
for (k, v) in legs {...}
// Numeric ranges with the closed range operator 
for i in 1...5 {...} // 12345
// Ignoring the value with _
for _ in 1...5 {...}
// Half open range operator
for i in 1..<5 {...} // 1234
// strides
for i in stride(from: 0, to: 60, by: 5) {...} // 0 5 10 ... 55
for i in stride(from: 0, through: 60, by: 5) {...} // 0 5 10 ... 60
```
* While loops
Performs the loop until the condition is false
```Swift
while x() {...}
repeat {...} while(x())
```
* Conditionals
```Swift
if x {...} else if {...} else {...}
guard x else {...} // tells swift x is true past this statement
```
Switch statements don't fall through, allow matching ranges, checks can overlap, handle sets, can name the value (value binding), use where to add a condition
```Swift
switch v {
  case v1: ... // You cannot have an empty case statement 
  case v2: ... fallthrough  // Unlike C, case statements don't flow into each other or need a break, but you can force it
  case v2, v3: ... // You can put 2 values to one action
  case v4..<v5: ... // or you can use interval matching
  default: ...
}

let x = (1,1) // Switch can test tuples
switch c {
  case (0,0): ... // The first matching case exits the switch
  case (_,0): ... // _ means any value
  case (-2...2, -2...2): ...
  case let (x, y): ... //bind values to x, y to use them
  case let (1, y): ...
  case (let d, 0), (0, let d): ... // if same type you can combine
  case let (c, y) where x == y: ... // can have a computed condition
  default: ...
}
```

Control transfer: **continue, break, fallthrough, return, throw, guard, #available**
```Swift
// Continue to skip to next iteration
for x in t {
  if done(x) { continue }
}
// Break exits a control flow
for x in t {
  if stop(x) { break }
}
switch v {
  case v1: ... if x { break } ... // if x aborts
  default: break // If we have an unknown case, aborts
}
// fallthrough makes a case execute the next case block
switch v {
  case v1: ... fallthrough // runs v1 and v2 code
  case v2: ...
}
// use a label to explicitly bread/continue a for/while block
labelx: for x in tx {
  labely: for y in ty {
     continue labely
     break labelx
  }
}

// use guards to clearly require conditions must be met. It must exit a block with continue, break, return, throw or fatalError(_:file:line:) which doesn't return
var x : String? = getx()
guard let x = x else { return }

// You can check for API features for iOS, macOS, watchOS, tvOS with if or guard 
if #available(iOS 13, macOS 11, *) {...} else {...}
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
* **Nested functions** are allowed, are hidden outside of the scope of the enclosing function, but can be exposed by being returned to escape the scope.

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


# Struct and Class

**Value types**: Basic types, enums, structs are copied on assignment. Collections like arrays, dictionaries ad strings do a lazy copy when modified.

Struct have built in initializers you can use.

**Reference type**: Classes are passed by pointer. To tell if two instances of a class are the same use **identity operators === !==**, vs == which check they have the same value

Classes don't have an implicit initializer. 

Swift provides pointers and buffer types only if you need to do manual memory management.

# Properties of Class, Struct and Enum
A property can be a var or a let constant, and can have **lazy** evaluation, and can be a **get/set computed property** with no stored value.
```Swift
struct SType {
  var v1 = 1 // set default values
  let c1 = 1
}
var s1 : SType // Initialized to default values
var s2 = SType(v1: 2, c1: 2) // Override the default values
s1.v1 = 3; s2.v1 = 3
 
clas CType {
   lazy var v1 = SType() // Lazy evaluation done on first use
  let c1 = "1"  
}
print(c1) // v1 not evaluated
print(v1) // SType() is created to initialize v1
// note: in multi-thread v1 might get initialized more than once!

// Computed properties don't actually store a value, but expose get/set
struct SType2 {
  var a = Int(0)
  var b = Int(0)
  var min: Int { // Read/Write computed property with get/set
    get { return a < b ? a : b }
    get { a < b ? a : b  } // For single expression "return" can be ommited
    set(v) {if a < b {a = v} else {b = v}} // Setter example with input value declared
    set {if a < b {a = newValue} else {b = newValue}} // Implicit input value is called newValue
  }
  var avg: Int { return (a+b)/2 } // Read only computed property
}
```
**Property observers willSet/didSet** are called before and after a value is set (even if it's set to the same value) with constant parameters being the new and old value. You can overwrite the new value in didSet. They can be added to to a property defined or inherited or computed property inherited (override it in the sub-class). For your computed property just use the set() to observe and respond.
Note: if the property is passed as an **inout** parameter to a function, the observers are called because the property will always be copied in and out of the function.
```Swift
class c {
  var t: Int  = 0 {
    willSet(newT) {...} // If newT is not specified parameter will be newValue
    didSet { ... oldValue } // and here it's oldValue
  }
}
```
**@propertyWrapper** separate how a property is stored from code that defines it (to do thread safety checks for example).
```Swift
// Define a wrapper
@propertyWrapper
struct TwelveOrLess {
    private var number = 0 // private enforces use of get/set, notice value is initialized
    var wrappedValue: Int { // This is a computed property with no storage used
        get { return number }
        set { number = min(newValue, 12) }
    }
}
// Use the wrapper
struct s1 {
  @TwelveOrLess var v1: Int
  @TwelveOrLess var v2: Int
}
var v1 = s1(v1: 32, v2:6) // Wrapper makes v1 set to 12.
```
For propertyWrapper to allow users to set initial values, like for v1, it must define an init() method(s).
