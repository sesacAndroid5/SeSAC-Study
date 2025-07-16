# SOLID 원칙과 MVVM 패턴의 적용

## SOLID란?

객체 지향 설계의 5가지 원칙으로, 유지보수성과 확장성을 높이는 데 도움을 준다.

- **S**: 단일 책임 원칙 (Single Responsibility Principle)
- **O**: 개방-폐쇄 원칙 (Open/Closed Principle)
- **L**: 리스코프 치환 원칙 (Liskov Substitution Principle)
- **I**: 인터페이스 분리 원칙 (Interface Segregation Principle)
- **D**: 의존 역전 원칙 (Dependency Inversion Principle)

---

## MVVM 패턴 구성

- **Model**: 데이터 계층. Repository, UseCase 등 비즈니스 로직 포함
- **ViewModel**: 상태 및 UI 로직 처리, View에 데이터 제공
- **View (Compose/Activity/Fragment)**: UI 표현. 상태 관찰만 수행

---

## SOLID 원칙별 MVVM 적용 예시

### 1. 단일 책임 원칙 (SRP)
- **정의**: 클래스는 하나의 책임만 가져야 한다.
- **MVVM 적용**:
  - ViewModel은 UI 상태 관리만 담당
  - UseCase는 도메인 로직만 수행
  - Repository는 데이터 소스 처리만 담당

> 예시:  
> `LoginViewModel`은 로그인 상태만 관리  
> `LoginUseCase`는 로그인 검증/처리만 수행  

---

### 2. 개방-폐쇄 원칙 (OCP)
- **정의**: 확장에는 열려 있고, 변경에는 닫혀 있어야 한다.
- **MVVM 적용**:
  - 새로운 기능은 클래스를 수정하지 않고 추가 클래스/컴포넌트로 구현
  - sealed class, interface 등을 통해 확장성 확보

> 예시:  
> `sealed class LoginEvent`를 도입해 새로운 이벤트 확장 가능  
> 기존 ViewModel 코드는 그대로 유지

---

### 3. 리스코프 치환 원칙 (LSP)
- **정의**: 자식 클래스는 부모 클래스의 행동을 대체할 수 있어야 한다.
- **MVVM 적용**:
  - Repository 구현체가 인터페이스 계약을 준수해야 함
  - 테스트 시 mock 대체가 가능해야 함

> 예시:  
> `AuthRepositoryImpl`은 `AuthRepository`를 완전히 대체 가능해야 함

---

### 4. 인터페이스 분리 원칙 (ISP)
- **정의**: 하나의 큰 인터페이스보다는 여러 개의 구체적인 인터페이스가 낫다.
- **MVVM 적용**:
  - Repository나 UseCase 인터페이스는 역할별로 분리
  - 필요한 기능만 주입받도록 구성

> 예시:  
> `AuthRepository`와 `ProfileRepository`를 분리하여 관심사 분리

---

### 5. 의존 역전 원칙 (DIP)
- **정의**: 고수준 모듈은 저수준 모듈에 의존하면 안 되고, 둘 다 추상화에 의존해야 한다.
- **MVVM 적용**:
  - ViewModel → Repository/UseCase는 인터페이스를 통해 의존
  - 실제 구현체는 DI(Hilt, Koin 등)로 주입

> 예시:  
> `LoginViewModel(authUseCase: AuthUseCase)`  
> → 실제 구현체는 Hilt/Koin으로 주입

---

## 정리

SOLID 원칙을 MVVM에 적용하면 유지보수하기 쉬운 구조를 만들 수 있다.  
아키텍처 설계 시 각 계층의 책임과 경계를 명확히 하여 **테스트**, **확장성**, **가독성** 모두 확보할 수 있다.
