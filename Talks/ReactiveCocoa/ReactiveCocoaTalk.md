#[fit]ReactiveCocoa入門

---

#FRPはどんな問題を解決しますか？
FRPじゃないiOSアプリの中で色んなパターンが有って：

- UIKitのIBActionメソッド
- NSNotificationCenter
- Delegate Pattern
- Callbacks

---

#IBActionの欠陥

UIイベントがある時に一つのメソッドに入ってそのメソッドの中に色んなオブジェクトに
イベント情報を送ります。

---

###例:「購入」ボタンを押して `purchaseButtonClicked`という`IBAction`メソッド呼ばれたら： 


1. データモデルを更新

2. 購入ボタンの見た目を更新して

3. ローディングスピナーを表示して

4. ネットワークリクエストを始めって

5. リクエストが完了したらCallbackでUIとモデルを更新する

---

#問題とは？ 

上記の*イベント*に対して５つのステップがあるけど３つのステップはUIに関係ないのに
IBActionメソッドの中からそのコードが書いてあります。

実は購入すると言うユーザーアクションはUIで検出するけどその時点の後にネットワークとモデルコードを独立した方がいいと思います。

---

# [fit] **RAC** FTW!

---

![inline](./ReactiveSignal.jpeg)

---

#　YES, Signals!

エベントを送りたいけどどういう仕組みでエベント送ると受けてれますか？
`Signal`でイベントを受け取って`Observer`とか`Disposable`でSignal操作が出来ます。

---

# Events...

```swift
/// Signals must conform to the grammar:
/// `Next* (Failed | Completed | Interrupted)?`
public enum Event<Value, Error : ErrorType> {
    /// A value provided by the signal.
    case Next(Value)
    /// The signal terminated because of an error. No further events will be
    /// received.
    case Failed(Error)
    /// The signal successfully terminated. No further events will be received.
    case Completed
    /// Event production on the signal has been interrupted. No further events
    /// will be received.
    case Interrupted
    /// Whether this event indicates signal termination (i.e., that no further
    /// events will be received).
}
``` 

---

#Signals...

すでに起こっているイベントストリーム

- UIコントロール

- APIリクエスト

- DB Read / Write

- UINotificationCenter

-　なんでも

---

### Signal作成と使い方
```swift
    let (userNameTextSignal, observer) = Signal<String, NoError>.pipe()
        
        userNameTextSignal.observeNext { userName in
            print("Next Event: \(userName)")
        }
        
        observer.sendNext("l")
        observer.sendNext("la")
        observer.sendNext("lat")
        observer.sendNext("latt")
        observer.sendNext("lattn")
        observer.sendNext("lattne")
        observer.sendNext("lattner")
	observer.sendCompleted()
	/*
		Next Event: l
		Next Event: la
		Next Event: lat
		Next Event: latt
		Next Event: lattn
		Next Event: lattne
		Next Event: lattner
	*/
```

---

# ReactiveCocoaとRxの違おうところ

- Naming Convention (Hot Signal = Signal, Cold Signal = SignalProducer)
  
- Cocoa専用API拡張

- もっとシムプル設計


---

### Signal's friends map, filter, reduce

http://neilpa.me/rac-marbles/

---

### Signal Operators in Action - filter

```swift
        let (userNameTextSignal, observer) = Signal<String, NoError>.pipe()
        let improvedUserNameTextSignal = userNameTextSignal.filter { $0.characters.count > 4 }
        
        improvedUserNameTextSignal.observeNext { userName in
            print("Next Event: \(userName)")
        }
        
        observer.sendNext("l")
        observer.sendNext("la")
        observer.sendNext("lat")
        observer.sendNext("latt")
        observer.sendNext("lattn")
        observer.sendNext("lattne")
        observer.sendNext("lattner")
        observer.sendCompleted()
```

---

### 新しいSignalを作成

```swift
        let (userNameTextSignal, observer) = Signal<String, NoError>.pipe()
        
        let userNameValidSignal = userNameTextSignal.map {
            return $0.characters.count > 5
        }
        
        let backgroundColorSignal = userNameValidSignal.map {
            return $0 ? UIColor.greenColor() : UIColor.redColor()
        }
```

---

### Signals & End of Life
swiftの`GeneratorType`と同じように一回`Signal`を使い終わったら再利用出来ない。

---

### SignalProducer (Cold Signals)

- SignalProducers are Signal Factories 
- No work is done until an observer is connected
- Signal Operators can be 'Lifted'

---

### SignalProducer - Nothing 

```swift
var userNameProducer = SignalProducer<String, NoError> { (observer, disposable) in
    ["foo", "bar", "zap", "bin", "fizz"].forEach { userName in
    print("Sending Username: \(userName)")
    observer.sendNext(userName) 
    }
}

```

---

### SignalProducer - Something

```swift
var userNameProducer = SignalProducer<String, NoError> { (observer, disposable) in
    ["foo", "bar", "zap", "bin", "fizz"].forEach { userName in
    print("Sending Username: \(userName)")
    observer.sendNext(userName) 
    }
}

    userNameProducer.startWithNext { userName in
            print("Received Username: \(userName)")
    }
    
    /*
    	Sending Username: foo
		Received Username: foo
		Sending Username: bar
		Received Username: bar
		Sending Username: zap
		Received Username: zap
		Sending Username: bin
		Received Username: bin
		Sending Username: fizz
		Received Username: fizz
	*/

```

---

### SignalProducer - SignalProducerからSignalProducer作る

```swift
var userNameProducer = SignalProducer<String, NoError> { (observer, disposable) in
    ["foo", "bar", "zap", "bin", "fizz"].forEach { userName in
    print("Sending Username: \(userName)")
    observer.sendNext(userName) 
    }
}

    userNameProducer.startWithNext { userName in
            print("Received Username: \(userName)")
    }
    
    let zCounterSignal = userNameProducer.map { return 		$0.containsString("z") }
       zCounterSignal.startWithNext {
       	if $0 {
     	     print("Contains z!")
          } else {
           print("No z here.")
                  }
      }
    
    /*
    	Sending Username: foo
		Received Username: foo
		Sending Username: bar
		Received Username: bar
		Sending Username: zap
		Received Username: zap
		Sending Username: bin
		Received Username: bin
		Sending Username: fizz
		Received Username: fizz
	*/

---





