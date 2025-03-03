# 1. Type

## 타입 추론

- Swift에서 변수와 상수에는 타입을 명시할 수도 있고 생각할 수도 있다.
- 타입을 생략하는 경우 컴파일러가 타입 추론 과정을 거쳐 타입을 지정한다.
- 타입을 생략하는 경우 컴파일 타입에 타입 추론을 하기 때문에 컴파일 시간이 더 걸릴 수 있다.

---

## Class와 Struct

- `class`는 reference 타입이고, `struct`는 value type이다.
- Swift에서 reference 타입은 ARC를 통해 메모리를 관리한다.
- `class`는 상속이 가능하지만, `struct`는 상속할 수 없다.

> Swift 가이드 라인에서 다음 조건 중 하나 이상 해당하면 struct를 사용하는 것을 권장한다.
> - 값의 집합을 캡슐화하는 목적일 때
> - 프로퍼티가 값 타입이고 참조보다 복사하는 것이 합당할 때
> - 상속할 필요가 없을 때

---

## Actor

- **Actor** Swift 5.5부터 등장한 새로운 타입이며 동시성 환경에서 데이터 경합을 방지하기 위해 등장하였다.
- `actor` 타입 또한 타입의 한 종류이므로 프로퍼티, 메서드, 이니셜라이저, 서브스크립트 등을 모두 사용할 수 있다.
- 또한 `protocol`을 채택하여 요구사항을 충족하거나, `extension`을 통해 동일하게 기능 확장이 가능하다.
- `actor`는 `class`와 같이 참조 타입이지만, `struct`와 같이 상속은 불가능하다.
- 기존 동시성 환경에서 데이터 경합을 방지하기 위해 작업이 끝나기 전까지 lock을 걸고, 작업이 완료되면 lock을 해제하는 등의 복잡한 과정이 필요했지만, **Actor**에서는 **actor isolation**이라는 것을 통해 동시성 환경에서 **Actor** 내부에 안전하게 접근할 수 있도록 한다.
- **Actor** 내부의 프로퍼티, 메서드를 외부에서 직접 접근할 수 없다. (기본적으로 `self`를 통해 내부에서만 접근이 가능하다.)
- `async` 함수 내부에서 **cross-actor reference**는 허용하며, 동시성 환경에서 여러 스레드가 동시에 접근하더라도 **데이터 동기화**를 보장하기 위해 `await` 키워드를 사용하여 프로퍼티와 메서드에 접근해야 한다.
- `actor` 타입은 암시적으로 `Sendable` protocol을 채택한다.

```swift
actor Counter {
    private var count: Int
    
    init(_ count: Int) {
		    self.count = count
    }
    
    func setCount(_ count: Int) {
        self.count = count
    }
    
    func getCount() -> Int {
        return self.count
    }
    
    func exchange(with other: Counter) async {
        let temp = self.count
        self.setCount(await other.getCount())
        await other.setCount(temp)
    }
}

let counter1: Counter = .init(10)
let counter2: Counter = .init(20)

Task {
    await counter1.exchange(with: counter2)
}

```

---

## Swift의 값 복사

- Swift는 **COW** 방식을 통해 값을 복사한다.
- **COW**는 원본과 복사본 중 수정되기 전까지 복사하지 않고 원본 리소스를 공유하다가, 둘 중 하나에서 수정이 일어나는 경우 값을 복사한다.
- 따라서 복사 후 첫 번째 수정이 발생하는 시점에 약간의 오버헤드가 발생한다.
- 또한 모든 value 타입에서 사용하는 것이 아니라 collection type의 복사 시 발생
(Swift의 대부분 collection type은 struct type으로 구현되어 있음)

---

## Static Dispatch vs Dynamic Dispatch

📌 **Dynamic Dispatch**

- class는 상속이 가능하기 때문에 상위 클래스와 하위 클래스 중 어떤 값 또는 함수를 호출할 지 런타임에 결정
- **런타임**에 **vTable**을 참조하여 실제 호출할 함수를 결정하기 때문에 성능 측면에서 손해 발생

