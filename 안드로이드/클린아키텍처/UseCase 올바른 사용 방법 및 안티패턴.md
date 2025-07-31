# UseCase 올바른 사용 방법 및 안티패턴 🛠️
---

## 1️⃣ 잘못된 예시 (Red Flags)

### ❌Generic UseCase
가장 흔한 안티패턴은 하나의 유스케이스에서 모든 관련 기능을 처리하려는 것입니다.

```kotlin
// ❌ 안티패턴: 모든 Product 관련 기능을 하나의 클래스에서 처리
class GenericUseCase(private val repository: ProductRepository) {
    suspend fun execute(action: String, data: Any?): Any? {
        return when (action) {
            "getAll" -> repository.getAllProducts()
            "insert" -> repository.insertProduct(data as Product)
            "delete" -> repository.deleteProduct(data as Product)
            else -> null
        }
    }
}
```

문제점:

- 단일 책임 원칙(SRP) 위반: 클래스가 너무 많은 책임을 갖게 됩니다.

- 낮은 테스트 용이성 및 재사용성: 특정 기능 하나만 테스트하거나 재사용하기 어렵습니다.

- 유지보수 어려움: 기능이 추가될수록 when 분기문이 늘어나며 코드가 복잡해집니다.

### ❌ 단순 Repository 호출 래퍼
Repository의 메서드를 단순히 한 번 감싸는 것만 하는 경우

```kotlin
class GetAllProductsUseCase(private val repository: ProductRepository) {
    suspend operator fun invoke(): List<Product> {
        return repository.getAllProducts()
    }
}
```
문제점:

- UseCase가 아무런 비즈니스 로직 없이 불필요한 계층이 됩니다.

- Repository를 직접 호출하는 것과 다르지 않습니다.

### ❌ Flow를 직접 반환만 하는 UseCase
Repository에서 받은 Flow를 아무런 가공 없이 그대로 반환만 하는 경우

```kotlin
class ObserveProductsUseCase(private val repository: ProductRepository) {
    operator fun invoke(): Flow<List<Product>> {
        return repository.observeProducts()
    }
}
```
문제점:

- 책임 모호성: UseCase는 "시작과 끝이 있는 단일 작업"을 표현해야 하지만, 단순히 Flow를 전달하면 데이터 스트림 전달자 역할만 하게 됩니다.

- 계층 누수 (Layer Leakage): Repository 구현이 도메인 계층에 그대로 노출되어 ViewModel에서 Repository를 직접 쓰는 것과 유사해집니다.

- 비즈니스 로직 부재: 중간에서 가공 없이 데이터를 그대로 전달하면 UseCase 계층이 의미가 없어집니다.
> 계층 누수 (Layer Leakage)란?
> 
>  각 계층은 자신보다 아래 계층의 "구현 세부 사항"을 몰라야 합니다.
>  현재 예시 코드에서 UseCase는 그냥 "중간 통로"로만 쓰고 있고, ViewModel이 Repository의 동작을 직접 아는 것과 다를 게 없어집니다.
>  즉, "도메인 계층(UseCase)"이 의미 없이 데이터 계층(Repository)의 API를 그대로 노출시키는 상황 → 이게 계층 누수입니다.

개선 방법:

```kotlin
class ObserveFilteredProductsUseCase(private val repository: ProductRepository) {
    operator fun invoke(): Flow<List<Product>> {
        return repository.observeProducts()
            .map { products -> products.filter { it.isAvailable } }
    }
}
```
> Flow를 반환하더라도, **비즈니스 로직(가공, 필터링, Validation 등)**이 포함되면 정상적인 UseCase로 간주할 수 있음.

### ❌ UseCase에서 UI 조작 금지
UseCase에서 UI를 직접 조작하는 경우
```kotlin
class ShowToastUseCase(private val context: Context) { // 안티패턴: Context(UI 관련 의존성)
    suspend operator fun invoke(message: String) {
        Toast.makeText(context, message, Toast.LENGTH_SHORT).show()
    }
}
```
문제점:

- UseCase가 UI 프레임워크(Context, Toast)에 의존 → Domain 계층이 Presentation 계층을 알게 됩니다.

- 재사용성 낮음: 특정 화면에만 적용되는 UI 모델을 반환하면 유스 케이스는 해당 화면에서만 사용 가능합니다.

------------------------------------

## 2️⃣ 올바른 예시 (Best Practices)
### ✅ 기능별 UseCase 분리
각 유스케이스는 오직 하나의 비즈니스 규칙만 책임져야 합니다.

