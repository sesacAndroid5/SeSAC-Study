# 🧾 Kotlin에서의 직렬화 - Serializable vs Parcelable

## 1️⃣ Serializable

### 📌 직렬화(Serialization)란?

- 객체의 **현재 상태를 바이트 스트림(Byte Stream)**으로 변환하여 파일이나 네트워크 등 **비휘발성 저장소**에 저장하는 것
- 이후 다시 메모리로 불러와 **원래 객체 상태를 복원**

> **영속성(Persistence)**: 객체의 상태가 프로그램 실행 수명을 넘어 유지되는 특성

- 객체 상태와 메타 정보는 내부적으로 트리(Tree) 구조로 표현됨
- JSON, XML, 바이너리 등 다양한 포맷으로 직렬화 가능

### 📦 Serializable이란?

- Java 표준 라이브러리에 포함된 (`java.io.Serializable`) 마커 인터페이스
- "해당 클래스의 객체는 직렬화될 수 있다" 라는 것을 가상 머신에게 알려주는 "마커(Marker)" 역할
- Kotlin에서도 Java와의 호환성 덕분에 사용 가능
- **메서드 없이 객체 직렬화 가능함을 표시**

```kotlin
data class User(
    val id: String,
    val name: String,
    val email: String
) : Serializable
```

**✅ 장점**
- 간단하게 `Serializable` 인터페이스 구현만으로 사용

**⚠️ 단점**
- 리플렉션을 광범위하게 사용 → 성능 저하 (느림)

---

## 🔧 kotlinx.serialization (추천)

- Kotlin 공식 직렬화 라이브러리
- Gradle 플러그인 설정 및 `@Serializable` 어노테이션 사용
- JSON, ProtoBuf, XML 등 다중 포맷 지원

### 설정 예시

```kotlin
// build.gradle.kts (Module)
plugins {
    id("org.jetbrains.kotlin.plugin.serialization") version "2.2.0"
}

dependencies {
    implementation("org.jetbrains.kotlinx:kotlinx-serialization-json:1.8.1")
}
```

### 사용 예시

```kotlin
// build.gradle에 의존성 추가
import kotlinx.serialization.Serializable
import kotlinx.serialization.json.Json
import kotlinx.serialization.encodeToString
import kotlinx.serialization.decodeFromString

// 직렬화/역직렬화를 적용할 클래스에 어노테이션 선언
@Serializable
data class Data(val a: Int, val b: String)

// 원하는 포맷으로 전환
fun main() {
    val json = Json.encodeToString(Data(42, "str"))
    val deserialized = Json.decodeFromString<Data>(json)
}
```

---

## 2️⃣ Parcelable

> 안드로이드 전용 직렬화 메커니즘  
> **IPC 통신(Inter Process Communication)**, 컴포넌트 간 객체 전달 시 사용

### 특징

- Android SDK에 정의된 `android.os.Parcelable` 인터페이스
- 데이터를 **Parcel 컨테이너**에 읽고/쓰기

### 레거시 구현 방식
Parcelable을 구현하는 클래스는 객체의 데이터를 Parcel에 쓰는 방법(writeToParcel 메서드)과
Parcel에서 데이터를 읽어 객체를 다시 생성하는 방법(CREATOR 필드)을 명시해야 합니다.

```kotlin
data class User(val id: String, val name: String, val email: String) : Parcelable {
    override fun writeToParcel(parcel: Parcel, flags: Int) {
        parcel.writeString(id)
        parcel.writeString(name)
        parcel.writeString(email)
    }
    override fun describeContents() = 0

    companion object CREATOR : Parcelable.Creator<User> {
        override fun createFromParcel(parcel: Parcel): User {
            return User(
                parcel.readString()!!,
                parcel.readString()!!,
                parcel.readString()!!
            )
        }
        override fun newArray(size: Int) = arrayOfNulls<User?>(size)
    }
}
```

### ✅ `@Parcelize` 사용 (추천)
수동으로 구현시 보일러플레이트 코드가 많아서 번거롭기 때문에 kotlin-parcelize 플로그인을 적용하여 손쉽게 구현이 가능합니다.

```kotlin
import kotlinx.parcelize.Parcelize
import android.os.Parcelable

@Parcelize
data class User(val id: String, val name: String, val email: String) : Parcelable
```

> 간단한 어노테이션 하나로 모든 Parcelable 구현 완료

### 사용처

- `Intent.putExtra()`로 Activity 간 객체 전달
- `Bundle`에 객체 저장/복원
- `Service`나 `BroadcastReceiver` 간 데이터 전달

---

## 📊 비교 요약

| 구분 | Serializable | Parcelable |
|------|--------------|------------|
| 플랫폼 | Java / Kotlin | Android 전용 |
| 사용 난이도 | 매우 간단 | 수동 구현 복잡 (Parcelize로 단순화 가능) |
| 성능 | 느림 (리플렉션 기반) | 빠름 (명시적 읽기/쓰기) |
| 주요 사용처 | 파일 저장, 네트워크 통신 | 컴포넌트 간 데이터 전달 |
| 포맷 확장성 | JSON, XML 등 다양 | Parcel 전용 (Android 내부 포맷) |
