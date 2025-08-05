# CoroutineDispatcher란?

**Dispatcher**란 Dispatch (보내다) + -er = 무언가를 보내는 주체.  
즉 **CoroutineDispatcher**는 ‘코루틴을 스레드로 보내는 주체’를 의미합니다.  
코루틴은 일시 중단이 가능한 ‘작업’이기 때문에 스레드로 보내야 실행이 가능합니다.

- 사용자가 코루틴 작업 요청  
→ `CoroutineDispatcher`가 실행 요청받은 코루틴을 작업 대기열에 적재  
→ 관리하는 스레드 중 비어있는 스레드가 있으면 작업 할당

➡️ 코루틴의 실행을 관리하는 주체로, 작업을 대기열에 적재해 있다가 자신이 관리하는 스레드에 코루틴을 실행시킵니다.

---

## 제한된 디스패처 vs 무제한 디스패처

- **제한된 디스패처**: 사용할 수 있는 스레드나 스레드풀이 제한된 디스패처  
- **무제한 디스패처**: 사용할 수 있는 스레드나 스레드풀이 제한되지 않은 디스패처

---

## 디스패처 생성 및 코루틴 작업 할당 예시

```kotlin
fun main() = runBlocking<Unit>(context = CoroutineName("Main")) {
    println("${Thread.currentThread().name} runBlocking 코루틴 실행")

    val dispatcher: CoroutineDispatcher = newSingleThreadContext(name = "myThread")
    val multiDispatcher: CoroutineDispatcher = newFixedThreadPoolContext(nThreads = 3, name = "multiThread")

    launch(context = dispatcher) {
        println("${Thread.currentThread().name} 실행")
    }

    launch(context = multiDispatcher) {
        println("${Thread.currentThread().name} 멀티 1 실행")
        Thread.sleep(1000)
    }
    launch(context = multiDispatcher) {
        println("${Thread.currentThread().name} 멀티 2 실행")
    }
}
```

**실행 결과**

```
main @Main#1 runBlocking 코루틴 실행
myThread @Main#2 실행
multiThread-1 @Main#3 멀티 1 실행
multiThread-2 @Main#4 멀티 2 실행
```

---

## 부모 코루틴의 CoroutineDispatcher 사용

- 코루틴은 내부에 자식 코루틴을 생성 가능
- 자식 코루틴에 dispatcher를 지정하지 않으면 부모의 dispatcher를 사용함

➡️ 같은 Dispatcher에서 여러 작업을 실행할 경우, **부모에 Dispatcher를 지정하고 자식 코루틴을 여러 개 만들자.**

---

## 직접 CoroutineDispatcher를 생성하는 것은 비효율적

### 단점

- 각 Dispatcher마다 스레드풀이 생성되므로 비효율
- 각 개발자가 Dispatcher를 만들면 리소스 낭비 (스레드 생성 비용이 높음)

➡️ **미리 정의된 CoroutineDispatcher 사용 권장**

---

## 주요 CoroutineDispatcher

- `Dispatchers.IO`: 네트워크 요청, 파일 입출력 등 I/O 작업에 최적화된 Dispatcher
- `Dispatchers.Default`: CPU를 많이 사용하는 연산 작업에 적합한 Dispatcher
- `Dispatchers.Main`: UI 작업을 위한 메인 스레드 Dispatcher (Android 환경에서 사용, 라이브러리 추가 필요)

---

## 입출력 vs 연산 작업 차이

- **입출력 작업**: 결과 반환까지 스레드가 비워져 다른 작업이 가능
- **연산 작업**: 지속적인 처리 필요 → 스레드를 계속 점유함

➡️ 입출력은 **코루틴 기반**에서 유리함, 연산은 큰 차이 없음

---

## limitedParallelism

스레드 수를 제한하여 병렬성을 제어할 수 있음

```kotlin
launch(Dispatchers.Default.limitedParallelism(2)) {
    repeat(10) {
        launch {
            println("${Thread.currentThread().name} Dispatcher.Default 에서 실행")
        }
    }
}
```

- `Dispatchers.IO.limitedParallelism(3)` 도 가능하지만  
기존 IO 스레드가 아닌 Default에서 스레드를 가져오므로 주의 필요

➡️ **특정 작업이 다른 작업에 영향 주지 않아야 할 때만 사용, 과도한 사용 지양**

---

## Dispatchers.Main

- UI 변경 등에 사용되는 메인 스레드용 Dispatcher  
- 사용하려면 `kotlinx-coroutines-android` 라이브러리 추가 필요
