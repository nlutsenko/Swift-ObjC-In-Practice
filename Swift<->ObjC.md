build-lists: true
autoscale: true

<!-- 
Copyright (c) 2016 Nikita Lutsenko

Swift Objective-C Interoperability in Practice by Nikita Lutsenko
Licensed under a Creative Commons Attribution-ShareAlike 4.0 International License.
 -->

<br>
# Swift <-> ObjC
# Interoperability in Practice
<br>
####  Nikita Lutsenko
####  @nlutsenko
#### Facebook, Parse, Banana!

---

## Swift <-> ObjC
## Interoperability

---

## Swift <-> ObjC
## I14y

---

![fit](Resources/bridge.jpg)

---

# Agenda

- ObjC -> Swift
- Swift <- ObjC
- Swift <-> ObjC

---

## ObjC -> Swift

---

![fit](Resources/itjustworks.png)

---

## ObjC -> Swift

- #ItJustWorks
- 100% of API is `@available()`
- Focus on:
  - Swifty Names
  - Type Safety
- Much better with Swift 3.0

---

## ObjC -> Swift 2.*

- Nullability Annotations
- Error Handling
- `NS_SWIFT_NAME`
- `NS_SWIFT_UNAVAILABLE`
- `NS_REFINED_FOR_SWIFT`
- `NS_SWIFT_NOTHROW`

---

ObjC
<br/>

```objectivec
- (BOOL)performRequest:(NSURLRequest *)request;
- (BOOL)performRequest:(NSURLRequest *)request 
                 error:(NSError **)error;
```

<br/>

Swift 2.*
<br/>

```swift
func performRequest(request: NSURLRequest!) -> Bool
func performRequest(request: NSURLRequest!, error: ()) throws
```

^ Introduction to example interface

---

ObjC

```objectivec
NS_ASSUME_NONNULL_BEGIN

- (BOOL)performRequest:(NSURLRequest *)request;
- (BOOL)performRequest:(NSURLRequest *)request 
                 error:(NSError **)error;

NS_ASSUME_NONNULL_END
```

<br/>
Swift 2.*

```swift
public func performRequest(request: NSURLRequest) -> Bool
public func performRequest(request: NSURLRequest, error: ()) throws
```

^ Nullability Annototations with NS_ASSUME_NONNULL_*

---

ObjC

```objectivec

- (BOOL)performRequest:(nullable NSURLRequest *)request;

- (BOOL)performRequest:(nullable NSURLRequest *)request
                 error:(NSError **)error;

```

<br/>
Swift 2.*

```swift
public func performRequest(request: NSURLRequest?) -> Bool
public func performRequest(request: NSURLRequest?, error: ()) throws
```

^ Nullability Annototations with `nullable`

---

ObjC

```objectivec

- (BOOL)performRequest:(nullable NSURLRequest *)request 
NS_SWIFT_NAME(perform(request:));

- (BOOL)performRequest:(nullable NSURLRequest *)request
                 error:(NSError **)error 
NS_SWIFT_NAME(perform(request:));

```

<br/>
Swift 2.*

```swift
public func perform(request request: NSURLRequest?) -> Bool
public func perform(request request: NSURLRequest?) throws
```

^ NS_SWIFT_NAME

---

ObjC

```objectivec

- (BOOL)performRequest:(nullable NSURLRequest *)request 
NS_SWIFT_UNAVAILABLE("");

- (BOOL)performRequest:(nullable NSURLRequest *)request
                 error:(NSError **)error 
NS_SWIFT_NAME(perform(request:));

```

<br/>
Swift 2.*

```swift
public func perform(request request: NSURLRequest?) throws
```

^ NS_SWIFT_UNAVAILABLE

---

ObjC

```objectivec

- (BOOL)performRequest:(nullable NSURLRequest *)request
NS_SWIFT_UNAVAILABLE("");

- (BOOL)performRequest:(nullable NSURLRequest *)request
                 error:(NSError **)error;

```

<br/>
Swift 2.*

```swift
public func performRequest(request: NSURLRequest?) throws
```

^ NS_SWIFT_UNAVAILABLE Doesn't need NS_SWIFT_NAME() anymore

---

ObjC

```objectivec

- (BOOL)performRequest:(nullable NSURLRequest *)request
NS_SWIFT_UNAVAILABLE("");

- (BOOL)performRequest:(nullable NSURLRequest *)request
                 error:(NSError **)error 
NS_SWIFT_NAME(perform(request:));

```

<br/>
Swift 2.*

