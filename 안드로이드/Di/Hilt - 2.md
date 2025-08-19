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

대표적인 컴포넌트 범위: - **SingletonComponent** → 앱 전체에서 단일
인스턴스 유지 (ex: Retrofit, DB 등) - **ActivityComponent** → Activity
단위에서 유지 - **FragmentComponent** → Fragment 단위 -
**ViewModelComponent** → ViewModel 단위 - **ServiceComponent** → Service
단위

------------------------------------------------------------------------

## 3. @Provides

-   **구체 객체를 생성해서 제공할 때** 사용.\
-   보통 외부 라이브러리 객체나, 인터페이스가 아닌 구체 클래스 의존성을
    만들 때 사용합니다.
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
