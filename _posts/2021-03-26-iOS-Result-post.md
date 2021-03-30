---
title: "Result"
date: 2021-03-26 11:17:10 -0400
categories: 졔 update
---


### Result


Swift5부터 우리는 Result를 사용할 수 있습니다

(👉 https://github.com/apple/swift-evolution/blob/master/proposals/0235-add-result.md)


사실 Result에 대해서 깊게 알아볼 게 무엇이 있을까요?

<br>

저는 누군가 이 코드에서 왜 Result를 사용했나요?란 질문에 Result의 강점에 대해 대답하지 못했습니다 ..



심지어 제네릭까지 사용해서 어디서든 가져다 쓸 수 있게 만들어 두곤 말문이 뙇! 막혀서 아무 말도 못했어서

<br>

겸사겸사 👀

회사에서 스터디도 해야 하고 Result를 한 번 보자!해서 이렇게 Result에 대해 글을 쓰게 되었습니다

비동기 API 코드를 작성하여 네트워킹을 할 경우 Result 타입을 많이 사용한다고 합니다

라고 해서 아래와 같은 코드를 준비한 거지 비동기 API 코드 외에도 Result를 활용할 수 있습니다 :)


<br>

우선 옵셔널을 사용한 예제코드를 보겠습니다!


```
func request(then handler: @escaping (Data?, Error?) -> Void) {
    //...
}
```

```
request { (data, error) in
    guard error == nil else { handleError(error!) }
    
    guard let data = data else { return } // Impossible? 🤔
    
    handleData(data)
}
```

<br>

여기서 success인 경우 data를 넘겨주고 fail인 경우 data를 nil로 넘겨주도록 만든다고 생각해 봅시다


이 경우
<br>
urlComponent를 정상적으로 잘 생성했는지
<br>
URLSession 요청이 잘된건지
<br>
Decoding이 잘못된 건지 알 수 없습니다

<br>

Swift는 실패 가능한 작업을 처리하기 위해 throws, try, catch를 제공했습니다

하지만!

이 방법으로 여러가지 예외 상황에 대해 대처하기 어려웠기 때문에 Swift5에서는 이런 점을 보완해 유연한 처리를 할 수 있도록 Result Type을 제공합니다

<br>

Result 타입을 사용하는 방법은 어렵지 않습니다


Swift의 Result Type은 success와 failure 두 가지 case가 있는 enum입니다
<br>
둘 다 제네릭을 사용하여 구현되므로 개발자가 정한 타입의 연관값을 가질 수 있습니다
<br>
하지만 failure의 연관값은 Swift의 Error를 채택해야 합니다 :)

```
enum Result<Success, Failure: Error> { 
	case success(Success) 
	case failure(Failure)
}
```

<br>

Result를 사용하면

```
func request(then handler: @escaping (Result<Data, SomeError>) -> Void) {
    //...
}

request { result in
    switch result {
    case .success(let data):
        // success 처리
        // load된 data 처리
    case .failure(let error):
        // error 처리
    }
}
```

명시적이고 간결하게 에러 처리를 할 수 있습니다
<br>
여기서 SomeError은 제가 만든 Error를 채택한 enum입니다

<br>

1. get()

이 메서드는 성공한 값이 있으면 반환하고 그렇지 않으면 에러를 throw합니다
<br>
일반적인 예외 throw를 사용하고 싶을 때 get()을 사용하면 됩니다

<br>

```
do {
    let (data, urlResponse) = try result.get()
		// ...
} catch let error {
    // error 처리
}
```

<br>

2. throwing closure를 받는 초기자


클로저가 값을 성공적으로 반환하면 success case의 연관값으로 저장하고
<br>
그렇지 않으면 throw된 에러를 failure case의 연관값으로 저장합니다


```
let result = Result { try String(contentsOfFile: someFile) }
```

<br>


자 지금까지 Result 타입에 대해서 알아봤습니다 :)
<br>
그럼 Result의 강점은 무엇이라고 생각하시나요?

<br>

Result를 사용하면 모호한 상태의 처리가 필요없고 꼭 필요한 상태만 처리하면 되니 보다 명시적이고 간결하게 에러처리를 할 수 있습니다
<br>
Result에 대해서 알아보려고 열심히 찾아보다가
<br>
재미난 것을 찾았습니다

<br>

다른 언어를 보면 이미 Result가 있는 언어도 있고 Result와 비슷한 일을 처리하는 것들이 존재합니다
<br>
오호!

<br>

그럼 Swift의 Result는 다른 언어에서 가져온건가?

<br>


그러다가

<br>

오잉? Result 라이브러리가 있습니다
<br>
심지어
<br>
README.md를 보니! 이번에 추가된 Result 타입과 너무나 비슷하게 생겼습니다

<br>

👀

<br>

아무래도 다른 언어에서 잘 사용하는 것을 Swift로 가져오지 않았을까란 생각이 들던 차에
<br>
Alamofire에서도 Result라는 타입을 만들어서 사용했었다는 사실을 듣고!
<br>
히스토리를 뒤졌습니다 :)



https://github.com/Alamofire/Alamofire/issues/2752

https://github.com/Alamofire/Alamofire/pull/2769


Swift5에서 Result 타입이 추가되면서 충돌이 생겼고 해당 이슈는 수정되었습니다


note.

Swift5에서 Result 타입이 생기면서 내부에서 충돌이 생겨서

기존에 사용하던 타입이 typealias AFResult <T> = Result <T, Error>로 대체됩니다

Result의 기존 확장도 AFResult와 함께 작동하도록 업데이트 되었습니다

또한 Result에 대한 모든 내부 참조가 AFResult에 대한 참조로 대체되었습니다




Swift에 Result 타입이 없었지만

라이브러리로도 사용하고 있었고

자체적으로도 만들어서 쓰고있던 걸

제안 받아서 추가하게 된 겁니다




사진 삭제
사진 설명을 입력하세요.


허허헣...

그럼 느낌상 다른 언어에서 잘 쓰고 있던 타입을

Swift화 시켜서 가져왔다고 생각해도 될까요? ��



생각보다 이것저것 알게되면서 재미난 이야기도 얻은것 같아 뿌듯하네요 :)




** 참고사이트

