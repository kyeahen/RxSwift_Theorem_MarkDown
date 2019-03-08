## 들어가기 전에

### 주의

이 글은 RxSwift를 처음부터 하나하나 친절하게 설명하는 글이 아닙니다(물론 내가 처음 시작하는건 맞음 ㅎ). 단지 공부하면서 든 의문들을 이해한 나름대로 정리한 글입니다. RxSwift의 기본적인 개념을 하나하나 원하신다면 제목에 속지 말고 돌아가세요..! 빠른 손절! 잘가요..!



####글을 쓰게 된 이유

RxSwift 를 공부하고 있다. 맞다. 사진의 얘.
![main](https://github.com/sujinnaljin/RxSwift_Theorem_MarkDown/blob/master/%EA%B0%95%EC%88%98%EC%A7%84/images/main.png)

​									근데 얜 뭘까..? 도롱뇽인가..?

RxSwift를 공부해본 사람, 아니 ReactiveX에 관심이 있다면 알겠지만 시작은 평범하게 달콤하게 **observable**과 **observer** 그리고 **subject**로 시작한다.

그러니까 이 세 개 (observable, observer, subject)가 RxSwift 를 시작하는데 필요한 기본 개념들이란 말이지…🤔 

![11](https://github.com/sujinnaljin/RxSwift_Theorem_MarkDown/blob/master/%EA%B0%95%EC%88%98%EC%A7%84/images/11.png)

​									구글 항상 고맙고 사랑해..



> **observable** 은 **구독 가능한 것**, **observer**는 **관찰자**, **subject**는 **observable이자 observer** (실제로 구글링하면 이렇게 가장 많이 나왔다)



​											**예???????????**

![12](https://github.com/sujinnaljin/RxSwift_Theorem_MarkDown/blob/master/%EA%B0%95%EC%88%98%EC%A7%84/images/12.png)

​						마음같아선 피눈물로 하고 싶었는데 너무 무서워서 수정했다

맞다. 보다시피 이 글은 기본 개념부터 이해가 안돼서 슬픈 개발자의 울면서 쓰는 RxSwift 이야기이다.



####무엇을 해결하고자 하는가

이 세 가지 개념을 들으면서 많고 많은 것들이 이해가 안됐지만, 그 중 가장 혼란스러웠던 것은 크게 두가지가 있다. 

![13](https://github.com/sujinnaljin/RxSwift_Theorem_MarkDown/blob/master/%EA%B0%95%EC%88%98%EC%A7%84/images/13.jpg)



이를 의식의 흐름대로 쓰자면 아래와 같다.

첫째, **observer**에 대하여.

좋아, 먼저 나오는 `observer`와 `observable`에 대해 생각해보자. `observable` 은 **관찰 가능한 것**이고 `observer`은 **관찰자**란 말이지,, 흠,,, 

`observable` 에 대해선 그래도 설명이 괜찮게 나와있는 편이다. 

> `observable`은 어떠한 이벤트를 계속 방출하고 우리는 이걸  관찰 할 수 있다

이 정도 이해하면 된다.

문제는 `observer`다. 도대체가 *`observer`는 관찰자* 라고만 하고 이거에 대해서 자세히 설명한걸 본적이 없다. 

![14](https://github.com/sujinnaljin/RxSwift_Theorem_MarkDown/blob/master/%EA%B0%95%EC%88%98%EC%A7%84/images/14.png)

​									거의 뭐 직독직해 수준;;

기껏 찾아봐도 `observer`가 `observable`을 관찰하고 `observable`은 `observer`에 이벤트를 알려준다는데… 보면서 이건 그냥 `observer`와 `observable` 의 영어 뜻풀인가..? 싶었다. 관찰자가 관찰가능한걸..! 관찰하겠지…!!

![15](https://github.com/sujinnaljin/RxSwift_Theorem_MarkDown/blob/master/%EA%B0%95%EC%88%98%EC%A7%84/images/15.jpg)



실제 코드를 보면

```swift 
observable.subscribe(onNext: { (element) in
        print(element)
  })
```

이런식으로 쓰게 되는데, 이렇게 하면 `observer`가 `observable`을 관찰 할 수 있다니!!!!

![16](https://github.com/sujinnaljin/RxSwift_Theorem_MarkDown/blob/master/%EA%B0%95%EC%88%98%EC%A7%84/images/16.JPG)

​									이게 무슨소리요 알엑스양반,,

~~내 거친 observable과 불안한 subscribe와 그걸 지켜보는 observer…~~ 내 눈에는 `observable`이랑 그걸 `subscribe`하는거 밖에 안나오는데 대체 뭔 `observer`가 있어서 관찰을 한다는거야!! 

관찰자면 예의상 나타나서 

![17](https://github.com/sujinnaljin/RxSwift_Theorem_MarkDown/blob/master/%EA%B0%95%EC%88%98%EC%A7%84/images/17.png)

​							누끼 왜케 열심히땃지..?

이런 과정이라도 있어야 하는거 아닌가?!!!!

`observer`가 관찰자라 지켜본다고 해놓고서 그놈의 `observer` 대체 어딨는데!! 쾅쾅!! 혹시,, 도시괴담인가,,?

대체 넌 어디있는거니,,,존재하긴 하는거니,,**옵저버를 찾아서,,떠나야겠다,,!!!!!!!!!!**

![18](https://github.com/sujinnaljin/RxSwift_Theorem_MarkDown/blob/master/%EA%B0%95%EC%88%98%EC%A7%84/images/18.gif)

​									전 옵저버를 찾아 떠납니다\~~~



둘째, **subject**에 대하여.

먼저 `observable`을 `create`할때 보면


```swift 
Observable<String>.create{ observer in
        observer.onNext("sujinnaljin")
        return Disposables.create()
}
```

이런식으로 escaping 클로저로 `AnyObserver<T>` 를 인자로 받아서 이 `observer`에 전달될 이벤트를  `observer.onNext("sujinnaljin")` 와 같이 써줄 수 있다. 

그런데  `subject`에서는 이런 코드를 쓸 수 있다.

```swift 
let subject = BehaviorSubject(value: "")
subject.onNext("sujinnaljin")
```

엥 뭔가 비슷하게 `.onNext`가 있네? ㅎㅎ

`구독(subscribe)` 하는 것도 보자.

```swift 
//observable에 대한 구독 
observable.subscribe(onNext: { (element) in
        print(element)
  })
  
//subject에 대한 구독
subject.subscribe(onNext: { (element) in
        print(element)
  })
```

오 똑같은데? 그럼 `observable`이랑 `subject`랑 같은건가?ㅎㅎ

![19](https://github.com/sujinnaljin/RxSwift_Theorem_MarkDown/blob/master/%EA%B0%95%EC%88%98%EC%A7%84/images/19.jpg)

​											과거의 나에게...



일단 위에 나와있던 `subject`에 대한 설명을 다시 생각해보자.

> observable 이자 observer

오키,, 일단,, `observable` 받고,, `observer`가 다시 나왔네,,,ㅎㅎ

**`observer`를 일단 확실히해야  `observable`이자 `observer`라는 `subject`를 이해할 수 있겠다. `observer` 공부한 다음에 `subject`가 `observer`이기 때문에 생기는 특성을 알아봐야 겠다..쉬익 쉬익.. 그게 바로 서브젝트와 옵저버블의 차이가 되겠지..**



------



이제 시작해보자. 원래 한 포스트에 때려넣으려고 했는데 서론부터 글이 너모너모 길어졌따. 다음 포스트에 나눠서 시작하겠다.

내 맘대로 이해하는 RxSwift!! 맘대로 이해한거니까 모든 정보는 걸러서 보는걸 추천!ㅎㅎ 잘못된 부분 알려주시면 압도적 감사! 그럼 RxSwift의 대환장 파티 속으로 고고링\~~\~~\~~\~~\~!

![1A](https://github.com/sujinnaljin/RxSwift_Theorem_MarkDown/blob/master/%EA%B0%95%EC%88%98%EC%A7%84/images/1A.jpeg)





해당 자료는 **미디엄**에서도 확인할 수 있습니다 - [[RxSwift]들어가기전에](https://medium.com/@rkdthd0403/rxswift-%EC%8B%9C%EC%9E%91-497dfada1e22)

