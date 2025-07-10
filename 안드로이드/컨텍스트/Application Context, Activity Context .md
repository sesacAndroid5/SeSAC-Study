# 🔍 Application Context vs Activity Context

안드로이드 개발에서 Context는 매우 자주 사용되며, 그 중에서도 `Application Context`와 `Activity Context`는 사용 목적과 수명이 다르다.  
잘못 사용할 경우 **메모리 누수(memory leak)** 등 심각한 문제가 발생할 수 있으므로, 각각의 차이를 정확히 이해하는 것이 중요하다.

---

## 1. Application Context란?

- `Application` 클래스에서 가져오는 Context (`applicationContext`)
- **앱 전체의 수명 동안 유지**되며, 모든 컴포넌트에서 동일한 인스턴스
- UI와 관련된 기능은 포함하지 않음
- 시스템 리소스, DB, SharedPreferences 등 앱 전반의 기능에 접근할 수 있음

### ✅ 특징
- 앱이 실행되는 동안 계속 살아있음
- 메모리 누수 위험이 적음
- UI를 만들거나 띄울 수 없음 (예: Toast는 가능하지만 Dialog는 안 됨)
---

## 2. Activity Context란?

- 현재 실행 중인 `Activity` 인스턴스에서 가져오는 Context (`this`)
- 해당 Activity의 생명 주기에 종속됨
- **UI와 관련된 작업**에 적합

### ✅ 특징
- Activity가 종료되면 GC에 의해 메모리에서 해제됨
- UI 리소스에 접근 가능
- **사용 시점이 Activity의 생명 주기를 초과하면 메모리 누수 위험**

---

## 3. 차이점 비교

| 항목 | Application Context | Activity Context |
|------|---------------------|------------------|
| 수명 | 앱 전체 | 특정 Activity 생명 주기 |
| UI 접근 | ❌ (불가능) | ✅ (가능) |
| 안전성 | 상대적으로 안전 | 생명 주기 관리 필요 |
| 사용 예 | DB, SharedPreferences, Toast 등 | View, Dialog, LayoutInflater 등 |
| 메모리 누수 위험 | 낮음 | 높음 (싱글톤에서 보관 시 위험) |

---

## 4. 언제 어떤 Context를 써야 할까?

### ✅ Application Context를 써야 할 때
- 컴포넌트의 수명보다 긴 객체에서 Context가 필요할 때 (ex. 싱글톤, ViewModel 등)
- UI와 무관한 작업: SharedPreferences, 파일 접근, DB 접근, Toast

### ✅ Activity Context를 써야 할 때
UI 작업이 필요한 경우
해당 작업이 Activity 생명 주기 내에만 사용될 경우


### 🔚 정리

UI 작업 → Activity Context

앱 전역, 리소스 접근, 싱글톤 객체 → Application Context

📌 항상 "이 Context가 얼마나 오래 살아남을지?"를 고려해서 선택해야한다.
특히 싱글톤이나 static 객체에 Context를 보관할 땐 절대 Activity Context를 저장하면 안된다.

