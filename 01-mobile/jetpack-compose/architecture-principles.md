---
tags: [jetpack-compose, architecture]
---

# Architecture Principles — Jetpack Compose

## UI / Domain / Data (MVVM)

Default architecture for every Jetpack Compose project, per [Android's official app architecture guide](https://developer.android.com/topic/architecture). This is not a per-project choice — the layers below are the one default. Dependencies always point downward, never upward:

``` text
UI (Composable + ViewModel) → Domain (UseCase, optional) → Data (Repository + DataSource)
```

- **UI layer** — Composable + `ViewModel`. The Composable is a pure function of state: no business logic, no direct data access, renders UI state and forwards user events upward. The `ViewModel` is the state holder — one per screen (or per cohesive feature) — owns UI state as a single immutable `StateFlow`/`State`, and handles the events the Composable forwards to it. Must be unit-testable without touching Compose.
- **Domain layer** *(optional)* — `UseCase`/Interactor classes. Add one only when logic is genuinely complex, reused across more than one ViewModel, or combines data from multiple repositories. Skip it otherwise — a ViewModel is allowed to call a Repository directly.
- **Data layer** — `Repository` + `DataSource`. A Repository is the single source of truth for one data type: it centralizes changes, resolves conflicts between data sources, and abstracts them from the rest of the app. A DataSource wraps exactly one external source (a Retrofit endpoint, a Room DAO, DataStore) — no business logic, no cross-source coordination. UI and Domain depend on Repositories only, never on a DataSource directly.

Two principles the official guide names explicitly and that apply here without exception:

- **Unidirectional Data Flow (UDF)**: state flows down (Repository → ViewModel → Composable), events flow up (Composable → ViewModel → Repository). Never mutate state from a lower layer in direct response to a UI event without routing it back up through this cycle.
- **Single Source of Truth (SSOT)**: every data type has exactly one owner that can mutate it — a Repository (or the database it wraps, for offline-first data) for app data, a ViewModel for pure UI state. Nothing outside the SSOT mutates that data directly.

```kotlin
// Data layer — DataSource: wraps one external source, no business logic
class UserRemoteDataSource(private val api: UserApi) {
    suspend fun fetchUser(id: String): UserDto = api.getUser(id)
}

// Data layer — Repository: single source of truth, exposed as an interface
interface UserRepository {
    val currentUser: Flow<User?>
    suspend fun refresh(id: String)
}

class UserRepositoryImpl(
    private val remote: UserRemoteDataSource,
) : UserRepository {
    private val _currentUser = MutableStateFlow<User?>(null)
    override val currentUser: Flow<User?> = _currentUser.asStateFlow()

    override suspend fun refresh(id: String) {
        val dto = remote.fetchUser(id)
        _currentUser.value = User(name = dto.name, email = dto.email) // raw -> domain model
    }
}

// UI layer — ViewModel: transforms repository data into UI state
class UserProfileViewModel(
    private val repository: UserRepository,
) : ViewModel() {
    val uiState: StateFlow<User?> = repository.currentUser
        .stateIn(viewModelScope, SharingStarted.WhileSubscribed(5_000), null)

    fun onRefresh(id: String) = viewModelScope.launch { repository.refresh(id) }
}
```

## General principles

- **Dependency direction**: a ViewModel or UseCase never depends on Android framework classes, Compose, or a `DataSource` directly — only on a `Repository`'s abstract interface. Infrastructure (Retrofit clients, Room, DataStore) sits behind that interface, not the other way around.
  
```kotlin
  // Bad — a ViewModel depends on a concrete infrastructure type
  class OrderViewModel(private val repository: OrderRepositoryImpl) // concrete implementation

  // Good — the ViewModel depends on an abstraction; the data layer implements it
  interface OrderRepository {
      suspend fun getById(id: String): Order?
  }

  class OrderViewModel(private val repository: OrderRepository)
```

- **Testability drives boundaries**: if a piece of logic can't be unit-tested without spinning up a database, an HTTP client, or Compose, the boundary is probably wrong.

```kotlin
  // Bad — the business rule can't be tested without a live data source
  class PricingService(private val dataSource: CustomerDataSource) {
      suspend fun calculateFinalPrice(customerId: String): Double {
          val customer = dataSource.fetchCustomer(customerId)
          return if (customer.isVip) customer.basePrice * 0.9 else customer.basePrice
      }
  }

  // Good — the rule is pure and testable in isolation; data access is separate
  fun calculateFinalPrice(isVip: Boolean, basePrice: Double): Double =
      if (isVip) basePrice * 0.9 else basePrice
```

- **Always depend on an interface, from the first implementation — testability alone justifies it.** Don't wait for a second real implementation before introducing the abstraction; needing to substitute a test double when unit-testing a consumer is reason enough on its own. Applies at every layer boundary — Repository, DataSource, UseCase alike — no carve-out for "it's simple" or "there's only one implementation today."

```kotlin
  interface EmailSender {
      suspend fun send(to: String, body: String)
  }

  class SmtpEmailSender : EmailSender {
      override suspend fun send(to: String, body: String) { /* ... */ }
  }

  class OrderConfirmationHandler(private val emailSender: EmailSender) {
      // Testable in isolation with a fake EmailSender — no real SMTP call needed
      // to verify a confirmation gets sent after an order is placed.
  }
```

- **Consistency within a project beats a "better" pattern mid-stream**: every feature follows the same UI/Domain/Data split. Add a Domain layer for one feature only when that feature's logic actually warrants a UseCase — not as an inconsistent house-style variation applied to some features and not others.
