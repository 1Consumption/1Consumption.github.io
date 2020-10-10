---
title: 개린이의 관점에서 Kingfisher 살펴보기(3) - Kingfisher의 주요 기능
author: 1Consumption
date: 2020-09-09 21:00:00 +0800
categories: [iOS, Library]
tags: [살펴보기, 둘러보기, 톺아보기, 주요 기능, 이미지 캐싱, 킹피셔, Kingfisher, 사용법]
toc: true


---

> 이 글은 개린이의 지식을 바탕으로 작성된 글입니다. 최대한 옳은 정보를 담으려고 노력하겠으나, 그럼에도 틀린 부분이 있을 수 있습니다. 혹여 발견하시면 댓글로 피드백 주시면 감사하겠습니다.

> 개린이의 관점에서 Kingfisher 살펴보기 시리즈
>
> 1. [개린이의 관점에서 Kingfisher 살펴보기(1) - 이미지 캐싱이란?](https://1consumption.github.io/posts/about-kingfisher(1))
> 2. [개린이의 관점에서 Kingfisher 살펴보기(2) - Kingfisher의 주요 기능](https://1consumption.github.io/posts/about-kingfisher(2))

# 머리말

안녕하세요ㅎ 오랜만에 글을 씁니다. 요새 개강도 했고, 따로 프로젝트도 하고 있어서 미루고 미루다가 다시 포스팅 합니다ㅎ 

아무튼! 이번 포스팅에서는 저번에 말했던 것처럼 아래의 기능에 대해 알아보도록 하겠습니다

* Multiple-layer hybrid cache for both memory and disk.
* Fine control on cache behavior. Customizable expiration date and size limit.
* Cancelable downloading and auto-reusing previous downloaded content to improve performance.
* Independent components. Use the downloader, caching system and image processors separately as you need.

# Kingfisher 주요 기능

## 1. Multiple-layer hybrid cache for both memory and disk.

Memory와 disk를 위한 다중 계층의 hybrid cache를 지원한다는 것 같네요? 이는 memory와 disk의 단점을 보완하기 위함이라고 생각 됩니다. 아래는 memory와 disk의 장단점을 분석한 표입니다.

|                             | Memory         | Disk     |
| :-------------------------- | -------------- | -------- |
| 속도                        | 빠르다.        | 느리다.  |
| 앱 종료 시 데이터 저장 여부 | 저장되지 않음. | 저장 됨. |
| 가격                        | 비싸다.        | 싸다.    |

따라서 서로의 단점을 보완하기 위해 memory와 disk에 캐싱하는 것입니다. 

하지만!! memory에만 캐싱하는 경우도 있습니다. 음... 정확히 어떤 경우인지 예시를 딱 들지는 못하겠습니다ㅎㅎ 제가 저런 상황에 처해보지 않아서요... 그래서 대략적인 상황을 생각해보자면, `굳이 disk에 저장하지 않아도 되며, 앱이 실행될 때마다 바뀌는 이미지들을 서버에서 받아 올 때 memory에만 캐싱하지 않을까 싶습니다...(이 부분은 잘 모르겠습니다ㅠㅠ)`

어떻게 하면 `Kingfisher`를 사용해 memory에만 캐싱할까요? 바로바로.... `KingfisherParsedOptionsInfo`의 `cacheMemoryOnly` 옵션을 활성화해서 memory에만 캐싱할 수 있습니다. `cacheMemoryOnly`의 기본 값은 `false` 입니다. 즉 해당 옵션을 활성화 하지 않으면 memory와 disk에 캐싱을 합니다.

``` swift
imageView.kf.setImage(with: URL, options: [.cacheMemoryOnly])
```

<br>

이를 설명하기 위해서는 `Kingfisher`가 어떻게 이미지를 저장하는지를 알아야 하는데요, 아래 다이어그램을 봐주세요.

<img src = "https://user-images.githubusercontent.com/37682858/92345050-fc19cf80-f102-11ea-9609-b4189c0ff58f.png" width = 600>

먼저 다이어그램에 대해 설명을 하자면 `KingfisherManager`는 `ImageDownloader`와 `ImageCache`를 연결합니다. 이 클래스를 사용하여 웹 또는 cache에서 지정된 URL을 통해 이미지를 검색 할 수 있습니다. 그리고 `ImageCache`는 `MemoryStorage.Backend`와`DiskStorage.Backend`로 구성된 하이브리드 캐싱 시스템입니다. 또한 이미지와 데이터를 메모리 및 디스크에 저장하고 다시 검색할 수 있습니다.

> 아니 그래서 이걸 왜 알려주나요?
>
> 바로 계층화가 되어있기 때문이죠! 사실 우리가 사용하는 `setImage(with:)` 메소드는 `KingfisherWrapper`의 메소드입니다. 하지만 `setImage(with:)`메소드만 실행되는 것이 아닙니다.
>
> `KingfisherWrapper`의 `setImage(with:)` 메소드 내부에는 `KingfisherManager`의 `retrieveImage(:with:options)` 메소드가 불리고, `KingfisherManager`의 `retrieveImage(:with:options)` 메소드 내부에서는 `ImageCahce`의 `store(_:)` 메소드가 불리는 것입니다.

<br>

네 그러면 실질적으로 캐싱을 하는 클래스인 `ImageCache` 내부의 `store(_:original:forKey:options:toDisk:completionHandler:)` 메소드를 살펴보겠습니다. 여기서 주목해야 할 매개변수는 `toDisk`입니다. 이 값은 `Bool` 타입이고, `KingfisherParsedOptionsInfo`의 `cacheMemoryOnly`에 따라 정해집니다. 

``` swift
// 메모리 캐싱 코드 들어가는 부분
        
guard toDisk else {
  	if let completionHandler = completionHandler {
      	let result = CacheStoreResult(memoryCacheResult: .success(()), diskCacheResult: .success(()))
      	callbackQueue.execute { completionHandler(result) }
    }
  	return
}
        
// 디스크 캐싱 코드 들어가는 부분
```

위 코드는 해당 메소드의 일부인데요, 대략적인 순서는 `메모리 캐싱 -> 디스크 캐싱 여부 확인 -> 디스크 캐싱` 순서입니다. 여기서 `디스크 캐싱 여부 확인` 부분은 `toDisk`라는 변수를 이용합니다. 이렇게 해서 메모리에만 캐싱할지, 디스크에도 캐싱할지 아는 것입니다!!!!

> 근데 `KingfisherParsedOptionsInfo`의 `cacheMemoryOnly` 의 기본값은 `false`라면서요. 그럼 해당 옵션을 활성화하면 `true`로 바뀌잖아요? 그러면 위 `guard`문을 통과하는 거 아닌가요? 그래서 Disk에도 캐싱이 될 것 같은데 그럼 의도한 바가 아니지 않나요?
>
> 아닙니다. `KingfisherManager` 클래스에서 `ImageCache`의 메소드를 호출할 때, `!options.cacheMemoryOnly` 이렇게 반전을 해서 매개변수에 전달합니다. 그래서 해당 옵션이 활성화되어 있다면, `toDisk`의 값은 `false`가 되어서 디스크캐싱이 안됩니다. 아마도 통일성을 위해 이렇게 한 것 같은데.... 좀 헷갈리게 해놨네유ㅎ



## 2. Fine control on cache behavior. Customizable expiration date and size limit.

Cache의 만료일, 크기를 제한할 수 있습니다. 왜 cache의 만료일, 크기를 제한해야 할까요? `크기를 제한하는 이유`는 우리가 쓸 수 있는 `자원(ram, disk)`이 한정적이기 때문입니다. 그리고 `만료일을 제한하는 이유`는 cache에는 유효한 값이 있어야 하기 때문에 일정 시간이 지났다면 유효하지 않은 값으로 판단하여(수정되었을 수도 있으니까) 지우고, `최신값`을 다시 받아오기 위함입니다.  그렇다면 한번 적용해볼까요?ㅎㅎ

먼저 `ImageCache` 인스턴스를 생성해 줍니다. 그러면 메모리 저장소를 의미하는 `memoryStorage` 프로퍼티와, `diskStorage` 프로퍼티에 접근할 수 있어요.

```swift
let cache = ImageCache(name: "myCache")
cache.memoryStorage // cache 인스턴스의 메모리 저장소에 접근할 수 있는 프로퍼티
cache.diskStorage // cache 인스턴스의 디스크 저장소에 접근할 수 있는 프로퍼티
```

그리고  `cache.memoryStorage`와 `cache.diskStorage`의 `Config` 타입의 프로퍼티 `config`를 통해 만료일, 크기 등을 제한할 수 있습니다. 

```swift
// memoryStorage 

// 저장소의 최대 용량(byte), 기본값은 프로세스가 할당받은 전체 메모리 / 4
cache.memoryStorage.config.totalCostLimit
// 만료된 item들을 지우는 주기(초), 상수이며, init 될 때 120으로 초기화 됨
cache.memoryStorage.config.cleanInterval 
// 저장소에 캐싱된 item들의 만료일, 기본값은 300초
cache.memoryStorage.config.expiration 
// 메모리에 캐싱할 수 있는 item의 수를 제한, 기본값은 Int tpye의 max
cache.memoryStorage.config.countLimit 

// diskStorage

// 디스크 저장소의 파일 크기 제한(byte), 0은 제한이 없음을 의미함. 기본값은 0
cache.diskStorage.config.sizeLimit 
// 저장소에 캐싱된 item들의 만료일, 기본값은 300초
cache.diskStorage.config.expiration 
// 캐싱된 item의 파일 확장자. 기본값은 nil, 캐시 파일에 파일 확장자가 없음을 의미
cache.diskStorage.config.pathExtension 
// 저장하기 전에 캐시 파일 이름이 해시됨을 의미. 기본값은 true
cache.diskStorage.config.usesHashedFileName 
```



다음은 [cheat sheet](https://github.com/onevcat/Kingfisher/wiki/Cheat-Sheet)에 있는 예시입니다.

``` swift
// memory cache의 크기를 300MB로 제한함.
cache.memoryStorage.config.totalCostLimit = 300 * 1024 * 1024

// memory cache에 최대 150개의 이미지가 있도록 제한함.
cache.memoryStorage.config.countLimit = 150

// disk cache의 크기를 1GB로 제한함.
cache.diskStorage.config.sizeLimit =  = 1000 * 1024 * 1024

// memory cache에 있는 이미지는 10분 뒤에 만료됨.
cache.memoryStorage.config.expiration = .seconds(600)

// disk cache에 있는 이미지는 만료되지 않음.
cache.diskStorage.config.expiration = .never

// 특정 이미지를 캐싱 할 때 다음 option을 사용하여 만료일을 재정의 할 수 있음.
// 해당 이미지는 memeory cache에서 만료되지 않음.
imageView.kf.setImage(with: url, options: [.memoryCacheExpiration(.never)])

// memory cache의 만료된 이미지들을 30초마다 확인하고, 삭제함.
cache.memoryStorage.config.cleanInterval = 30
```



## 3. Cancelable downloading and auto-reusing previous downloaded content to improve performance.

이미지를 다운로드하는 중에 취소할 수 있고, 이전에 다운로드되었던 컨텐츠를 자동으로 재사용해서 성능을 향상해준다고 합니다. 먼저 `이미지를 다운로드하는 중에 취소할 수 있다.` 부터 보겠습니다.

[이전 포스팅](https://1consumption.github.io/posts/about-kingfisher(2))에서 `Resource` protocol을 준수하는 객체를 `setImage(with:)` 메소드의 매개변수로 넘기면, `Source`로 변환해주고, 패턴 매칭을 통해 네트워크에서 이미지를 다운받거나 로컬에서 데이터를 가져온다고 했었습니다! 아래는 `Source` 타입의 변수 `source`로 패턴 매칭하는 `KingfisherManager` 내부 코드입니다.

``` swift
switch source {
case .network(let resource):
		let downloader = options.downloader ?? self.downloader
  	let task = downloader.downloadImage(
      	with: resource.downloadURL, options: options, completionHandler: _cacheImage
		)
  
  	if let task = task {
      	return .download(task)
    } else {
      	return nil
    }

case .provider(let provider):
  	// ...
```

`source`가 `network` 라고 판단되면 이미지를 네트워크에서 받아오기 위해 `DownloadTask`를 생성합니다. 이를 위해 `downloader`라는 프로퍼티를 사용하는데요, 이는 `ImageDownloader`타입으로 서버에서 URL로 이미지를 요청하는 다운로드 관리자를 나타냅니다. 그래서 `setImage(with:)` 메소드의 반환값이 옵셔널 타입의 `DownloadTask` 였던 것입니다. 

그런데 항상 `setImage(with:)` 메소드를 통해 `DownloadTask`를 얻을 수 있을까요? `UIImageView`에 이미지를 설정하지 않고 이미지만 따로 다운 받을 수는 없는 걸까요? 아닙니다! `ImageDownloader` 인스턴스를 생성하거나 싱글톤으로 선언된 `ImageDownloader.default`를 통해 이미지만 따로 다운받을 수 있고, 해당 task를 취소할 수 있습니당ㅎ

``` swift
// let downloader = ImageDownloader.default -> singletone
let downloader = ImageDownloader(name: "myDownloader")
let task = downloader.downloadImage(with: url) { result in
    // ...
    case .failure(let error):
        if error.isTaskCancelled {
        		// 이 부분에 task가 취소되었을 때 동작을 넣으면 됨.
        }
    }

}

// task를 취소할 수 있음!
task?.cancel()
```

<br>

그리고 `이전에 다운로드되었던 컨텐츠를 자동으로 재사용해서 성능을 향상해준다.` 이건 무슨 말일까요? 음... 예를 들어 우리가 `A`라는 이미지를 네트워크에서 다운받고, `processor`를  통해 `A′` 을 만들었다고 가정합시다. 그런데 `A`이미지를 가공해서 `A′′`라는 이미지로 만들고 싶을 때, `A`이미지를 다시 네트워크에서 받아와야 할까요? 아닙니다! `.cacheOriginalImage` 옵션을 활성화하면 원본 이미지를 캐싱해서 재사용할 수 있습니다ㅎㅎ. 함 보시죠~~~

```swift
let p1 = MyProcessor()
imageView.kf.setImage(with: url, options: [.processor(p1), .cacheOriginalImage])

let p2 = AnotherProcessor()
imageView.kf.setImage(with: url, options: [.processor(p2)])
```

이미지를 받아오고 `p1 프로세서`로 가공할 때 `.cacheOriginalImage` 옵션을 활성화했습니다! 그러면 `p2 프로세서`로 동일한 이미지를 가공할 때는 네트워크에 다녀오지 않고, 캐싱된 원본 이미지를 불러와서 가공하는 것입니다!!!



## 4. Independent components. Use the downloader, caching system and image processors separately as you need.

`downloader`, `caching system`, `processor`등을 당신이 원하는 대로 분리해서 사용할 수 있습니다. 이 기능은 제가 다른 기능을 설명하면서 종종 나왔었는데요, 먼저 예시를 보시죠.

``` swift
let downloader = ImageDownloader(name: "myDownloader")
imageView.kf.setImage(with: url, options: [.downloader(downloader)])
        
let cache = ImageCache(name: "myCache")
imageView.kf.setImage(with: url, options: [.targetCache(cache)])
        
let processor = RoundCornerImageProcessor(cornerRadius: 10)
imageView.kf.setImage(with: url, options: [.processor(processor)])
```

이렇게 저만의 `downloader`, `cache`, `processor`들을 만들어서 적용할 수 있습니다. 생각보다 쉽네요...?ㅎ

> 그런데 저는 그동안 따로 `downloader`, `cache`, `processor`를 지정한 적이 없는데 어떻게 동작하는 거죠?
>
> 우리가 평소에 이미지를 설정할 때 사용하던 `setImage(with:)` 메소드는 `KingfisherManager.shared` 라는 싱글톤 변수에 있는 메소드를 내부에서 호출합니다. 그리고 내부에서 이미지 다운로드나, 캐싱, 프로세싱할 때도 기본값으로 `DefaultImageProcessor.default`, `ImageDownloader.default`, `ImageCache.default` 라는 싱글톤 변수들을 사용하기 때문에 동작할 수 있습니다.
>
> 하지만 따로 `downloader`, `cache`, `processor`를 지정한다면 이런 기본값으로 설정되어있는 싱글톤 변수들 대신 자신의 입맛에 맞게 동작을 수행할 수 있겠죠.



-----------------------------------------------------------------

아 너무 많아요 많아 진짜 으ㅓ아아아아악 아직도 포스팅할 기능이 반이나 남았어요..... 기능 하나를 살펴보려하면 그 기능을 구성하고 있는 요소들을 타고타고 들어가고 그 요소를 또 타고타고 들어가게 되는 아주 머리가 복잡한 상황이에요 흑흑 그래도 뭔가 얻는게 있는거같고 막 그래요....

그러면 다음 포스팅 때는 아래 기능에 대해 알아보겠습니다!!!!!!!!!!!!!!!!

* Prefetching images and showing them from cache to boost your app.
* View extensions for `UIImageView`, `NSImageView`, `NSButton` and `UIButton` to directly set an image from a URL.
* Built-in transition animation when setting images.
* Customizable placeholder and indicator while loading images.
* Extensible image processing and image format easily.

~~SwiftUI 관련 기능은 제가 할줄 모르는 관계로 스킵하겠습니다..ㅎ~~

# 출처

* [Kingfisher github](https://github.com/onevcat/Kingfisher)