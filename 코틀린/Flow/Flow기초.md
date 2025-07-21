## Kotlin의 Flow란?

Flow는 비동기 스트림(데이터 흐름) 을 처리하기 위한 Kotlin 코루틴의 구성요소이다.
쉽게 말해,
"시간이 지나면서 하나씩 나오는 값을 순서대로 처리하는 파이프라인"이라고 생각하면 된다. 

### 🚘 자동차 조립 공장 vs Kotlin Flow
| 자동차 조립 공장                    | Kotlin Flow                              |
| ---------------------------- | ---------------------------------------- |
| 각 작업자가 정해진 부품을 조립하며 차량을 완성함  | 각 연산자가 데이터를 가공하며 최종 결과를 생성함              |
| 벨트 위에서 차량이 앞 단계 → 다음 단계로 이동  | Flow는 upstream → downstream으로 데이터를 흐름 처리 |
| 중간에서 품질 검사 또는 분기 작업도 가능      | Flow에서 `filter`, `map`, `takeWhile` 등 가능 |
| 공정마다 지연이 생길 수 있음 (예: 도장, 건조) | `delay`나 `suspend`로 비동기 작업 처리 가능         |
| 생산 완료 시 출고되며 기록이 남음          | `collect`을 호출하면 실제로 데이터가 소비됨             |

#### 🔧 단계별 비교 (Step by Step)

1. 🚗 초기 부품이 조립 라인에 투입됨 → flow { emit(...) }

```kotlin
val carAssemblyLine = flow {
    emit("프레임") 
    emit("엔진")
    emit("바퀴")
}
```

2. 🔧 각 공정에서 부품을 가공하거나 조립함 → map, filter
 
```kotlin
carAssemblyLine
    .map { part -> "도색된 $part" }      // 도장 공정
    .filter { part -> part != "도색된 엔진" } // 엔진은 도색 제외
```

3. 🧪 검사 및 테스트 → takeWhile, onEach

```kotlin
carAssemblyLine
    .onEach { println("$it 검사 중...") } // 중간 검사
    .takeWhile { it != "바퀴" }           // 바퀴까지 조립하고 중단
```

4. 🕐 대기/지연 → delay, flowOn
 
```kotlin
carAssemblyLine
    .map {
        delay(1000) // 공정 시간
        it
    }
```

5. ✅ 최종 출고 → collect { ... }
6. 
```kotlin
carAssemblyLine.collect { println("$it 출고 완료") }
```

#### ✅ 요약
Kotlin의 Flow는 자동차 조립 공장의 벨트형 파이프라인처럼 동작한다.

각 공정(연산자)은 데이터를 순차적으로 가공하고, 최종적으로 collect를 해야 비로소 조립이 완료된다.

즉, **Flow는 "조립 라인을 설계하는 것"이고, collect는 "기계를 돌리는 스위치"**이다.

### Flow를 사용하는 이유 
- 비동기 데이터 처리가 필요할때 (ex: API 응답, DB스트림, UI상태 변홛 등등)
- LiveData보다 유연한 흐름제어가 가능하다. 
