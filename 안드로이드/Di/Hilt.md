# DI(Dependency Injection) & Hilt

## DI(Dependency Injection)란?

의존성 주입(Dependency Injection, DI)은 **객체가 필요로 하는 다른 객체를 직접 생성하는 대신 외부에서 필요한 객체를 전달받는 방식**의 디자인 패턴이다.  
이를 통해 **객체 간의 결합도를 낮추고 유지보수와 테스트에 유리**하게 만든다.  
즉, 객체의 생성과 사용을 분리하며, 사용하는 쪽은 객체의 생성 방식이나 구성에 대해 알 필요가 없다.

**안드로이드에서의 장점**
- 공통 객체(DB, 네트워크 등)를 한 곳에서 관리
- 각 컴포넌트에서 객체 생성 불필요
- 코드 재사용성, 리팩터링, 테스트 용이

---

## DI가 필요한 이유
다른 클래스의 참조가 필요한 경우 객체를 얻는 방법은 3가지:
1. 클래스가 직접 인스턴스를 생성  
2. 다른 곳에서 가져와 사용  
3. 매개변수로 제공받아 사용 (**→ DI 방식**)

### 예시
#### ❌ 직접 생성
```kotlin
class Car {
    private val engine = Engine()
    fun start() {
        engine.start()
    }
}
```
→ Car와 Engine이 강하게 결합되어 변경에 취약.

#### ✅ 생성자 주입
```kotlin
class Car(private val engine: Engine) {
    fun start() {
        engine.start()
    }
}
```
→ Car는 Engine이 어떻게 생성되는지 몰라도 동작.

---

## Android에서 DI 실행 방법
1. **생성자 삽입** : 생성자에 의존성 전달
2. **필드 삽입** : 시스템이 인스턴스화하는 클래스(액티비티, 프래그먼트)에서 사용

```kotlin
class Car {
    lateinit var engine: Engine
    fun start() {
        engine.start()
    }
}
```

---

## 자동 의존성 주입 - Dagger & Hilt
- **Dagger** : 종속 항목 그래프를 자동으로 생성/관리
- **Hilt** : Android 공식 권장 DI 라이브러리  
  (Dagger 기반, Android 컴포넌트 수명 주기에 맞춰 의존성 관리)

---

## Hilt 설정

### 1. Gradle 설정  
<img width="894" height="752" alt="스크린샷 2025-08-13 오후 11 39 55" src="https://github.com/user-attachments/assets/651d96a7-7f32-4a26-ad69-1cc219902da9" />

**1) plugins > hilt.android**  
Hilt의 Gradle 플러그인을 프로젝트에 적용하겠다는 의미. Gradle 플러그인은 Hilt를 사용하기 위해 필요한 코드 생성과 설정 작업을 자동으로 처리해준다.  

**2) dependencies > hilt-android**  
Hilt 라이브러리의 런타임 부분을 프로젝트에 포함시킨다. 앱이 실행될 때 Hilt의 기능들을 사용할 수 있게 해주는 핵심 라이브러리.   
해당 어노테이션을 추가하면 Hilt의 어노테이션들을 코드에서 사용할 수 있다.  
  
**3) dependencies > hilt-android-compiler**  
Hilt의 어노테이션 프로세서를 추가한다. 컴파일 타임에 Hilt 어노테이션들을 분석해서 의존성 주입에 필요한 코드를 자동으로 생성해주는 도구

**4) plugins > devtools.ksp**  
Kotlin 코드를 분석하여 컴파일 시점에 추가적인 코드를 생성하는 데 사용되는 도구
Dagger, Room, Moshi 와 같은 라이브러리들은 어노테이션을 기반으로 코드를 자동으로 생성하는데 ksp가 이 과정을 담당

---

### 2. @HiltAndroidApp
- Application 클래스에 선언하여 **앱 전체 DI 컨테이너** 생성
```kotlin
@HiltAndroidApp
class ExampleApplication : Application()
```

---

### 3. @AndroidEntryPoint
- Android 컴포넌트(Activity, Fragment 등)에 DI 가능하게 함
- 상위 컴포넌트에도 반드시 어노테이션 필요
```kotlin
@AndroidEntryPoint
class ExampleActivity : AppCompatActivity()
```

지원 대상:
- Application (@HiltAndroidApp)
- Activity 
- Fragment
- View
- Service
- BroadcastReceiver
- ViewModel (@HiltViewModel)  
<img width="736" height="291" alt="image" src="https://github.com/user-attachments/assets/a2052011-3f0d-400b-9548-0a797d5bf296" />

---

### 4. @Inject
- 의존성 주입이 필요한 필드나 생성자에 붙임

#### 생성자 주입 (권장)
```kotlin
class Engine @Inject constructor()
class Car @Inject constructor(private val engine: Engine)
```

#### 필드 주입
```kotlin
@AndroidEntryPoint
class ExampleActivity : AppCompatActivity() {
    @Inject lateinit var analytics: AnalyticsAdapter
}
```

## ViewModel에서 Hilt 적용
1. 액티비티 or 프래그먼트에서 ViewModel을 사용하는 경우 @AndroidEntryPoint 주석을 통해 의존성 주입을 Hilt에게 알림  
<img width="370" height="66" alt="image" src="https://github.com/user-attachments/assets/e7713ad2-770d-48cd-9f84-36c08585e5d9" />  

2. 사용할 ViewModel에 @HiltViewModel 주석을 통해 Hilt에게 알려준다. 만약 Repository나 Usecase를 의존성 주입해야하는 경우 @Inject를 Constructor 앞에 달아 생성자에
   필요한 의존성을 주입한다. 없는경우 @Inject는 생략 가능  
<img width="410" height="101" alt="image" src="https://github.com/user-attachments/assets/afc2c347-fe75-45b1-bd1a-67d99787f555" />
  
  당연히 ViewModel에 넣어줄 Repository나 Usecase도 의존성 주입을 Hilt에게 알려줘야 한다.  
<img width="394" height="70" alt="image" src="https://github.com/user-attachments/assets/c1b0de5c-9286-4199-aa01-f390627a95cb" />  

4. ViewModel을 의존성 주입해준다.  
- by viewModels() : XML View를 사용하는 액티비티, 프래그먼트에서 사용  
<img width="613" height="194" alt="image" src="https://github.com/user-attachments/assets/900e5328-371e-44b0-94cb-6ff0bc429e2f" />  

- hiltViewModel() : Compose 전용함수로 ViewModel 요청 시 Hilt가 자동으로 ViewModelFactory 생성 및 인스턴스 제공  
<img width="523" height="139" alt="image" src="https://github.com/user-attachments/assets/88a78f51-6adb-445e-96d1-ceae233d33d2" />

---
