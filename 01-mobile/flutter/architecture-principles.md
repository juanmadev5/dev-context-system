---
tags: [flutter, architecture]
---

# Architecture Principles — Flutter

## UI / Domain / Data (MVVM)

Default architecture for every Flutter project, per [Flutter's official app-architecture guide](https://docs.flutter.dev/app-architecture/case-study). This is not a per-project choice — the layers below and the folder layout they imply are the one default. Three layers, dependencies always pointing downward, never upward:

```
View → ViewModel → (Use-Case →) Repository → Service
```

- **UI layer** — `View` + `ViewModel`, one ViewModel per View (1:1). Views are dumb widgets: only layout, animation, and simple conditionals — no business logic, no data access except through their ViewModel. ViewModels expose UI state (streams/values) and **Commands** (callback methods for user interactions); they must be unit-testable without pumping a widget tree.
- **Domain layer** *(optional)* — `Use-Case`s. Add one only when logic is genuinely complex, reused across more than one ViewModel, or combines data from multiple repositories. Skip it otherwise — a ViewModel is allowed to call a Repository directly.
- **Data layer** — `Repository` + `Service`. A Repository is the single source of truth for one data type: it transforms raw models into domain models and owns caching/retry/refresh logic. A Service is a stateless wrapper around exactly one external data source (a REST endpoint, a platform channel, local storage) — it holds no state and makes no business decisions. UI and Domain depend on Repositories only, never on a Service directly.

Cardinality: View↔ViewModel is 1:1; ViewModel→Repository/Use-Case is many:1; Repository↔Service is many:many; Repository→Repository never happens — combine data from two repositories in the ViewModel or a Use-Case, not by having one repository call another.

```dart
// Data layer — Service: stateless wrapper, one external source, no business logic
class UserApiService {
  UserApiService(this._httpClient);
  final http.Client _httpClient;

  Future<UserApiModel> fetchUser(String id) async { /* ... */ }
}

// Data layer — Repository: source of truth, owns caching/transform, exposed as an interface
abstract class UserRepository {
  Stream<User?> get currentUser;
  Future<void> refresh(String id);
}

class UserRepositoryRemote implements UserRepository {
  UserRepositoryRemote(this._service);
  final UserApiService _service;
  final _controller = StreamController<User?>.broadcast();

  @override
  Stream<User?> get currentUser => _controller.stream;

  @override
  Future<void> refresh(String id) async {
    final apiModel = await _service.fetchUser(id);
    _controller.add(User(name: apiModel.name, email: apiModel.email)); // raw -> domain model
  }
}

// UI layer — ViewModel: transforms repository data into UI state, exposes Commands
class UserProfileViewModel {
  UserProfileViewModel(this._repository);
  final UserRepository _repository;

  Stream<User?> get user => _repository.currentUser;
  Future<void> refreshCommand(String id) => _repository.refresh(id);
}
```

### Folder organization: hybrid

The UI layer is organized **by feature** (vertical slice); the Domain and Data layers are organized **by type** (layered):

```
lib/
  ui/
    core/                 # shared widgets, theme
    <feature_name>/
      view_models/
      widgets/
  domain/
    models/
    use_cases/             # only the features that actually need one
  data/
    repositories/
    services/
    models/                 # raw/API models, distinct from domain models
```

## General principles

- **Dependency direction**: a ViewModel or Use-Case never depends on Flutter widgets, and never depends on a `Service` directly — only on a `Repository`'s abstract interface. Infrastructure (HTTP clients, local storage, platform channels) sits behind that interface, not the other way around.
  ```dart
  // Bad — a ViewModel depends on a concrete infrastructure type
  class OrderViewModel {
    final OrderRepositoryRemote repository; // concrete implementation
    OrderViewModel(this.repository);
  }

  // Good — the ViewModel depends on an abstraction; the data layer implements it
  abstract class OrderRepository {
    Future<Order?> getById(String id);
  }

  class OrderViewModel {
    final OrderRepository repository;
    OrderViewModel(this.repository);
  }
  ```
- **Testability drives boundaries**: if a piece of logic can't be unit-tested without spinning up a database, an HTTP server, or the widget tree, the boundary is probably wrong.
  ```dart
  // Bad — the business rule can't be tested without a live data source
  class PricingService {
    double calculateFinalPrice(String customerId) {
      final customer = dataSource.fetchCustomer(customerId);
      return customer.isVip ? customer.basePrice * 0.9 : customer.basePrice;
    }
  }

  // Good — the rule is pure and testable in isolation; data access is separate
  double calculateFinalPrice({required bool isVip, required double basePrice}) =>
      isVip ? basePrice * 0.9 : basePrice;
  ```
- **Always depend on an interface, from the first implementation — testability alone justifies it.** Don't wait for a second real implementation before introducing the abstraction; needing to substitute a test double when unit-testing a consumer is reason enough on its own. Applies at every layer boundary — Repository, Service, Use-Case alike — no carve-out for "it's simple" or "there's only one implementation today."
  ```dart
  abstract class EmailSender {
    Future<void> send(String to, String body);
  }

  class SmtpEmailSender implements EmailSender {
    Future<void> send(String to, String body) async { /* ... */ }
  }

  class OrderConfirmationHandler {
    final EmailSender emailSender;
    OrderConfirmationHandler(this.emailSender);
    // Testable in isolation with a fake EmailSender — no real SMTP call needed
    // to verify a confirmation gets sent after an order is placed.
  }
  ```
- **Consistency within a project beats a "better" pattern mid-stream**: every feature follows the same UI/Domain/Data split. Add a Domain layer for one feature only when that feature's logic actually warrants a Use-Case — not as an inconsistent house-style variation applied to some features and not others.

## See also

- [[01-mobile/flutter/flutter|flutter]]
- [[01-mobile/flutter/coding-standards|coding-standards]]
- [[01-mobile/flutter/code-review|code-review]]
