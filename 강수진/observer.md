## Observer

역시 시작은 공식 문서다. [Observable 도큐멘트](http://reactivex.io/documentation/observable.html) 로 가보면 맨 첫문장에 이렇게 써있다. 

> *ReactiveX에서 observer는 observeable 을 구독한다. observer는 observable이 방출하는 모든 아이템(들)에 대해 반응한다.* 

여기서 나오는 키워드 세가지, 바로 `observer`, `observable` 그리고 `subscribe(구독)` .

자, 자주 쓰이는 일반적인 코드에서 시작해보자.

```swift
observable.subscribe(onNext: { (element) in
        print(element)
  })
```

맞아,, 옵저버블을 이런식으로 구독하지,, 좋아,, `observable`, `subscribe` 두 개 다 찾았다. 그런데.. `observer`은? 이제부터 본격적으로 찾아보자.

![21](https://github.com/sujinnaljin/RxSwift_Theorem_MarkDown/blob/master/%EA%B0%95%EC%88%98%EC%A7%84/images/21.jpg)



위의 `subscribe` 함수는  `ObservableType`의 `extension` 에 이런식으로 정의 되어있다.

cf) `ObservableType` 은 `protocol` 로  `Observable`은 이를 따른다.

```swift
public func subscribe(onNext: ((E) -> Void)? = nil, onError: ((Swift.Error) -> Void)? = nil, onCompleted: (() -> Void)? = nil, onDisposed: (() -> Void)? = nil)
        -> Disposable
```

구현부를 뜯어보자.

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
            
            //1) 안에서 자체적으로 observer 생성
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
                //2) 'observer'에 대한 subscription. 이렇게 해서 해당 옵저버블에 대해 옵저버가 관찰을 할 수 있다.
                self.asObservable().subscribe(observer),
                disposable
            )
    }
```



여기서 주목해야할 부분은 주석으로 표시했듯 두가지가 있다. 1번부터  보도록하자.

1) 안에서 자체적으로 `observer`를 생성.

![22](https://github.com/sujinnaljin/RxSwift_Theorem_MarkDown/blob/master/%EA%B0%95%EC%88%98%EC%A7%84/images/22.jpeg)

​										찾았다 옵저버!

대체 어디서 만들어질까.. 찾아헤메던 `observer`는  `observable`을 `subscribe`하게 되면 `subscribe` 구현부에서 각각 생성되고 있었다.

이제 보자 2번.

2) observer'에 대한 subscription

```swift
  return Disposables.create(
       self.asObservable().subscribe(observer),
       disposable
  )
```

 이 코드가 의미하는것은 무엇일까.

우선 `asObservable`은 `ObservableType`의 `extension`에 정의되어 있다.

```swift 
extension ObservableType {
    
    /// Default implementation of converting `ObservableType` to `Observable`.
    public func asObservable() -> Observable<E> {
        // temporary workaround
        //return Observable.create(subscribe: self.subscribe)
        return Observable.create { o in
            return self.subscribe(o)
        }
    }
}
```

한마디로 `ObservableType`을 `Observable`로 변환해준다는 거다.

그럼 그 뒤에 붙는 `subscribe(observer)` 는? 이것도 `Observable`이 따르고 있는 `protocol`인 `ObservableType`에서 해답을 찾을 수 있다.

그 기본적으로 형태를 보면 이러하다. 

```swift 
public protocol ObservableType : ObservableConvertibleType {
   
    func subscribe<O: ObserverType>(_ observer: O) -> Disposable where O.E == E
}
```

안에 `subscribe` 라는 함수를 정의해 놓은 형태이다. 해당 subscribe에 대한 설명을 보면 아래와 같다.

```**이 시퀀스에서의 이벤트들을 받기 위해 observer를 구독한다.**
**문법**

* 시퀀스는 0개 혹은 그 이상의 요소들을 생산할 수 있고 이러한 'Next' 이벤트들이 observer로 전달될 수 있다.
* `Error`나 `Completed` 이벤트가 발생했을땐 해당 시퀀스는 종료되고 아무런 요소들도 생산하지 않는다.

이벤트들이 각기 다른 스레드에서 전달되어 올 수는 있지만, 두개의 이벤트가 동시에 observer로 전달될 수는 없다.

**리소스 관리**

* 시퀀스가 `Complete` 나 `Error` 이벤트를 보내면 해당 시퀀스의 요소를 다루었던 모든 내부적 자원은 해제된다. 
* 시퀀스의 요소 방출을 막고 자원을 즉각 해제하기 위해서 반환되는 subscription에서 dispose를 호출한다.
  

**반환 값**

시퀀스가 요소들을 생산하는 것을 취소하고, 자원을 해제하는데 쓰일 수 있는 'observer'에 대한 subscription.
```

어쩌다 다 해석하게 됐는데 다 무시하고 반환 값만 보자. 그러니까 `observer`를 받아서 그에 대한 `subscription`을 반환한다는 거다. 그리고 이는 자원 해제를 할 수 있는 형태(`Disposable`)라고 써있다.

다시 돌아와보자. 

```swift
  return Disposables.create(
       //observer에 대한 subscription. 타입은 Disposable(protocol).
       self.asObservable().subscribe(observer),
       disposable
  )
```

그러니까 `self.asObservable().subscribe(observer)` 은 **`observable.subscribe`할때 마다 각각 만들어지는 `observer`들에 대한 `Disposable` 타입의 `subscription`**이라는 거다. 

서론에서 이런 말을 한 적이 있다.

> 실제 코드를 보면
>
> ```swift
> observable.subscribe(onNext: { (element) in
>         print(element)
>   })
> ```
>
> 이런식으로 쓰게 되는데, 이렇게 하면 `observer`가 `observable`을 관찰 할 수 있다니!!!!

이게 가능한 이유가 `observable`을 `subscribe`하게 되면 `subscribe` 구현부에서 각각의 `observer`를 생성하고 그 `observer`에 대한 `subscription`을 만들기 때문이었다.
