# UseCase를 사용해야 하는 경우와 이유

유스케이스(UseCase, Interactor)는 **도메인 계층**의 핵심 요소로, 애플리케이션의 특정 비즈니스 로직을 단일 책임으로 캡슐화한 클래스입니다.  
이를 통해 ViewModel은 UI 상태 관리에만 집중하고, 비즈니스 로직은 유스케이스에서 처리하여 코드의 유지보수성과 재사용성을 높입니다.

---

## 1️⃣ 계층 구조와 흐름

일반적인 MVVM + 클린 아키텍처 흐름은 다음과 같습니다.

View → ViewModel → UseCase → Repository → DataSource

- **View**: UI를 렌더링하고 사용자 입력을 ViewModel로 전달
- **ViewModel**: UI 상태 관리, 유스케이스 호출
- **UseCase**: 단일 비즈니스 로직을 캡슐화 (재사용성과 테스트 용이성 제공)
- **Repository**: 데이터 소스를 추상화 (API, DB 등)
- **DataSource**: 실제 네트워크나 DB 접근을 담당

### 단순 작업일 경우
작업이 단순하다면 UseCase를 생략하고 ViewModel에서 Repository를 직접 호출하기도 합니다.

View → ViewModel → Repository → DataSource

예) 단순히 `userRepository.getUser(userId)`를 호출해 UI에 표시할 때

---

## 2️⃣ UseCase를 사용하는 이유

### 복잡한 작업의 기준

| 상황 | 설명 | 예시 |
|------|------|------|
| 여러 Repository 조합 | 2개 이상의 데이터 소스를 합쳐야 할 때 | `UserRepository`에서 사용자 정보 → `ProductRepository`에서 재고 확인 → `OrderRepository`에서 주문 생성 |
| 복잡한 비즈니스 규칙 | 순차적, 조건부 로직이 필요할 때 | 결제 시 할인 쿠폰 적용 → 포인트 적립 → 배송비 계산 → 결제 승인 |
| 비동기 흐름 제어 | API 순차 호출, 조건부 실행 필요 | 로그인 후 토큰 저장 → 사용자 정보 요청 |
| 여러 ViewModel에서 재사용 | 같은 기능을 여러 화면에서 공유 | `FetchUserInfoUseCase`에서 유저 정보 가져오기 → 프로필, 설정 화면 등에서 공통적으로 호출 |
| 데이터 가공 및 변환 | 데이터 후처리 필요 | API 응답 데이터를 도메인 모델로 매핑해 추가 계산이나 필터링 수행, 여러 API 호출 후 평균값이나 합계를 계산해서 통계 데이터 집계 |

### 단순 작업의 기준
- Repository 호출만으로 충분한 경우
- 비즈니스 규칙이 거의 없는 경우
- 로직이 한 ViewModel에서만 사용되는 경우

---

## 3️⃣ 여러 ViewModel에서 재사용되는 경우의 장점

### 유스케이스 없는 경우
ViewModel A → Repository

ViewModel B → Repository

- 공통 로직을 각각의 ViewModel에서 구현
- 예외 처리, 데이터 가공 로직이 중복 발생
- 로직 변경 시 모든 ViewModel을 수정해야 함 → 유지보수 부담

### 유스케이스 사용하는 경우
ViewModel A → UseCase → Repository

ViewModel B → UseCase → Repository

- 공통 로직을 UseCase에서 캡슐화
- ViewModel에서는 결과만 받아 UI 반영
- 로직 변경 시 UseCase만 수정하면 모든 ViewModel에 반영

> 예) `AddBookmarkUseCase`에서 에러 처리 및 데이터 변환 로직을 수행 → 각 ViewModel은 성공 여부만 확인

---

## 4️⃣ 정리
- 단순 작업은 ViewModel → Repository로도 충분

- 복잡한 작업은 UseCase를 통해 로직을 캡슐화하여 재사용성과 유지보수성을 확보

- UseCase는 UI와 데이터 계층을 분리해 도메인 로직을 안정적으로 보호하는 핵심 계층

