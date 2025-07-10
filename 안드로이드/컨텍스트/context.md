# 📱 Android에서의 Context란?

## 1. Context란?

`Context`는 Android 시스템에서 구현된 추상 클래스(Abstract Class)로,  
앱과 안드로이드 시스템이 **소통하기 위한 통로** 역할을 한다.  
즉, 앱의 환경 정보에 접근할 수 있는 **마스터 키** 같은 존재이다.

> 📌 Context가 없다면 앱은 아무것도 할 수 없다.  
> 예: 리소스 접근, 화면 전환, 시스템 알림, 파일/DB 작업 등

---

## 2. Context의 상속 구조

안드로이드의 핵심 컴포넌트(`Application`, `Activity`, `Service`)들은 모두 Context를 상속받고 있다.  
실제로는 `ContextWrapper`를 상속하고 있으며, 내부적으로는 `ContextImpl`이 기능을 구현하고 있다.

```
Context (추상 클래스)
 ├── ContextImpl        // 실제 기능을 구현한 클래스
 └── ContextWrapper     // ContextImpl을 감싸는 Wrapper
       └── Application / Activity / Service ...
```

따라서, `Activity`나 `Service` 안에서는 `this` 키워드로 `Context`를 사용할 수 있다.

---

## 3. Context가 제공하는 주요 기능

| 기능 | 설명 |
|------|------|
| 🔹 리소스 접근 | `getString()`, `getDrawable()`, `getResources()` 등 |
| 🔹 시스템 서비스 접근 | `getSystemService(Context.CONNECTIVITY_SERVICE)` 등 |
| 🔹 애플리케이션 전반 제어 | 화면 전환, 브로드캐스트, 인텐트 전송 등 |
| 🔹 파일 및 데이터베이스 접근 | `openFileInput()`, `getSharedPreferences()` 등 |

---

## 4. Context 사용 시 주의사항 ⚠️ (메모리 누수)

Context를 잘못 사용하면 **치명적인 메모리 누수**가 발생할 수 있다.

### 예시: 싱글톤 객체에 Activity Context 저장 ❌

```kotlin
object SingletonManager {
    var context: Context? = null  // ❌ Activity Context 참조 → 메모리 누수 발생 가능
}
```

- `SingletonManager`는 앱이 종료될 때까지 메모리에 남아 있습니다.
- 내부에 `Activity`를 참조하면, Activity가 종료되었음에도 GC가 해제하지 못합니다.
- 결국 `Activity` 및 그 안의 리소스들이 메모리에 붙잡혀 앱 메모리 누수로 이어집니다.

---

🔚 정리

Context란?	시스템과 앱이 소통할 수 있게 해주는 추상 클래스
상속 구조	Context → ContextWrapper → Activity, Service, Application
주요 기능	리소스, 시스템 서비스, 파일, DB 등 앱 전반 제어
주의사항	Activity Context를 잘못 보관하면 메모리 누수 발생
