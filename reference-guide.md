# Swift Value Semantics Reference

## The Mutation Game - Value Semantic, or Reference Semantic?
If we have some type Mystery, we can play the following simple game to determine if it's value semantic!


```swift
var mystery: Mystery = Mystery() // Player 1 choses the type to test
var copy = mystery // they assign another var to have the same value

let start = valueOf(mystery)

/*
Player 2 can do anything he wants here operating only on the copy
*/

let end = valueOf(mystery)

// Player 1 wins if this passes, otherwise player 2 wins
assert(start == end)
```

If player 1 wins, the type is **Value Semantic**.
If player 2 wins, the type is **Reference Semantic**.


If changing the reference can in any way change the original, your type is reference semantic and *cannot leverage the copy on write optimization*.

## Can I Use the Copy on Write Optimazation?
If your type is a value semantic collection from the standard library, you get copy on write optimization for free! These types include:

- Strings
- Arrays
- Dictionaries
- Sets

Otherwise, you have to...

## Make a Simple Struct Type Copy On Write
Starting with the struct `Car`:

```swift
struct Car {
  var speed = 2
  var color = "blue"
}
```

We need to add the following code to make a copy-on-write version of car:

```swift
struct CopyOnWriteCar {
  final class CarRef {
    var car: Car
  }

  private var ref: CarRef

  init(car: Car) {
    self.ref = CarRef(car: car)
  }

  var value: Car {
    get { return ref.car }
    set {
      guard isKnownUniquelyReferenced(&ref) else {
        ref = CarRef(car: newValue)
        return
      }
      ref.car = newValue
    }
  }
}
```

Here, we create a box for the car which all copies of the Car can hold on to. When the value is retreived, we simply give back the reference. When someone sets the value, then we go through the work of making a copy of the structure.

## Creating a Copy on Write Wrapper
You can also genericize this to create a wrapper that allows any value semantic data type to leverage the copy on write optimization as follows:

```
struct CopyOnWrite<ValueType> {
  final class Ref<ValueType> {
    var value: Value
  }

  private var ref: Ref<ValueType>

  init(value: ValueType) {
    self.ref = Ref(value: value)
  }

  var value: ValueType {
    get { return ref.value }
    set {
      guard isKnownUniquelyReferenced(&ref) else {
        ref = Ref(value: newValue)
        return
      }
      ref.value = newValue
    }
  }
}
```