```swift
public func perform(request request: NSURLRequest?) throws
```

^ NS_SWIFT_NAME is still useful, since it provides Swiftier API

---

ObjC

```objectivec
- (void)getFirstName:(NSString **)firstName
            lastName:(NSString **)lastName;

```

<br/>
Swift 2.*

```swift
func getFirstName(firstName: AutoreleasingUnsafeMutablePointer<NSString?>,
                  lastName: AutoreleasingUnsafeMutablePointer<NSString?>)
```

^ Bad API to import into Swift

---

ObjC

```objectivec
- (void)getFirstName:(NSString **)firstName
            lastName:(NSString **)lastName 
NS_SWIFT_NAME(get(firstName:lastName:));

```

<br/>
Swift 2.*

```swift
func getFirstName(firstName: AutoreleasingUnsafeMutablePointer<NSString?>,
                  lastName: AutoreleasingUnsafeMutablePointer<NSString?>)
```

^ We might have gone this way and still used this API.

---

ObjC

```objectivec
- (void)getFirstName:(NSString **)firstName
            lastName:(NSString **)lastName NS_REFINED_FOR_SWIFT;

```

<br/>
Swift 2.*

```swift
extension Person {
  var names: (firstName: String?, lastName: String?) {
      var firstName: NSString? = nil
      var lastName: NSString? = nil
      __getFirstName(&firstName, lastName: &lastName)
      return (firstName: firstName as String?, lastName: lastName as String?)
  }
}
```

^ Use `NS_REFINED_FOR_SWIFT` and create nice Swift wrapper

---

## ObjC -> Swift 3.0

- "The Grand Rename"
- Objective-C Lightweight Generics
- `NS_NOESCAPE`/`CF_NOESCAPE`
- `NS_EXTENSIBLE_STRING_ENUM`

---

ObjC

```objectivec
@interface MyNetworkController<Request: NSURLRequest *> : NSObject

- (void)performRequest:(nullable Request)request;

@end

```

<br/>
Swift 2.*

```swift
public class MyNetworkController : NSObject {
    public func performRequest(request: NSURLRequest?)
}
```

^ Swift 2 does not import ObjC Generics

---

ObjC

```objectivec
@interface MyNetworkController<Request: NSURLRequest *> : NSObject

- (void)performRequest:(nullable Request)request;

@end

```

<br/>
Swift 3

```swift
public class MyNetworkController<Request : NSURLRequest> : NSObject {
    public func perform(_ request: Request?)
}

```

^ Swift 3.0 imports ObjC Generics

---

ObjC

```objectivec
- (void)executeActions:(void (^)(void))actions;
```

<br/>
Swift 3

```swift
func executeActions(_ actions: () -> Void)
```

^ NS_NOESCAPE

---

ObjC

```objectivec
- (void)executeActions:(void (NS_NOESCAPE ^)(void))actions;
```

<br/>
Swift 3

```swift
func executeActions(_ actions: @noescape () -> Void)
```

^ NS_NOESCAPE

---

ObjC

```objectivec
- (void)executeActions:(void (NS_NOESCAPE ^)(void))actions
NS_SWIFT_NAME(execute(actions:));
```

<br/>
Swift 3

```swift
func execute(actions: (@noescape () -> Void))
```

^ NS_NOESCAPE

---

ObjC

```objectivec
typedef NSString *NSRunLoopMode;
extern NSRunLoopMode const NSDefaultRunLoopMode;
```


^ NS_EXTENSIBLE_STRING_ENUM

---

ObjC

```objectivec
typedef NSString *NSRunLoopMode NS_EXTENSIBLE_STRING_ENUM;
extern NSRunLoopMode const NSDefaultRunLoopMode;
```

<br/>
Swift 3

```swift
struct RunLoopMode : RawRepresentable { }

extension RunLoopMode {
  public static let defaultRunLoopMode: RunLoopMode
}

```

^ NS_EXTENSIBLE_STRING_ENUM

---

# ObjC -> Swift

- U
- U
- R
- R
- R
- R

---

# ObjC -> Swift

- Use Nullability Annotations
- Use Objective-C Generics
- Rename (`NS_SWIFT_NAME`)
- Refine (`NS_REFINED_FOR_SWIFT`)
- Rinse (`NS_SWIFT_UNAVAILABLE`)
- Repeat

---

![fit](Resources/bridge_job.jpg)

---

## Swift -> ObjC

---

# Swift -> ObjC

- No Tuples
- No Generics
- No Typealiases
- No Swift Enums
- No Swift Structs
- No Global Variables
- No Free-standing Functions
- Only ObjC Runtime Compatible

