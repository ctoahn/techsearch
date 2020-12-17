rxswift에서 flatmap 사용하는 이유
---

flatmap을 사용하는 이유를 알기전에 우선 map과 flatamp의 차이를 먼저 알아야 합니다.

map은 이벤트를 다른 이벤트 타입으로 변경합니다. ( E 타입의 이벤트를 R타입의 이벤트로 변환)
이벤트가 발생할때 마다 수행되고 이벤트를 다른 형태로 가공 or 변경 사용하고 싶을때 주로 사용합니다.

```swift
public func map<R>(_ transform: @escaping (Self.E) throws -> R) -> RxSwift.Observable<R>
```

flatmap은 이벤트를 다른 observable로 바꿈니다.
이벤트를 비동기처리해야 한 후 결과를 전달해야 할겨우 사용합니다.
```swift
public func flatMap<O: ObservableConvertibleType>(_ selector: @escaping (E) throws -> O)
        -> Observable<O.E>
```

map과 flatamp모두 입력된 이벤트를 처리해서 다른 형태로 변환해서 전달하는 비슷한 일을 하는것 같은데 왜 다를까요? 핵심은 비동기처리

map은 구독중인 observable의 이벤트 스트림을 그대로 유지하며 발생하는 이벤트의 값만 변경하고 싶은 경우로 주로 사용됩니다.  테이터의 형변환이나, 즉시 결과값이 반환되는 타입의 변환이나 동기화처리된 계산식의 결과를 반환하는 경우에 사용됩니다. 

그런데 만약에 이벤트의 변환이나 처리에 시간이 필요해서 비동기로 처리해야 한다면 어떻게 될까요? 만약 이벤트를 받은 후 서버에 정보를 요청하고 그 결과를  다음으로 전달해야 한다면? map으로는 정상적인 처리가 불가능 합니다. 그래서 비동기처리를 위해 사용하는 객체인 observable을 전달해야 하고 이런 역할을 하는 함수가 flatmap입니다.

### map의 사용 예
1. 단순 타입의 변환
```swift
slider.rx.value.asObservable()
        .map { String($0) }
        .bind(to: label.rx.text)
        .disposed(by: disposeBag)
```

2. 단순 타입변환 (json to Objct)
```swift
API.default.request(.getPost)
      .map {json in Post(json:json)}
      .objserveOn(MainScheduler.instance)
      .subscrib(onNext: {[week self] post in 
        self?.display(post:post)
			})
 			.disposed(by: disposeBag)
```


### flatmap의 사용 예
1. 여러 비동기 이벤트를 통합 처리 후 반환 하는 경우   
```swift
button.rx.tap.asObservable()
	.flatmap{
		return Obsevable.zip(API.default.request(.getPost), API.default.request(.getExtra) ){ postJson,extraJson in 		
	     return (Post(json:postJson), Extra(json:extraJson))
		}
	}
	.objserveOn(MainScheduler.instance)
  .subscrib(onNext: {[week self] (post, extra) in 
			self?.display(post:post, extra:extra)
  })
  .disposed(by: disposeBag)

```