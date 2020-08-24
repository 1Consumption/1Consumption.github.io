---
title: 개린이의 관점에서 Kingfisher 살펴보기(1)
author: 1Consumption
date: 2020-08-25 01:00:00 +0800
categories: [iOS, Library]
tags: [개린이, 살펴보기, 이미지 캐싱]
toc: true

---

> 이 글은 개린이의 지식을 바탕으로 작성된 글입니다. 최대한 옳은 정보를 담으려고 노력하겠으나, 그럼에도 틀린 부분이 있을 수 있습니다. 혹여 발견하시면 댓글로 피드백 주시면 감사하겠습니다.

# 머리말

우리가 사용하는 앱에서는 수많은 이미지를 사용하고, 서버로부터 다운로드받습니다. 그런데 매번 이미지를 사용할 때마다 서버에서 다운로드를 받아야 할까요? 만약 매번 다운로드한다면 서버와 통신하는 비용이 매우 많이 들 것입니다. 이 문제점을 이미지 캐싱을 통해 해결할 수 있는데, 그렇다면 이미지 캐싱 이미지 캐싱이란 무엇일까요?

# 이미지 캐싱

> **캐시**(cache)는 [컴퓨터 과학](https://ko.wikipedia.org/wiki/컴퓨터_과학)에서 데이터나 값을 미리 복사해 놓는 임시 장소를 가리킨다. 캐시는 캐시의 접근 시간에 비해 원래 데이터를 접근하는 시간이 오래 걸리는 경우나 값을 다시 계산하는 시간을 절약하고 싶은 경우에 사용한다. 캐시에 데이터를 미리 복사해 놓으면 계산이나 접근 시간 없이 더 빠른 속도로 데이터에 접근할 수 있다.
>
> 출처: 위키백과 - [캐시](https://ko.wikipedia.org/wiki/캐시)

위는 캐시에 대한 정의인데, 이를 이미지와 접목해본다면 `매번 서버에서 이미지를 받아오기에는 시간이 오래 걸리니 임시적인 장소에 저장해놨다가, 필요할 때 꺼내서 쓰겠다.`라는 개념이 될 것입니다. 즉 로컬 스토리지에 저장해놨다가 필요할 때 불러온다면 서버와 통신하는 비용을 줄일 수 있을 것입니다.

# Kingfisher

`Kingfisher`는 이미지 캐싱을 손쉽게 할 수 있게 해주는 라이브러리입니다. view extension을 기반으로 만들어진 API이며 `UIImageView`, `NSImageView`, `UIButton`, `NSButton`에서 사용 가능합니다. 

<br>

``` swift
let url = URL(string: "https://example.com/image.jpg")
imageView.kf.setImage(with: url)
```

위 코드는 `Kingfisher`에서 제공하는 샘플 코드입니다. 생각보다 정말 간단하네요...? 이 코드는 다음과 같은 6단계로 실행됩니다.

1. `url.absoluteString`을 key로 캐싱 된 이미지가 있는지 확인합니다.
2. 만약 cache(memory나 disk)에서 이미지를 찾으면 `imageView.image`에 설정해줍니다.
3. 그렇지 않다면, request를 생성하고 `url`로 부터 다운로드합니다.
4. 다운로드된 데이터를 `UIImage` 객체로 변환합니다.
5. memory cache에 이미지를 cache하고 disk cache에 데이터를 저장합니다.
6. 이미지를 표시하기 위해 `imageView.image`를 설정하세요.

후에, 같은 `url`로 `setImage`를 호출한다면, cache가 제거되지 않는 한 처음 두 단계만 수행됩니다. 

<br>

-----

이번 포스트에서는 간단하게 이미지 캐싱이란 무엇이고, Kingfisher란 무엇인지에 대해 알아보고 어떤 식으로 이미지를 불러오는지 알아봤습니다.

사실 이번 포스팅에서 Kingfisher의 주요 기능도 알아보려 했으나 내용이 너무 많을 것 같아서 다음 포스팅에서 알아보도록 하겠습니다.ㅎㅎ

# 출처

* [Kingfisher github](https://github.com/onevcat/Kingfisher)