---

![fit](Resources/nofun.jpg)

---

# Swift -> ObjC

- Subclasses of `NSObject`
- Not `private` (`fileprivate`)
- Not `@nonobjc`
- `@objc` protocols

---

# `@objc & @nonobjc`

- Subclasses of `NSObject`
- Explicit export to ObjC
- Concrete extensions
- `@objc` protocols
- `Int` enums

---

# `@objc` protocols

- Special case of protocol
- Weaker and not-Swifty
- Supports `optional`
- Extensions are inaccessible from ObjC

---

## Value Types via `_ObjectiveCBridgeable`

- Powers Foundation
- Implementation Detail Protocol
- ObjC Reference Type <-> Swift Types

---

## `_ObjectiveCBridgeable`

```swift
public protocol _ObjectiveCBridgeable {
  associatedtype _ObjectiveCType : AnyObject

  static func _isBridgedToObjectiveC() -> Bool

  func _bridgeToObjectiveC() -> _ObjectiveCType

  static func _forceBridgeFromObjectiveC(_ source: _ObjectiveCType, result: inout Self?)

  static func _conditionallyBridgeFromObjectiveC(
    _ source: _ObjectiveCType,
    result: inout Self?
  ) -> Bool

  static func _unconditionallyBridgeFromObjectiveC(_ source: _ObjectiveCType?) -> Self
}
```

---

## Swift <-> ObjC

---

## Swift <-Pitfalls-> ObjC

---

### Swift <-Pitfalls-> ObjC

![right](Resources/pitfall.png)


- Type Compatibility
- Objective-C Runtime
- Closure -> Block Performance

---

## Type Compatibility

<br/>
<br/>

```objectivec
NSArray<NSString *> *names = @[ @"John", @"Jane" ];
NSArray<id<NSCopying>> *array = names;
```

^ In ObjC generics are compile-time only and are type erased.

---

## Type Compatibility

<br/>

```swift
let names: [String] = ["John", "Jane"]
let array: [CustomDebugStringConvertible] = names
```

^ Let's use the same paradighm. We upcast the array of strings into an array of a protocol that is implemented by String.
^ For example, CustomDebugStringConvertible

---

## Type Compatibility

<br/>

```swift
let names: [String] = ["John", "Jane"]
let array: [CustomDebugStringConvertible] = names
```

## `fatal error: can't unsafeBitCast between types of different sizes`

^ Turns out that this fails with a runtime exception, but doesn't warn you about anything.
^ The reason for it, is that `CustomDebugStringConvertible` is a separate type in Swift, since protocols in Swift have a different memory layout
^ compared to a String. Where String has a struct memory layout, where CustomStringConvertible uses a witness table for each value on top of the storage.
^ So, in our particular case - since you are creating an array of strings - compiler doesn't create extra space for witness table, as it's not needed
^ But when you cast it to CustomDebugStringConvertible - it's actually required.

---

## Type Compatibility

<br/>

```swift
let names: [String] = ["John", "Jane"]
let array: [CustomDebugStringConvertible] = names.map({
  $0 as CustomDebugStringConvertible
})
```

^ You fix it by explicitly casting every member of the array to the explicit type, which adds the witness table behind the scenes

---

## Objective-C Runtime in Swift

```swift
@objc class Animal: NSObject {
    let name = "Animal"
    override init() {
        super.init()
    }
}
```

```objectivec
@interface Plant: NSObject
@end

@implementation Plant

- (NSString *)name {
    return @"Plant";
}

@end
```

---

## Objective-C Runtime in Swift

```objectivec
// ObjC
SEL selector = @selector(name);

Method originalMethod = class_getInstanceMethod([Animal class], selector);
Method swizzledMethod = class_getInstanceMethod([Plant class], selector);

method_exchangeImplementations(originalMethod, swizzledMethod);

// ObjC
NSLog(@"I am %@!", name);

// Swift
print("I am \(name)!")
```


---

## Objective-C Runtime in Swift

```objectivec
// ObjC
SEL selector = @selector(name);

Method originalMethod = class_getInstanceMethod([Animal class], selector);
Method swizzledMethod = class_getInstanceMethod([Plant class], selector);

method_exchangeImplementations(originalMethod, swizzledMethod);

// ObjC
NSLog(@"I am %@!", name); // I am Plant!

// Swift
print("I am \(name)!") // I am Animal!
```

---

![fit](Resources/animal.png)

---

## Objective-C Runtime in Swift

