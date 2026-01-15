# Eisaal App - Architecture Recommendations

## Current vs Recommended Structure

### Module Consolidation

**Before (28 separate modules):**
```
lib/modules/
├── auth/
├── baby_names/
├── blood_donation/
├── chatbot/
├── duas/
├── hijri_calendar/
├── home/
├── khums_calculator/
├── matrimonial/
├── mohr_e_ameen/
├── nifas_calculator/
├── notifications/
├── prayer_times/
├── profile/
├── qaza_namaz/
├── qaza_roza/
├── quran/
├── shajra/
├── teachers/
├── wasiyat/
... (and more)
```

**After (Logical grouping by feature area):**
```
lib/modules/
├── today/                      # TODAY tab features
│   ├── home/
│   ├── prayer_times/
│   ├── hijri_calendar/
│   └── aamaal/
│
├── resources/                  # RESOURCES tab features
│   ├── quran/
│   ├── duas/
│   └── knowledge_base/
│
├── community/                  # COMMUNITY tab features
│   ├── matrimonial/
│   ├── blood_donation/
│   ├── shajra/
│   ├── teachers/
│   └── chatbot/
│
├── calculators/                # All calculation features
│   ├── shared/                 # Shared widgets, models
│   │   ├── widgets/
│   │   │   ├── marja_selector.dart
│   │   │   ├── calculation_input_card.dart
│   │   │   └── calculation_result_card.dart
│   │   └── models/
│   │       └── marja.dart
│   ├── khums/
│   ├── qaza_namaz/
│   ├── qaza_roza/
│   └── nifas/
│
├── tools/                      # Utility tools
│   ├── mohr_e_ameen/
│   ├── wasiyat/
│   └── baby_names/
│
├── auth/                       # Authentication
├── profile/                    # User settings
└── notifications/              # Notification system
```

### Benefits of This Structure

1. **Related features share code** - All calculators can share `marja_selector` widget
2. **Clearer mental model** - Matches the 3-tab navigation structure
3. **Easier for Claude** - Understands where to put things
4. **Scalable** - New features fit into existing categories

---

## Clean Architecture Within Each Module

```
lib/modules/{category}/{feature}/
├── data/
│   ├── datasources/
│   │   ├── {feature}_local_datasource.dart
│   │   └── {feature}_remote_datasource.dart
│   ├── models/
│   │   └── {feature}_model.dart       # JSON serializable
│   └── repositories/
│       └── {feature}_repository_impl.dart
│
├── domain/
│   ├── entities/
│   │   └── {feature}_entity.dart      # Pure Dart/Freezed
│   ├── repositories/
│   │   └── {feature}_repository.dart  # Abstract contract
│   └── usecases/
│       └── get_{feature}.dart
│
└── presentation/
    ├── pages/
    │   └── {feature}_page.dart
    ├── widgets/
    │   └── {feature}_card.dart
    └── providers/
        └── {feature}_provider.dart    # Riverpod
```

---

## Layer Dependency Rules

```
┌─────────────────────────────────────────┐
│             PRESENTATION                │
│  (Pages, Widgets, Providers)            │
│                                         │
│  Can import: domain, core               │
│  Cannot import: data (directly)         │
└────────────────┬────────────────────────┘
                 │ uses
                 ▼
┌─────────────────────────────────────────┐
│               DOMAIN                    │
│  (Entities, Repository Contracts,       │
│   Use Cases)                            │
│                                         │
│  Can import: core (errors only)         │
│  Cannot import: data, presentation      │
└────────────────┬────────────────────────┘
                 │ implemented by
                 ▼
┌─────────────────────────────────────────┐
│                DATA                     │
│  (Models, DataSources, Repo Impl)       │
│                                         │
│  Can import: domain, core               │
│  Cannot import: presentation            │
└─────────────────────────────────────────┘
```

---

## Python Services Integration

### Recommended Structure

```
lib/
├── core/
│   └── services/
│       └── python_api/
│           ├── python_api_client.dart
│           ├── python_api_config.dart
│           └── python_api_exception.dart
```

