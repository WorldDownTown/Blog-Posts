#Making Sequences work for you 

今日のトピックはSwift2.0以上のSequenceとSequenceの内部的な動きです。
カスタム`class`とか`struct`に`Sequence`の`protocol`を採用したら`for in`ループとか`map`,`filter`などを使えるようになります。

さあ、始まりましょう！

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

この`Unique`というクラスはすごく簡単で色んなデータアイテムを`append`したらそのアイテムがセットに存在してなかったら追加します。

じゃ`Unique`をテストしましょう！


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

じゃ動物の名前は十分だと思います。
ある名前を`print`しようぜ！

```swift

for animal in mySet {
	println(animal)
}
```


あれ？上手く行けなかったね。。。:
![]()

なんで上手く行けなかったかというとつまり`for in`は短縮形です。
`for in`を書くと下記の書き方と一緒に成ります。

```swift
let vowels = ["A","E","I","O","U"]

var gen = vowels.generate()
while let letter = gen.next() {
    print(letter)
}

```

`Unique`は`SequenceType`protocolを採用してないので動かないです。
そうして`SequenceType`protocolの必要な条件を見って実装しましょう！

```swift
public protocol SequenceType {
    associatedtype Generator : GeneratorType
    public func generate() -> Self.Generator
}
```
`generate()`のリタンタイプで`Generator`のタイプを推論出来るので明確な`typealias Generator = ...`文を書く必要はないです。

今のステップは`SequenceType`protocolを採用して`generate()`関すを実装して`GeneratorType`のオブジェクトを`return`するべきだけどまだ`GeneratorType`を見ってないです。

そうして`GeneratorType`をチェックしましょう！

## Generator　とは？
```swift
public protocol GeneratorType {
    associatedtype Element
    public mutating func next() -> Self.Element?
}

```

`SequenceType`と同じように一つの`func`しかないし、`next()`のリタンタイプで`Element`のタイプを推論ができます！
`Generator`は順番にオブジェクトの持ってるデータをリタンして最後のデータ項目をリタンしたら次に`nil`をリタンします。
`GeneratorType`を一回使って終わったら再利用出来ないので同じデータセットをもう一回`GeneratorType`で読みたければ新しい`GeneratorType`を使わなければならないです。
そうして`generate()`を実装する時に`return`された`GeneratorType`と他の`GeneratorType`変数のステートをシエアしないようにしなければならないです。

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

`SequenceType`と`GeneratorType`を学んだので`Unique`クラスの専用`GeneratorType`を作りましょう！

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

今回はすでに学んだことで`Unique`の`GeneratorType`の実装出来ましたがもうちょっと行数が少ない書き方があります：
`AnyGenerator`というGenericタイプでもっと短い書き方書けます：

```swift
extension Unique: SequenceType {
    func generate() -> AnyGenerator<T> {
        var iteration = 0

        return anyGenerator {
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

もう一度`for in`ルプを書きましょう！

```swift
for item in mySet {
    print(item)
}
```

今回やっと出来ました！

`map`と`filter`も動いてる！
やった！

```swift
let cnt = mySet.map { Int($0.characters.count) } //[14, 15, 13, 12, 31, 14]
mySet.filter { $0.characters.first != "A" } //["Petty Parrot", "North American Reckless Raccoon"]
```


#Controlling Sequences with Sequences
偶に無限シーケンスを使いたいけどそのまま使えば永遠にループが止まれられないです。
どうやってその問題を解決するかというとリミットされるシーケンスを作って無限か大きすぎるシーケンスを入れてリミットを決まってコントロール出来るようになります。


例えばこのシーケンスは無限です。
繰り返すに`0`か`1`を`return`します。


```swift
class ThePattern: SequenceType {
    func generate() -> AnyGenerator<Int> {
        var isOne = true
        return anyGenerator {
            isOne = !isOne
            return isOne ? 1 : 0
        }
    }
}

```

このパータンは無限ですからリミットしなければならないです。
先話したリミットクラスを実装しましょう！


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
        
        return anyGenerator {
            defer { self.counter += 1 }
            guard self.counter < self.limit else { return nil }
            return self.generator.next()
        }
    }
}
```

いいね！`First(n, sequence: s)`　を呼び出すと`n`まで`s`のエレメントを取れます。`s`は無限だとうしても最初の`n`エレメントしか取らないです。

```swift
for item in First(5, sequence: ThePattern()) {
    print("\(item)\n)) //prints
}
```