```swift
@objc class Animal: NSObject {
    let name = "Animal"
    override init() {
        super.init()
    }
}
```

```objectivec
@interface Plant: NSObject
@end

@implementation Plant

- (NSString *)name {
    return @"Plant";
}

@end
```

---

## Objective-C Runtime in Swift

```swift
@objc class Animal: NSObject {
    dynamic let name = "Animal"
    override init() {
        super.init()
    }
}
```

```objectivec
@interface Plant: NSObject
@end

@implementation Plant

- (NSString *)name {
    return @"Plant";
}

@end
```

---

## Objective-C Runtime in Swift

```objectivec
// ObjC
SEL selector = @selector(name);

Method originalMethod = class_getInstanceMethod([Animal class], selector);
Method swizzledMethod = class_getInstanceMethod([Plant class], selector);

method_exchangeImplementations(originalMethod, swizzledMethod);

// ObjC
NSLog(@"I am %@!", name);

// Swift
print("I am \(name)!")
```

---

## Objective-C Runtime in Swift

```objectivec
// ObjC
SEL selector = @selector(name);

Method originalMethod = class_getInstanceMethod([Animal class], selector);
Method swizzledMethod = class_getInstanceMethod([Plant class], selector);

method_exchangeImplementations(originalMethod, swizzledMethod);

// ObjC
NSLog(@"I am %@!", name); // I am Plant!

// Swift
print("I am \(name)!") // I am Plant!
```

---

![inline](Resources/plant.jpg)

# Yes, this is dog!

---

## Swift <-> ObjC

- You don't need this!
- Migrate to Swift!
- OpenGL Graphics
- Interacting with C++
- System calls (server side ftw!)
- Super-über-high-performance
- Support for both languages

---

## Swift <-> ObjC
## @ Facebook

- Swift-taylored Facebook SDK
- Builds upon Existing ObjC SDK
- Native Swift Interface & Types
- No exposure to ObjC APIs

![right fit](Resources/facebook-swift.png)

---

## Swift <-> ObjC
## @ Facebook

- Structs over Classes
- Concrete-typed vs Stringly-typed
- Bridging protocols (`***Delegate`) to Closures
- Protocol-oriented (+ default implementation)

---

## Structs over Classes

```swift
// Create a profile request
let request = FBSDKGraphRequest(graphPath: "me",
                                parameters: ["fields" : "name"])

```

---

## Structs over Classes

```swift
// Create a profile request
let request = FBSDKGraphRequest(graphPath: "me",
                                parameters: ["fields" : "name"])

// Start a request
request.startWithCompletionHandler({ _, result, _ in
  // ...
  print(result) // [ 'name': 'Nikita Lutsenko' ]
})
```

---

## Structs over Classes

```swift
// Create a profile request
let request = FBSDKGraphRequest(graphPath: "me",
                                parameters: ["fields" : "name"])

// Start a request
request.startWithCompletionHandler({ _, result, _ in
  // ...
  print(result) // ...
})

request.parameters = ["fields": "id"]
```

---

## Structs over Classes

```swift
// Create a profile request
let request = FBSDKGraphRequest(graphPath: "me",
                                parameters: ["fields" : "name"])

// Start a request
request.startWithCompletionHandler({ _, result, _ in
  // ...
  print(result) // ¯\_(ツ)_/¯
})

request.parameters = ["fields": "id"]
```

---

## Structs over Classes

![inline](Resources/banana.png)

---

## Structs over Classes

```swift
// Create a profile request
let request = GraphRequest(graphPath: "me",
                           parameters: ["fields" : "name"])
```

---

## Structs over Classes

```swift
// Create a profile request
let request = GraphRequest(graphPath: "me",
                           parameters: ["fields" : "name"])

request.start({ _, result in
  // ...
  print(result) // ...
})
```

---

## Structs over Classes

```swift
// Create a profile request
let request = GraphRequest(graphPath: "me",
                           parameters: ["fields" : "name"])

request.start({ _, result in
  // ...
  print(result) // [ 'name' : 'Nikita Lutsenko' ]
})
```

---

## Structs over Classes

```swift
// Create a profile request
let request = GraphRequest(graphPath: "me",
                           parameters: ["fields" : "name"])

request.start({ _, result in
  // ...
  print(result) // [ 'name' : 'Nikita Lutsenko' ]
})

request.parameters = ["fields" : "id"]
// ...
```

---

## Structs over Classes

