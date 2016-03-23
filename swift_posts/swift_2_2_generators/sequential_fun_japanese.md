#Making Sequences work for you 

今回のトピックはSwift2.0以降の`SequenceType`というプロトコルと、その内部的な動きについて紹介します。`class`や`struct`を`SequenceType`プロトコルに準拠させると、`for in`ループや`map`, `filter`などを使えるようになります。

さあ、始めましょう！

```swift
struct Unique<T: Comparable> {
    private var backingStore: [T] = []

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

この`Unique`という構造体はとてもシンプルです。`Comparable`に準拠した型の要素を`append`すると内部的な配列に要素を追加します。配列中に同じ要素が存在する場合は追加しません。

それでは`Unique`をテストしてみましょう！

```swift
var mySet = Unique<String>()

// Our set can
mySet.setHas("A Careless Cat") //false
mySet.append("A Careless Cat")
mySet.setHas("A Careless Cat") //true

mySet.append("A Careless Cat")　//すでにある文字列
mySet.count //まだ1です！

//もうちょっと項目追加しましょう！
mySet.append("A Dangerous Dog")
mySet.append("A Diamond Dog")
mySet.append("Petty Parrot")
mySet.append("North American Reckless Raccoon")
mySet.append("A Monadic Mole")
```

動物は十分入ったので、名前を`print`してみよう！

```swift
for animal in mySet {
	println(animal)
}
```

あれ？うまくいきませんね。。。:
![]()

なぜうまくいかなかったかというと、`for in`を使うために必要な実装が足りないからです。
Stringの配列で`for in`を書くのは、下記の書き方と同じです。

```swift
let vowels = ["A","E","I","O","U"]

var gen = vowels.generate()
while let letter = gen.next() {
    print(letter)
}
```

`generate()`は`SequenceType`プロトコルのメソッドです。
`Unique`は`SequenceType`プロトコルに準拠してないので`for in`が動かないのです。
それでは、`SequenceType`プロトコルに必要な条件を見て実装してみましょう！

```swift
public protocol SequenceType {
    associatedtype Generator : GeneratorType
    public func generate() -> Self.Generator
}
```

`generate()`の戻り値の型は型推論が効くので、`typealias Generator = ...`を書く必要はありません。

`Unique`を`SequenceType`プロトコルに準拠させて、`generate()`メソッドを実装するべきですが、まだ`GeneratorType`のことをよく知りません。
それでは`GeneratorType`をチェックしましょう！

##Generator とは？

```swift
public protocol GeneratorType {
    associatedtype Element
    public mutating func next() -> Self.Element?
}
```

`SequenceType`と同じように一つのメソッドしかありません。`next()`の戻り値の型は、`Element`型となっていますが型推論が効きます。`[String]`であれば、実際には`String`になります。
`Generator`は`next()`メソッドで、保持しているデータを順番に返します。最後のデータを返したあと再度`next()`を呼ぶと`nil`を返します。

`GeneratorType`を使い終わると再利用できません。同じデータセットをもう一度`GeneratorType`で読みたければ、新しく`GeneratorType`を生成する必要があります。

そして、`generate()`を実装する時に戻り値の`GeneratorType`と他の`GeneratorType`変数の状態を共有しないようにしなければいけません。

基本的な上限がある`GeneratorType`:

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

`SequenceType`と`GeneratorType`を学んだので、`Unique`専用の`GeneratorType`を作りましょう！

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

ここまで学んだことで、`Unique`の`GeneratorType`の実装ができましたが、もう少し短く書けます。
`AnyGenerator`というGenericタイプを使うと`UniqueGenerator`を作る必要がありません。

```swift
extension Unique: SequenceType {
    func generate() -> AnyGenerator<T> {
        var iteration = 0

        return AnyGenerator {
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

もう一度`for in`ループを書きましょう！

```swift
for item in mySet {
    print(item)
}
```

やっと動きました！

`map`と`filter`も動きます！やった！

```swift
let cnt = mySet.map { Int($0.characters.count) } //[14, 15, 13, 12, 31, 14]
mySet.filter { $0.characters.first != "A" } //["Petty Parrot", "North American Reckless Raccoon"]
```


#Controlling Sequences with Sequences

`SequenceType`の実装次第で、とても大きいリストや無限リストを作ることができます。
そういったリストから先頭から`n`個取り出そうとすると、そのまま使うと無限ループになってしまいます。

例えば下記の`ThePattern`は無限リストです。
`for in`で使うと、`Generator`が`nil`を返さないため、永遠に`0`か`1`を返します。

```swift
class ThePattern: SequenceType {
    func generate() -> AnyGenerator<Int> {
        var isOne = true
        return AnyGenerator {
            isOne = !isOne
            return isOne ? 1 : 0
        }
    }
}

// 無限ループ
for i in ThePattern() {
    print(i)
}
```

この無限リストから

```swift
class First<S: SequenceType>: SequenceType {

    private let limit: Int
    private var counter: Int = 0
    private var generator: S.Generator

    init(_ limit: Int, sequence: S) {
        self.limit = limit
        self.generator = sequence.generate()
    }

    func generate() -> AnyGenerator<S.Generator.Element> {
        return AnyGenerator {
            defer { self.counter += 1 }
            guard self.counter < self.limit else { return nil }
            return self.generator.next()
        }
    }
}

for item in First(5, sequence: ThePattern()) {
    print(item) // 0 1 0 1 0
}
```

いいですね！
`First(n, sequence: s)` を呼び出すと、`s`の先頭`n`個の要素を取り出せます。
`s`が無限リストだとしても最初の`n`個しかチェックしないので効率的です。
