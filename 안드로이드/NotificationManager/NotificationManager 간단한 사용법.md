# NotificationManager 간단한 사용법

NotificationManager는 알림을 표시하고 관리하는 안드로이드 시스템 서비스입니다. 단순히 알림을 띄우는 것뿐만 아니라, 사용자와의 상호작용을 지원하고 알림을 세분화해 관리할 수 있도록 도와줍니다.  

---

## 1️⃣ 알림 채널(NotificationChannel)

### 채널의 목적: 사용자 제어권 제공
Android 8.0(API 26) 이상을 타겟으로 하는 앱은 반드시 하나 이상의 알림 채널을 생성해야 합니다.  
알림 채널은 앱의 알림을 카테고리별로 그룹화하여 사용자가 개별적으로 알림을 제어할 수 있도록 돕습니다.  

예시) 메신저 앱
- **메시지 채널**: 소리와 진동 활성화
- **프로모션 채널**: 무음

### 중요도 수준 (Importance Levels)
채널 생성 시 설정하는 중요도는 알림 표시 방식과 사용자 경험을 결정합니다.

| 중요도              | 설명                                                           |
|----------------------|--------------------------------------------------------------|
| `IMPORTANCE_HIGH`    | 소리를 내고 Heads-up 알림으로 표시 (앱이 포그라운드이거나 허용 시) |
| `IMPORTANCE_DEFAULT` | 소리를 내지만 Heads-up 알림 없음                              |
| `IMPORTANCE_LOW`     | 소리 없이 상태 표시줄에만 표시                                |
| `IMPORTANCE_MIN`     | 소리 및 아이콘 표시 없음, 알림 패널에서만 확인 가능           |

> ⚠️ **중요:** 채널은 한 번 생성되면 중요도를 포함한 설정을 코드로 변경할 수 없으며, 사용자가 시스템 설정에서 직접 변경해야 합니다.  

---

## 2️⃣ 알림 상호작용과 액션

### 알림 탭(Tap) 처리: PendingIntent
사용자가 알림을 탭했을 때 특정 화면으로 이동시키려면 **PendingIntent**가 필요합니다.

```kotlin
val notificationManager = getSystemService(NOTIFICATION_SERVICE) as NotificationManager

val intent = Intent(this, MainActivity::class.java).apply {
    flags = Intent.FLAG_ACTIVITY_NEW_TASK or Intent.FLAG_ACTIVITY_CLEAR_TASK
}
val pendingIntent = PendingIntent.getActivity(
    this, 0, intent, PendingIntent.FLAG_IMMUTABLE
)

val builder = NotificationCompat.Builder(this, CHANNEL_ID)
    .setContentTitle("새 메시지")
    .setContentText("새 메시지가 도착했습니다.")
    .setSmallIcon(R.drawable.ic_notification)
    .setContentIntent(pendingIntent)
    .setAutoCancel(true)

notificationManager.notify(1, builder.build())
```

### 액션 버튼 추가: addAction()
알림에 버튼을 추가해 빠른 동작을 수행할 수 있습니다.

```kotlin
val snoozeIntent = Intent(this, SnoozeReceiver::class.java)
val snoozePendingIntent = PendingIntent.getBroadcast(
    this, 0, snoozeIntent, PendingIntent.FLAG_IMMUTABLE
)

val builder = NotificationCompat.Builder(this, CHANNEL_ID)
    .addAction(R.drawable.ic_snooze, "나중에 보기", snoozePendingIntent)
```

---

## 3️⃣ 알림 업데이트 및 취소
- 업데이트: 같은 ID를 사용해 notify(id, notification)를 호출하면 기존 알림을 새 내용으로 업데이트합니다. (예: 다운로드 진행률 업데이트)

- 취소: cancel(id)를 호출하면 특정 알림만 취소할 수 있습니다.

- 전체 취소: cancelAll() 호출 시 앱의 모든 알림을 제거합니다.

## 📌 정리
- 알림 채널은 Android 8.0 이상에서 필수
- 채널은 한번 생성되면 중요도를 포함한 설정을 코드로 변경할 수 없고, 중요도는	알림 표시 방식을 결정하는 것이다 
- PendingIntent를 이용해	알림 클릭 및 액션 버튼 이벤트에 대한 처리 가능
- 동일 ID로 notify() 실행시 새로운 알림 생성이 아닌 업데이트가 된다
- Android 13부터는	POST_NOTIFICATIONS 런타임 권한 필요

## 📚 참고 자료
- [알림 정보](https://developer.android.com/develop/ui/views/notifications?hl=ko)
- [알림 만들기](https://developer.android.com/develop/ui/views/notifications/build-notification?hl=ko#kts)
- [알림 채널 만들기 및 관리](https://developer.android.com/develop/ui/views/notifications/channels?hl=ko)
- [(추천) velog 정리 글](https://velog.io/@thundevistan/Android-Notification-%EC%A0%95%EB%A6%AC)
