---
layout: post
title: "[Swift] @autoclosure란?"
category:
  - Swift
tags:
  - Closure
comments: true
published: true
---

## 정의
`@autoclosure`는 함수의 인자로 전달되는 코드를 감싸서 자동으로 클로저로 만들어 줍니다. 
다시말해 일반 표현의 코드를 클로저 표현의 코드로 만들어 주는 역할을 합니다. 

이때 사용되는 클로저는 인자가 없고 리턴값만 존재해야 합니다. 

`@autoclosure`는 기본적으로 non-escaping 클로저로써 만약 여기에 사용되는 클로저가 escaping 클로저라면 `@escaping`을 함께 붙여줘야합니다.

## 예제 코드
### `@autoclosure`를 사용하지 않은 경우
다음 코드는 클로저를 인자로 받는 함수에 `@autoclosure`를 사용하지 않은 경우입니다.

```swift
func normalPrint(_ closure: () -> Void) {
    closure()
}

normalPrint({ print("I'm Normal Expression") })
```

이 함수를 호출할 때 클로저 부분은 다음과 같이 대괄호 `{...}`로 묶어서 인자를 넣어야 합니다.

```swift
normalPrint({ print("I'm Normal Expression") })
```

### `@autoclosure`를 사용한 경우
`@autoclosure`를 이용하면 함수에 클로저를 인자로 사용할때 일반 표현을 사용할 수 있습니다.

```swift
func autoClosurePrint(_ closure: @autoclosure () -> Void) {
    closure()
}

autoClosurePrint(print("I'm AutoClosure Expression"))
```

위에 `@autoclosure`를 사용하지 않은 코드보다 함수에 클로저를 인자로 넣는 코드가 간결해 졌음을 확인할 수 있습니다.

```swift
autoClosurePrint(print("I'm AutoClosure Expression"))
```

## `@autoclosure`의 사용 예
Swift에서 `@autoclosure`를 사용하는 대표적인 예로 `assert()`함수가 있습니다.

`assert()` 함수는 대략 다음과 같이 정의 돼 있는데요. (`@autoclosure`와 상관없는 코드는 생략)

```swift
public func assert(_ condition: @autoclosure () -> Bool, _ message: @autoclosure () -> String = String())
```

`assert()` 함수는 다음과 같이 호출합니다.

```swift
assert(false, "Error Occurred!")
assert(0 < 1, "Error Occurred!")
assert(false, (statusCode == .fileNotFound) ? "File Not Found!" : "Something going wrong!")
```

이 코드에서 `@autoclsoure` 를 제거해 보겠습니다.

```swift
func assert(_ condition: () -> Bool, _ message: String = String()) { }
```

그리고 똑같이 위에서 호출한 `assert()`함수를 모두 호출해 보겠습니다.

```swift
assert({ false }, { "Error Occurred!" })
assert({ 0 < 1 }, { "Error Occurred!" })
assert({ false }, {(statusCode == .fileNotFound) ? "File Not Found!" : "Something going wrong!"})
```

호출 부분의 코드가 좀 더 복잡해진 것을 확인할 수 있습니다.

이렇듯 `@autoclosure`는 일반 표현의 코드를 클로저 표현의 코드로 변환해서 함수에서 클로저를 인자로 사용할때 보다 쉬운 표현으로 클로저를 사용할 수 있게 해줍니다.

## `@autoclosure`의 다양한 활용

다음은 `@autoclosure`를 이용해 코드를 간결하게 만든 (예) 입니다.

### 1. `UIView.animate`
- `@autoclosure` 사용전

```swift
UIView.animate(withDuration: 0.5) {
   self.view.frame.origin.x = 200
}
```

- `@autoclosure` 사용하는 `UIView`를 animate 하는 함수 구현

```swift
func animate(_ animation: @autoclosure @escaping () -> Void, duration: TimeInterval = 0.5) {
   UIView.animate(withDuration: duration, animations: animation)
}
```

이 함수를 사용하면 `{...}` 구문 없이 같은 에니메이션 코드를 수행할 수 있습니다.

```swift
animate(self.view.frame.origin.x = 200)
```

### 2. `Dictionary`
다음과 같은 `dictionary`가 있다고 했을 때

```swift
let fruits = ["Apple": 20, "Banana": 30, "Orange": 40]
```

Apple의 갯수를 얻고자 한다면 dictionary에서 특정 key에 대한 값이 없을 수 있기 때문에 `Optional`로 반환됩니다.  옵셔널인 `Int?` 대신 `Int`로 값을 저장하고 싶다면 `??`을 사용해 옵셔널일때의 값을 지정해야 합니다.

```swift
let apple = (fruits["Apple"]) ?? 0
```

코드가 좀 장황해 보입니다. 이 부분을 `@autoclosure` 사용해 개선할 수 있습니다. 다음과 같은 익스텐션을 생성합니다.

**[Int 버전]**

```swift
extension Dictionary where Value == Int {
    func value(forKey key: Key, defaultValue: @autoclosure () -> Int) -> Int {
        guard let value = self[key] else { return defaultValue() }

        return value
    }
}
```

**[Generic 버전]**

```swift
extension Dictionary {
    func value<T>(forKey key: Key, defaultValue: @autoclosure () -> T) -> T {
        guard let value = self[key] as? T else { return defaultValue() }

        return value
    }
}
```

이 익스텐션을 사용하면 위의 `Dictionary`에서 특정 키값에 접근하고, 키값이 없을시 기본 값을 갖는 코드를 다음과 같이 표현할 수 있습니다.

```swift
let apple = fruits.value(forKey: "Apple", defaultValue: 0)
```

## 정리
`@autoclosure`를 사용하면 장황해 보이는 코드를 보다 간결하게 보이도록 만들 수 있습니다. 하지만 남용하면 오히려 코드의 가독성을 해칠 수 있기 때문에 꼭 필요한 곳에 적절히 사용하는 것이 좋습니다.

이상으로 `@autoclosure`에 대해 살펴봤습니다.

👨🏻‍💻지식이 +2 늘었다. 다음 포스트에서 또 만나요 🚀😄

**[참 고]**

- [Using @autoclosure when designing Swift APIs](https://www.swiftbysundell.com/articles/using-autoclosure-when-designing-swift-apis/)

- [What is the autoclosure attribute?](https://www.hackingwithswift.com/example-code/language/what-is-the-autoclosure-attribute)

- [[Swift]Autoclosure란?](http://minsone.github.io/mac/ios/what-is-autoclosure-in-swift)

- [Swift - @autoclosure 이야기](http://seorenn.blogspot.com/2016/04/swift-autoclosure.html)

- [Closures](https://docs.swift.org/swift-book/LanguageGuide/Closures.html)
