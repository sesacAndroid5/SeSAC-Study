# WorkManager 작업 종류 및 사용 방법 가이드 🛠️

## 🔹 1. WorkManager의 작업 종류
WorkManager에서 예약 가능한 작업은 **실행 횟수 기준**으로 아래 두 가지로 나뉩니다.

### 📌 1-1. 일회성 작업 (OneTimeWorkRequest)
한 번만 실행되면 되는 작업을 예약할 때 사용합니다.

**특징**
- `.then()`으로 체이닝하여 순차 실행 가능
- 실패 시 재시도 설정 가능

**예시 사용 사례**
- 사진 업로드 후 필터 적용
- 서버에 로그 업로드

```kotlin
val uploadWorkRequest = OneTimeWorkRequestBuilder<MyUploadWorker>().build()
WorkManager.getInstance(context).enqueue(uploadWorkRequest)
```
### 📌 1-2. 주기적 작업 (PeriodicWorkRequest)
일정 간격마다 반복되어야 하는 작업을 예약할 때 사용합니다.

**특징**
- 최소 반복 주기: 15분
- 정확한 실행 시각은 보장되지 않고, 시스템 리소스를 고려해 최적 시점에 실행됨

**예시 사용 사례**
- 하루 1회 콘텐츠 동기화
- 주기적인 로그 업로드

```kotlin
val syncWorkRequest = PeriodicWorkRequestBuilder<MySyncWorker>(
    12, TimeUnit.HOURS
).build()
WorkManager.getInstance(context).enqueue(syncWorkRequest)
```
## 🔹 2. 작업 실행 방식: 기본 실행 vs. Expedited 실행
작업 예약 시, 시스템 리소스 상황에 따라 빠르게 실행할지, 지연해도 괜찮을지 설정할 수 있습니다.

### 📌 2-1. 지연 가능한 실행 (Deferrable Work)
WorkManager의 기본 동작입니다.

정확한 시간보다 작업 실행 자체의 보장에 초점

제약 조건 (Constraints)을 설정하면 조건이 만족될 때까지 지연 실행

```kotlin
val constraints = Constraints.Builder()
    .setRequiredNetworkType(NetworkType.UNMETERED) // Wi-Fi 상태
    .setRequiresCharging(true)                     // 충전 중
    .build()

val backupWorkRequest = OneTimeWorkRequestBuilder<MyBackupWorker>()
    .setConstraints(constraints)
    .build()

WorkManager.getInstance(context).enqueue(backupWorkRequest)
```
### 📌 2-2. 빠른 실행 요청 (Expedited Work)
사용자에게 중요한 작업은 가능한 한 빨리 실행되도록 요청할 수 있습니다.

즉시 실행을 시도하지만, 시스템 리소스 상황에 따라 지연될 수 있음

```kotlin
val expeditedWorkRequest = OneTimeWorkRequestBuilder<MyUrgentWorker>()
    .setExpedited(OutOfQuotaPolicy.RUN_AS_NON_EXPEDITED_WORK_REQUEST)
    .build()

WorkManager.getInstance(context).enqueue(expeditedWorkRequest)
```
>🔍 OutOfQuotaPolicy란?
>Expedited 작업은 시스템이 허용하는 빠른 실행 quota를 초과할 경우 제한됩니다. 이때 어떻게 처리할지 결정하는 정책입니다.

| 옵션 | 설명 |
|--------------------|--------------------------|
|RUN_AS_NON_EXPEDITED_WORK_REQUEST | 일반 작업으로 처리 (지연 가능, 기본값)|
|DROP_WORK_REQUEST | 작업을 실행하지 않고 취소 (권장 X)|

## 🔹 3. WorkManager 사용 3단계
✅ 1단계: 작업 정의 (Worker 클래스 만들기)
작업 로직을 담은 클래스를 정의합니다.
일반적으로 CoroutineWorker를 권장합니다.

>왜 CoroutineWorker를 권장하나요?
> - suspend 함수 사용 가능 → 자연스러운 비동기 처리
> - 구조화된 코루틴 사용으로 안정성과 가독성 향상
> - Android 공식 문서에서도 우선 사용을 권장
>   
>필요 시 Worker (동기 처리), RxWorker (RxJava 기반)도 선택 가능.

```kotlin
class MyUploadWorker(
    context: Context,
    params: WorkerParameters
) : CoroutineWorker(context, params) {

    override suspend fun doWork(): Result {
        /*
        WorkManager를 통해 실행될 작업 정의
        */
        return Result.success()
    }
}
```
| 반환값	| 의미 |
|-------------|---------------|
|Result.success() |	작업 성공 |
|Result.failure()	| 실패로 종료 |
|Result.retry()	| 나중에 재시도 요청|

✅ 2단계: 작업 요청 생성 (WorkRequest)
```kotlin
val uploadWorkRequest = OneTimeWorkRequestBuilder<MyUploadWorker>().build()
```
✅ 3단계: 작업 예약 (WorkManager에 전달)
```kotlin
WorkManager.getInstance(context).enqueue(uploadWorkRequest)
```
### 📝 요약 정리
| 항목 | 설명 | 예시 |
|------|------|------|
| OneTimeWorkRequest | 한 번만 실행되는 작업 | 로그 업로드, 이미지 저장 |
| PeriodicWorkRequest| 주기적으로 반복되는 작업 | 데이터 동기화, 콘텐츠 확인 |
| Expedited Work | 빠르게 실행되도록 요청 | 사용자 입력 직후 중요한 처리 |
| Constraints | 실행 조건 지정 | Wi-Fi 연결 시, 충전 중 등 |
| CoroutineWorker | 코루틴 기반 비동기 처리 | 대부분 권장되는 방식 |

✅ 마무리
WorkManager는 **정확한 시각보다 ‘작업의 보장된 실행’**이 필요한 경우에 적합한 Android 공식 백그라운드 작업 도구입니다.

즉각적인 실행 또는 장기 실행이 필요한 경우에는 ForegroundService를 고려하는 것이 좋습니다.
