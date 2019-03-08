## Subject

서브젝트다. 서브젝트랑 서브스크라이브랑 서브로 시작해서 그런지 자주 용어를 혼동해서 사용하곤 했다. 내가 아는 서브로 시작하는 단어는…서브병뿐인데…

![31](https://github.com/sujinnaljin/RxSwift_Theorem_MarkDown/blob/master/%EA%B0%95%EC%88%98%EC%A7%84/images/31.jpg)

다음웹툰 취향저격 그녀. 작가님 적게 일하고 많이버세요.



다시 주제로 돌아와서 ~~주제? 서브젝트? ㅎㅎ 라임 뺌!~~ `Subject`와 `Observable` 의 차이를 모르겠어서 열심히 구글링 한 결과 [여기](https://arnoldyoo.tistory.com/18 )에서 아래와 같은 설명을 봤다.

> Subject는 observable과 observer의 역할을 모두 할 수 있는 bridge/proxy Observable이라 생각하면 됩니다. 그렇기 때문에 Observable이나 Subject 모두 Subscribe를 할 수 있습니다. 다만 subscribe의 차이가 있다면 Subject는 multicast방식이기 때문에 여러개의 observer를 subscribe할 수 있습니다. 단순 observable은 unicast방식이기 때문에 observer하나만을 subscribe할 수 있습니다. 아래 그림을 보시면 이해가 될 것입니다.
>
> ![32](https://github.com/sujinnaljin/RxSwift_Theorem_MarkDown/blob/master/%EA%B0%95%EC%88%98%EC%A7%84/images/32.png)

비록 RxJs 에 관한 글이지만 ReactiveX의 개념은 언어를 막론하고 비슷하니까! 참고할 수 있어서 좋았다. 

![33](https://github.com/sujinnaljin/RxSwift_Theorem_MarkDown/blob/master/%EA%B0%95%EC%88%98%EC%A7%84/images/33.jpg)

​									땡큐! 알엑스! 땡큐!

하지만 슬프게도 아래 그림을 보면 이해가 될거라는 글과 다르게 사실 잘 이해가 안됐다. [다른 글](https://code.i-harness.com/ko-kr/q/25aa1aa)을 통해 조금 더 살펴 보자.  

`Observable`과 `Subject`는 하나의 매우 중요한 차이를 가진다. `Observable`은 단지 하나의 함수이기 때문에 어떤 상태도 가지지 않으므로 모든 새로운 `Observer`에 대해 관찰 가능한 `create` 코드를 반복해서 실행한다. 코드는 각 관찰자에 대해 실행되므로 HTTP 호출인 경우 각 관찰자에 대해 호출된다. 이로 인해 주요 버그와 비효율이 발생한다.

반면 `Subject`는 관찰자 세부 정보를 저장하고 코드를 한 번만 실행하고 모든 관찰자에게 결과를 제공한다.

이제 예시를 통해 unicast와 multicast의 차이를 알아 보자.

우선 `observable`의 unicast를 보자. [이 블로그](https://medium.com/@luukgruijs/understanding-rxjs-subjects-339428a1815b)에선 unicast란 각각 subscribed된  `observer`가 `observable`에 대해 독립적인 실행을 갖는것이라고 설명한다.

뭔말이지. 코드로 얘기하자. 저 블로그에 있는 RxJs코드 RxSwift로 바꿨다.

```swift 
// --- Observable ---
    let randomNumGenerator1 = Observable<Int>.create{ observer in
        observer.onNext(Int.random(in: 0 ..< 100))
        return Disposables.create()
    }
    
    randomNumGenerator1.subscribe(onNext: { (element) in
        print("observer 1 : \(element)")
    })
    randomNumGenerator1.subscribe(onNext: { (element) in
        print("observer 2 : \(element)")
    })
    
    --------------------print------------------
    
observer 1 : 54
observer 2 : 69
```

다른 숫자가 출력된다. 왜일까. `observer`가 해당 `observable`에 대해 독자적인 실행을 갖기 때문에, 동일한 `observable` 구독을 통해 생성된 두개의 `observer`라고해도  `observable`이 각각 실행되면서 `observer`에게 서로 다른 값이 가는 것이다.

이제 `subject`의 multicast를 보자. multicast가 된다는건 하나의 `observable` 실행이 여러 `subscriber`에게 공유되는걸 뜻한다.

```swift
  // ------ BehaviorSubject/ Subject
    let randomNumGenerator2 = BehaviorSubject(value: 0)
    randomNumGenerator2.onNext(Int.random(in: 0..<100))
    
    randomNumGenerator2.subscribe(onNext: { (element) in
        print("observer subject 1 : \(element)")
    })
    randomNumGenerator2.subscribe(onNext: { (element) in
        print("observer subject 2 : \(element)")
    })

    --------------------print------------------

observer subject 1 : 92
observer subject 2 : 92
```

구독해서 생성될 `observer`에게 `observable`의 동일한 실행이 간다. `observable`의 실행이 공유되면서 같은 값 프린트된다.

다시 정리하자면, `observable`에서 `subscribe`를 하면 이벤트로 전달되는 것은 항상 새로운 것. (`observable`의 `create` 코드는 항상 새롭게 실행된 결과. cf)Observables are just functions that set up observation). `subject`에서 `subscribe`하면 이벤트로 전달되는거는 `subscribe` 전에 수행 된 `onNext` 등의 하나의 공통된 것.

`Observable`과 `Subject`의 차이를 이렇게 정리할 수 있다.

| Observable                                              | Subject                                                      |
| :------------------------------------------------------ | :----------------------------------------------------------- |
| 그냥 단지 함수이다. state 존재 x                        | state를 가진다. data를 메모리에 저장.                        |
| 각각의 옵저버에 대해 코드가 실행.                       | 같은 코드 실행. 모든 옵저버에 대해 오직 한번만.              |
| 오직 옵저버블만 생성 (data producer)                    | 옵저버블 생성 및 그것을 관찰할 수 있음 (data producer and consumer) |
| 용도 : 하나의 옵저버에 대한 간단한 observable 필요할 때 | 용도 : 1. 자주 데이터를 저장하고 수정할때, 2. 여러개의 옵저버가 데이터를 관찰해야 할 때, 3. 옵저버와 옵저버블 사이의 프록시 역할 |

cf )주로 `observable`을 data producer, `subject`는 producer & consumer라고 표현하는데 `subject`를 data consumer로 이용함으로서 `observable`을 unicast에서 multicast로 바꿀 수 있는 작업을 할 수 있다는데..! 잘 모르겠으니 더 보고 싶은 사람은 [여기](https://medium.com/@luukgruijs/understanding-rxjs-subjects-339428a1815b)에서 보기! 역시 수련이 필요해!



![34](https://github.com/sujinnaljin/RxSwift_Theorem_MarkDown/blob/master/%EA%B0%95%EC%88%98%EC%A7%84/images/34.png)

​										그란데 말입니다...

흠,, 원래 궁금했던 걸로 다시 돌아가보자..! `subject`는 `observable`이자 `observer`이다. 여기서 비롯되는 `observable`과의 차이점이 뭘까,,? 좋아 코드를 보자,,! 

`BehaviorSubject`로 글을 시작하겠다. 굳이  `BehaviorSubject` 를 쓴 이유? 없다.

```swift 
public final class BehaviorSubject<Element>
    : Observable<Element>
    , SubjectType
    , ObserverType
    , SynchronizedUnsubscribeType
    , Disposable {
    //blah blah~~
    }
```

근데 들어가자마자  `BehaviorSubject`은 `Observable` 과 `ObserverType` 을 따르고 있는걸 확인할 수 있다. 그래서 `Subject`가 `Observable`이자 `Observer`이라고 한거구나..! 

![35](https://github.com/sujinnaljin/RxSwift_Theorem_MarkDown/blob/master/%EA%B0%95%EC%88%98%EC%A7%84/images/35.jpeg)

​										드디어 깨달은 사실..

`ObserverType`을 따르고 있기 때문에 단순 `Observable`과는 차이가 있다. 어떤 차이일까. `ObserverType`으로 가보자

 ```swift 
public protocol ObserverType {
    /// The type of elements in sequence that observer can observe.
    associatedtype E

    /// Notify observer about sequence event.
    ///
    /// - parameter event: Event that occurred.
    func on(_ event: Event<E>)
}
 ```

`ObserverType`은 프로토콜인데 안에 `on` 이라는 메소드가 정의되어있다. 그러니까 `ObserverbleType`을 따르면 `func on(_ event: Event<E>)` 을 구현해줘야한다는 의미다. `Subject` 로 가면 실제로 해당 메소드가 구현되어있다. 

서론에서 이런말을 한적이 있다.

> 그런데  `subject`에서는 이런 코드를 쓸 수 있다.
>
> ```swift 
> let subject = BehaviorSubject(value: "")
> subject.onNext("sujinnaljin")
> ```

`Subject`는 `ObserverType`으로 내부에 `on` 함수를 구현하고 있기 때문에 저렇게 바로 `.onNext`를 호출 할 수 있던 것이다.

`Observer.onNext` 같은 형태라서 처음에는 옵저버가 이벤트를 방출하는건줄 알았다. 분명 이벤트를 방출하는건 `Observable`인데..?라는 생각에 혼란스러웠다. 하지만 `ObserverType` 의 `on` 함수 위에 설명 덕분에 생각을 정리할 수 있었다. 

> Notify observer about sequence event.

그러니까 `observer`가 이벤트를 발생시키는게 아니라 `observer`에게 이벤트를 알려준다는 의미이다.

아래는 `Subject`에 실제 구현된  `on` 함수이다.

- BehaviorSubject / PublishSubject / ReplaySubject

```swift 
/// Notifies all subscribed observers about next event.
    ///
    /// - parameter event: Event to send to the observers.
    public func on(_ event: Event<E>) {
        #if DEBUG
            _synchronizationTracker.register(synchronizationErrorMessage: .default)
            defer { _synchronizationTracker.unregister() }
        #endif
        dispatch(_synchronized_on(event), event)
    }
```

그런데 여기서 주목할만한 설명이 있다.

> Notifies all subscribed observers about next event.

해석하자면 모든 `subscribe` 된 `observer`들에게 `nextEvent`를 전달한다는 건데… observer's'? 그러니까 `Subject`는 각각 `observer`을 갖고 있는게 아니라 `observer`들을 갖고있다는 말인가? 를 생각할 수 있다. 

아니나 다를까 `Subject` 안에는 ` _observers`라는 변수가 있어서 여기에 다수의 관찰자들에 대한 정보를 저장해 놓는다.

```swift 
private var _observers = Observers()
```

그렇다면 `on` 은 어떤 식으로 이 모든 `_observers`에게 `nextEvent`를 전달한다는 것일까?

`Subject`의 `on` 함수 하단에서는  `dispatch(_synchronized_on(event), event) ` 를 통해 `_synchronized_on` 함수를 호출하고 있다.

- BehaviorSubject

```swift
//BehaviorSubject
func _synchronized_on(_ event: Event<E>) -> Observers {
        _lock.lock(); defer { _lock.unlock() }
        if _stoppedEvent != nil || _isDisposed {
            return Observers()
        }
        
        switch event {
        case .next(let element):
            _element = element
        case .error, .completed:
            _stoppedEvent = event
        }
        
        return _observers
    }
```

- PublishSubject

```swift
//PublishSubject
func _synchronized_on(_ event: Event<E>) -> Observers {
        _lock.lock(); defer { _lock.unlock() }
        switch event {
        case .next(_):
            if _isDisposed || _stopped {
                return Observers()
            }
            
            return _observers
        case .completed, .error:
            if _stoppedEvent == nil {
                _stoppedEvent = event
                _stopped = true
                let observers = _observers
                _observers.removeAll()
                return observers
            }

            return Observers()
        }
    }
```

- ReplaySubject

```swift 
//ReplaySubject
func _synchronized_on(_ event: Event<E>) -> Observers {
        _lock.lock(); defer { _lock.unlock() }
        if _isDisposed {
            return Observers()
        }
        
        if _isStopped {
            return Observers()
        }
        
        switch event {
        case .next(let element):
            addValueToBuffer(element)
            trim()
            return _observers
        case .error, .completed:
            _stoppedEvent = event
            trim()
            let observers = _observers
            _observers.removeAll()
            return observers
        }
    }
```

` _synchronized_on` 함수의 구현부는 `subject`마다 조금씩 다르지만 정상적인 상황일때 공통적으로 `_observers` 을 반환한다.

따라서 첫번째 인자로 해당 서브젝트의 모든 `observers`을 담고있고, 두번째 인자로 nextEvent를 갖고 있는 `dispatch(_synchronized_on(event), event)` 는 모든 `observers`들이 `nextEvent`를 받는 것을 가능케한다.

![36](https://github.com/sujinnaljin/RxSwift_Theorem_MarkDown/blob/master/%EA%B0%95%EC%88%98%EC%A7%84/images/36.jpg)

​										였다면 좋았을텐데..^^

한편 이 `_observers`는 `subject`가 `subscribe`될때마다 생기는 `observer`들이 모인 것일텐데, 이는 어떤 과정을 거쳐 만들어진건지 살펴보자.

`Observable`과 `Subject` 기본적으로 둘다 `ObserveableType`을 상속받기 때문에 `subscribe` 함수를 호출 할때 같은 코드가 실행된다.

```swift 
public func subscribe(onNext: ((E) -> Void)? = nil, onError: ((Swift.Error) -> Void)? = nil, onCompleted: (() -> Void)? = nil, onDisposed: (() -> Void)? = nil)
        -> Disposable {
            let disposable: Disposable
            
            if let disposed = onDisposed {
                disposable = Disposables.create(with: disposed)
            }
            else {
                disposable = Disposables.create()
            }
            
            #if DEBUG
                let synchronizationTracker = SynchronizationTracker()
            #endif
            
            let callStack = Hooks.recordCallStackOnError ? Hooks.customCaptureSubscriptionCallstack() : []
            
            let observer = AnonymousObserver<E> { event in
                
                #if DEBUG
                    synchronizationTracker.register(synchronizationErrorMessage: .default)
                    defer { synchronizationTracker.unregister() }
                #endif
                
                switch event {
                case .next(let value):
                    onNext?(value)
                case .error(let error):
                    if let onError = onError {
                        onError(error)
                    }
                    else {
                        Hooks.defaultErrorHandler(callStack, error)
                    }
                    disposable.dispose()
                case .completed:
                    onCompleted?()
                    disposable.dispose()
                }
            }
            return Disposables.create(
             	//주목
                self.asObservable().subscribe(observer),
                disposable
            )
    }
```

여기서 주목해야할 부분은 `Disposable`을 `create`하고 return 하는 과정에서 불리는 `self.asObservable().subscribe(observer)` 함수다. 이전 포스팅에서도 설명했듯  `ObservableType` 은 protocol이고  `subscribe` 라는 메소드가 있다. 그리고 해당 프로토콜을 따르는 각 `Subject`를 살펴보면 그 안에 `override` 를 통해 `.subscribe`함수를 구현해 놓고 있음을 확인할 수 있다.

- BehaviorSubject / PublishSubject / ReplaySubject

```swift
     override func subscribe<O : ObserverType>(_ observer: O) -> Disposable where O.E == Element {
        _lock.lock()
        let subscription = _synchronized_subscribe(observer)
        _lock.unlock()
        return subscription
    }
```

이 함수가 불리면서 각기 다른 구현부를 가진 `_synchronized_subscribe(observer)` 호출한다.

- BehaviorSubject

```swift
// BehaviorSubject
func _synchronized_subscribe<O : ObserverType>(_ observer: O) -> Disposable where O.E == E {
        if _isDisposed {
            observer.on(.error(RxError.disposed(object: self)))
            return Disposables.create()
        }
        
        if let stoppedEvent = _stoppedEvent {
            observer.on(stoppedEvent)
            return Disposables.create()
        }
        
        
    	//_observers 에 observe insert 한 것
        let key = _observers.insert(observer.on)
        observer.on(.next(_element))
    
        return SubscriptionDisposable(owner: self, key: key)
    }
```

- PublishSubject

```swift
//PublishSubject 
func _synchronized_subscribe<O : ObserverType>(_ observer: O) -> Disposable where O.E == E {
        if let stoppedEvent = _stoppedEvent {
            observer.on(stoppedEvent)
            return Disposables.create()
        }
        
        if _isDisposed {
            observer.on(.error(RxError.disposed(object: self)))
            return Disposables.create()
        }
        
    	//_observers 에 observe insert 한 것
        let key = _observers.insert(observer.on)
        return SubscriptionDisposable(owner: self, key: key)
    }
```

- ReplaySubject

```swift 
//ReplaySubject
func _synchronized_subscribe<O : ObserverType>(_ observer: O) -> Disposable where O.E == E {
        if _isDisposed {
            observer.on(.error(RxError.disposed(object: self)))
            return Disposables.create()
        }
     
        let anyObserver = observer.asObserver()
        
        replayBuffer(anyObserver)
        if let stoppedEvent = _stoppedEvent {
            observer.on(stoppedEvent)
            return Disposables.create()
        }
        else {
            //_observers 에 observe insert 한 것
            let key = _observers.insert(observer.on)
            return SubscriptionDisposable(owner: self, key: key)
        }
    }
```

구현은 조금씩 다르지만 정상적인 상황일때 모두 `let key = _observers.insert(observer.on)` 를 통해 `_observers` 에  `observe`를 추가한 `key`라는 변수를 만들고  `subscription`으로 만들어 리턴한다. 

다시 한번 아래 코드를 보자.

```swift
return Disposables.create(
         self.asObservable().subscribe(observer),
         disposable
   )
```

결국 이러한 과정을 통해 `self.asObservable().subscribe(observer)` 에는 해당 `subject`의 모든 `observers`에 대한 `subscription`이 담기게 되고(`observable`이었을땐 단일 `observer`에 대한 `subscription`) 므로 `.on`이 실행될때 모든 `observer`가 `nextEvent`를  받을 수 있는 환경을 마련한다.



### 마무리 

세상 길어졌따! 한 20시간은 족히쓴거 같은 느낌…? 역시 정리는 함부로 하는게 아니라는걸 깨달았다 ㅎㅠㅠ

![37](https://github.com/sujinnaljin/RxSwift_Theorem_MarkDown/blob/master/%EA%B0%95%EC%88%98%EC%A7%84/images/37.jpg)

제대로 플젝 시작도 안했는데 이렇게 길어질줄 나도 몰랐지! ㅎㅎ 그래도 헷갈리던 개념잡고 조금이나마 정리가 된거 같아 기분이 좋다. 잘아는 사람이 보면 왜 헛소리를 써놨지,,싶을 수 있겠지만,, 틀리더라도 이렇게 이해한걸 정리해놔야 누군가 보고 틀렸다고 알려줄 수 있을테니까! 만약 보인다면 알려주세여! 그리고 저와 비슷한 고민을 안고 있던 분들께 조금이나마 도움이 되었으면 좋겠슴둥! 그럼 20000!

![38](https://github.com/sujinnaljin/RxSwift_Theorem_MarkDown/blob/master/%EA%B0%95%EC%88%98%EC%A7%84/images/38.jpeg)

### 참고 사이트

https://arnoldyoo.tistory.com/18 

https://stackoverflow.com/questions/45141151/what-are-the-differences-between-observable-and-subject-in-rxswift

https://code.i-harness.com/ko-kr/q/25aa1aa

https://medium.com/@luukgruijs/understanding-rxjs-subjects-339428a1815b