```Kotlin

// ✅ 모범 사례: 각 기능에 따라 클래스를 분리
class GetAllProductsUseCase(private val repository: ProductRepository) {
    suspend operator fun invoke(): List<Product> = repository.getAllProducts()
}

class InsertProductUseCase(private val repository: ProductRepository) {
    suspend operator fun invoke(product: Product) = repository.insertProduct(product)
}

class DeleteProductUseCase(private val repository: ProductRepository) {
    suspend operator fun invoke(product: Product) = repository.deleteProduct(product)
}
```
### ✅ UseCase 묶어서 관리하기
관련된 유스케이스가 많아지면, data class로 묶어서 의존성 주입을 간소화할 수 있습니다.

```Kotlin

// ✅ 모범 사례: 관련된 유스케이스들을 하나의 클래스로 그룹화
data class ProductUseCases(
    val getAllProducts: GetAllProductsUseCase,
    val insertProduct: InsertProductUseCase,
    val deleteProduct: DeleteProductUseCase
)
```
이 ProductUseCases를 ViewModel에 주입하면, 여러 유스케이스를 개별적으로 주입할 필요가 없어 코드가 깔끔해집니다.

```Kotlin

// ✅ 모범 사례: ViewModel은 그룹화된 UseCases를 주입받아 사용
class ProductViewModel(
    private val productUseCases: ProductUseCases
) : ViewModel() {

    fun loadProducts() {
        viewModelScope.launch {
            val products = productUseCases.getAllProducts()
            // UI 업데이트
        }
    }
}
```

------------------------------------

## 3️⃣ 복잡한 UseCase 구성 방법
### ✅ 하나의 UseCase에서 여러 Repository 사용
하나의 비즈니스 로직을 위해 여러 데이터 소스가 필요한 경우, 한 유스케이스에서 여러 Repository를 조합할 수 있습니다. 이는 SRP를 위반하지 않습니다.

```Kotlin

// 예시: 원격 서버 데이터를 가져와 로컬 DB에 저장하는 동기화 작업
class SyncUserDataUseCase(
    private val localRepo: LocalUserRepository,
    private val remoteRepo: RemoteUserRepository
) {
    suspend operator fun invoke(userId: String) {
        val remoteUser = remoteRepo.fetchUser(userId)
        localRepo.saveUser(remoteUser)
    }
}
```
이유: '사용자 데이터 동기화'라는 단일 책임을 위해 여러 데이터 소스를 조합하는 것은 자연스러운 비즈니스 로칙입니다.

## ✅ 다른 UseCase를 조합해서 사용
더 큰 비즈니스 플로우를 위해, 하나의 상위 유스케이스가 여러 하위 유스케이스를 오케스트레이션(orchestration)할 수 있습니다.
> 오케스트레이션(orchestration) : 여러 개별 작업(또는 컴포넌트)을 순서와 조건에 맞게 조율해서 하나의 큰 흐름을 완성하는 것

```Kotlin

// 예시: '구매 완료'라는 큰 흐름을 위해 여러 단계를 조합
class CompletePurchaseUseCase(
    private val validateStockUseCase: ValidateStockUseCase,
    private val processPaymentUseCase: ProcessPaymentUseCase,
    private val saveOrderUseCase: SaveOrderUseCase
) {
    suspend operator fun invoke(order: Order): Boolean {
        if (!validateStockUseCase(order)) return false
        if (!processPaymentUseCase(order)) return false
        saveOrderUseCase(order)
        return true
    }
}
```
이유: 상위 유스케이스가 여러 하위 유스케이스를 조율하여 전체 비즈니스 플로우를 담당합니다. 이를 통해 각 단계의 로직을 재사용하고 테스트하기 용이해집니다.

## 📌 출처
- ✅ [안드로이드 디벨로퍼 도메인 레이어](https://developer.android.com/topic/architecture/domain-layer?hl=ko&_gl=1*1ahkdal*_up*MQ..*_ga*MTUxMzgzMzM4NC4xNzUzOTY4MjQx*_ga_6HH9YJMN9M*czE3NTM5NjgyNDAkbzEkZzAkdDE3NTM5NjgyNDAkajYwJGwwJGg0NDAwNTA1NTA.)
- 🔧 [UseCase Red Flags and Best Practices in Clean Architecture (Teknasyon Engineering)](https://engineering.teknasyon.com/usecase-red-flags-and-best-practices-in-clean-architecture-76e2f6d921eb)
