A guide to our Swift style and conventions.

This is an attempt to encourage patterns that accomplish the following goals (in
rough priority order):

 1. Increased rigor, and decreased likelihood of programmer error
 2. Increased clarity of intent
 3. Reduced verbosity
 4. Fewer debates about aesthetics

If you have suggestions, please see our [contribution guidelines](CONTRIBUTING.md),
then open a pull request. :zap:

----

#### Whitespace

* Spaces, not tabs. This should be specified in the `.xcodeproj`'s settings to override user preferences, which can be done by selecting the project navigator, and then modifying _Text Settings_ in the file inspector (⌘⌥1).
* End files with a newline.
* Make liberal use of vertical whitespace to divide code into logical chunks.
* Don’t leave trailing whitespace.
* Opening braces should be on the same line as their associated `if` statement / function definition / etc.


#### Prefer `let`-bindings over `var`-bindings wherever possible

Use `let foo = …` over `var foo = …` wherever possible (and when in doubt). Only use `var` if you absolutely have to (i.e. you *know* that the value might change, e.g. when using the `weak` storage modifier).

_Rationale:_ The intent and meaning of both keywords is clear, but *let-by-default* results in safer and clearer code.

A `let`-binding guarantees and *clearly signals to the programmer* that its value is supposed to and will never change. Subsequent code can thus make stronger assumptions about its usage.

It becomes easier to reason about code. Had you used `var` while still making the assumption that the value never changed, you would have to manually check that.

Accordingly, whenever you see a `var` identifier being used, assume that it will change and ask yourself why.

#### Avoid Using Force-Unwrapping of Optionals

If you have an identifier `foo` of type `FooType?` or `FooType!`, don't force-unwrap it to get to the underlying value (`foo!`) if possible.

Instead, prefer this:

```swift
if let foo = foo {
    // Use unwrapped `foo` value in here
} else {
    // If appropriate, handle the case where the optional is nil
}
```

Alternatively, you might want to use Swift's Optional Chaining in some of these cases, such as:

```swift
// Call the function if `foo` is not nil. If `foo` is nil, ignore we ever tried to make the call
foo?.callSomethingIfFooIsNotNil()
```

_Rationale:_ Explicit `if let`-binding of optionals results in safer code. Force unwrapping is more prone to lead to runtime crashes.

#### Avoid Using Implicitly Unwrapped Optionals

Where possible, use `let foo: FooType?` instead of `let foo: FooType!` if `foo` may be nil (Note that in general, `?` can be used instead of `!`).

_Rationale:_ Explicit optionals result in safer code. Implicitly unwrapped optionals have the potential of crashing at runtime.

#### If a method has no side effects and doesn’t take parameters, consider making it a computed property

For a particular class of method, specifically pure functions that don't take parameters, it makes more sense to express these as properties

Take this example `Person` type:

```swift
struct Person {
  let firstName: String
  let lastName: String
}
```

Now we could define `fullName` as a function that takes no parameters and returns a `String` like so:

```swift
extension Person {
  func fullName() -> String {
    return "\(firstName) \(lastName)"
  }
}
```

But in this example, the computation being done is trivial, there are no side effects, and no inputs. We can say that these characteristics fulfill the _property contract_ and it would make more sense to express this function as a computed property:

```swift
extension Person {
  var fullName: String {
    return "\(firstName) \(lastName)"
  }
}
```

_Rationale:_ Computed properties make it clear to developers that calling them will involve no side-effects, making it easier to reason about code's behaviour

#### Prefer implicit getters on read-only properties and subscripts

When possible, omit the `get` keyword on read-only computed properties and
read-only subscripts.

So, write these:

```swift
var myGreatProperty: Int {
	return 4
}

subscript(index: Int) -> T {
    return objects[index]
}
```

… not these:

```swift
var myGreatProperty: Int {
	get {
		return 4
	}
}

subscript(index: Int) -> T {
    get {
        return objects[index]
    }
}
```

_Rationale:_ The intent and meaning of the first version is clear, and results in less code.

#### Always specify access control explicitly for top-level definitions

Top-level functions, types, and variables should always have explicit access control specifiers:

```swift
public var whoopsGlobalState: Int
internal struct TheFez {}
private func doTheThings(things: [Thing]) {}
```

However, definitions within those can leave access control implicit, where appropriate:

```swift
internal struct TheFez {
	var owner: Person = Joshaber()
}
```

_Rationale:_ It's rarely appropriate for top-level definitions to be specifically `internal`, and being explicit ensures that careful thought goes into that decision. Within a definition, reusing the same access control specifier is just duplicative, and the default is usually reasonable.

#### When specifying a type, always associate the colon with the identifier

When specifying the type of an identifier, always put the colon immediately
after the identifier, followed by a space and then the type name.

```swift
class SmallBatchSustainableFairtrade: Coffee { ... }

let timeToCoffee: NSTimeInterval = 2

func makeCoffee(type: CoffeeType) -> Coffee { ... }
```

_Rationale:_ The type specifier is saying something about the _identifier_ so
it should be positioned with it.