```swift
// Create a profile request
let request = GraphRequest(graphPath: "me",
                           parameters: ["fields" : "name"])

request.start({ _, result in
  // ...
  print(result) // [ 'name' : 'Nikita Lutsenko' ]
})

request.parameters = ["fields" : "id"]
// Error: Cannot assign to property
```

---

## Structs over Classes

![inline](Resources/strawberry.png)

---

## Stringly-typed

- Using `String` (`NSString`) for types
- Has limitations on values
- Provides loose runtime guarantees
- ObjC-flavor and JS-taste

---

## Stringly-typed

```swift
class FBSDKGraphRequest : NSObject {
  
  var graphPath: String! { get }
  
  var HTTPMethod: String! { get }
  
  var apiVersion: String! { get } 
}
```

---

## Stringly-typed

```swift
class FBSDKGraphRequest : NSObject {
  
  var graphPath: String! { get } // "me"
  
  var HTTPMethod: String! { get } // GET/POST/DELETE
  
  var apiVersion: String! { get } // "v2.7"
}
```

---

## Stringly-typed

```swift
let request = FBSDKGraphRequest(graphPath: "me",
                                parameters: [],
                                httpMethod: "GET")
                                
request.startWithCompletionHandler({ _, result, _ in
    print(result) // ...
})
```

---

## Stringly-typed

```swift
let request = FBSDKGraphRequest(graphPath: "me",
                                parameters: [],
                                httpMethod: "GET")
                                
request.startWithCompletionHandler({ _, result, _ in
    print(result) // [ "name" : "Nikita Lutsenko" ]
})
```

---

## Stringly-typed

```swift
let request = FBSDKGraphRequest(graphPath: "me",
                                parameters: [],
                                httpMethod: "GET")
                                
request.startWithCompletionHandler({ _, _, _ in
    print(result) // ...
})
```

---

## Stringly-typed

```swift
let request = FBSDKGraphRequest(graphPath: "me",
                                parameters: [],
                                httpMethod: "YOLO")
                                
request.startWithCompletionHandler({ _, _, _ in
    print(result) // Error!
})
```

---

## Stringly-typed

```swift
let request = FBSDKGraphRequest(graphPath: "me",
                                parameters: [],
                                httpMethod: "BANANA")
                                
request.startWithCompletionHandler({ _, _, _ in
    print(result) // 🍌
})
```

---

## Stringly-typed

![inline](Resources/banana.png)

---

## Concrete-typed

```swift
struct GraphRequest: GraphRequestProtocol {

  let graphPath: String

  let httpMethod: GraphRequestHTTPMethod

  let apiVersion: GraphAPIVersion
}
```

---

## Concrete-typed

```swift
struct GraphRequest: GraphRequestProtocol {

  let graphPath: String

  let httpMethod: GraphRequestHTTPMethod

  let apiVersion: GraphAPIVersion
}
```

---

## Concrete-typed

```swift
enum GraphRequestHTTPMethod: String {
  
  case GET = "GET"
  case POST = "POST"
  case DELETE = "DELETE"
  
  init?(string: String)
}
```

---

## Concrete-typed

```swift
struct GraphAPIVersion {
  
  let stringValue: String
  
  init(stringLiteral: StringLiteralType)
  
  init(floatLiteral value: FloatLiteralType)
}
```

---

## Concrete-typed

```swift
let request = GraphRequest(graphPath: "me",
                           httpMethod: "GET")
                                
request.startWithCompletionHandler({ _, _, _ in
    print(result) // ...
})
```

---

## Concrete-typed

```swift
let request = GraphRequest(graphPath: "me",
                           httpMethod: "GET") // Error
                                
request.startWithCompletionHandler({ _, _, _ in
    print(result) // ...
})
```

---

## Concrete-typed

```swift
let request = GraphRequest(graphPath: "me",
                           httpMethod: .GET)
                                
request.startWithCompletionHandler({ _, _, _ in
    print(result) // [ "name" : "Nikita Lutsenko"]
})
```

---

## Concrete-typed

```swift
let method = GraphRequestHTTPMethod("YOLO") ?? .GET

let request = GraphRequest(graphPath: "me",
                           httpMethod: method)
                                
request.startWithCompletionHandler({ _, _, _ in
    print(result) // [ "name" : "Nikita Lutsenko"]
})
```

---

## Swift <-> ObjC in Practice

- Large existing codebase
- Support for 2 languages
- Support for latest and greatest
- ???
- Profit

---

# Thank you!
## Questions?

#### github.com/nlutsenko
#### twitter.com/nlutsenko
#### facebook.com/nsunimplemented