### Python API Client

```dart
// lib/core/services/python_api/python_api_client.dart

import 'package:dio/dio.dart';

class PythonApiClient {
  final Dio _dio;
  
  PythonApiClient({required String baseUrl})
      : _dio = Dio(BaseOptions(
          baseUrl: baseUrl,
          connectTimeout: Duration(seconds: 30),
          receiveTimeout: Duration(seconds: 30),
        ));

  Future<T> post<T>(
    String endpoint,
    Map<String, dynamic> data,
    T Function(Map<String, dynamic>) fromJson,
  ) async {
    try {
      final response = await _dio.post(endpoint, data: data);
      return fromJson(response.data);
    } on DioException catch (e) {
      throw PythonApiException.fromDioError(e);
    }
  }
}
```

### Usage in Calculator Module

```dart
// lib/modules/calculators/khums/data/datasources/khums_python_datasource.dart

class KhumsPythonDataSource {
  final PythonApiClient _client;

  KhumsPythonDataSource(this._client);

  Future<KhumsResultModel> calculate(KhumsInputModel input) async {
    return _client.post(
      '/khums/calculate',
      input.toJson(),
      KhumsResultModel.fromJson,
    );
  }
}
```

---

## Shared Components

### When to Share

| Scenario | Location |
|----------|----------|
| Used by 2+ modules in same category | `modules/{category}/shared/` |
| Used across categories | `core/widgets/` or `core/utils/` |
| Feature-specific | Keep in feature module |

### Calculator Shared Components

```
lib/modules/calculators/shared/
├── widgets/
│   ├── marja_selector_widget.dart    # All calculators need this
│   ├── calculation_card.dart
│   └── result_display_widget.dart
├── models/
│   └── marja.dart                    # Marja enum/class
└── utils/
    └── islamic_date_utils.dart
```

---

## File Removal Recommendations

**Remove empty placeholder folders:**
```bash
# If these are empty, delete them
rm -rf lib/data/
rm -rf lib/domain/
```

Only recreate when you have truly shared domain/data that doesn't belong to any module.

---

## Migration Steps

### Phase 1: Restructure Folders (Do Now)

```bash
# Create new structure
mkdir -p lib/modules/today
mkdir -p lib/modules/resources
mkdir -p lib/modules/community
mkdir -p lib/modules/calculators/shared
mkdir -p lib/modules/tools

# Move existing modules
mv lib/modules/home lib/modules/today/
mv lib/modules/prayer_times lib/modules/today/
mv lib/modules/hijri_calendar lib/modules/today/

mv lib/modules/quran lib/modules/resources/
mv lib/modules/duas lib/modules/resources/

mv lib/modules/matrimonial lib/modules/community/
mv lib/modules/blood_donation lib/modules/community/
mv lib/modules/shajra lib/modules/community/

mv lib/modules/khums_calculator lib/modules/calculators/khums
mv lib/modules/qaza_namaz lib/modules/calculators/qaza_namaz
mv lib/modules/qaza_roza lib/modules/calculators/qaza_roza
mv lib/modules/nifas_calculator lib/modules/calculators/nifas

mv lib/modules/mohr_e_ameen lib/modules/tools/
mv lib/modules/wasiyat lib/modules/tools/
mv lib/modules/baby_names lib/modules/tools/
```

### Phase 2: Update Imports

After moving, update all imports. Use VS Code's "Update imports on move" or:

```bash
# Find imports that need updating
grep -rn "import.*modules/home" lib/
grep -rn "import.*modules/prayer_times" lib/
# etc.
```

### Phase 3: Extract Shared Components

As you build calculators, extract common widgets to `calculators/shared/`.

---

## Checklist

- [ ] Restructure folders as recommended
- [ ] Update imports
- [ ] Delete empty `lib/data/` and `lib/domain/` if unused
- [ ] Create `calculators/shared/` for common calculator widgets
- [ ] Setup `core/services/python_api/` for Python integration
- [ ] Update router paths to match new structure
