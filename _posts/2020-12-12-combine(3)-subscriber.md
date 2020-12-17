---
title: 개린이의 관점에서 Combine 살펴보기(3) - Subscriber
author: 1Consumption
date: 2020-12-12 08:00:00 +0800
categories: [iOS, Framework]
tags: [살펴보기, 둘러보기, 톺아보기, 주요 기능, Combine, 사용법, SwiftUI]
toc: true


---

# 머리말

저번 포스팅에서 `Publisher` 에 대해 알아봤습니다. 그러면 `Subscriber` 에 대해서도 알아봐야겠죠?ㅎㅎ 왜냐하면 `Publisher`가 방출한 값을 받는 애가 `Subscriber`니까ㅎ ~~아 스포~~

# Subscriber

## Subscriber란?

Publisher로부터 Input을 받을 수 있는 타입을 선언하는 프로토콜입니다. 이전 포스팅에서 말했던 것처럼 Subscriber 의 Input, Failure는 Publisher의 Output, Failure와 일치해야합니다. 그리고 Subscriber 인스턴스는 Publisher로 부터 데이터의 흐름을 받습니다.

Subscriber 가 Publisher의 데이터를 받는 것 또한 [combine(2) - publisher](https://1consumption.github.io/posts/combine(2)-publisher/)에서 설명했었군요. 간단히 다시 알아보겠습니다.

Publisher의 `subscribe(_:)` 메소드를 호출해서 Publisher와 Subscriber를 연결하고, 이 메소드가 불리고 난 후 publisher는 subscriber의 `receive(subscription:)`메소드를 통해 `Subscription` 인스턴스를 받아 publisher로부터 데이터를 요구할 수 있고, 구독을 취소할 수도 있습니다. 

그다음 publisher는 subscriber의 `receive(_:)`메소드를 비동기로 호출하고 새로 발행된 데이터를 전달합니다.

만약 publisher가 발행을 멈추면, subscriber의 `receive(completion:)`메소드를 호출하고 `Subscribers.Completion` 타입의 매개변수를 통해 정상적으로 종료되었는지, 오류가 나서 종료되었는지를 나타냅니다. 아래 그림은 publisher와 subscriber의 관계를 도식화 한 것입니다.



<img width="406" alt="image" src="https://user-images.githubusercontent.com/37682858/101635118-e652d800-3a6c-11eb-9543-a87d5da23ab7.png">

<br>

## 그래서 어떻게 만드는데?

친절하게도 애플에서는 이미 Subscriber를 제공하고 있습니다! 바로 `Publisher`타입의 오퍼레이터를 통해서요. 두 가지 오퍼레이터가 있는데 `sink(receiveCompletion:receiveValue:)`와 `assign(to:on:)` 입니다. 이 두 오퍼레이터는 각각 `Subscribers.Sink`, `Subscribers.Assign` 타입의 인스턴스를 생성합니다. 그럼 각 오퍼레이터에 대해 알아볼까요?  

<br>

### 1. `sink(receiveCompletion:receiveValue:)`

closure 기반 동작으로 subscriber를 연결하는 오퍼레이터입니다.

 `receiveCompletion` 매개변수는 completion 되었을 때 실행되는 클로저입니다. 이 클로저는 매개변수로 `Subscribers.Completion` 타입을 가지고, 정상적으로 종료된 `finished`, 에러가 발생해 종료된 `failure(Failure)` 를 내부 case로 가집니다.

`receiveValue` 매개변수는 pubisher로 부터 Output을 받을 때마다 실행되는 클로저입니다.

그리고!  `AnyCancellable` 타입 인스턴스를 반환합니다. 이를 통해 subscription(stream 이죠)을 취소할 수 있습니다.

<br>

### 2. `assign(to:on:)` 

Publisher의 Output을 특정 인스턴스의 key path에 쓸 수 있게 해주는 오퍼레이터입니다. 이 오퍼레이터는 publisher의 Failure가 Never일 때만 사용 가능합니다.

`to` 매개변수에는 publisher로부터 받은 Output이 할당될 key path가 들어가고, `on` 매개변수에는 `to` 매개변수에 할당된 key path에 해당하는 프로퍼티를 포함하는 객체입니다. Subscriber는 새로운 값을 받을 때 해당 프로퍼티에 받은 값을 할당합니다. 또한 `AnyCancellable` 타입 인스턴스를 반환합니다.

이 오퍼레이터에 의해 생성된 `Subscribers.Assign` 인스턴스는 `on` 매개변수에 할당된 객체에 강한 참조를 유지합니다. 그리고, upstream에서 publish를 완료하면 참조를 끊습니다. 흠... 그러니까 이 subscriber가 살아 있다면, `on` 매개변수에 할당된 객체는 메모리에서 해제되지 않는다는 소리인가? 이 부분은 잘 모르겠네요ㅎㅎㅎㅎㅎㅎㅎ

<br>

## Publisher를 subscribe 해보기

```swift
let passthroughSubject = PassthroughSubject<Int, Never>()
let subscriber = passthroughSubject.sink { state in
    print(state)
} receiveValue: { output in
    print(output)
}

passthroughSubject.send(1)
passthroughSubject.send(2)
passthroughSubject.send(3)
passthroughSubject.send(4)
passthroughSubject.send(completion: .finished)
passthroughSubject.send(5)
passthroughSubject.send(6)

// 1
// 2
// 3
// 4
// finished
```

<br>

전 포스팅에서 써봤던 `PassthroughSubject`를 사용해서 publisher를 만들고, `sink(receiveCompletion:receiveValue:)` 오퍼레이터를 통해 subscriber를 만들어 줬습니다.  되게 간단하고 이해하기 쉬운 코드죠? 

1. publisher를 만든다.
2. publisher를 subscriber가 구독한다.
3. publisher가 값을 방출하면 subscriber가 받아서 동작을 취한다.
4. publisher가 finished 되면 정상적으로 종료되었는지가 receiveComplete의 매개변수를 통해 제공된다.
5. publisher가 finished 된 이후에는 값을 방출하지 않는다.

이렇게 간단하게 사용할 수 있습니다!

<br>

------------------------------------------

<br>

이번 포스팅에서는 `Subscriber`에 대해서 알아보고, 어떻게 `Publisher`를 구독하는지 간단하게 실습 해봤습니다ㅎㅎ 재미있군여... 이 combine은.... 뭔가 알 듯 말 듯하지만 편하다는 것만은 알겠네요ㅎㅎㅎㅎㅎㅎ

다음 포스팅에서는 `Publisher`가 방출한 값을 잘 가공해서 `Subscriber`에게 넘겨줄 수 있는 `Operator`에 대해서 알아보도록 하겠습니다!!!

> 이 글은 개린이의 지식을 바탕으로 작성된 글입니다. 최대한 옳은 정보를 담으려고 노력하겠으나, 그럼에도 틀린 부분이 있을 수 있습니다. 혹여 발견하시면 댓글로 피드백 주시면 감사하겠습니다.

# 출처

* [WWDC 19 Introducing Combine](https://developer.apple.com/videos/play/wwdc2019/722/)
* Apple Developer Documentation 
  * [Subscriber](https://developer.apple.com/documentation/combine/subscriber)
  * [sink(receiveCompletion:receiveValue:)](https://developer.apple.com/documentation/combine/publisher/sink(receivecompletion:receivevalue:))
  * [assign(to:on:)](https://developer.apple.com/documentation/combine/publisher/assign(to:on:))
  * [Processing Published Elements with Subscribers](https://developer.apple.com/documentation/combine/processing-published-elements-with-subscribers)

