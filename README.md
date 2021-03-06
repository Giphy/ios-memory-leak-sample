# Optional Enum (annotated with @objc) + Mutable Array Leak (Swift 3) 

* **UPDATE**: As of Xcode Version 9.0 beta 6 (9M214v) -- issue has ben resolved.

## How to reproduce the leak

When a Swift class contains an optional Enum property that is marked with @objc, and this same class also contains a mutable (var) array holding at least one element, there is an unexpected memory leak.

* **Important**: *(XC 9.0 beta required)* This code has been tested on both XC 8.3+ and 9.0+. However, since we used XC 9.0 beta 4 most recently the project file seems to be modified by XCode which **breaks the compatibility with XC 8.3.x** -- if you ran into this problem, you can just create a new Single View application and replace the ```ViewController.swift``` with the code below:

```swift
// Without @objc this enum won't leak
// however when this enum is included in a class
// which contains an array, it will leak
@objc enum leakingObjCMarkedEnum: Int {
    
    // Just some random cases.
    case apple, orange
}

// Wrapper class which contains an enum and Array
// The class needs to contain the the Array in order for
// the Enum to leak.
class WrapperClass {
  
  // Optional enums marked with @objc will leak.
  var leakyOptionalEnum: leakingObjCMarkedEnum?
  
  // Include an array to trigger this behaviour.
  // Empty arrays won't cause the leak, so lets add an arbitrary Int
  var myArray: [Int] = [80]
}

class ViewController: UIViewController {
  
  // Hang on to a reference to our Wrapper Class instance.
  var wc: WrapperClass?
  
  override func viewDidLoad() {
    super.viewDidLoad()
    
    // Allocate an instance of our class
    // and things will start leaking at this point.
    wc = WrapperClass()
  }
}
```

#### Command + I -> Leaks Instrument -> Record

![Leaks Instrument](leaking.png)

## How to prevent the leak

If we convert ```leakOptionalEnum``` optional var to a non-optional var leak will disappear. 

```swift
// Let's convert the optional property to a non-optional
var leakOptionalEnum: leakingObjCMarkedEnum = .apple
```
#### Command + I -> Leaks Instrument -> Record

![Leaks Instrument](no_leaks.png)

## Similar reported leaks

We found a Stack Overflow question which addresses the root cause of this issue without the @objc marker. (https://stackoverflow.com/questions/42602301/swift-3-enums-leak-memory-when-the-class-contains-an-array ) This particular case seems to be fixed in later versions of XC and Swift.

```swift
enum LeakingEnum: Int, RawRepresentable {
    case
    LeakCase,
    AnotherLeakCase
}

class Primitive {
    var lightingType: LeakingEnum?
    var mysub : [Int] = []
    init() {
        mysub.append(80)
    }
}

class ViewController: UIViewController {
    var prim: Primitive?
    override func viewDidLoad() {
        super.viewDidLoad()
        prim = Primitive()
    }
}
```

## Conclusion

We believe this is a compiler related issue and are reporting a Radar to Apple.

![Leaking](https://media.giphy.com/media/l3q2MDnkLri1t7i5a/giphy.gif)
