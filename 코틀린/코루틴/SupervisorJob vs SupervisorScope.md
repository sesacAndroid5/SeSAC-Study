# SupervisorJob vs SupervisorScope 정리
출처: [10분 테코톡](https://youtu.be/3DNbRnl0im4?si=0wBxMGTw97vPJuyN)

참고: [velog](https://velog.io/@murjune/kotlin-Coroutine-supervisorScope-vs-SupervisorJob-%EC%96%B4%EB%96%A4%EA%B1%B8-%EC%82%AC%EC%9A%A9%ED%95%98%EB%9D%BC%EB%8A%94%EA%B1%B0%EC%A7%80)

## SupervisorJob
- 단순히 `Job`의 일종으로, **자식 코루틴 간 예외 전파를 막는 역할**을 한다.  
- 그냥 생성하면 **root Job**이 되며, 부모와 연결되지 않는다. → 필요하다면 `parent`를 명시적으로 설정해야 한다.  
- 모든 자식이 끝나더라도 자동으로 완료되지 않는다.  
  → 따라서 명시적으로 `cancel()`하거나 `complete()` 호출이 필요하다.  
- 일반적으로는 `CoroutineScope(SupervisorJob() + Dispatcher)` 형태로 사용하며, 스코프의 생명주기는 `cancel()` 호출로 제어한다.  
- **사용 위치**: 최상위 스코프 생성 시

```kotlin
val scope = CoroutineScope(SupervisorJob() + Dispatchers.Main)

scope.launch {
    // 자식1
}

scope.launch {
    // 자식2
    throw RuntimeException("예외 발생") // 다른 자식에게 전파되지 않음
}

// 모든 자식이 끝나도 scope는 자동으로 완료되지 않음
// 필요 시 명시적으로 scope.cancel() 또는 scope.coroutineContext[Job]?.complete()
scope.cancel()
```

---

## SupervisorScope
- 내부적으로 SupervisorJob을 포함한다.
- 부모와 자동으로 연결되므로 parent를 수동으로 지정할 필요가 없다.
- 자식 코루틴이 모두 완료되면 해당 스코프도 자동으로 완료 상태가 된다.
- 사용 위치: suspend 함수 내부나 이미 존재하는 코루틴 안에서 일시적으로 독립적인 자식 코루틴 그룹을 묶고 싶을 때

```kotlin
suspend fun fetchData() = supervisorScope {
    launch {
        // 자식1
    }

    launch {
        // 자식2
        throw RuntimeException("예외 발생") // 다른 자식에게 전파되지 않음
    }
    // 모든 자식이 끝나면 supervisorScope 도 자동 종료됨
}
```

---

## ViewModelScope
- viewModelScope는 내부적으로 SupervisorJob을 사용한다.
- 따라서 자식 코루틴 중 하나가 실패해도 다른 자식 코루틴에 영향을 주지 않는다.
- 별도의 complete() 호출이 필요하지 않으며, ViewModel의 onCleared() 시점에 자동으로 cancel()된다.

```kotlin
class MyViewModel : ViewModel() {
    fun loadData() {
        viewModelScope.launch {
            // 안전하게 여러 자식 코루틴 실행 가능
        }
    }
}
```

|구분 |	SupervisorJob	 | SupervisorScope |
|-----|--------|-------|
|부모 |연결	없음 → 필요시 parent 설정	|자동으로 부모와 연결|
|예외 |전파	자식 간 전파 없음|	자식 간 전파 없음|
|완료 처리|	자식이 끝나도 자동 완료되지 않음 → cancel/complete 필요|	자식 완료 시 자동 완료|
|주 사용 위치|	최상위 스코프 |	suspend 함수 내부, 블록 단위 독립 작업|
|예시|	CoroutineScope(SupervisorJob() + Dispatchers.Main)|	supervisorScope { ... }|

---

## 핵심 요약
- 최상위 스코프를 만들 때 → SupervisorJob (단, 명시적으로 cancel 또는 complete 호출 필요)
- 코루틴 블록 내부에서 독립적인 자식 작업을 묶고 싶을 때 → supervisorScope
- ViewModelScope는 내부적으로 SupervisorJob을 사용하므로 따로 신경 쓸 필요 없음