> **vTable**
> - dynamic dispatch를 지원하기 위해 프로그래밍 언어에서 사용하는 매커니즘
> - 클래스 마다 가지고 있는 가상의 테이블을 말하며 메서드의 포인터 값을 가지고 있음
> - 하위 클래스는 상위 클래스의 vTable 복사본을 가지고, 오버라이딩 하는 경우에는 오버라이딩 한 메서드를 가리키는 함수 포인터 저장

📌 **Static Dispatch**

- 컴파일 타임에 호출될 함수를 결정하고 런타임에 그대로 실행
- value 타입에서 Static Dispatch 사용

하지만 class에서 `final`을 사용하면 상속이 불가능하기 때문에 Static Dispatch 사용 → 오버헤드가 줄어 성능 향상

`private` 접근 제어자 또한 상속이 불가능하기 때문에 Static Dispatch로 동작

---

## Type Casting

- Swift의 타입 캐스팅은 타입을 바꾸는 것이 아닌, 사용 범위를 전환하여 다른 타입처럼 사용할 수 있도록 한다.
- **업캐스팅**은 하위 클래스가 상위 클래스로 전환하는 것, **다운캐스팅**은 상위 클래스가 하위 클래스 타입으로 전환하는 것
- 업캐스팅은 무조건 전환 가능하기 때문에 `as` 사용
- 다운캐스팅을 하는 경우 실패 가능성이 있으므로 `as?` 또는 `as!` 사용
- 인스턴스의 타입을 확인할 때는 `is`를 사용

```swift
class Person {}
class Student: Person {}

let student: Any = Student()

if let person = student as? Person {
    //타입이 변경 되는 것이 아닌, 타입의 범위를 제한
    print(type(of: person))    // Prints: Student
}
```

---

# 2. 접근 제어자

Swift는 `open`, `public`, `internal`, `fileprivate`, `private` 다섯 가지의 접근 제어자를 제공한다.

- `open` 키워드는 클래스와 클래스 멤버에서만 사용할 수 있다.
- 접근 제어자는 상위 요소보타 하위 요소가 더 높은 접근 수준을 가질 수 없다.
- `open`과 `public`은 외부 모듈을 통해 접근을 허용할 수 있음
- `open`은 모듈 외부에서 상속과 오버라이딩이 가능하지만, `public`은 접근만 가능하다.
- `fileprivate`는 소스파일 내부에서 접근이 가능하고, `private`는 선언된 타입 내부에서만 접근이 가능하다.

---

# 3. Initializer

## Convenience Initializer

- `init` 앞에 `convenience` 키워드를 사용하여 편의 이니셜라이저를 나타낸다.
- **편의 이니셜라이저**는 **지정 이니셜라이저**를 도와 클래스 초기화를 쉽게할 수 있도록 도와주는 역할을 한다.
- **편의 이니셜라이저**는 또 다른 **편의 이니셜라이저**를 호출할 수 있지만, 초기화 과정 중 반드시 마지막에는 **지정 이니셜라이저**를 호출해야 한다.
- 이니셜라이저가 다른 이니셜라이저를 호출하는 행위를 초기화를 **위임**한다라고 한다.

## Designated Initializer

- 상속 관계에서 **지정 이니셜라이저**는 반드시 상위 클래스의 **지정 이니셜라이저**를 호출해야 한다.

## Required Initializer

- **요구 이니셜라이저**는 상속받은 모든 하위 클래스에서 필수적으로 **재정의** 해야하는 이니셜라이저이다.
- 이니셜라이저를 **재정의** 하는 역할이지만, `override` 키워드를 사용하지 않는다.
- 하위 클래스에서 상위 클래스의 이니셜라이저로 초기화가 모두 가능한 경우, 상위 클래스의 이니셜라이저가 **자동 상속** 되므로 요구 이니셜라이저를 구현하지 않아도 된다.
- 하지만 하위 클래스에서 어떠한 종류의 이니셜라이저라도 구현하는 경우 상위 클래스의 이니셜라이저가 자동 상속 되지 않으므로 이런 경우 반드시 **요구 이니셜라이저**를 구현해야 한다.

