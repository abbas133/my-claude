---
name: dart-flutter-expert
description: Expert Flutter/Dart guidance. Use when working with Flutter projects, implementing BLoC/Riverpod patterns, writing Dart code, optimizing performance, or following Clean Architecture. Provides architecture patterns, widget best practices, and testing strategies.
---

# Dart & Flutter Expert

## Architecture (Clean Architecture)

```
lib/modules/feature/
├── data/           # DTOs, datasources, repository impl
├── domain/         # Entities, contracts, use cases
└── presentation/   # Pages, widgets, providers
```

## State Management

| Complexity | Use |
|------------|-----|
| Simple | setState, ValueNotifier |
| Medium | Provider, Riverpod |
| Complex | BLoC, Riverpod with codegen |

## Widget Best Practices

```dart
// Always use const
const MyWidget({super.key});

// Extract widgets, not methods
class _UserAvatar extends StatelessWidget { ... }

// Use freezed for models
@freezed
class User with _$User {
  const factory User({required String id}) = _User;
}

// Selective rebuilds
BlocSelector<AuthBloc, AuthState, bool>(
  selector: (state) => state is AuthLoading,
  builder: (context, isLoading) => ...,
)
```

## Performance

1. Use `const` everywhere possible
2. Use `ListView.builder` for lists
3. Use `CachedNetworkImage` for images
4. Avoid FutureBuilder with inline futures

## Testing

```dart
// BLoC test
blocTest<AuthBloc, AuthState>(
  'emits [Loading, Success] on login',
  build: () => AuthBloc(useCase: mockUseCase),
  act: (bloc) => bloc.add(LoginRequested()),
  expect: () => [AuthLoading(), AuthSuccess(user)],
);
```

## Commands

```bash
dart run build_runner build --delete-conflicting-outputs
flutter test --coverage
flutter analyze
```
