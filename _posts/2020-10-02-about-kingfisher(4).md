---
title: 개린이의 관점에서 Kingfisher 살펴보기(4) - Kingfisher의 주요 기능
author: 1Consumption
date: 2020-10-02 21:00:00 +0800
categories: [iOS, Library]
tags: [개린이, 살펴보기, 이미지 캐싱]
toc: true


---

> 이 글은 개린이의 지식을 바탕으로 작성된 글입니다. 최대한 옳은 정보를 담으려고 노력하겠으나, 그럼에도 틀린 부분이 있을 수 있습니다. 혹여 발견하시면 댓글로 피드백 주시면 감사하겠습니다.

> 개린이의 관점에서 Kingfisher 살펴보기 시리즈
>
> 1. [개린이의 관점에서 Kingfisher 살펴보기(1) - 이미지 캐싱이란?](https://1consumption.github.io/posts/about-kingfisher(1))
> 2. [개린이의 관점에서 Kingfisher 살펴보기(2) - Kingfisher의 주요 기능](https://1consumption.github.io/posts/about-kingfisher(2))
> 3. [개린이의 관점에서 Kingfisher 살펴보기(3) - Kingfisher의 주요 기능](https://1consumption.github.io/posts/about-kingfisher(3)/)

# 머리말

하하... 저번 포스팅 때도 미루고 미루고 어쩌구 저쩌구로 시작했던 것 같은데.... 정말 오랜만에 글을 씁니다...ㅎㅎ 글을 안 써 버릇 하니까 계속 쓰기 싫고... 참 큰일이네요ㅎ

오늘은 아래의 기능에 대해 알아보고 Kingfisher의 주요 기능에 대한 포스팅을 마무리 하겠습니다.

* Prefetching images and showing them from cache to boost your app.
* View extensions for `UIImageView`, `NSImageView`, `NSButton` and `UIButton` to directly set an image from a URL.
* Built-in transition animation when setting images.
* Customizable placeholder and indicator while loading images.
* Extensible image processing and image format easily.

# Kingfisher 주요 기능

## 1. Prefetching images and showing them from cache to boost your app.

Cache로부터 이미지를 미리 가져와서 보여줌으로써 앱의 성능을 향상시켜 줍니다. 엥 근데 내가 어떤 이미지를 필요로할 줄 알고 미리 cache에서 가져온다는 걸까요? 음... 그건 아무도 모릅니다. 하지만!! 내가 필요한 이미지 리스트가 주어진다면? 미리 가져올 수 있겠죠?ㅎㅎ `Kingfisher`에서도 `This is useful when you know a list of image resources you know they would probably be shown later.` 라고 말하고 있네요. 

우리는 `ImagePrefetcher`를 사용해서 이미지를 미리 가져올 수 있습니다. 아래 코드는 [cheat sheet](https://github.com/onevcat/Kingfisher/wiki/Cheat-Sheet)에 있는 예시 코드입니다. cheet sheet 최고ㅎㅎ

```swift
let urls = ["https://example.com/image1.jpg", "https://example.com/image2.jpg"]
						.map { URL(string: $0)! }
let prefetcher = ImagePrefetcher(urls: urls) { skippedResources, failedResources, completedResources in
    print("These resources are prefetched: \(completedResources)")
}
prefetcher.start()

// Later when you need to display these images:
imageView.kf.setImage(with: urls[0])
anotherImageView.kf.setImage(with: urls[1])
```

<br>

사용법은 간단합니다. `ImagePrefetcher` 생성자에 prefetch할 `URL` 또는 `Resource` 를 넘겨주면 됩니다. 

> 생성자의 매개변수
>
> 1. 이미지를 캐싱할 때 추가로 설정을 할 수 있는 `KingfisherOptionsInfo`
> 2. 이미지를 prefetching 하는 경과에 대한 작업을 할 수 있는 `PrefetcherProgressBlock`
> 3. Prefetch를 완료하고 난 후의 동작을 받는 `PrefetcherCompletionHandler` 



`ImagePrefetcher` 인스턴스를 만들었으니 `start()` 메소드를 호출해서 prefetch를 하면 우리가할 일은 끝납니다. 굳굳ㅎ 이라고 넘어가고 싶지만 `start()` 메소드의 동작 방식을 안알아보고 가면 서운하겠죠?ㅎ

 `start()` 메소드가 실행되면 먼저 3가지를 검사합니다.

1.  `ImagePrefetcher` 가 중지 되었는지
2. 최대 동시 다운로드 수가 1 보다 작은지(기본값은 5 )
3. prefetch할 대상이 비어있는지

그런데 이 조건 중 최대 동시 다운로드 조건이 잘 이해가 안 가는데, 기본값이 있기 때문에 사용자가 값을 지정하지 않는 이상 1보다 작아질 수 없을 텐데... 더더욱 사용자가 동시에 다운로드 가능한 수를 1보다 작게 해둘 리도 없고요...흠흠 잘 모르겠네요ㅎ

위 조건을 모두 통과했다면 prefetching이 시작되는데요, 한 번에 `maxConcurrentDownloads` 와 prefetching 할 source의 수 중에서 최솟값만큼 prefectching 합니다. 그리고 `CacheType` 을 활용해 현재 prefetching 할 source가 memory에 cache 되어 있는지, disk에 cache 되어 있는지, 둘 다 아니라서 network에서 받아와야 할지 판단합니다. 

> `CacheType`  이란?
>
> Kingfisher에서 제공하는 enum 타입으로 none, memory, disk 세가지 케이스를 가짐.
>
> cached 프로퍼티를 통해 memory 또는 disk 캐싱이 되어있는지, 아니면 안되어 있는지를 반환함.

<br>

이렇게 prefetch를 완료했습니다. 그런데 이미지를 미리 가져오는 것이 어떤 UI Element를 사용할 때 효과적일까요? 바로... 스크롤을 통해 여러 요소를 볼 수 있는 `UICollectionView`나 `UITableView` 입니다.  `UITableViewDataSourcePrefetching` `UICollectionViewDataSourcePrefetching` 프로토콜을 통해 prefetch 될 cell들을 알 수 있기 때문이죠.

`UITableViewDataSourcePrefetching`과 `UICollectionViewDataSourcePrefetching` 에 대해서는 다음에 꼭!!! 포스팅하도록 하겠습니다. 이번 포스트에서는  `UICollectionViewDataSourcePrefetching` 로 설명하겠습니다. 어차피 둘이 하는 일은 똑같아요ㅎㅎ

```swift
override func viewDidLoad() {
    super.viewDidLoad()
    collectionView?.prefetchDataSource = self
}

extension ViewController: UICollectionViewDataSourcePrefetching {
    func collectionView(_ collectionView: UICollectionView, prefetchItemsAt indexPaths: [IndexPath]) {
        let urls = indexPaths.flatMap { URL(string: $0.urlString) }
        ImagePrefetcher(urls: urls).start()
    }
}
```

<br>

`prefetchItemsAt`  메소드 내에서 `ImagePrefetcher`를 생성과 동시에 `start()` 메소드를 실행합니다. 이 과정을 통해 곧 표시될 셀에 대한 이미지를 미리 받아올 수 있습니다. 그런데 만약에 유저가 스크롤을 너무 빨리해서 prefetch 중인 이미지가 쓸모가 없어 취소를 해야 한다면 어떻게 해야 할까요? 그 시점을 우리는 `cancelPrefetchingForItemsAt` 메소드를 통해 알 수 있습니다. 해당 메소드 내에서 `ImagePrefetcher().stop()` 메소드를 통해 prefetch를 취소할 수 있습니다. 

<br>

## 2. View extensions for `UIImageView`, `NSImageView`, `NSButton` and `UIButton` to directly set an image from a URL.

Extension을 통해 `UIImageView`, `NSImageView`, `NSButton`,  `UIButton`에 직접 이미지를 설정할 수 있습니다. 이를 통해서 당신의 코드를 심플하고 우아하게 만들어 준대요. 

이게 무슨 말이냐? 어렵지 않습니다. 우리는 이것을 계속 썼어요. 엥???? 내가 언제??? 라고 말씀하실 수도 있습니다. 하지만 사실인걸요... 

바로... `kf` 라는 키워드를 통해서요! `UIImageView`에 이미지를 지정해줄 때 어떻게 했죠? `imageView.kf.setImage(with: url)` 이렇게 했죠! 아래의 코드를 보시면, `kf`가 `KingfisherWrapper` 인스턴스를 반환하네요. 여기서 `KingfisherCompatible`는 `protocol`입니다. 이를 `UIImageView`, `NSImageView`, `NSButton`,  `UIButton`들이 채택하고 있는 것입니다. 

``` swift
extension KingfisherCompatible {
    /// Gets a namespace holder for Kingfisher compatible types.
    public var kf: KingfisherWrapper<Self> {
        get { return KingfisherWrapper(self) }
        set { }
    }
}
```

<br>

그리고 `KingfisherWrapper` 는 Generic을 사용합니다. Base 타입의 `base` 프로퍼티를 가지고 있는데, `self`로 넘어온 주소가 `base` 프로퍼티에 저장이 됩니다. 그래서 `KingfisherWrapper` 구조체에 extension을 통해 구현된 여러 메소드 내에서 `base`프로퍼티를 통해 `UIImageView`의 주소에 접근해 이미지를 설정해줄 수 있는 것입니다.

```swift
public struct KingfisherWrapper<Base> {
    public let base: Base
    public init(_ base: Base) {
        self.base = base
    }
}
```

<br>

-----------------------------------------------------------------

엥? 왜 벌써 끝?????

아.... 음.... 쓰다 보니 양이 좀 많네요ㅎ 그리고 아래의 내용들은 스크릿샷과 함께 설명을 해야 할 텐데 그럼 양이 더 많아질 거고요ㅎ... 그래서 다음 포스팅 때 쓰도록 하겠습니다 하하핳

* Built-in transition animation when setting images.
* Customizable placeholder and indicator while loading images.
* Extensible image processing and image format easily.

# 출처

* [Kingfisher github](https://github.com/onevcat/Kingfisher)