---

## Safety-Check

- 클래스 초기화 중 안전 확인 조건을 만족해야 하는 이유는 의도하지 않는 값으로 프로퍼티가 초기화 되는 것을 방지하기 위함이다.
- 자식 클래스의 **지정 이니셜라이저**는 부모 클래스의 **지정 이니셜라이저**로 초기화를 위임하기 전 자식 클래스만이 가지는 프로퍼티가 모두 초기화 되었는지 확인 후 초기화를 위임해야 한다.
- 자식 클래스의 **지정 이니셜라이저**에서 부모 클래스의 **지정 이니셜라이저**로 반드시 초기화를 위임 후 상속 받은 프로퍼티를 사용자 정의로 초기화할 수 있다.
- 1단계 초기화를 마치기 전까지 이니셜라이저에서 인스턴스 **메서드**, 인스터스 **프로퍼티**를 호출할 수 없으며, `self` 키워드 또한 사용할 수 없다.

## Class의 초기화 2단계

**1단계 초기화**

- 클래스가 지정 또는 편의 이니셜라이저를 호출한다.
- **지정 이니셜라이저**는 자신 클래스가 가지는 모든 **저장 프로퍼티**를 초기화한다.
- **지정 이니셜라이저**는 자신의 프로퍼티를 모두 초기화한 후 상위 클래스의 이니셜라이저에 초기화를 위임한다.
- 최상위 부모 클래스에 도달할 때까지 위의 과정을 반복한다.

**2단계 초기화**

- 최상위 부모 클래스부터 최하위 클래스까지 지정 이니셜라이저를 통해 체인처럼 내려오면서 프로퍼티를 사용자 정의한다.
- 1단계를 마친 후, `self` 키워드, 인스턴스 프로퍼티, 인스턴스 메서드 등을 사용할 수 있다.

---

## Deinit

- 클래스의 인스턴스가 메모리에서 해제될 때 필요한 작업을 수행해야할 때 deinitializer를 사용
- `deinit` 키워드를 통해 구현하면, 인스턴스가 메모리에서 해제될 때 자동으로 호출

---

# 4. 데이터 타입

## Optional Type

- 옵셔널 타입은 값이 있을 수도, 없을 수도 있음을 나타내는 타입이다.

## Optional Type 사용 이유

- 변수 또는 상수에 `nil`을 할당하여 값이 없음을 의미할 수 있다.
- 함수의 parameter에서 값을 전달하지 않아도 되는 경우 옵셔널 타입을 사용할 수 있다.

## Optional의 정의

