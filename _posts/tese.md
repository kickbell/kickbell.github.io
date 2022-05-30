---
layout: post
title: "[Algorithm] 순환참조를 피하는 방법 - 신이섭"
tags: [weak , unowned]
comments: true
---


<br>
<br/>

[jhjo-tech : 순환참조를 피하는 방법
<br>
<br>
<br/>
순환참조를 피하는 방법

순환참조는 무조건 피해야 합니다. 순환참조를 피하는 방법은 weak(약한 참조) / unowned(미소유 참조) 가 있다.
 weak(약한 참조) / unowned(미소유 참조) 모두 reference count를 증가시키지 않는다
<br>
<br/>
 

``` 
weak var seob: Person?

unowned let seob = Person()
```

위 샘플소스를 보게 되면  Optional 여부가 다르다.
Optional 여부가 다르다라는건 사라지지 않을거라는 보장이 있는 객체라는 판단이 중요하다
<br>
<br/>
1. 순환참조가 발생하는 경우는 Closure , Delegate Pattern 가 있다 

<br>
<br/>
1. Closure 에서 약참조/미소유 참조 사용
# Linear time

```
closure = { [weak self] newPoint in
  self?.point = newPoint
}
closure = { [weak self] newPoint in 
  guard let self = self else { return } 
  self.point = newPoint
}
```
2. Closure 에서 약참조/미소유 참조 사용
```
closure = { [unowned self] newPoint in
  self.point = newPoint 
}
``` 
<br>
<br/>
 
<br>
<br/>

# 요약

-   순환참조가 발생 한 경우 영구적으로 메모리가 해제되지 않을 수 있다.
-   프로퍼티를 선언한 이후, 나중에 nil이 할당된다는 점으로 보아 , weak는 무조건 옵셔널 타입의 변수 여야한다.
-   string 
   ▪ 강한 순환 참조로 인해 Memory leak 이 발생할 수 있음
-   weak
   ▪ 참조하던 인스턴스가 해제되면 자동으로 nil을 할당
- unowned
   ▪  참조하던 인스턴스가 먼저 메모리에서 해제되면, 해제된 주소값을 계속 들고 있음 (에러로 이어질 가능성 높음)
- 이상 순환참조에 대해 알아봤다. string, weak, unowned 를 잘써서 메모리 관리에도 신경쓰길 바란다 
<br>
<br/>
 



