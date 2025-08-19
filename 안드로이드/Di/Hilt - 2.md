# Android Hilt 주요 개념 정리 2  
  
개발자가 만든 클래스의 경우 @Inject 어노테이션을 통해 의존성 주입이 가능해집니다.  
그렇다면 직접 코드 변경이 불가능한 외부 라이브러리나 인터페이스는 어떻게 의존성 주입을 해야하나?  
**-> @Module을 통해 Hilt에게 직접 인스턴스화하는 방법을 알려줘야 합니다.**  

------------------------------------------------------------------------

## 1. @Module

-   **의존성을 제공하는 곳**을 모아두는 클래스에 붙이는 어노테이션
-   Hilt는 의존성 객체를 직접 생성하는 대신, `Module`에서 정의된
    메서드를 통해 객체를 생성합니다.
-   Hilt는 Module을 통해 의존성을 주입할 때 기본적으로 모듈 내 반환 타입과 일치하는 메서드를 사용합니다.
-   일반적으로 `object` 또는 `abstract class` 형태로 작성합니다.

### Module 언제 생성해야 하는가?
1) Hilt가 직접 생성할 수 없는 객체를 제공할 때
Retrofit, OkHttpClient, Room DB, SharedPreferences, 외부 SDK 등을 제공

2) 인터페이스와 구현체간 바인딩이 필요할 때
Repository와 RepositoryImpl를 매칭할 때

``` kotlin
@Module
@InstallIn(SingletonComponent::class) // 어느 범위에 설치할지 지정
object NetworkModule {
    @Provides
    fun provideOkHttpClient(): OkHttpClient {
        return OkHttpClient.Builder().build()
    }
}
```

------------------------------------------------------------------------

## 2. @InstallIn

-   `@Module`이 **어느 범위(Component)**에 설치될지를 명시하는
    어노테이션.
-   내가 정의한 Module을 어느 생명주기 범위에 붙여서, 그 범위 동안 의존성을 관리할지 지정
-   예: `SingletonComponent`, `ActivityComponent`, `ViewModelComponent`
    등

[ 대표적인 컴포넌트 범위 ]  
- **SingletonComponent** → 앱 전체에서 단일 인스턴스 유지 (ex: Retrofit, DB 등)  
- **ActivityComponent** → Activity 단위에서 유지(액티비티가 새로 만들어질 때마다 내부 모듈의 인스턴스는 새로 생성)  
- **FragmentComponent** → Fragment 단위  
- **ViewModelComponent** → ViewModel 단위(같은 ViewModel 내에서는 같은 인스턴스를 사용하지만 다른 ViewModel에서는 새로운 인스턴스 생성)
- **ServiceComponent** → Service 단위  

Hilt는 내부적으로 Dagger의 Component 개념을 적용    
Component란? : "객체 그래프(Object Graph)를 관리하는 컨테이너". 즉, 어느 범위에서 객체를 생성하고 언제까지 유지할지를 결정

<img width="765" height="469" alt="image" src="https://github.com/user-attachments/assets/a67e41d2-b5c3-48ba-a99c-aca4b96c2c57" />  


------------------------------------------------------------------------

## 3. @Provides

-   **구체 객체를 생성해서 제공할 때** 사용.\
-   보통 외부 라이브러리 객체, Builder 패턴, 팩토리 메서드처럼 Hilt가 직접 생성할 수 없는 경우 사용
-   메서드에 붙여서 반환 타입을 Hilt가 주입할 수 있게 해줍니다.

``` kotlin
@Module
@InstallIn(SingletonComponent::class)
object AppModule {

    @Provides
    fun provideRetrofit(client: OkHttpClient): Retrofit {
        return Retrofit.Builder()
            .baseUrl("https://example.com")
            .client(client)
            .build()
    }
}
```

------------------------------------------------------------------------

## 4. @Binds

-   **인터페이스와 구현체를 연결할 때** 사용.\
-   `@Provides`와 달리 **구현체 생성 로직을 직접 쓰지 않고** 단순히
    매핑만 합니다.\
-   `abstract class` 안에서 `abstract fun`으로 선언해야 합니다.

``` kotlin
interface UserRepository {
    fun getUser(): String
}

class UserRepositoryImpl @Inject constructor(): UserRepository {
    override fun getUser() = "홍길동"
}

@Module
@InstallIn(SingletonComponent::class)
abstract class RepositoryModule {
    @Binds
    abstract fun bindUserRepository(
        impl: UserRepositoryImpl
    ): UserRepository
}
```