- 아래는 [Swift 깃허브 리포지토리](https://github.com/swiftlang/swift/blob/main/stdlib/public/core/Optional.swift)에 정의 되어 있는 옵셔널 타입이다. (Swift 5.9+)

```swift
@frozen
public enum Optional<Wrapped: ~Copyable>: ~Copyable {
  case none

  /// The presence of a value, stored as `Wrapped`.
  case some(Wrapped)
}
```

- 기본적으로 Swift의 옵셔널 타입은 `enum`으로 정의 되어 있다.
- `nil`을 할당하는 경우 `.none`값이, 값이 있는 경우 `.some`이 된다.
- 옵셔널 타입은 `@frozen`으로 정의되어 있기 때문에 새로운 case가 추가되지 않는 것을 보장한다.

## Optional Chaining

- Swift는 런타임 안전성을 확보하기 위해 옵셔널 타입의 경우 값이 있는지 없는지 항상 체크해야 한다.
- **옵셔널 체이닝**(`?` 키워드)을 사용하면, 옵셔널 타입의 값이 존재하는 경우에만 메서드를 호출하거나 프로퍼티에 접근할 수 있다.

```swift
if imagePaths["star"]?.hasSuffix(".png") == true {
	print("The star image is in PNG format")
}
```

## Optional Binding

- 옵셔널 타입의 값을 안전하게 사용하기 위해서는 **옵셔널 바인딩**을 사용할 수 있다.

```swift
var optionalValue: Int? = 10

if let value = optionalValue {
	print(value)
}
```

> 기본적으로 옵셔널 타입 간의 **비교 연산자**는 사용할 수 없다.
따라서 옵셔널 타입 간에 값을 비교하기 위해서는 **옵셔널 바인딩**으로 값을 가져온 뒤, **비교 연산자**를 사용해야 한다.
*예외적으로 `==`, `!=`  **비교 연산자**는 옵셔널 타입 간 연산이 가능하다.
> 

## Implicitly Unwrapped Optional

- 옵셔널 바인딩을 하지 않고 **암시적 언래핑**을 통해 옵셔널 내부의 값을 가져올 수 있다.
- 하지만 옵셔널 내부에 값이 존재하지 않고, 예외 처리를 하지 않으면 런타임 에러가 발생한다.

---

## enum Type

- **열거형**은 연관된 항목을 묶어서 표현할 수 있는 데이터 타입이다.
- **열거형** 타입에는 **RawValue(원시값)**을 지정할 수 있다.

> Swift에서 기본적으로 `RawRepresentable` **protocol**을 따르는 `String`, `Int`, `Float`, `Double` 타입은 **원시값**을 명시적으로 지정하지 않은 경우 `String` 타입은 case 이름과 동일하게, number 타입은 0부터 1씩 증가하는 형태의 기본 원시값을 가진다.
`RawRepresentable` **protocol**을 따르지 않는 타입 중 **원시값**으로 지정할 수 있는 타입은 `Character` 타입이다.
> 
- **원시값**을 통해 열거형 값을 지정할 수 있다. 이때 원시값이 존재하지 않을 수 있기 때문에 결과 값은 **옵셔널** 타입이다.

```swift
enum Number: Int {
    case one = 1
    case two = 2
    case three
}

var num: Number? = Number(rawValue: 3)
```

- **원시값**은 미리 지정 되어 있는 한 가지 타입의 한 가지 값만 가질 수 있기 때문에, **열거형**은 **연관값**을 가질 수 있다.
- 모든 **열거형**의 `case`가 **연관값**을 가지는 것은 아니다.

```swift
enum ServerError: Error {
	case internalServerError(message: String)
}

let serverError: ServerError = .internalServerError(message: "서버 에러 메시지")
```
---
## Array
- 배열은 같은 자료형의 데이터를 나열하여 저장하는 형태의 컬렉션 타입이다.
- Swift의 배열은 `struct` 타입으로 선언되어 있다. (value 타입)

### Preallocation
- 배열에 element를 자주 append하는 경우 메모리를 지속적으로 할당 해야하기 때문에 미리 일정 크기만큼의 메모리를 미리 할당하는 방식을 말한다.
- 미리 할당된 크기를 모두 사용하는 경우 더 큰 메모리 영역을 할당하고 이전 저장소에서 새로운 저장소로 복사한다. (재할당)
- 이때, 늘어나는 배열의 capacity는 이전 스토리지의 배수, 즉 지수 형태로 늘어난다.
- 따라서 element를 추가하는 빈도가 많아질 수록 발생 빈도는 줄어든다.
- `Array` 타입의 `capacity` 프로퍼티를 통해 크기를 확인할 수 있다.
```swift
var arr: [Int] = Array(repeating: 0, count: 3)
print(arr.count)    //3
print(arr.capacity)    //4

arr.append(0)
print(arr.count)    //4
print(arr.capacity)    //4

arr.append(0)
print(arr.count)    //5
print(arr.capacity)    //8

```
- 만약 재할당을 방지하고자 한다면, `reserveCapacity(_:)` 메서드를 통해 capacity를 고정할 수 있다.