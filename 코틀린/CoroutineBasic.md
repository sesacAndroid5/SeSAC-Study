# 📌 [Coroutine] 코루틴의 개념, 구조, 스레드와의 차이, 사용법 정리

---

## 1. ✅ Coroutine 등장 배경: AsyncTask의 한계

- 과거에는 **AsyncTask**를 통해 손쉽게 비동기 작업 처리 가능
- 그러나 다음과 같은 문제로 인해 Android API 30에서 `Deprecated`
  - 메모리 누수
  - 복잡한 스레드 처리
  - Lifecycle 미연동
- **대체제로 Kotlin Coroutine**이 강력히 권장됨

---

## 2. ✅ 루틴(Routine) 개념 정리

###  메인 루틴 (Main Routine)
- `main()` 함수에서 시작되는 함수의 흐름
  
###  서브 루틴 (sub Routine)
- 메인 함수에서 안에서 실행되는 개별 함수
  
### 🌀 코루틴 (Coroutine)
- `suspend` 키워드를 사용하여 중간에 **일시정지(suspend)** 가능
- **재시작(resume)** 지점이 존재하는 함수 흐름
- **Thread가 아닌 메모리 기반 흐름**  
→ “함수에 가까운 비동기 단위”

---

## 3. ✅ Coroutine vs Thread

| 항목 | Coroutine | Thread |
|------|----------|--------|
| 메모리 구조 | 프로세스의 힙 메모리 공유 | 각 스레드마다 스택 메모리 할당 |
| 실행 방식 | **비선점형 멀티태스킹** | **선점형 멀티태스킹** |
| Context Switching | 없음 (동일 메모리) | 있음 (스택 전환 필요) |
| 동시성 | 있음 (빠른 전환) | 있음 |
| 병행성 | 없음 | 있음 |
| 오버헤드 | 작음 | 큼 |
| 생성/삭제 비용 | 낮음 | 높음 |

> 코루틴은 빠른 전환으로 **동시성은 있지만 병렬성은 없다**  
> 한 스레드에서 여러 코루틴을 실행 가능

| Thread | Coroutine |
|:--:|:--:|
|<img width="250" height="250" alt="스크린샷 2025-08-03 오후 9 52 51" src="https://github.com/user-attachments/assets/1b31a6df-e189-44c8-991a-2a2fd3c3de4f" />|<img width="250" height="250" alt="스크린샷 2025-08-03 오후 9 53 58" src="https://github.com/user-attachments/assets/a283a0fb-b576-4278-ba96-547a8cba03f8" />|
|스레드는 동일한 시간에 동시에 작업을 할 수 있다. 이걸 병행성이라고 한다|코루틴은 동일한 시간에 동시에 작업이 진행되지 않는다. 하나가 실행하면 하는 멈추고 다만 코루틴이 전환되면서 수행되는 속도가 빨라서 외부에서 볼떄는 동시에 수행되는것처럼 보인다. 따라서 동시성은 있지만 병행성은 없다.|
---

## 4. ✅ Coroutine 구조

### 📌 1) Coroutine Scope
 <img width="623" height="100" alt="스크린샷 2025-08-03 오후 10 07 12" src="https://github.com/user-attachments/assets/57fbeb96-8324-4aeb-8fdf-09d385ee3e6c" />
 
- **Coroutine Scope**는 인터페이스 이고 스코프가 결정되면 특정한 디스패처를 지정해서 동작이 실행될 영역을 제한할 수 있다
- **코루틴이 살아있는 범위**
- 가장 큰 portion이다.
- 대표 Scope:
  - `CoroutineScope`
  - `GlobalScope` (지양)

```kotlin
CoroutineScope(Dispatchers.IO).launch {
    // 코루틴 본문
}
```

### 📌 2) Coroutine Context
- **디스패처 + Job 등 코루틴의 실행 환경 구성**
- 주요 Dispatcher:
  - `Dispatchers.Main`: UI 작업
  - `Dispatchers.IO`: 네트워크, 파일 I/O
  - `Dispatchers.Default`: CPU 연산
  - `Dispatchers.Unconfined`: 코루틴이 자신을 생성한 스레드에서 즉시 실행

