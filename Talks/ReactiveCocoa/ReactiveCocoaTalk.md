#[fit]ReactiveCocoa入門
Signals, SignalProducers and Events! Oh my!

---

###自己紹介
VasilyのiOSエンジニアにこらすと申します。
最近Swift Evolution SE-0053を提案してSwift3.0に入りました。
偶にOSSとかSwift Evolutionに貢献するので興味が有れば github.com/**nirma**
とかtwitterで[@din0sr](https://twitter.com/din0sr)でフォローしてください。

---

###FRP(Functional Reactive Programming)のメリットとは？
* コードノイズ減ります
* IBAction, NSNotificationCenterおよびCallback/Delegateのパターンにはもっとシンプルな設計でSignalとイベントのパターンで書けます。

---

###例: リフレッシュボタンのクリックイベント (MVCとFRPではないの場合)

```swift

   @IBAction private func refreshButtonClicked(sender: AnyObject) {
   updateViewForState(.Loading)
   performNetworkRequest() { 
   		updateModel()
   		dispatch_async(dispatch_get_main_queue()) {
       	updateViewForState(.Success)
       }
   }
   
 }
 
 ```
 
---

#OOP+MVCの問題点: Case Study
一つずつ順番にコードで命令型っぽい感じで「どうやってタスクAをする」というコードを書けなければいけないので順番ではないです。
もっとコンサイスなコードを書きたいです。

---

今の書き方をロープの書き方に比べたらこんな感じになると思います。
```swift
	var counter = 0 
	var animationImages = [UIImage]()
	
	while counter < 10 {
		let imageString = "animation_image_\(counter)"
		if let image = UIImage(named: imageString) {
			animationImages += [image]
		}
	}
```

---

こういう風にVCかVMのコード書きたいです：

```swift
(0..<10).flatMap { UIImage(named: "animation_image_\($0)") }
```

---


###例:ボタンを押して `refreshButtonClicked`という`IBAction`メソッド呼ばれたら： 


1. ボタンの見た目を更新して

2. ローディングスピナーを表示して

3. ネットワークリクエストを始めって

4. リクエストが完了したらCallbackでUIとモデルを更新する

---

# [fit] **RAC** FTW!

# ReactiveCocoaとRxの違おうところ

- Naming Convention (Hot Signal = Signal, Cold Signal = SignalProducer)
  
- Cocoa専用API拡張

- もっとシムプル設計

- etc 

---

![inline](./ReactiveSignal.jpeg)

---

#　YES, Signals!

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

#Signals, cheap, 

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


### Signal's friends map, filter, reduce

[RAC Marbles](http://neilpa.me/rac-marbles/)

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

### 既存Signalから新しいSignalを作る

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
```

---

### レガシィRACとRAC4のAPI変更

- RACSignalが無く成りました。（This is a lie.）
- RAC3.0からRACSignal(HOTとCOLD)SignalとSignalProducer
- ~~Global Functions~~ `Protocol Extensions`
- `|>`(Pipe-Left)は普通の「Dot Operator」に成りました

まだ`RACSignal`タイプリタンしてるメソッドが多いけど`.toSigna`
```swift
searchBox.rac_textSignal().toSignalProducer()
```
[RAC change log](https://github.com/ReactiveCocoa/ReactiveCocoa/blob/master/CHANGELOG.md)

---

### まとめ
- RAC4はいいFRPライブラリーとして設計が良くって、コミュニティもうヘルシーし、進化も早いけどまとめだRACSignalとがレガシィシィコードが残ってるから'5/5'を上げられないです。
- Cocoaの拡張は便利けどちょっと肥大化
- Learning Curve is steep



