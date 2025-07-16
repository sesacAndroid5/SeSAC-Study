# Enum Class와 Sealed Class의 차이


### 1. Enum Classes (열거형 클래스)
Enum 클래스는 미리 정의된 상수들의 집합을 표현하는 타입이고 각 열거형 상수는 Enum 클래스의 인스턴스인 객체다.

- 모든 열거형 상수는 동일한 프로퍼티와 메서드를 공유
```kotlin
enum class Color(val rgb: Int) {
    RED(0xFF0000),
    GREEN(0x00FF00),
    BLUE(0x0000FF)
}
```

- 다른 클래스를 상속할 수 없지만(java.lang.Enum을 암시적으로 상속), 인터페이스는 구현 가능

- 각 상수는 익명 클래스처럼 자신만의 멤버를 가질 수 있다
```kotlin
enum class ProtocolState {
    WAITING {
        override fun signal() = TALKING
    },

    TALKING {
        override fun signal() = WAITING
    };

    abstract fun signal(): ProtocolState
}
```
------------------------------------------------
### 2. Sealed Classes (봉인된 클래스)
Sealed 클래스는 제한된 클래스 계층을 표현합니다. 특정 클래스를 상속받는 하위 클래스의 종류를 미리 정의하여 제한할 수 있다

- Sealed 클래스 자체는 abstract 클래스이며 직접 인스턴스화 할 수 없다

- (Kotlin 1.5 이상) 모든 하위 클래스는 반드시 같은 패키지 내에 있어야 한다 (이전에는 같은 파일)

- 하위 클래스는 class, data class, object 등 다양한 형태를 가질 수 있으며, 각각 서로 다른 상태와 데이터를 가질 수 있다
```kotlin
// Sealed Class: 각 상태가 다른 데이터를 가질 수 있음
sealed class Result {
    data class Success(val data: String) : Result() // 성공 시에는 데이터를 가짐
    data class Error(val code: Int) : Result()      // 실패 시에는 에러 코드를 가짐
    object Loading : Result()                       // 로딩 중에는 데이터가 없음 (싱글턴)
}
  ```

- 다른 클래스나 인터페이스를 상속받을 수 있다

------------------------------------------------
### 3. 주요 차이점
| 구분	| Enum Class | Sealed Class |
|------|----------------|------------------------|
|인스턴스 |	각 상수가 싱글턴 인스턴스	| 하위 클래스를 class(여러 인스턴스) 또는 object(싱글턴)로 선택 가능 |
|데이터 저장	|모든 상수가 동일한 구조의 데이터만 가짐|	하위 클래스마다 서로 다른 데이터와 구조를 가질 수 있음|
|상속	|클래스 상속 불가, 인터페이스 구현만 가능|	클래스, 인터페이스 상속 가능|

------------------------------------------------

### 4. 핵심 목적과 주요 사용처
#### Enum Class (열거형 클래스)
서로 관련 있는 상수 값들의 집합을 정의할 때 사용하며 모든 값이 동일한 타입과 구조를 가지며, 주로 단순하고 명확하게 구분되는 상태나 옵션을 나타낼 때 적합

- 요일 (MONDAY, TUESDAY...)
- 방향 (NORTH, SOUTH, EAST, WEST)
- 사용자 역할 (ADMIN, GUEST, USER)

------------------------------------------------
#### Sealed Class (봉인된 클래스)
각 하위 클래스는 서로 다른 데이터나 구조를 가질 수 있어, 더 복잡하고 다양한 상태를 표현할 때 적합

- 네트워크 요청의 결과 (Loading, Success, Error)
- 복잡한 화면의 상태 (Content, Loading, Error)
- 사용자 이벤트의 종류 (Click, Scroll, LongPress)
