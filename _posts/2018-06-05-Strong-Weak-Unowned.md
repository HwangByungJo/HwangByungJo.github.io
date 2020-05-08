---
title: strong, weak, unowned - Reference Counting in Swift
author: HwnagByungJo
layout: post
---

## strong, weak, unowned - Reference Counting in Swift

이 아티클에서는 스위프트에서의 Apple의 메모리 관리 방법을 설명합니다. 
대부분 자동으로 처리 되더라도 여전히 몇 가지 함정이 있습니다. 
객체 간의 관계를 설명하는 올바른 참조 유형을 선택하면 메모리 누수를 피할 수 있습니다.


### Automatic Reference Counting

자동 추적 및 메모리 사용 관리를위한 Apple의 구현을 ARC (Automatic Reference Counting)라고합니다.
객체가 더 이상 참조되지 않을 때 자동으로 메모리를 비웁니다.
어떤 오브젝트가 아직 사용 중인지 알기 위해 참조 카운트를 증가 및 감소시켜 오브젝트 간의 관계를 추적합니다. 오브젝트는 참조 횟수가 0 일 때만 할당을 취소 할 수 있습니다.ARC는 클래스의 인스턴스에만 적용됩니다. 
구조체, 열거 형 및 값 형식은 참조로 전달되지 않습니다. 
이것은 가능할 때마다 클래스 대신에 Structs를 사용하는 또 하나의 큰 이유입니다.

### Strong

Swift에서는 속성의 기본 선언이기 때문에 강력한 참조를 찾을 수 있습니다. 
이것은 선형 참조 흐름(linear reference flow)이 있을 때 문제가 되지 않습니다. 
부모를 할당 해제하고 보유 수를 줄이면 모든 자식도 감소합니다. 다음은 강력한 참조의 예입니다.



```swift
import Foundation

class Car {

    var brand: String

init(brand: String) {

        self.brand = brand

        print("Car of the brand (brand) allocated")

    }

deinit {

        print("Car of the brand (brand) is being deallocated")

    }

}

do {

    let tesla = Car(brand: "tesla")

}

```



Car 객체의 초기화는 do 블록에 래핑됩니다. 

이렇게하면 주위에 범위가 만들어집니다. 범위의 끝에서 객체의 할당이 해제되고 모든 것이 잘됩니다.



> Car of the brand tesla allocated
>
> Car of the brand tesla is being deallocated



그러나 클래스 인스턴스간에 강력한 참조주기가 있는 코드를 작성할 수 있습니다.

이전 예제의이 수정 된 버전은 참조주기를 보여줍니다.



```swift
import Foundation

class Car {
    var brand: String
    var owner: Owner?
 
    init(brand: String) {
       self.brand = brand
       print("Car of the brand \(brand) allocated")
    }
 
    deinit {
        print("Car of the brand \(brand) is being deallocated")
    }
}
class Owner {
    var name: String
    var car: Car?
 
    init(name: String) {
        self.name = name
        print("Owner \(name) allocated")
    }
 
    deinit {
        print("Owner \(name) deallocated")
    }
}
do {
    let tesla = Car(brand: "tesla")
    let misterX = Owner(name: "Mister X")
    tesla.owner = misterX
    misterX.car = tesla
}
```



 car 객체는 이제 Owner에 대한 선택적 참조를 가지며 Owner는 Car에 대한 선택적 참조를가집니다. 

출력은 다음과 같습니다.



> Car of the brand tesla allocated
>
> Owner Mister X allocated



그러나 클래스 인스턴스간에 강력한 참조주기가있는 코드를 작성할 수 있습니다. 이전 코드의 수정된 버전은 다음과 같은 참조주기를 보여줍니다.



```swift
import Foundation

class Car {
    var brand: String
    var owner: Owner?
 
    init(brand: String) {
       self.brand = brand
       print("Car of the brand \(brand) allocated")
    }
 
    deinit {
        print("Car of the brand \(brand) is being deallocated")
    }
}
class Owner {
    var name: String
    var car: Car?
 
    init(name: String) {
        self.name = name
        print("Owner \(name) allocated")
    }
 
    deinit {
        print("Owner \(name) deallocated")
    }
}
do {
    let tesla = Car(brand: "tesla")
    let misterX = Owner(name: "Mister X")
    tesla.owner = misterX
    misterX.car = tesla
}
```



 car 객체는 이제 Owner에 대한 선택적 참조를 가지며 Owner는 Car에 대한 선택적 참조를가집니다. 출력은 다음과 같습니다.



> Car of the brand tesla allocated
>
> Owner Mister X allocated



강력한 참조주기는 보유 수를 0으로 설정하여 할당을 해제하고 메모리를 확보합니다. 이 문제를 해결하려면 약한 참조가 필요합니다.

### Weak

Strong 참조 과  달리 Weak은 보유 수를 1만큼 증가시키지 않습니다. 그래서 그들은 객체가 ARC에 의해 할당 해제되지 않도록 보호하지 않습니다. 할당 해제시 Weak 참조는 자동으로 nil로 설정됩니다. 이것이 모든 약한 참조가 선택적 변수를 변형시키는 이유입니다. 상수를 약한 것으로 정의하는 것은 불가능합니다.



앞의 예를 살펴 보겠습니다. 자동차와 소유자 사이의 Weak한 참조를 사용하면 할당 취소 문제를 해결할 수 있습니다.



```swift
class Car {
    weak var owner: Owner?
    ...
}
class Owner {
    weak var car: Car?
    ... 
}
...
```



이제 ARC는 모든 객체를 올바르게 할당 해제합니다.

> Car of the brand tesla allocated 
>
> Owner Mister X allocated 
>
> Owner Mister X deallocated 
>
> Car of the brand tesla is being deallocated



 참고 :  Car 또는 Owner 클래스에서 하나의 Weak 참조만 사용하면 문제가 해결되어 강력한 참조 사이클이 해제됩니다.



### Unowned

 Unowned 참조는 약한 참조와 비슷합니다. 보유 수를 1만큼 증가시키지 않습니다. Unowned 참조와 달리 할당 해제시 nil로 자동 설정되지 않기 때문에 Optional 일 필요는 없습니다. 객체가 설정되면 객체가 전혀 삭제되지 않는다는 사실을 알고있을 때만 Unowned 참조를 사용하는 것이 중요합니다.



 Apple에 따르면 참조와 참조 된 코드가 동시에 할당 해제 될 때 사용하는 것이 가장 좋습니다.



### weak vs. unowned - an advice from Apple

> "그 참조가 평생 동안 어떤 시점에서 무효가 될 때마다 weak 참조를 사용하십시오. 반대로 초기화가 완료되면 참조가 무조건 유지된다는 사실을 알게 될 때 unowned 참조를 사용하십시오. " Apple Documentation



### fin

Apple의 자동 메모리 관리로 인해 메모리 누출이 발생하지 않습니다. 비선형 강한 참조(Non-linear strong references) 및 참조주기는 보유 수가 0이되지 않도록 방지 할 수 있습니다. weak 및 unowned 참조를 사용하면이 문제를 방지하고 앱 메모리를 균형있게 유지할 수 있습니다.

원문 : <https://medium.com/@chris_dus/strong-weak-unowned-reference-counting-in-swift-5813fa454f30>