➡️ `UserRepository`를 필요로 하는 곳에서 Hilt가 `UserRepositoryImpl`을
주입해줍니다.


------------------------------------------------------------------------

## 5. @Provides And @Binds  
1. 공통점  
- 둘 다 @Module 안에서 사용  
- Hilt에게 "이 타입을 필요로 하는 경우 어떤 객체를 제공해야 하는지" 알려줌

2. 차이점
**Provides** : 객체 생성 로직을 직접 작성해서 반환, 개발자가 구체적인 생성 방법을 명시해야 함
**Binds** : 단순히 매핑하는 방식, 인터페이스와 구현체를 매핑할 때 사용

3. 정리  
<img width="795" height="482" alt="image" src="https://github.com/user-attachments/assets/bc9ac35d-b5d1-45a5-a016-c412fea8db47" />  

------------------------------------------------------------------------

## 6. @Qualifier
의존성 주입 시에 동일한 타입의 여러 객체를 구분하기 위한 식별자

### 왜 사용하는가?  
Hilt의 모듈에서 메서드명이 아닌 반환 타입을 기준으로 의존성을 주입합니다.  
그런데 같은 반환 타입이 두 개 이상인 경우 Hilt에게 구분해줄 필요가 생깁니다.  

``` kotlin
@Module
@InstallIn(SingletonComponent::class)
object NetworkModule {

    @Provides
    fun provideBaseRetrofit(): Retrofit =
        Retrofit.Builder()
            .baseUrl("https://api.base.com")
            .build()

    @Provides
    fun provideAuthRetrofit(): Retrofit =
        Retrofit.Builder()
            .baseUrl("https://api.auth.com")
            .build()
}
```  
여기서 Retrofit 타입이 2개이기 때문에 Hilt에게 상황에 맞게 알려줘야 합니다.  

### 사용법
1. 사용할 Qualifier를 정의할 파일을 생성하고 정의한다.  
``` kotlin
// Qualifier들을 모아두는 Kotlin 파일 ex) Qualifiers.kt
@Qualifier
@Retention(AnnotationRetention.BINARY)
annotation class BaseRetrofit

@Qualifier
@Retention(AnnotationRetention.BINARY)
annotation class AuthRetrofit
```
  
2. 구분이 필요한 곳에 만들어둔 Qualifier 어노테이션을 추가한다.  
``` kotlin
@Module
@InstallIn(SingletonComponent::class)
object NetworkModule {

    **@BaseRetrofit**
    @Provides
    fun provideBaseRetrofit(): Retrofit =
        Retrofit.Builder()
            .baseUrl("https://api.base.com")
            .build()

    **@AuthRetrofit**
    @Provides
    fun provideAuthRetrofit(): Retrofit =
        Retrofit.Builder()
            .baseUrl("https://api.auth.com")
            .build()
}
```

3. 의존성 주입 시 구분하여 사용한다.  
``` kotlin
class UserRepository @Inject constructor(
    @BaseRetrofit private val baseRetrofit: Retrofit,
    @AuthRetrofit private val authRetrofit: Retrofit
) {
    // baseRetrofit → 일반 API 호출용
    // authRetrofit → 인증 API 호출용
}
```  
  
### <참고> Named
<img width="906" height="566" alt="image" src="https://github.com/user-attachments/assets/1ab8733e-bd02-4b78-af8f-474de8acf33e" />  

### <참고> @ApplicationContext  
<img width="795" height="317" alt="image" src="https://github.com/user-attachments/assets/7c91df1f-6d6b-4fe2-ae49-2cb7816fad1f" />


------------------------------------------------------------------------

## 정리

-   **@Module**: 의존성을 제공하는 모음집
-   **@InstallIn**: Module을 어느 컴포넌트 범위에 설치할지 지정
-   **@Provides**: 직접 객체를 만들어 반환하는 방식 (주로 외부
    라이브러리, 구체 클래스)
-   **@Binds**: 인터페이스 ↔ 구현체 매핑 (Hilt가 생성 가능해야 함, 즉
    생성자에 `@Inject`가 있어야 함)

------------------------------------------------------------------------

👉 정리하면,\
- **직접 객체 생성이 필요한 경우 → `@Provides`**\
- **인터페이스 구현체 연결만 필요한 경우 → `@Binds`**
