## Testing Providers

### Unit Testing
```dart
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:flutter_test/flutter_test.dart';

void main() {
  test('counterProvider increments', () {
    final container = ProviderContainer();
    addTearDown(container.dispose);

    // Initial state
    expect(container.read(counterProvider), 0);

    // After increment
    container.read(counterProvider.notifier).increment();
    expect(container.read(counterProvider), 1);
  });

  test('asyncProvider handles errors', () async {
    final container = ProviderContainer(
      overrides: [
        userRepositoryProvider.overrideWith((ref) => MockUserRepository()),
      ],
    );
    addTearDown(container.dispose);

    final result = await container.read(currentUserProvider.future);
    expect(result, isA<User>());
  });
}
```

### Widget Testing
```dart
testWidgets('Widget uses provider', (tester) async {
  await tester.pumpWidget(
    ProviderScope(
      child: MaterialApp(
        home: MyWidget(),
      ),
    ),
  );

  expect(find.text('0'), findsOneWidget);
  
  await tester.tap(find.byIcon(Icons.add));
  await tester.pump();
  
  expect(find.text('1'), findsOneWidget);
});
```

## Performance Optimization

### Select Specific Fields
```dart
// ❌ Rebuilds on any settings change
final settings = ref.watch(settingsProvider);
return Text(settings.theme);

// ✅ Rebuilds only when theme changes
final theme = ref.watch(settingsProvider.select((s) => s.theme));
return Text(theme);
```

### Use Family for Parameterized Providers
```dart
@riverpod
Future<Dua> duaById(Ref ref, String id) async {
  return ref.watch(duaRepositoryProvider).getById(id);
}

// Usage - automatically cached by id
ref.watch(duaByIdProvider('123'));
```

### Keep Alive Important State
```dart
@Riverpod(keepAlive: true)
class AuthState extends _$AuthState {
  // Won't be disposed when no listeners
}
```

## Common Patterns

### Pagination
```dart
@riverpod
class DuaList extends _$DuaList {
  int _page = 1;
  final List<Dua> _items = [];
  bool _hasMore = true;

  @override
  Future<List<Dua>> build() async {
    return _fetchPage(_page);
  }

  Future<List<Dua>> _fetchPage(int page) async {
    final repo = ref.read(duaRepositoryProvider);
    final items = await repo.getPage(page);
    _items.addAll(items);
    _hasMore = items.length == 20; // Page size
    return _items;
  }

  Future<void> loadMore() async {
    if (!_hasMore) return;
    _page++;
    state = const AsyncLoading();
    state = await AsyncValue.guard(() => _fetchPage(_page));
  }

  Future<void> refresh() async {
    _page = 1;
    _items.clear();
    _hasMore = true;
    ref.invalidateSelf();
    await future;
  }
}
```

### Search with Debounce
```dart
@riverpod
class DuaSearch extends _$DuaSearch {
  Timer? _debounce;

  @override
  Future<List<Dua>> build() async {
    return [];
  }

  void search(String query) {
    _debounce?.cancel();
    _debounce = Timer(const Duration(milliseconds: 500), () {
      _performSearch(query);
    });
  }

  Future<void> _performSearch(String query) async {
    if (query.isEmpty) {
      state = const AsyncData([]);
      return;
    }

    state = const AsyncLoading();
    state = await AsyncValue.guard(() async {
      final repo = ref.read(duaRepositoryProvider);
      return repo.search(query);
    });
  }

  @override
  void dispose() {
    _debounce?.cancel();
    super.dispose();
  }
}
```