## Flow의 종류 

### 1. Flow

#### ✅ 개념
- 가장 기본적인 Flow
- 데이터를 지연(lazy) 생성하고, 수집(collect) 되기 전까지 아무 일도 안 일어남

#### ✅ 특징
| 항목     | 값                             |
| ------ | ----------------------------- |
| 타입     | Cold Stream                   |
| 소비자 수  | 단일 (`collect`한 곳만 수신)         |
| 작동 조건  | `collect()` 호출 시 작동 시작        |
| 예시 사용처 | 리스트 가공, API 호출 결과 처리, DB 쿼리 등 |

#### ✅ 예시

```kotlin
flow<Int> {
            delay(1000L)
            emit(1)
            delay(2000L)
            emit(2)
            delay(3000L)
            emit(3)
        }.collect {
            println("Current value: $it")
        }
```

### 2. SharedFlow

#### ✅ 개념
- 여러 소비자에게 동시에 이벤트를 브로드캐스트할 수 있는 흐름
- collect() 하지 않아도 작동하는 Hot Stream

#### ✅ 특징
| 항목     | 값                             |
| ------ | ----------------------------- |
| 타입     | Hot Stream                    |
| 소비자 수  | 여러 개 가능                       |
| 상태 저장  | 기본 없음 (`replay` 설정으로 버퍼 가능)   |
| 예시 사용처 | 이벤트, 알림, 네비게이션 트리거, WebSocket |

#### ✅ 예시

```kotlin
fun sharedFlowDemo() {
    val sharedFlow = MutableSharedFlow<Int>(
        extraBufferCapacity = 3,
        onBufferOverflow = BufferOverflow.DROP_OLDEST
    )

    GlobalScope.launch {
        sharedFlow.onEach {
            println("Collector 1: $it")
            delay(5000L)
        }.launchIn(GlobalScope)

        sharedFlow.onEach {
            println("Collector 2: $it")
        }.launchIn(GlobalScope)
    }

    GlobalScope.launch {
        repeat(10) {
            delay(500L)
            sharedFlow.emit(it)
        }
    }
}
```

### 3. StateFlow

#### ✅ 개념
- 항상 현재 상태를 저장하고 있는 Hot Stream

- UI에서 ViewModel 상태를 관리할 때 많이 사용

#### ✅ 특징
| 항목     | 값                            |
| ------ | ---------------------------- |
| 타입     | Hot Stream                   |
| 소비자 수  | 여러 개 가능                      |
| 상태 저장  | 항상 최신값 1개 보존                 |
| 초기값 필요 | ✅ 반드시 필요                     |
| 예시 사용처 | 화면 상태(state), 폼 입력값, 로딩 상태 등 |

#### ✅ 예시

```kotlin
fun stateFlowDemo() {
    val stateFlow = MutableStateFlow(0)

    stateFlow.onEach {
        println("Value is $it")
    }.launchIn(GlobalScope)

    GlobalScope.launch {
        repeat(10) {
            delay(500L)
            stateFlow.value = it
        }
        stateFlow.onEach {
            println("Value from collector 2 is $it")
        }.launchIn(GlobalScope)
    }
}
```

### 4. callbackFlow

#### ✅ 개념
- 콜백 기반 API(예: 위치 업데이트, 센서 이벤트)를 Flow로 감싸는 도구

- 비동기 콜백을 emit으로 전환할 수 있게 해줌

#### ✅ 특징

| 항목    | 값                                      |
| ----- | -------------------------------------- |
| 타입    | Cold Stream                             |
| 초기값   | ❌ 필요 없음                                |
| 값 유지  | ❌ (기본)                                 |
| 소비자 수 | 보통 1명 (1:1 구조)                         |
| 대표 용도 | 콜백 기반 API → Flow 변환 (예: 위치, 센서, BLE 등) |

#### ✅ 예시
```kotlin
class LocationObserver(
    private val context: Context
) {
    private val client = LocationServices.getFusedLocationProviderClient(context)

    fun observeLocation(interval: Long): Flow<Location> {
        return callbackFlow {
            println("Flow is being collected.")
            val locationManager = context.getSystemService<LocationManager>()!!

            var isGpsEnabled = false
            var isNetworkEnabled = false

            while(!isGpsEnabled && !isNetworkEnabled) {
                isGpsEnabled = locationManager.isProviderEnabled(LocationManager.GPS_PROVIDER)
                isNetworkEnabled = locationManager.isProviderEnabled(LocationManager.NETWORK_PROVIDER)

                if(!isGpsEnabled && !isNetworkEnabled) {
                    delay(3000L)
                }
            }

            val hasFineLocationPermission = ActivityCompat.checkSelfPermission(
                context,
                Manifest.permission.ACCESS_FINE_LOCATION
            ) == PackageManager.PERMISSION_GRANTED
            val hasCoarseLocationPermission = ActivityCompat.checkSelfPermission(
                context,
                Manifest.permission.ACCESS_COARSE_LOCATION
            ) == PackageManager.PERMISSION_GRANTED

            if(hasFineLocationPermission && hasCoarseLocationPermission) {
                val request = LocationRequest.Builder(
                    Priority.PRIORITY_HIGH_ACCURACY,
                    interval
                ).build()

                val callback = object : LocationCallback() {
                    override fun onLocationResult(result: LocationResult) {
                        super.onLocationResult(result)
                        result.locations.lastOrNull()?.let { location ->
                            trySend(location)
                        }
                    }
                }

                client.requestLocationUpdates(request, callback, context.mainLooper)

                awaitClose {
                    println("Flow was closed.")
                    client.removeLocationUpdates(callback)
                }
            } else {
                close()
            }
        }
    }
}
```

### 🔍 비교

| 항목        | `SharedFlow`             | `StateFlow`        | `callbackFlow`       |
| --------- | ------------------------ | ------------------ | -------------------- |
| Stream 타입 | Hot                      | Hot                | Cold     |
| 값 저장      | ❌ 기본 없음 (`replay` 설정 가능) | ✅ 항상 하나            | ❌                    |
| 초기값 필요 여부 | ❌                        | ✅ 필요               | ❌                    |
| 소비자 수     | ✅ 여러 개                   | ✅ 여러 개             | 보통 1개 (1:1 구조)       |
| 주요 목적     | UI 이벤트, 알림               | UI 상태, 입력값         | 콜백 API 흐름 처리         |

### 💡 언제 어떤 걸 써야 할까?

| 상황                      | 추천 Flow 종류     |
| ----------------------- | -------------- |
| 화면 상태를 계속 유지해야 한다       | `StateFlow`    |
| "이벤트성" 메시지를 전달하고 싶다     | `SharedFlow`   |
| 콜백 기반 API를 Flow로 바꾸고 싶다 | `callbackFlow` |
