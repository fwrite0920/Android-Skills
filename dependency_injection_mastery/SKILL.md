---
name: Dependency Injection Mastery
description: Hilt 进阶用法、Custom Components 与 Multi-binding 模式
---

# Dependency Injection Mastery (依赖注入专精)

## Instructions
- 确认问题属于 DI 架构或 Scope 设计
- 依照下方章节顺序套用
- 一次只调整一种注入模式或 module
- 完成后对照 Quick Checklist

## When to Use
- Scenario A：新项目 DI 架构创建
- Scenario C：旧项目现代化与模块拆分
- Scenario F：KMP 共用模块的 DI 调整

## Example Prompts
- "请参考 Assisted Injection，替 ViewModel 加入动态参数"
- "依照 Custom Hilt Components 创建 User Session Scope"
- "请用 Multi-binding 章节设计插件式支付模块"

## Workflow
1. 先确认 Assisted Injection 与 Scope 需求
2. 再整理 Module Organization 与 Qualifier
3. 最后用 Quick Checklist 验收

## Practical Notes (2026)
- 多模块采 API/impl 分离，避免跨模块直接依赖实作
- 跨模块导航与 Service 以 interface + EntryPoint 统一
- Scope 设计先画出生命周期，再落地到 Module

## Minimal Template
```
目标: 
Scope: 
Module 结构: 
注入模式: 
验收: Quick Checklist
```

---

## Assisted Injection

当 ViewModel 或 Worker 需要同时接收 DI 的依赖与 Runtime 参数时使用。

### ViewModel with SavedStateHandle + Custom Args

```kotlin
@HiltViewModel(assistedFactory = DetailViewModel.Factory::class)
class DetailViewModel @AssistedInject constructor(
    @Assisted private val productId: String,
    @Assisted savedStateHandle: SavedStateHandle,
    private val repository: ProductRepository
) : ViewModel() {
    
    @AssistedFactory
    interface Factory {
        fun create(productId: String, savedStateHandle: SavedStateHandle): DetailViewModel
    }
}

// 在 Compose 中使用
@Composable
fun DetailScreen(productId: String) {
    val viewModel = hiltViewModel<DetailViewModel, DetailViewModel.Factory> { factory ->
        factory.create(productId, createSavedStateHandle())
    }
}
```

### WorkManager with Assisted Injection

```kotlin
@HiltWorker
class SyncWorker @AssistedInject constructor(
    @Assisted context: Context,
    @Assisted params: WorkerParameters,
    private val repository: Repository
) : CoroutineWorker(context, params) {
    
    override suspend fun doWork(): Result {
        repository.sync()
        return Result.success()
    }
}
```

---

## Custom Hilt Components (Scopes)

创建自定义的生命周期范围，例如 User Session。

### 定义 Custom Scope

```kotlin
@Scope
@Retention(AnnotationRetention.RUNTIME)
annotation class UserSessionScope

@DefineComponent(parent = SingletonComponent::class)
@UserSessionScope
interface UserSessionComponent

@DefineComponent.Builder
interface UserSessionComponentBuilder {
    fun setUser(@BindsInstance user: User): UserSessionComponentBuilder
    fun build(): UserSessionComponent
}
```

### 管理 Component 生命周期

```kotlin
@EntryPoint
@InstallIn(UserSessionComponent::class)
interface UserSessionEntryPoint {
    fun paymentService(): PaymentService
}

@Singleton
class UserSessionManager @Inject constructor(
    private val componentBuilder: Provider<UserSessionComponentBuilder>
) {
    private var component: UserSessionComponent? = null
    
    fun login(user: User) {
        component = componentBuilder.get().setUser(user).build()
    }
    
    fun logout() {
        component = null
    }
    
    fun getPaymentService(): PaymentService {
        val currentComponent = checkNotNull(component) { "User not logged in" }
        return EntryPoints.get(currentComponent, UserSessionEntryPoint::class.java)
            .paymentService()
    }
}
```

---

## Multi-binding (Set & Map)

实作 Plugin 架构，例如多种支付方式。

### Set Multibinding

```kotlin
interface PaymentProcessor {
    fun process(amount: Double): Result
}

@Module
@InstallIn(SingletonComponent::class)
abstract class PaymentModule {
    
    @Binds
    @IntoSet
    abstract fun bindCreditCard(impl: CreditCardProcessor): PaymentProcessor
    
    @Binds
    @IntoSet
    abstract fun bindPayPal(impl: PayPalProcessor): PaymentProcessor
}

// 注入所有实作
class PaymentService @Inject constructor(
    private val processors: Set<@JvmSuppressWildcards PaymentProcessor>
) {
    fun processAll(amount: Double) {
        processors.forEach { it.process(amount) }
    }
}
```

### Map Multibinding (with Key)

```kotlin
enum class PaymentType { CREDIT_CARD, PAYPAL, GOOGLE_PAY }

@MapKey
annotation class PaymentTypeKey(val value: PaymentType)

@Module
@InstallIn(SingletonComponent::class)
abstract class PaymentModule {
    
    @Binds
    @IntoMap
    @PaymentTypeKey(PaymentType.CREDIT_CARD)
    abstract fun bindCreditCard(impl: CreditCardProcessor): PaymentProcessor
}

// 按 Key 取得特定实作
class PaymentService @Inject constructor(
    private val processors: Map<PaymentType, @JvmSuppressWildcards PaymentProcessor>
) {
    fun process(type: PaymentType, amount: Double) {
        processors[type]?.process(amount)
    }
}
```

---

## Module Organization

### 分层架构

```
di/
├── AppModule.kt           # Application-level
├── NetworkModule.kt       # Retrofit, OkHttp
├── DatabaseModule.kt      # Room
├── RepositoryModule.kt    # Repository bindings
└── UseCaseModule.kt       # UseCase bindings
```

### Qualifier 使用

```kotlin
@Qualifier
@Retention(AnnotationRetention.BINARY)
annotation class IoDispatcher

@Module
@InstallIn(SingletonComponent::class)
object DispatcherModule {
    @Provides
    @IoDispatcher
    fun provideIoDispatcher(): CoroutineDispatcher = Dispatchers.IO
}
```

---

## Quick Checklist

- [ ] ViewModel 动态参数使用 Assisted Injection
- [ ] 避免在 Module 中使用 `@Provides` 创建复杂逻辑
- [ ] Qualifier 用于区分相同类型的不同实例
- [ ] 避免 Circular Dependencies
- [ ] 测试时使用 `@UninstallModules` + `@TestInstallIn`
