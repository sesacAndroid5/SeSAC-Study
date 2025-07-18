# 의존성 주입(DI)과 DI 라이브러리(Hilt/Koin)의 필요성

## 의존성 주입(DI)이란?

> 의존성 주입(Dependency Injection)은 객체가 필요한 의존성을 외부에서 주입받도록 하여, 객체 간의 결합도를 낮추고 테스트와 유지보수를 쉽게 해주는 설계 원칙이다.

---

## DI를 사용하지 않았을 때의 문제점

### 1. **높은 결합도**

* 객체가 직접 다른 객체를 생성하거나 초기화할 경우, 두 객체는 강하게 연결됨.
* 변경이나 확장이 어렵고 유지보수가 어렵다.

```kotlin
class UserRepository {
    private val api = UserApiService() // 직접 생성 → 테스트 어렵음
}
```

---

### 2. **재사용성 저하**

* 의존 객체가 하드코딩되면 재사용이 어렵고, 다양한 환경에 맞게 유용하게 구성할 수 없음.

---

### 3. **테스트 어렵음**

* 테스트 시 가짜 객체(Mock)를 주입하기 힘들음 → 단위 테스트(Unit Test) 작성이 불편.

---

### 4. **확장성 부족**

* 앱 구조가 커지면 의존성 관리가 복잡해지고, 클래스 간 연결 관계를 파악하기 어렵다.

---

## DI 라이브러리(Hilt, Koin)를 사용하면 좋은 점

### 1. **결합도 감소 & 유연한 설계**

* 의존성을 외부에서 주입 → 클래스 간 관계가 느슨해지는 구조
* 기능 변경이나 테스트 시 쉽게 대체 가능

---

### 2. **의존성 자동 주입**

* 생성자 주입 또는 필드 주입으로 객체 생성 및 주입 자동화
* 객체 생성 관리 시 스코프 ID 없이 구현 가능

```kotlin
@HiltViewModel
class MainViewModel @Inject constructor(
    private val userRepository: UserRepository
) : ViewModel()
```

---

### 3. **테스트 용이**

* 테스트 환경에서는 실제 구현 대신 Mock 객체 주입 가능
* 단위 테스트 / UI 테스트를 더 쉽게 작성 가능

---

### 4. **전역 관리 및 생명주기 기반 주입**

* Singleton, ViewModel, Activity 등 생명주기에 따라 필요한 객체를 적절히 주입
* 메모리 누수 또는 중복 생성 방지

---

### 5. **생산성 향상**

* 반복적인 객체 생성 코드 제거 → 코드 간결성 증가
* 프로젝트 규모가 커질 경우 효율적인 구조 제공

---

## 주요 DI 라이브러리 비교

| 항목          | Hilt                                               | Koin                   |
| ----------- | -------------------------------------------------- | ---------------------- |
| **언어 기반**   | Annotation 기반 (Java/Kotlin)                        | Kotlin DSL 기반          |
| **주입 방식**   | 컴파일 타임 (정적 분석: KAPT/KSP)                           | 런타임 (동적 분석)            |
| **성능**      | 빠름 (컴파일 타임 최적화)                                    | 느릴 수 있음 (런타임 분석)       |
| **학습 곡선**   | 다소 높음 (Annotation 기반 구성 필요)                        | 비교적 쉬움 (DSL 기반 선언적 구성) |
| **테스트 지원**  | 우수 (Mock 주입 가능)                                    | 우수 (Mock 주입 가능)        |
| **생명주기 지원** | Android 생명주기와 통합 (Activity, Fragment, ViewModel 등) | 별도 생명주기 지원 구성 필요       |
| **공식 지원**   | Android 공식 지원 (Google 제공)                          | 커뮤니티 중심 지원             |

---

## Hilt vs 수동 주입 코드 비교

### Hilt 사용 시

```kotlin
@HiltViewModel
class MyViewModel @Inject constructor(
    private val repository: MyRepository
) : ViewModel()
```

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object AppModule {
    @Provides
    fun provideRepository(): MyRepository = MyRepositoryImpl()
}
```

* ViewModel에 필요한 의존성이 자동 주입됨
* 테스트 및 유지보수 용이

---

### 수동 주입 시

```kotlin
class MyViewModel(
    private val repository: MyRepository
) : ViewModel()
```

```kotlin
val repository = MyRepositoryImpl()
val viewModel = MyViewModel(repository)
```

* 객체 생성 시 일일이 의존성을 전달해야 함
* 테스트 시 대체 구현 전달 가능하지만, 프로젝트 규모가 커지면 복잡도 증가

---

## 결론

의존성 주입을 사용하지 않으면 **유지보수, 테스트, 확장성, 재사용성** 측면에서 부담이 크다.
**Hilt**나 **Koin** 같은 DI 라이브러리를 활용하면 앱 구조를 더 **유지보수 가능하고 유연한 구조**로 만들 수 있으며, **생산성도 향상**된다.

---
