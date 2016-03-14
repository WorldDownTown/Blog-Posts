#Making Sequences work for you

For this post I want to talk about Generators, Sequences and how you can simplify your programming life with them.
Adopting the SequenceType protocol on a class or struct type that you wish to use with things like for-in loops, map and filter 
will surely make your code shorter and more concise.

Lets start with a class that just maintains a unique set ignoring any duplicates we might add to it.

```swift
struct Unique<T: Comparable> {
    private var backingStore: [T] = [T]()

    var count: Int {
        return backingStore.count
    }

    func setHas(element: T) -> Bool {
        return backingStore.filter { $0 == element }.count != 0
    }

    mutating func append(element: T) {
        guard !setHas(element) else { return }

        backingStore.append(element)
    }
}
```

Ok, so our Unique class seems to be in order.
Basic functionality exists, we can safely append new elements to the set and be assured that there will be no duplicates.
Lets give it a go!

```swift
var mySet = Unique<String>()

// Our set can
mySet.setHas("A Careless Cat") //false
mySet.append("A Careless Cat")
mySet.setHas("A Careless Cat") //true

mySet.append("A Careless Cat")
mySet.count //Still 1, our uniqueness test passed

//lets grow the set some more
mySet.append("A Dangerous Dog")
mySet.append("A Diamond Dog")
mySet.append("Petty Parrot")
mySet.append("North American Reckless Raccoon")
mySet.append("A Monadic Mole")
```

Ok our animals have fairly colorful names.
Lets say we want to list out what we have.

```swift

for animal in mySet {
	print(animal)
}
```


Wow that didnt work as expected, we get this error:
![]()

loop over this one in a for-in loop.

The deal is, for - in is simply shorthand for this:

```swift
let vowels = ["A","E","I","O","U"]

var gen = vowels.generate()
while let letter = gen.next() {
    print(letter)
}

```

So the conclusion is that we need a generator, and all that is is an object that has a next method that will return the next object we wish to pluck from the **series** each time we call it. Woops I got a little ahead of myself there!
    
But what about... THE END? Well... When you are tired and don't feel like you can go on much longer, you can allways just return `nil` and call it quits, but you dont have to.

That is if you dont return nil, the generator will go on forever.
Lets get down to it:

```swift
struct CountToGenerator: GeneratorType {

    private var limit: Int
    private var currentCount = 0

    init(limit: Int) {
        self.limit = limit
    }
    mutating func next() -> Int? {
        guard currentCount < limit else { return nil }

        defer { currentCount += 1 }
        return currentCount
    }
}

var goTillTen = CountToGenerator(limit: 10)
while let num = goTillTen.next() {
    print(num)
}
```

Ok great, looks like we got that one in the bag.
   We pulled off a simple generator that counts from 0 to a number we tell it.
   Lets go ahead and try to make one for our Unique class.

```swift

class UniqueGenerator<T>: GeneratorType {
    private var _generationMethod: () -> T?
    init(_generationMethod: () -> T?) {
        self._generationMethod = _generationMethod
    }

    func next() -> T? {
        return _generationMethod()
    }
}

extension Unique: SequenceType {
    func generate() -> UniqueGenerator<T> {
        var iteration = 0

        return UniqueGenerator {
            if iteration < self.backingStore.count {
                let result = self.backingStore[iteration]
                iteration += 1
                return result
            }

            return nil
        }
    }
}
```

Here all we do is implement the generate() method required by SequenceType and return our the custom Generator we wrote.
Fortunately we really didn't even need to create a `UniqueGenerator` class at all, there already exists a generic generator class that does pretty much the same thing!
Its called GeneratorOf and we can use it to shorten our implementation like so:


Now the syntactic sugar of the for - in loop works as expected.

```swift
for item in mySet {
    print(item)
}
```

We also get map and filter functionality at no additional cost since they are protocol extensions of Sequence!

```swift
let cnt = mySet.map { Int($0.characters.count) } //[14, 15, 13, 12, 31, 14]
mySet.filter { $0.characters.first != "A"} //["Petty Parrot", "North American Reckless Raccoon"]
```



