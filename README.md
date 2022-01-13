# SwiftNotes
My repo of notes on Swift

# Enumerations

```
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



