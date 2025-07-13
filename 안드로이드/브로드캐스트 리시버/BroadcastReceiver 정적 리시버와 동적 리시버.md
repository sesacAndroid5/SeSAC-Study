# BroadcastReceiver 정적 리시버와 동적 리시버
BroadcastReceiver는 안드로이드 시스템이나 다른 앱에서 발생하는 이벤트를 수신하는 컴포넌트입니다.

### 1. BroadcastReceiver 등록 방식의 종류
#### 정적 리시버 (Static Receiver)
AndroidManifest.xml 파일에 <receiver> 태그를 사용하여 등록하는 방식

앱이 실행중이 아닐때도 이벤트를 수신 받을 수 있다.

```kotlin
<receiver
    android:name=".MyBootReceiver"
    android:enabled="true"
    android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.BOOT_COMPLETED" />
    </intent-filter>
</receiver>
```
```kotlin
class MyBootReceiver : BroadcastReceiver() {

    override fun onReceive(context: Context, intent: Intent) {
        if (intent.action == Intent.ACTION_BOOT_COMPLETED) {
            // 여기에 원하는 로직을 구현합니다.
        }
    }
}
```
🚨 중요: Android 8.0 (API 26) 이상 제약사항
배터리 소모와 시스템 성능 문제로 인해, Android 8.0부터 대부분의 암시적 브로드캐스트는 정적 리시버로 수신할 수 없습니다. ACTION_BOOT_COMPLETED와 같이 앱이 꼭 받아야 하는 일부 예외적인 이벤트만 정적 등록이 허용됩니다. 따라서 배터리 상태나 네트워크 변경 등은 이제 동적 리시버로 구현해야 합니다.

-------------------------------------------------------

#### 동적 리시버 (Dynamic Receiver)
코드 내에서 Context 객체를 통해 직접 리시버를 등록하는 방식입니다. 주로 Activity나 Service 같은 컴포넌트의 생명주기(Lifecycle)에 맞춰 사용됩니다.

장점: 앱이 실행 중일 때만 필요한 이벤트를 수신하므로 시스템에 부담이 적습니다.

주의사항: 컴포넌트가 소멸될 때 반드시 unregisterReceiver()를 호출하여 리시버를 해제해야 합니다. 그렇지 않으면 **메모리 릭(Memory Leak)**이 발생할 수 있습니다.

```kotlin
val broadcastReceiver = remember {
        object : BroadcastReceiver() {
            override fun onReceive(context: Context?, intent: Intent?) {
                // 여기에 원하는 로직을 구현합니다.
            }
        }
    }

DisposableEffect(Unit) {
        val filter = IntentFilter(Intent.ACTION_AIRPLANE_MODE_CHANGED)
        context.registerReceiver(broadcastReceiver, filter)
        onDispose {
            context.unregisterReceiver(broadcastReceiver)
        }
    }
```

-------------------------------------------------------

### 2. 정적 리시버와 동적 리시버의 생명주기

| 구분 | 정적 리시버 | 동적 리시버 |
|------|----------------|------------------------|
| 관리 주체 | 안드로이드 OS | 개발 |
| 생존 기간 | `onReceive()` 실행 동안 (일회성) | 컴포넌트 생존 동안 (지속성) |
| 관리 코드 | 필요 없음 | 필수 (DisposableEffect 등) |

- 정적 리시버는 평소에는 비활성화 상태로 있다가, AndroidManifest.xml에 등록된 특정 이벤트가 발생했을 때만 안드로이드 시스템에 의해 잠시 깨어납니다.
- 동적 리시버는 개발자가 코드를 통해 직접 등록합니다. 이 리시버는 해제 되기 전까지 계속 활성 상태를 유지하며 이벤트를 수신할 수 있습니다.

-------------------------------------------------------

### 3. 핵심 요약📝

BroadcastReceiver: 시스템 이벤트를 수신하는 안드로이드 컴포넌트입니다.

정적 리시버: AndroidManifest파일에 정의하여 사용하며, 안드로이드 OS에 의해 해당 이벤트 발생시에 깨어나 onReceive 동작을 수행한 후 종료된다.

동적 리시버: 코드를 통해 직접 등록되어 사용하며, 리시버를 해제하기 전까지 계속 활성화 되어 이벤트를 수신하며 리시버를 직접 해제해줘야 한다.