- **Job & Deferred**:
  - `Job`: 코루틴 자체를 객체화
  - `Deferred`: 값 반환 가능한 Job

```kotlin
val job = scope.launch { ... }
val deferred = scope.async { return@async 123 }
```

### 📌 3) Coroutine Builder
- 코루틴을 실행하는 함수

| Builder | 설명 | 반환값 |
|---------|------|--------|
| `launch` | 값 반환 X | `Job` |
| `async` | 값 반환 O | `Deferred` |
| `runBlocking` | 메인 스레드 BLOCK | 지양 |
| `withContext` | 디스패처 변경 | 결과값 |

---

## 5. ✅ Coroutine LifeCycle
<img width="380" height="164" alt="New (optional initial state)" src="https://github.com/user-attachments/assets/44996b62-b0c9-4911-9398-d318117b0944" />

<img width="700" height="350" alt="1sdb3Y6ckQGLcuekR8RD4Ew" src="https://github.com/user-attachments/assets/979b4130-ae1a-4db8-8072-132f9f445db7" />

- 코루틴은 일시정지 시킬 수 있는 작업의 흐름이기 떄문에 Job은 코루틴의 여러가지 상태를 반영할 수 있도록 설계되어 있다. 
- 여러 state가 있고 state마다 Flag가 있어서 현재 어떤 State에 Job이 있는지 Flag를 체크함으로써 확인을 할 수 있다.
- 
### Job 메서드
- start: 새로만들어진 객체를 시작
- cancel: 작업중인 코루틴을 Cancelling으로 바꾼다
- join: cancel된 작업을 병렬처리하지 않고 완전히 끝날때까지 기다리게 하는 메서드 
---

## 6. ✅ Coroutine 사용법 요약

### Suspend Function
```kotlin
suspend fun fetchData(): String { ... }
```

- `suspend` → 일시정지 가능한 함수
- `launch`, `async`, `withContext` 안에서만 호출 가능

---

## 7. ✅ Coroutine 지연 처리

| API | 설명 |
|-----|------|
| `delay(ms)` | 일정 시간 동안 지연 (`sleep`과 다름) |
| `join()` | `launch`된 Job이 종료될 때까지 대기 |
| `await()` | `async`된 Deferred 값 기다리기 |

---

## 8. ✅ Coroutine 취소 처리

| 메서드 | 설명 |
|--------|------|
| `cancel()` | 코루틴을 `Cancelling` 상태로 |
| `cancelAndJoin()` | 취소 후 완료될 때까지 대기 |
| `withTimeout(ms)` | 시간 초과 시 `TimeoutCancellationException` 발생 |
| `withTimeoutOrNull(ms)` | 시간 초과 시 `null` 반환 (Exception 없음) |

---

## 9. ✅ Coroutine 예외 처리

- `CoroutineExceptionHandler` 사용
- `launch`/`actor`: 예외 발생 시 즉시 전파
- `async`/`produce`: `await()` 시 예외 발생

### ExceptionHandler 예시
```kotlin
val handler = CoroutineExceptionHandler { _, throwable ->
    Log.e("Exception", throwable.message ?: "")
}

scope.launch(handler) {
    throw RuntimeException("Test")
}
```

> Structured Concurrency 원칙에 따라  
> **자식 Coroutine 예외 발생 시 부모도 모두 취소됨**

---

## 10. ✅ 요약

- **Coroutine**은 비동기 작업을 구조적이고 메모리 효율적으로 처리할 수 있게 해주는 Kotlin 표준 도구
- **스레드 대비 메모리 사용량이 낮고, 구조적 동시성 제공**
- `launch`와 `async`로 작업 정의, `suspend` 함수와 `withContext`로 흐름 제어
- 취소, 예외, 스코프 등 **Lifecycle 관리가 용이**

---

## 🔗 참고 자료
- [Kotlin Coroutine 공식 문서](https://kotlinlang.org/docs/coroutines-overview.html)
- [Google Coroutine Best Practices](https://developer.android.com/kotlin/coroutines)
