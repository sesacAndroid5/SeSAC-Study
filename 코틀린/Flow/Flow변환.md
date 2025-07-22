## ColdFlow를 HotFlow로 변환하는 방법
기본 Flow는 Cold Stream이라 collect() 전까지 아무 일도 일어나지 않음

→ stateIn이나 shareIn을 사용하면 Hot Stream으로 변해, 즉시 값을 흘릴 수 있음

### 🧭 1. stateIn: Flow → StateFlow

#### ✅ 사용 시기
- UI 상태값을 지속적으로 유지하고 싶을 때

- 초기값이 필요하며 항상 최신값을 유지해야 할 때

#### 📌 주요 파라미터
| 파라미터           | 설명                            |
| -------------- | ----------------------------- |
| `scope`        | 코루틴 스코프 (보통 `viewModelScope`) |
| `started`      | 언제부터 실행할지 전략                  |
| `initialValue` | 초기값 (필수)                      |

#### SharingStarted 전략
- WhileSubscribed(timeout) → 구독자가 있을 때만 동작, 없으면 timeout 후 중단

- Eagerly → 즉시 실행 시작

- Lazily → 첫 구독자 생길 때까지 대기

#### 예시

```kotiln
fun flowDemo() {
    val flow = flow<Int> {
        println("Collection started!")
        delay(1000L)
        emit(1)
        delay(2000L)
        emit(2)
        delay(3000L)
        emit(3)
    }.stateIn(
        GlobalScope,
        SharingStarted.WhileSubscribed(),
        0
    )

    flow.onEach {
        println("Collector 1: $it")
    }.launchIn(GlobalScope)

    GlobalScope.launch {
        delay(5000L)
        flow.onEach {
            println("Collector 2: $it")
        }.launchIn(GlobalScope)
    }
}
```
### 🧭 2. shareIn: Flow → SharedFlow

#### ✅ 사용 시기
- 이벤트를 여러 곳에 동시에 공유해야 할 때

- Flow를 다중 소비자용 SharedFlow로 전환할 때

#### 예시

```kotiln
fun flowDemo() {
    val flow = flow<Int> {
        println("Collection started!")
        delay(1000L)
        emit(1)
        delay(2000L)
        emit(2)
        delay(3000L)
        emit(3)
    }.shareIn(
        GlobalScope,
        SharingStarted.Eagerly,
    )

    flow.onEach {
        println("Collector 1: $it")
    }.launchIn(GlobalScope)

    GlobalScope.launch {
        delay(5000L)
        flow.onEach {
            println("Collector 2: $it")
        }.launchIn(GlobalScope)
    }
}
```

#### 📌 주요 파라미터
| 파라미터      | 설명                    |
| --------- | --------------------- |
| `scope`   | 코루틴 스코프               |
| `started` | 언제부터 실행할지 전략          |
| `replay`  | 새 구독자에게 재전송할 이전 이벤트 수 |

### stateIn vs shareIn 요약 비교
| 항목       | `stateIn`       | `shareIn`               |
| -------- | --------------- | ----------------------- |
| 변환 결과    | `StateFlow`     | `SharedFlow`            |
| 초기값 필요   | ✅ 필요            | ❌ 불필요 (`replay`로 유사 기능) |
| 상태 저장 여부 | ✅ 항상 하나 저장      | ❌ 기본 없음, `replay`로 조절   |
| 용도       | 상태 유지           | 이벤트 공유                  |
| 대표 예시    | UI 상태 (`State`) | 알림, 네비게이션, 메시지          |
