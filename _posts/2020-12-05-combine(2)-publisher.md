---
title: 개린이의 관점에서 Combine 살펴보기(2) - Publisher
author: 1Consumption
date: 2020-12-05 01:00:00 +0800
categories: [iOS, Framework]
tags: [살펴보기, 둘러보기, 톺아보기, 주요 기능, Combine, 사용법, SwiftUI]
toc: true


---

# 머리말

저번 포스팅에서 `Combine`의 3가지 키 컨셉에 대해서 알아 봤었죠. 그 중에 하나인 `Publisher`를 이번 포스팅에서 알아보려고 합니다.

# Publisher

## Publisher란?

`Publisher`는 `Subscriber`에게 하나 이상의 값을 전달하는 인스턴스 입니다. `Subscriber` 의 Input, Failure는 `Publisher`의 Output, Failure와 일치해야합니다. ~~난 노래 유튜버를 구독했는데 갑자기 산악등반 컨텐츠를 하면 당황스러우니께...~~ 

그리고 `receive(subscriber:)`를 구현해야합니다. 이 메소드는 매개변수로 넘어온 subscriber와 publisher를 연결해주는 메소드입니다. 당연한 얘기지만 연결이 된 후에 publisher가 방출하는 결과를 받을 수 있습니다!! 그런데 똑같은 역할을 하는 `subscribe(_:)` 메소드가 또 있습니다. 머여 장난하는거여......?

<br>

> **subscribe(_:)**
>
> Always call this function instead of `receive(subscriber:)`. Adopters of `Publisher` must implement `receive(subscriber:)`. The implementation of `subscribe(_:)` provided by `Publisher` calls through to `receive(subscriber:)`.

<br>

아하~ `receive(subscriber:)`는 Publisher를 채택한 친구가 구현을 해야하는거고, 그 메소드를 부르고 싶으면 `subscribe(_:)` 메소드를 부르라고 하네요. 

왜 굳이 이렇게 해놨을까 모르겠네요ㅠ 주어의 차이일까요? publisher의 입장에서 보면 subscriber를 받는(receive)것이니까 내부에서 실행되는 것이고, subscriber의 입장에서 보면 publisher를 구독(subscribe) 하니까 외부에서 부르는 메소드로 사용이 되는 것 같기도하고... 이 부분은 잘 모르겠습니다ㅠ

<br>

아무튼~ `receive(subscriber:)` 메소드를 통해 subscriber를 받으면 `publisher`는 `subscriber`의 3가지 메소드를 호출할 수 있습니다. 간단하게 알아볼게요.

1. `receive(subscription:)`: subscribe가 되었음을 확인하고, 매개변수로 넘어온 `Subscription` 인스턴스를 통해 한번에 몇개의 아이템을 받을지 여부(demand)를 정할 수 있고, 구독을 취소할 수 있습니다.
2. `receive(_:)`: publisher로 부터 하나의 input을 받습니다. 반환 값을 조정해서 subscriber가 받을 것으로 예상되는 요소의 수를 변경할 수 있습니다.
3. `receive(completion:)`: subscriber에게 publishing이 정상적으로 종료되었거나 오류가 발생해서 종료되었음을 알립니다.

<br>

## 그래서 어떻게 만드는데?

애플에서는 직접 `Publisher` 프로토콜을 채택해서 만드는 방법보다 `Combine`에서 제공하는 3가지 방법을 권장합니다.

1. `Subject`을 채택한 클래스 사용
2.  Convenience Publishers 사용
3. 프로퍼티에 `@Published` 어노테이션 추가

이번 포스팅에서는 첫번째 방법에 대해 알아보도록 하겠습니다.

`Subject`는 외부에서 값을 주입할 수 있는 `Publisher` 프로토콜 입니다. `send(_:)` 메소드를 통해 스트림에 값을 주입할 수 있습니다. 이 `Subject` 프로토콜을 채택한 구현체인  `PassthroughSubject`, `CurrentValueSubject`이 기본 제공 됩니다.

 `PassthroughSubject`와 `CurrentValueSubject`의 차이점은 초기값이 있냐 없냐, 그리고 직전에 방출한 값을 저장을 하냐 안하냐로 나뉩니다. `CurrentValueSubject`은 초기값이 있고,  직전에 방출한 값을 저장하는 반면,  `PassthroughSubject`는 초기값이 없고 직전에 방출한 값을 저장하지 않습니다. 아래와 같이 선언하고, `send(_:)` 메소드를 통해 값을 주입할 수 있습니다. 

``` swift
let passthroughSubject = PassthroughSubject<Int, Never>()
let currentValueSubject = CurrentValueSubject<Int, Never>(1) // value 1

passthroughSubject.send(2) // value 2
passthroughSubject.send(3) // value 3
passthroughSubject.send(.finished) // publish finished

print(currentValueSubject.value) // 1
currentValueSubject.send(2) // value 2
print(currentValueSubject.value) // 2
currentValueSubject.send(3) // value 3
print(currentValueSubject.value) // 3
currentValueSubject.send(.finished) // publish finished
```

<img src = "https://user-images.githubusercontent.com/37682858/101308298-e5fbe680-388c-11eb-8dba-afc31ee5c511.gif" width = 900>

<br>

------------------------------------------

<br>

이번 포스팅에서는 `Publisher`에 대해서 전반적으로 알아봤습니다. `Publisher`에서 방출한 값을 어떻게 사용하는지까지 포스팅하고 싶었지만, 이건 `Subscriber`에 대해서 더 자세히 알아야 설명할 수 있기 때문에 다음 포스팅에서  `Subscriber`를 알아보고 마저 포스팅 하겠습니다ㅎㅎ

> 이 글은 개린이의 지식을 바탕으로 작성된 글입니다. 최대한 옳은 정보를 담으려고 노력하겠으나, 그럼에도 틀린 부분이 있을 수 있습니다. 혹여 발견하시면 댓글로 피드백 주시면 감사하겠습니다.

# 출처

* Apple Developer Documentation 
  * [Publisher](https://developer.apple.com/documentation/combine/publisher)
  * [Subject](https://developer.apple.com/documentation/combine/subject)
    * [PassthroughSubject](https://developer.apple.com/documentation/combine/passthroughsubject)
    * [CurrentValueSubject](https://developer.apple.com/documentation/combine/currentvaluesubject)