Also, when specifying the type of a dictionary, always put the colon immediately
after the key type, followed by a space and then the value type.

```swift
let capitals: [Country: City] = [ Sweden: Stockholm ]
```

#### Only explicitly refer to `self` when required

When accessing properties or methods on `self`, leave the reference to `self` implicit by default:

```swift
private class History {
	var events: [Event]

	func rewrite() {
		events = []
	}
}
```

Only include the explicit keyword when required by the language—for example, in a closure, or when parameter names conflict:

```swift
extension History {
	init(events: [Event]) {
		self.events = events
	}

	var whenVictorious: () -> () {
		return {
			self.rewrite()
		}
	}
}
```

_Rationale:_ This makes the capturing semantics of `self` stand out more in closures, and avoids verbosity elsewhere.

#### Only use type annotations where type inference is insufficient

Type annotations should be avoided in cases where the compiler can infer types itself:

```swift
// Good
let squares = numbers.map { number in
  return number * number
}

// Bad
let squares: [Int] = numbers.map { (number: Int) -> Int in
  return number * number
}
```

_Rationale:_ Type annotations are redundant outside of appeasing the type system, as a variable's type can be checked at any time by holding ⌥ and clicking it


#### Prefer structs over classes

Unless you require functionality that can only be provided by a class (like identity or deinitializers), implement a struct instead.

Note that inheritance is (by itself) usually _not_ a good reason to use classes, because polymorphism can be provided by protocols, and implementation reuse can be provided through composition.

This idea can be generalized and emphasized as: Prefer protocols to inheritance and value types to reference types.

For example, this class hierarchy:

```swift
class Vehicle {
    let numberOfWheels: Int

    init(numberOfWheels: Int) {
        self.numberOfWheels = numberOfWheels
    }

    func maximumTotalTirePressure(pressurePerWheel: Float) -> Float {
        return pressurePerWheel * Float(numberOfWheels)
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
}

func maximumTotalTirePressure(vehicle: Vehicle, pressurePerWheel: Float) -> Float {
    return pressurePerWheel * Float(vehicle.numberOfWheels)
}

struct Bicycle: Vehicle {
    let numberOfWheels = 2
}

struct Car: Vehicle {
    let numberOfWheels = 4
}
```

_Rationale:_ Value types are simpler, easier to reason about, and behave as expected with the `let` keyword. Using protocols supports encapsulation, reduces tight coupling and separates concerns.  

#### Avoid using `default` when switching on `enum` types

When writing `switch` statements that deal with `enum` types, it's preferable to explicitly handle every possible case instead of falling back on `default`.

Consider the example enum `ButtonStyle`:

```swift
enum ButtonStyle {
    case Default
    case Primary
    case Danger
}
```

Let's pretend we're using this enum to style a `UIButton` instance, part of that code might look like this:

```swift
switch buttonStyle {
case .Primary:
    button.tintColor = UIColor.primaryButtonColor
case .Danger:
     button.tintColor = UIColor.dangerColor
default:
    break
}
```

This works great for now, but if we were to add another kind of `ButtonStyle` in the future, we'd have to manually go hunt through our codebase and find every one of these `switch` statements so we can handle that new case. It's preferable to write this switch statement the following way instead:

```swift
switch buttonStyle {
case .Primary:
    button.tintColor = UIColor.primaryButtonColor
case .Danger:
     button.tintColor = UIColor.dangerColor
case .Default:
    break
}
```

_Rationale:_ Explicitly handling every possible case means we'll see compiler errors if we ever introduce a new one, helping us ensure that the new case is handled properly throughout our codebase

#### Make classes `final` by default

Classes should start as `final`, and only be changed to allow subclassing if a valid need for inheritance has been identified. Even in that case, as many definitions as possible _within_ the class should be `final` as well, following the same rules.

_Rationale:_ Composition is usually preferable to inheritance, and opting _in_ to inheritance hopefully means that more thought will be put into the decision.


#### Omit type parameters where possible

Methods of parameterized types can omit type parameters on the receiving type when they’re identical to the receiver’s. For example:

```swift
struct Composite<T> {
	…
	func compose(other: Composite<T>) -> Composite<T> {
		return Composite<T>(self, other)
	}
}
```

could be rendered as:

```swift
struct Composite<T> {
	…
	func compose(other: Composite) -> Composite {
		return Composite(self, other)
	}
}
```

_Rationale:_ Omitting redundant type parameters clarifies the intent, and makes it obvious by contrast when the returned type takes different type parameters.

#### Use whitespace around operator definitions

Use whitespace around operators when defining them. Instead of:

```swift
func <|(lhs: Int, rhs: Int) -> Int
func <|<<A>(lhs: A, rhs: A) -> A
```

write:

```swift
func <| (lhs: Int, rhs: Int) -> Int
func <|< <A>(lhs: A, rhs: A) -> A
```

_Rationale:_ Operators consist of punctuation characters, which can make them difficult to read when immediately followed by the punctuation for a type or value parameter list. Adding whitespace separates the two more clearly.
