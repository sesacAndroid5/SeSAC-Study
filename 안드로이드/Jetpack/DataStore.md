# DataStore란?

Jetpack 라이브러리

기존의 SharedPreferences를 대체하기 위해 만들어진 데이터 저장 솔루션
<img width="884" height="183" alt="스크린샷 2025-07-31 오후 3 52 00" src="https://github.com/user-attachments/assets/54399bd8-42a4-49a3-80b8-0d3bad9de4d3" />

---

## ✅ 간단 정리
- Key - Value 데이터나 타입 세이프한 설정값을 저장하기 위한 Jetpack 라이브러리  
- **비동기/코루틴 기반** → SharedPreferences처럼 UI 스레드를 막지 않음  
- **Flow 제공** → Compose와 궁합이 좋음 (반응형 UI 가능)

---

## 📂 종류
1️⃣ **Preferences DataStore**  
- SharedPreferences와 비슷하게 Key - Value 형태로 데이터 저장  
- Key를 `PreferencesKey`로 정의 (예: `booleanPreferencesKey`, `stringPreferencesKey`)

2️⃣ **Proto DataStore**  
- ProtoBuf(Protocol Buffers) 사용 → 정해진 데이터 구조(스키마) 기반으로 타입 안정성 보장  
- 복잡한 구조(클래스, 중첩 데이터)를 안전하게 저장할 때 유리

---

## 🔍 SharedPreferences보다 DataStore가 좋은 이유
- 비동기 처리
- 타입 안정성 (Proto DataStore)
- Flow 지원 → UI와 자연스럽게 연결 가능
- 데이터 손상 가능성 적음

---

## 📌 사용법 (Preferences DataStore)

### 1️⃣ DataStore 정의
preferencesDataStore로 만든 속성 위임을 사용하여 DataStore<Preferences>의 객체를 생성
코틀린 파일의 최상위 수준에서 인스턴스 호출 후 앱의 나머지 부분에서 이 속성을 통해 객체에 접근
```kotlin
// At the top level of your kotlin file
val Context.dataStore: DataStore<Preferences> by preferencesDataStore(name = "user_prefs")
```

### 2️⃣ Key 정의
```kotlin
object UserKeys {
    // Boolean 값에 대응하는 키값 설정
    val IS_LOGGED_IN = booleanPreferencesKey("is_logged_in")
    // String 값에 대응하는 키값 설정
    val USER_NAME = stringPreferencesKey("user_name")
}
```

### 3️⃣ 데이터 읽기 (Flow)
```kotlin
// .data 속성을 사용하여 Flow로 저장 값을 읽기
val userNameFlow: Flow<String> = context.dataStore.data
    .map { prefs -> prefs[UserKeys.USER_NAME] ?: "" }
```

### 4️⃣ 데이터 쓰기
```kotlin
suspend fun saveUserName(name: String) {
    // .edit 함수를 통해 데이터를 트랜잭션 방식으로 업데이트
    context.dataStore.edit { prefs ->
        prefs[UserKeys.USER_NAME] = name
    }
}
```

---
### 📌 **DataStore를 쓸 때 꼭 지켜야 할 규칙 요약**

1️⃣ **같은 파일로 DataStore 인스턴스를 두 번 만들지 말 것**

- 같은 프로세스에서 동일한 파일(`user_prefs.pb` 같은 파일명)을 DataStore로 두 번 이상 만들면 ❌
- 이렇게 하면 **데이터 읽기/쓰기 중에 `IllegalStateException`** 발생

---

2️⃣ **DataStore에 쓰는 데이터 타입은 “불변(immutable)” 이어야 함**

- DataStore에 저장하는 타입을 변경하면(Data class 변경 등) DataStore의 안정성이 깨짐
- **Proto DataStore(Protocol Buffers)** 사용을 권장 → 자동으로 불변성 & 직렬화 제공
- ✅ **즉, DataStore는 “읽은 값이 외부에서 수정되지 않는다”는 전제를 깔고 설계됨**

---

3️⃣ **SingleProcessDataStore와 MultiProcessDataStore를 같은 파일에 혼용 금지**

- **앱이 여러 프로세스**에서 DataStore에 접근한다면 **MultiProcessDataStore**만 사용해야 함
- 두 개를 섞으면 데이터 충돌 가능성 있음

| 구분 | **SingleProcessDataStore** | **MultiProcessDataStore** |
| --- | --- | --- |
| **전제 조건** | 한 프로세스에서만 접근 | 여러 프로세스에서 동시에 접근 가능 |
| **동기화 처리** | 간단한 락(lock)만 사용 | 프로세스 간 IPC 락, 파일 동기화 필요 |
| **캐시 메커니즘** | 메모리 내 캐시만으로 충분 | 파일 기반 동기화 + 캐시 관리 필요 |
| **속도** | 빠름 (락, 동기화 부담 거의 없음) | 다소 느림 (락·IPC 동기화 비용 있음) |
| **사용 사례** | 기본적인 앱 (Activity, Fragment 등) | 위젯, 백그라운드 서비스, 다중 프로세스 앱 |
| **비유** | **개인 다이어리** – 혼자 쓰니까 잠금장치 필요 없음 → **빠름** | **공동 일기** – 여러 명이 쓰니까 자물쇠 걸고 열쇠 주고받으며 작성 → **안전하지만 느림** |
---

## 🛠 Gradle 의존성
```kotlin
// Preferences DataStore
implementation("androidx.datastore:datastore-preferences:1.1.7")

// Proto DataStore
implementation("androidx.datastore:datastore:1.1.7")
```

---

## 🗂 Protocol Buffers(ProtoBuf)
구글이 만든 데이터 직렬화 기술

- JSON, XML보다 **작고 빠름**
- .proto 파일로 데이터 스키마 정의 후 코드 자동 생성
- 타입 안정성 제공

```proto
syntax = "proto3";

message User {
  string name = 1;
  int32 age = 2;
  bool is_active = 3;
}
```

---

## 🔵 트랜잭션(Transaction) 방식

- **edit { }** 블록 내부 작업은 **하나의 트랜잭션**으로 처리
- 중간 오류 시 → **롤백(rollback)**
- 데이터 일관성(Consistency) 보장

✅ **장점**
- 중간 저장 없이 “모두 저장 or 전혀 저장 안함” 보장
- 스레드 안전(Thread-Safe)

