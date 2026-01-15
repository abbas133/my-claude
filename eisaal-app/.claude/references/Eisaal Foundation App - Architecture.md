# Eisaal Foundation App - Architecture & Implementation Plan

A comprehensive modular Flutter application using Clean Architecture with Supabase backend, featuring 20+ modules across three main categories (Today, Resources, Community). The plan emphasizes strict module isolation, offline-first capability, multi-language RTL support, and headless Python services for complex calculations.

---

## Project Directory Structure

```
eisaal_app/
├── lib/
│   ├── core/                          # Shared infrastructure
│   │   ├── config/                    # App configuration
│   │   │   ├── app_config.dart
│   │   │   ├── supabase_config.dart
│   │   │   └── env_config.dart
│   │   ├── constants/                 # App-wide constants
│   │   │   ├── api_endpoints.dart
│   │   │   ├── asset_paths.dart
│   │   │   ├── hive_boxes.dart
│   │   │   └── string_constants.dart
│   │   ├── di/                        # Dependency injection setup
│   │   │   └── injection_container.dart
│   │   ├── errors/                    # Error handling
│   │   │   ├── exceptions.dart
│   │   │   └── failures.dart
│   │   ├── extensions/                # Dart extensions
│   │   │   ├── context_extensions.dart
│   │   │   ├── date_extensions.dart
│   │   │   ├── string_extensions.dart
│   │   │   └── hijri_extensions.dart
│   │   ├── network/                   # API client, network info
│   │   │   ├── api_client.dart
│   │   │   ├── network_info.dart
│   │   │   └── interceptors/
│   │   │       ├── auth_interceptor.dart
│   │   │       └── error_interceptor.dart
│   │   ├── utils/                     # Utilities
│   │   │   ├── hijri_utils.dart
│   │   │   ├── prayer_time_utils.dart
│   │   │   ├── validators.dart
│   │   │   └── date_formatters.dart
│   │   └── widgets/                   # Shared widgets
│   │       ├── arabic_text_widget.dart
│   │       ├── loading_indicator.dart
│   │       ├── error_widget.dart
│   │       ├── shimmer_loading.dart
│   │       └── responsive_builder.dart
│   │
│   ├── modules/                       # Standalone module packages
│   │   │
│   │   ├── theme/                     # 🎨 THEME MODULE
│   │   │   ├── data/
│   │   │   │   ├── models/
│   │   │   │   │   └── theme_preference_model.dart
│   │   │   │   └── datasources/
│   │   │   │       └── theme_local_datasource.dart
│   │   │   ├── domain/
│   │   │   │   ├── entities/
│   │   │   │   │   └── theme_state.dart
│   │   │   │   └── repositories/
│   │   │   │       └── theme_repository.dart
│   │   │   └── presentation/
│   │   │       ├── themes/
│   │   │       │   ├── divine_reflection_theme.dart
│   │   │       │   ├── crimson_tear_theme.dart
│   │   │       │   ├── inner_sanctum_theme.dart
│   │   │       │   ├── grounded_earth_theme.dart
│   │   │       │   └── persian_sky_theme.dart
│   │   │       ├── providers/
│   │   │       │   └── theme_provider.dart
│   │   │       └── app_theme.dart
│   │   │
│   │   ├── localization/              # 🌐 TRANSLATION MODULE
│   │   │   ├── l10n/
│   │   │   │   ├── app_en.arb
│   │   │   │   ├── app_ur.arb
│   │   │   │   ├── app_hi.arb
│   │   │   │   └── app_gu.arb
│   │   │   ├── providers/
│   │   │   │   └── locale_provider.dart
│   │   │   ├── utils/
│   │   │   │   └── rtl_helper.dart
│   │   │   └── app_localizations.dart
│   │   │
│   │   ├── auth/                      # Authentication Module
│   │   │   ├── data/
│   │   │   │   ├── datasources/
│   │   │   │   │   ├── auth_remote_datasource.dart
│   │   │   │   │   └── auth_local_datasource.dart
│   │   │   │   ├── models/
│   │   │   │   │   └── user_model.dart
│   │   │   │   └── repositories/
│   │   │   │       └── auth_repository_impl.dart
│   │   │   ├── domain/
│   │   │   │   ├── entities/
│   │   │   │   │   └── user.dart
│   │   │   │   ├── repositories/
│   │   │   │   │   └── auth_repository.dart
│   │   │   │   └── usecases/
│   │   │   │       ├── sign_in_with_google.dart
│   │   │   │       ├── sign_out.dart
│   │   │   │       └── get_current_user.dart
│   │   │   └── presentation/
│   │   │       ├── pages/
│   │   │       │   ├── splash_page.dart
│   │   │       │   ├── login_page.dart
│   │   │       │   └── profile_setup_page.dart
│   │   │       ├── widgets/
│   │   │       │   └── google_sign_in_button.dart
│   │   │       └── providers/
│   │   │           └── auth_provider.dart
│   │   │
│   │   ├── home/                      # Today Tab (Dashboard)
│   │   │   ├── data/
│   │   │   ├── domain/
│   │   │   └── presentation/
│   │   │       ├── pages/
│   │   │       │   └── home_page.dart
│   │   │       └── widgets/
│   │   │           ├── prayer_time_card.dart
│   │   │           ├── hijri_date_card.dart
│   │   │           ├── daily_aamaal_widget.dart
│   │   │           ├── today_aamaal_widget.dart
│   │   │           ├── verse_of_day_card.dart
│   │   │           └── hadees_of_day_card.dart
│   │   │
│   │   ├── prayer_times/              # Namaz Timing & Reminders
│   │   │   ├── data/
│   │   │   │   ├── datasources/
│   │   │   │   │   ├── prayer_times_remote_datasource.dart
│   │   │   │   │   └── prayer_times_local_datasource.dart
│   │   │   │   ├── models/
│   │   │   │   │   └── prayer_time_model.dart
│   │   │   │   └── repositories/
│   │   │   │       └── prayer_times_repository_impl.dart
│   │   │   ├── domain/
│   │   │   │   ├── entities/
│   │   │   │   │   └── prayer_time.dart
│   │   │   │   ├── repositories/
│   │   │   │   │   └── prayer_times_repository.dart
│   │   │   │   └── usecases/
│   │   │   │       ├── get_prayer_times.dart
│   │   │   │       └── get_next_prayer.dart
│   │   │   └── presentation/
│   │   │       ├── pages/
│   │   │       │   └── prayer_times_page.dart
│   │   │       ├── widgets/
│   │   │       │   └── prayer_time_tile.dart
│   │   │       └── providers/
│   │   │           └── prayer_times_provider.dart
│   │   │
│   │   ├── hijri_calendar/            # Hijri Date & Events
│   │   │   ├── data/
│   │   │   ├── domain/
│   │   │   └── presentation/
│   │   │
│   │   ├── quran/                     # Quran Reader with Tajweed
│   │   │   ├── data/
│   │   │   │   ├── datasources/
│   │   │   │   │   └── quran_local_datasource.dart
│   │   │   │   ├── models/
│   │   │   │   │   ├── surah_model.dart
│   │   │   │   │   ├── ayah_model.dart
│   │   │   │   │   └── bookmark_model.dart
│   │   │   │   └── repositories/
│   │   │   │       └── quran_repository_impl.dart
│   │   │   ├── domain/
│   │   │   │   ├── entities/
│   │   │   │   │   ├── surah.dart
│   │   │   │   │   ├── ayah.dart
│   │   │   │   │   └── bookmark.dart
│   │   │   │   ├── repositories/
│   │   │   │   │   └── quran_repository.dart
│   │   │   │   └── usecases/
│   │   │   │       ├── get_surah_list.dart
│   │   │   │       ├── get_ayahs.dart
│   │   │   │       ├── verify_quran_integrity.dart
│   │   │   │       └── manage_bookmarks.dart
│   │   │   └── presentation/
│   │   │       ├── pages/
│   │   │       │   ├── surah_list_page.dart
│   │   │       │   └── quran_reader_page.dart
│   │   │       ├── widgets/
│   │   │       │   ├── tajweed_text_widget.dart
│   │   │       │   ├── verse_widget.dart
│   │   │       │   └── translation_toggle.dart
│   │   │       └── providers/
│   │   │           └── quran_provider.dart
│   │   │
│   │   ├── duas/                      # Duas & Ziyarat Library
│   │   │   ├── data/
│   │   │   ├── domain/
│   │   │   └── presentation/
│   │   │
│   │   ├── mohr_e_ameen/              # Sajda Counter (ML Kit)
│   │   │   ├── data/
│   │   │   ├── domain/
│   │   │   └── presentation/
│   │   │       ├── pages/
│   │   │       │   └── mohr_e_ameen_page.dart
│   │   │       ├── widgets/
│   │   │       │   ├── sajda_counter_widget.dart
│   │   │       │   └── rakat_display.dart
│   │   │       └── providers/
│   │   │           └── sajda_detection_provider.dart
│   │   │
│   │   ├── qibla/                     # Qibla Compass
│   │   │   ├── data/
│   │   │   ├── domain/
│   │   │   └── presentation/
│   │   │
│   │   ├── khums_calculator/          # Khums with Marja Logic
│   │   │   ├── data/
│   │   │   │   ├── datasources/
│   │   │   │   │   └── khums_remote_datasource.dart
│   │   │   │   └── models/
│   │   │   │       ├── khums_input_model.dart
│   │   │   │       └── khums_result_model.dart
│   │   │   ├── domain/
│   │   │   │   ├── entities/
│   │   │   │   │   ├── khums_input.dart
│   │   │   │   │   └── khums_result.dart
│   │   │   │   ├── repositories/
│   │   │   │   │   └── khums_repository.dart
│   │   │   │   └── usecases/
│   │   │   │       └── calculate_khums.dart
│   │   │   └── presentation/
│   │   │       ├── pages/
│   │   │       │   ├── khums_input_page.dart
│   │   │       │   └── khums_result_page.dart
│   │   │       ├── widgets/
│   │   │       │   └── marja_selector.dart
│   │   │       └── providers/
│   │   │           └── khums_provider.dart
│   │   │
│   │   ├── qaza_namaz/                # Qaza Namaz Tracker
│   │   │   ├── data/
│   │   │   ├── domain/
│   │   │   └── presentation/
│   │   │
│   │   ├── qaza_roza/                 # Qaza Roza Tracker
│   │   │   ├── data/
│   │   │   ├── domain/
│   │   │   └── presentation/
│   │   │
│   │   ├── nifas_calculator/          # Nifas Calculator
│   │   │   ├── data/
│   │   │   ├── domain/
│   │   │   └── presentation/
│   │   │
│   │   ├── fiqh_masail/               # Fiqhi Q&A
│   │   │   ├── data/
│   │   │   ├── domain/
│   │   │   └── presentation/
│   │   │
│   │   ├── wasiyat/                   # Will Generator
│   │   │   ├── data/
│   │   │   ├── domain/
│   │   │   └── presentation/
│   │   │
│   │   ├── baby_names/                # Baby Name Suggestions
│   │   │   ├── data/
│   │   │   ├── domain/
│   │   │   └── presentation/
│   │   │
│   │   ├── matrimonial/               # Matrimonial Service
│   │   │   ├── data/
│   │   │   │   ├── datasources/
│   │   │   │   │   └── matrimonial_remote_datasource.dart
│   │   │   │   └── models/
│   │   │   │       ├── matrimonial_profile_model.dart
│   │   │   │       └── wali_verification_model.dart
│   │   │   ├── domain/
│   │   │   │   ├── entities/
│   │   │   │   │   └── matrimonial_profile.dart
│   │   │   │   ├── repositories/
│   │   │   │   │   └── matrimonial_repository.dart
│   │   │   │   └── usecases/
│   │   │   │       ├── search_profiles.dart
│   │   │   │       ├── create_profile.dart
│   │   │   │       └── verify_wali.dart
│   │   │   └── presentation/
│   │   │       ├── pages/
│   │   │       │   ├── matrimonial_browse_page.dart
│   │   │       │   ├── matrimonial_profile_page.dart
│   │   │       │   └── matrimonial_create_page.dart
│   │   │       ├── widgets/
│   │   │       │   ├── profile_card.dart
│   │   │       │   └── privacy_tier_indicator.dart
│   │   │       └── providers/
│   │   │           └── matrimonial_provider.dart
│   │   │
│   │   ├── blood_donation/            # Blood Donor Network
│   │   │   ├── data/
│   │   │   │   ├── datasources/
│   │   │   │   │   └── blood_donation_remote_datasource.dart
│   │   │   │   └── models/
│   │   │   │       ├── blood_donor_model.dart
│   │   │   │       └── blood_request_model.dart
│   │   │   ├── domain/
│   │   │   │   ├── entities/
│   │   │   │   │   ├── blood_donor.dart
│   │   │   │   │   └── blood_request.dart
│   │   │   │   ├── repositories/
│   │   │   │   │   └── blood_donation_repository.dart
│   │   │   │   └── usecases/
│   │   │   │       ├── find_nearby_donors.dart
│   │   │   │       ├── register_as_donor.dart
│   │   │   │       └── create_blood_request.dart
│   │   │   └── presentation/
│   │   │       ├── pages/
│   │   │       │   ├── donor_map_page.dart
│   │   │       │   └── donor_registration_page.dart
│   │   │       └── providers/
│   │   │           └── blood_donation_provider.dart
│   │   │
│   │   ├── teachers/                  # Teacher/Student Directory
│   │   │   ├── data/
│   │   │   ├── domain/
│   │   │   └── presentation/
│   │   │
│   │   ├── live_classes/              # Najaf Classes
│   │   │   ├── data/
│   │   │   ├── domain/
│   │   │   └── presentation/
│   │   │
│   │   ├── aamaal_marhomeen/          # Aamaal Service Requests
│   │   │   ├── data/
│   │   │   ├── domain/
│   │   │   └── presentation/
│   │   │
│   │   ├── shajra/                    # Genealogy Tree
│   │   │   ├── data/
│   │   │   │   ├── datasources/
│   │   │   │   │   └── shajra_remote_datasource.dart
│   │   │   │   └── models/
│   │   │   │       └── family_node_model.dart
│   │   │   ├── domain/
│   │   │   │   ├── entities/
│   │   │   │   │   └── family_node.dart
│   │   │   │   ├── repositories/
│   │   │   │   │   └── shajra_repository.dart
│   │   │   │   └── usecases/
│   │   │   │       └── get_family_tree.dart
│   │   │   └── presentation/
│   │   │       ├── pages/
│   │   │       │   ├── shajra_explorer_page.dart
│   │   │       │   └── shajra_submit_page.dart
│   │   │       ├── widgets/
│   │   │       │   └── tree_graph_widget.dart
│   │   │       └── providers/
│   │   │           └── shajra_provider.dart
│   │   │
│   │   ├── chat/                      # RAG Chatbot (Flowise)
│   │   │   ├── data/
│   │   │   ├── domain/
│   │   │   └── presentation/
│   │   │
│   │   ├── notifications/             # Push Notifications
│   │   │   ├── data/
│   │   │   ├── domain/
│   │   │   └── presentation/
│   │   │
│   │   └── settings/                  # User Preferences
│   │       ├── data/
│   │       ├── domain/
│   │       └── presentation/
│   │           ├── pages/
│   │           │   └── settings_page.dart
│   │           └── widgets/
│   │               ├── language_selector.dart
│   │               ├── theme_selector.dart
│   │               └── notification_toggles.dart
│   │
│   ├── data/                          # Shared data layer
│   │   ├── datasources/
│   │   │   ├── local/
│   │   │   │   ├── hive_datasource.dart
│   │   │   │   └── sqlite_datasource.dart
│   │   │   └── remote/
│   │   │       └── supabase_client.dart
│   │   ├── models/
│   │   │   └── base_model.dart
│   │   └── repositories/
│   │       └── base_repository.dart
│   │
│   ├── domain/                        # Shared domain layer
│   │   ├── entities/
│   │   │   └── base_entity.dart
│   │   ├── repositories/
│   │   │   └── base_repository_contract.dart
│   │   └── usecases/
│   │       └── usecase.dart
│   │
│   ├── services/                      # App-wide services
│   │   ├── notification_service.dart
│   │   ├── location_service.dart
│   │   ├── sync_service.dart
│   │   ├── ml_kit_service.dart
│   │   └── python_api_service.dart    # Headless Python API client
│   │
│   ├── router/                        # Navigation
│   │   ├── app_router.dart
│   │   └── route_guards.dart
│   │
│   └── main.dart
│
├── assets/
│   ├── fonts/
│   │   ├── Amiri-Regular.ttf
│   │   ├── Amiri-Bold.ttf
│   │   ├── NotoNaskhArabic-Regular.ttf
│   │   └── NotoNaskhArabic-Bold.ttf
│   ├── images/
│   │   ├── logo.png
│   │   ├── splash.png
│   │   └── icons/
│   ├── db/
│   │   ├── quran.db                   # Bundled Tanzil SQLite
│   │   └── duas.db                    # Bundled Duas database
│   └── patterns/
│       └── arabesque_pattern.svg
│
├── python_services/                   # Headless Python Microservices
│   ├── shared/
│   │   ├── requirements.txt
│   │   └── utils/
│   │       └── hijri_converter.py
│   │
│   ├── khums_service/
│   │   ├── main.py
│   │   ├── requirements.txt
│   │   ├── strategies/
│   │   │   ├── base_strategy.py
│   │   │   ├── sistani_strategy.py
│   │   │   ├── makarem_strategy.py
│   │   │   └── khamenei_strategy.py
│   │   └── models/
│   │       └── khums_models.py
│   │
│   ├── qaza_calculator_service/
│   │   ├── main.py
│   │   ├── requirements.txt
│   │   └── calculators/
│   │       ├── namaz_calculator.py
│   │       └── roza_calculator.py
│   │
│   └── hijri_service/
│       ├── main.py
│       ├── requirements.txt
│       └── converters/
│           └── hijri_gregorian.py
│
├── test/
│   ├── unit/
│   │   ├── core/
│   │   ├── modules/
│   │   └── services/
│   ├── widget/
│   │   └── modules/
│   └── mocks/
│
├── integration_test/
│   ├── auth_flow_test.dart
│   ├── prayer_times_test.dart
│   └── offline_sync_test.dart
│
├── pubspec.yaml
├── l10n.yaml
├── analysis_options.yaml
└── README.md
```

---

## Implementation Steps

### Step 1: Project Initialization & Core Infrastructure (Week 1-2)

**1.1 Initialize Flutter Project**

```bash
flutter create eisaal_app --org io.eisaal --platforms android,ios,web
cd eisaal_app
```

**1.2 Configure `pubspec.yaml` with Dependencies**

```yaml
dependencies:
  flutter:
    sdk: flutter
  flutter_localizations:
    sdk: flutter
    
  # State Management
  flutter_riverpod: ^3.1.0
  riverpod_annotation: ^2.3.0
  
  # Backend
  supabase_flutter: ^2.8.0
  
  # Local Storage
  hive_flutter: ^1.1.0
  sqflite: ^2.3.0
  path: ^1.8.0
  path_provider: ^2.1.0
  
  # Network
  dio: ^5.4.0
  connectivity_plus: ^6.0.0
  
  # Location & Maps
  geolocator: ^12.0.0
  flutter_compass: ^0.8.0
  
  # Auth
  google_sign_in: ^6.2.0
  
  # UI
  flutter_svg: ^2.0.0
  cached_network_image: ^3.3.0
  shimmer: ^3.0.0
  
  # Navigation
  go_router: ^14.0.0
  
  # Utilities
  intl: ^0.19.0
  freezed_annotation: ^2.4.0
  json_annotation: ^4.8.0
  uuid: ^4.3.0
  crypto: ^3.0.0          # For SHA-256 verification
  
  # ML/AI (Mohr-e-Ameen)
  google_mlkit_pose_detection: ^0.10.0
  sensors_plus: ^4.0.0    # Proximity sensor
  
  # PDF Generation (Wasiyat)
  pdf: ^3.10.0
  printing: ^5.12.0
  
  # Fonts
  google_fonts: ^6.2.0
  
dev_dependencies:
  flutter_test:
    sdk: flutter
  build_runner: ^2.4.0
  riverpod_generator: ^2.6.0
  freezed: ^2.4.0
  json_serializable: ^6.7.0
  hive_generator: ^2.0.0
  riverpod_lint: ^2.3.0
  flutter_lints: ^3.0.0
  mockito: ^5.4.0
  mocktail: ^1.0.0
```

**1.3 Create Core Directory Structure**

- Set up all folders under `lib/core/`
- Create base classes for exceptions, failures, and use cases
- Configure dependency injection with Riverpod

**1.4 Configure Supabase**

- Create Supabase project
- Set up Google OAuth provider
- Configure Row Level Security (RLS) policies
- Create initial database schema

---

### Step 2: Theme Module (Week 2)

**2.1 Create Theme Definitions**

| Theme Name | Purpose | Primary Color | Background |
|------------|---------|---------------|------------|
| Divine Reflection | Default | `#C5A059` (Royal Gold) | `#F5F7FA` (Crystal White) |
| Crimson Tear | Muharram/Aza | `#7F1D1D` (Garnet Red) | `#121212` (Midnight Black) |
| Inner Sanctum | Reading Mode | `#FFB300` (Warm Amber) | `#FFF8E1` (Warm Cream) |
| Grounded Earth | Neutral | `#795548` (Cocoa) | `#FFF8E1` (Off-White) |
| Persian Sky | Alternative | `#546E7A` (Slate Blue) | `#FFFDF5` (Cream) |

**2.2 Implement ThemeProvider**

- Support light/dark mode toggle per theme
- Auto-switch to Crimson Tear during Muharram (Hijri month 1) and Safar (month 2)
- Persist theme preference in Hive

**2.3 Font Configuration**

- Bundle Arabic fonts (Amiri, Noto Naskh Arabic)
- Configurable font size for Arabic text
- Support for KFGQPC Uthmanic Script for Quran

---

### Step 3: Localization Module (Week 2-3)

**3.1 Configure l10n.yaml**

```yaml
arb-dir: lib/modules/localization/l10n
template-arb-file: app_en.arb
output-localization-file: app_localizations.dart
output-class: AppLocalizations
```

**3.2 Create ARB Files for All Languages**

| Language | Code | Direction | Status |
|----------|------|-----------|--------|
| English | `en` | LTR | Primary |
| Urdu | `ur` | RTL | Full support |
| Hindi | `hi` | LTR | Full support |
| Gujarati | `gu` | LTR | Full support |

**3.3 Implement RTL Support**

- Automatic RTL detection for Urdu
- Mixed LTR/RTL for Arabic text within LTR interfaces
- Proper text alignment for all UI elements

**3.4 Content Localization Strategy**

| Content Type | Approach |
|--------------|----------|
| UI Labels | ARB files with code generation |
| Quran Translation | Bundled in SQLite per language |
| Duas/Ziyarat | Database with language column |
| Fiqhi Masail | Database with language column |

---

### Step 4: Authentication Module (Week 3)

**4.1 Implement Google OAuth**

- PKCE flow for security
- Native Google Sign-In for mobile
- Web OAuth fallback
- Secure token storage

**4.2 Profile Creation Flow**
Collect required user information:

| Field | Purpose | Required |
|-------|---------|----------|
| Name | Identification | Yes |
| Gender | Qaza/Nifas calculators | Yes |
| City/State/Country | Prayer times, events | Yes |
| Preferred Matan Language | Quran/Dua text | Yes |
| Preferred Description Language | Translations | Yes |
| Khums Date | Annual reminder | Optional |

**4.3 Multi-Profile Support**

- Blood Donor profile
- Matrimonial profile
- Service Provider profile
- Teacher profile
- Student profile

---

### Step 5: Static Screens & Navigation

**Objective**
Establish app shell and navigation structure with static screens.

**Instructions**

- Create static versions of all screens as per the requirement document.
- Favor UI matching design mockups to pixel-perfect accuracy.
- Favor performance optimizations (lazy loading, widget recycling).
- Favor maintainability and reusability (modular widgets, clear separation of concerns).
- Instead of hardcoding strings, use localization keys.
- Instead of generic placeholders, use contextually relevant placeholder content.
- Use generic icons from `flutter_svg` or `Icons` where applicable.
- Use stock images and add them to `assets/images/` if needed.

**5.1 Implement Static Screens**

- About Us
- Contact Us
- Privacy Policy
- Terms of Service

**5.2 Implement App Shell with static screens**

- Ensure responsiveness across devices (mobile, tablet, web), across orientations and across different platforms (Android, iOS, Web).
- Ensure accessibility compliance (screen readers, font scaling, color contrast).

**5.3 Configure Navigation**

- Use `go_router` for declarative routing
- Support deep linking
- Handle authentication guards
- Implement nested navigation for modules

---

### Step 6: Phase 1 Core Features (Week 4-8)

#### 6.1 Prayer Times Module

- Calculate prayer times using location (Aladhan API or local calculation)
- Support for Shia-specific calculation methods
- Namaz reminders (Awwal Waqt, Ada time remaining)
- Mark prayer as completed

#### 6.2 Hijri Calendar Module

- Accurate Hijri date display
- Important events database (Wiladat, Shahadat, Historic)
- Event notifications
- Integration with theme auto-switch

#### 6.3 Quran Module

**Data Source**: Tanzil Project XML/SQL

**Features**:

- Surah list with Arabic and translated names
- Verse-by-verse reading
- Tajweed highlighting (Ghunna, Qalqala, Ikhfa color codes)
- Translation toggle (Urdu, Hindi, Gujarati, English)
- Transliteration support
- Bookmark management
- Configurable Arabic font and size

**Integrity Verification**:

```dart
// On first launch, verify Quran database
final dbBytes = await rootBundle.load('assets/db/quran.db');
final hash = sha256.convert(dbBytes.buffer.asUint8List());
final isValid = await verifyWithServer(hash.toString());
```

#### 6.4 Duas & Ziyarat Module

**Data Sources**: Duas.org, Mafatihul Jinan, Sahifa Sajjadia

**Features**:

- Categorized library (by book, occasion, purpose)
- Daily recitation suggestions based on Hijri date
- Day-of-week specific duas/ziyarat
- Audio playback (optional)
- Bookmark and favorites

#### 6.5 Digital Mohr-e-Ameen

**Primary Method**: Proximity sensor

**Algorithm**:

1. Screen turns black when sensor covered (battery saver)
2. Event trigger: distance < 1cm
3. Debounce: covered > 1.5s AND uncovered > 1s
4. Every 2 valid sajdah = 1 rakat

**Fallback**: Google ML Kit pose detection (for devices without proximity sensor)

**Integration**:

- Link to current namaz (mark as complete)
- Link to Qaza tracker (reduce count)

---

### Step 7: Headless Python Services (Week 6-8)

#### 7.1 Service Architecture

```
┌─────────────────┐     HTTPS/REST      ┌─────────────────────┐
│  Flutter App    │ ◄─────────────────► │  FastAPI Services   │
│                 │                      │  (Railway/Render)   │
└─────────────────┘                      └─────────────────────┘
                                                   │
                                         ┌─────────┴─────────┐
                                         │                   │
                                   ┌─────┴─────┐       ┌─────┴─────┐
                                   │  Khums    │       │  Qaza     │
                                   │  Service  │       │  Service  │
                                   └───────────┘       └───────────┘
```

#### 7.2 Khums Calculator Service

**Strategy Pattern for Marja-specific rules**:

```python
# strategies/base_strategy.py
class KhumsStrategy(ABC):
    @abstractmethod
    def calculate(self, income: float, savings: float, expenses: dict) -> KhumsResult:
        pass

# strategies/sistani_strategy.py
class SistaniStrategy(KhumsStrategy):
    def calculate(self, income, savings, expenses):
        # Ayatollah Sistani's specific deduction rules
        ...
        
# strategies/makarem_strategy.py
class MakaremStrategy(KhumsStrategy):
    def calculate(self, income, savings, expenses):
        # Ayatollah Makarem Shirazi's specific rules
        ...
```

**API Endpoint**:

```python
@app.post("/khums/calculate")
async def calculate_khums(
    income: float,
    savings: float,
    expenses: dict,
    marja: MarjaType
) -> KhumsResult:
    strategy = get_strategy(marja)
    return strategy.calculate(income, savings, expenses)
```

#### 7.3 Qaza Calculator Service

- Input: missed days, gender, start date
- Female logic: exclude estimated menses periods
- Output: total prayers/fasts, completion strategies

#### 7.4 Deployment

- **Recommended**: Railway or Render (free tier for non-profit)
- Fallback: Self-hosted on a VPS
- Authentication: API key or Supabase JWT verification

---

### Step 8: Phase 2 Calculator Features (Week 9-12)

#### 8.1 Khums Calculator Module

- Input: Annual income, savings, deductible expenses
- Marja selection (Sistani, Makarem, Khamenei, etc.)
- Call Python service for calculation
- Display: Sahme-Imam, Sahme-Sadat amounts
- Payment integration (link to Ijaza)
- Annual reminder based on user's Khums date

#### 8.2 Qaza Namaz Tracker

- Dashboard: Self, Father, Mother, Other Momin
- Input: Estimated missed days
- Gender-aware calculation (excludes menses for female)
- Completion strategies:
  - Time-based: "Complete in 90 days"
  - Availability-based: "2 days of qaza daily"
- Integration with Mohr-e-Ameen

#### 8.3 Qaza Roza Tracker

- Similar to Namaz tracker
- Notes about pregnancy rules
- Suggested fasting patterns (alternate, weekend, recommended days)

#### 8.4 Nifas Calculator

- Based on Tauzihul Masail rules
- Input: relevant dates
- Output: Nifas period calculation

#### 8.5 Wasiyat (Will) Generator

**Phase I Features**:

- Model will based on Ayatullah Marashi Najafi's wasiyat nama
- Auto-include Qaza Namaz/Roza sections
- External document links
- Video attachment (user testimony + witnesses)
- PDF export

**Disclaimer**: "This is a Sharia-valid will but may not stand as a legal document in court."

#### 8.6 Baby Names Module

- Database of Shia baby names
- Search by meaning, origin
- Hijri date significance: prioritize names matching historic events
- "Suggest a Name" form (admin approval required)

---

### Step 9: Phase 3 Community Features (Week 13-20)

#### 9.1 Matrimonial Service

**Subscription Model**: Paid-only for authenticity

**Profile Creation**:

- Female profiles require Wali verification (email link)
- Searchable only after verification

**Privacy Tiers**:

| Tier | Use Case | Photo | Name | Contact |
|------|----------|-------|------|---------|
| Tier 1 | Nikah-e-Daimi | Blurred until match | Visible | After match |
| Tier 2 | Nikah-e-Mutah | Blurred | Masked | Strict privacy |

**Search Filters**:

- State/City
- Age group
- Nikah type (Daimi/Mutah/Both)

**Add-ons**:

- Background verification (chargeable)

#### 9.2 Blood Donation Module

**Registration Flow**:

1. User registers with blood group
2. Consents to emergency notifications
3. Location stored with PostGIS

**"Uber Model" for Privacy**:

1. Requester posts need: "O+ needed at City Hospital"
2. Push notification to verified O+ donors within 10km (PostGIS geofencing)
3. Donor accepts request
4. Masked chat/VoIP call (no phone numbers exchanged)

**Supabase PostGIS Query**:

```sql
SELECT * FROM blood_donors
WHERE blood_group = 'O+'
AND ST_DWithin(
  location::geography,
  ST_MakePoint(longitude, latitude)::geography,
  10000  -- 10km radius
);
```

#### 9.3 Shajra (Genealogy) Module

- Interactive family tree visualization
- Submit family tree form
- Recursive SQL queries for tree traversal
- Graph visualization library (graphview or custom)

#### 9.4 Teacher Directory

- Register as Teacher/Student
- Profile: Online/Offline, City, Gender
- Search and filter
- Enrollment system

#### 9.5 Live Classes

- Google Meet links to Najaf classes
- Historic class recordings
- Access management TBD

#### 9.6 Aamaal for Marhomeen

- Request: Quran recitation, Qaza Namaz, Qaza Roza
- Service Provider registration (requires Aalim verification)
- Admin approval workflow

---

### Step 10: Chat & Notifications (Week 18-22)

#### 10.1 RAG Chatbot (Flowise + pgvector)

**Architecture**:

```
User Query → Flowise → pgvector (Supabase) → Verified Shia Sources → Response
```

**Skill-based Routing**:

- Hajj/Umrah → Hajj specialist
- Fiqh questions → Online Aalim
- Matrimonial → Moderator

**Escalation**: Bot → Human agent when needed

#### 10.2 Notification System

| Notification Type | Trigger | Data Source |
|-------------------|---------|-------------|
| Namaz Time | Time of day | Local config |
| Awwal Waqt Remaining | 30 min after Azan | Local config |
| Ada Time Remaining | Before next namaz | Local config |
| Important Dates | Hijri date | Database |
| Daily Hadees | Configured time | Database |
| Geolocation Events | Admin push | City/State/Country |
| Moon Sighting | Admin push | Geographic region |

**Implementation**: Firebase Cloud Messaging (FCM) or Supabase Edge Functions with push

---

## Naming Conventions & Coding Standards

### File Naming

```
lowercase_with_underscores.dart

Examples:
- prayer_time_widget.dart
- khums_calculator.dart
- supabase_auth_datasource.dart
```

### Class Naming

```dart
UpperCamelCase

Examples:
- PrayerTimeWidget
- KhumsCalculator
- SupabaseAuthDatasource
```

### Variables & Functions

```dart
lowerCamelCase

Examples:
- prayerTime
- calculateKhums()
- currentUser
```

### Constants

```dart
lowerCamelCase (NOT SCREAMING_CAPS)

Examples:
- defaultTimeout
- maxRetries
- apiBaseUrl
```

### Private Members

```dart
Leading underscore

Examples:
- _internalState
- _calculateInternal()
- _cache
```

### Provider Naming

```dart
descriptiveSuffixProvider

Examples:
- prayerTimesProvider
- currentHijriDateProvider
- themeControllerProvider
```

### Use Case Naming

```dart
VerbNoun (action-based)

Examples:
- CalculateKhums
- GetPrayerTimes
- SyncQazaCounts
- VerifyQuranIntegrity
```

### Model vs Entity Naming

```dart
// Data Layer (DTO)
UserModel
PrayerTimeModel

// Domain Layer (Business Object)
User
PrayerTime
```

### Import Order

```dart
// 1. Dart SDK
import 'dart:async';
import 'dart:convert';

// 2. Flutter SDK
import 'package:flutter/material.dart';

// 3. External packages
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:supabase_flutter/supabase_flutter.dart';

// 4. Internal packages (alphabetical)
import 'package:eisaal_app/core/errors/failures.dart';
import 'package:eisaal_app/modules/auth/domain/entities/user.dart';

// 5. Relative imports
import '../widgets/prayer_widget.dart';
import 'prayer_time_model.dart';
```

### Architecture Rules

1. **Layer Separation**:
   - Presentation → Domain → Data (one-way dependency)
   - UI never directly accesses datasources
   - Domain layer has no Flutter dependencies

2. **Module Isolation**:
   - Each module is self-contained
   - Cross-module communication via domain layer contracts
   - Shared code lives in `core/`

3. **Repository Pattern**:
   - Domain defines interfaces
   - Data implements interfaces
   - UI depends only on interfaces

4. **Use Cases**:
   - Single responsibility
   - One public method: `call()` or `execute()`
   - Return `Either<Failure, Success>` using `dartz` or `fpdart`

5. **Error Handling**:
   - Catch exceptions at repository level
   - Convert to `Failure` objects
   - Never throw from domain layer

---

## Database Schema (Supabase/PostgreSQL)

### Core Tables

```sql
-- Users
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email TEXT UNIQUE NOT NULL,
  name TEXT NOT NULL,
  gender TEXT CHECK (gender IN ('male', 'female')),
  city TEXT,
  state TEXT,
  country TEXT,
  location GEOGRAPHY(POINT),
  matan_language TEXT DEFAULT 'ar',
  description_language TEXT DEFAULT 'en',
  khums_date DATE,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Qaza Tracking
CREATE TABLE qaza_namaz (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  entity_type TEXT CHECK (entity_type IN ('self', 'father', 'mother', 'other')),
  entity_name TEXT,
  fajr_remaining INT DEFAULT 0,
  dhuhr_remaining INT DEFAULT 0,
  asr_remaining INT DEFAULT 0,
  maghrib_remaining INT DEFAULT 0,
  isha_remaining INT DEFAULT 0,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Matrimonial Profiles
CREATE TABLE matrimonial_profiles (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  is_active BOOLEAN DEFAULT false,
  gender TEXT NOT NULL,
  age INT,
  city TEXT,
  state TEXT,
  country TEXT,
  open_for TEXT[] CHECK (open_for <@ ARRAY['daimi', 'mutah']),
  photo_url TEXT,
  wali_email TEXT,
  wali_verified BOOLEAN DEFAULT false,
  subscription_expires_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Blood Donors
CREATE TABLE blood_donors (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  blood_group TEXT NOT NULL,
  location GEOGRAPHY(POINT),
  city TEXT,
  state TEXT,
  is_available BOOLEAN DEFAULT true,
  last_donation_date DATE,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Enable PostGIS
CREATE EXTENSION IF NOT EXISTS postgis;

-- Index for geo queries
CREATE INDEX blood_donors_location_idx ON blood_donors USING GIST (location);
```

---

## Testing Strategy

### Unit Tests

- Use cases
- Repository implementations
- Utility functions
- State providers

### Widget Tests

- Individual widgets
- Page layouts
- User interactions

### Integration Tests

- Authentication flow
- Offline sync
- Prayer time calculation
- Khums calculation API

### Coverage Target

- Core: 90%+
- Features: 80%+
- UI: 70%+

---

## Deployment & CI/CD

### Mobile (Android/iOS)

- Flutter build for Android APK/AAB
- iOS via Xcode/TestFlight
- Distribution: Google Play, App Store

### Web

- Flutter web build
- Deploy to Vercel/Netlify/Firebase Hosting

### Python Services

- Docker containers
- Deploy to Railway/Render
- Environment variables for secrets

### CI/CD Pipeline (GitHub Actions)

```yaml
- Run tests
- Build for all platforms
- Deploy Python services
- Deploy web app
- Publish to stores (on release tags)
```

---

## Further Considerations / Open Questions

### 1. State Management Confirmation

**Recommended**: Riverpod

- Compile-time safety
- Less boilerplate than Bloc
- Code generation support
- Easy testing

**Alternative**: Bloc (if team prefers)

### 2. Dart Package Dependencies

The following packages require confirmation before adding:

- `supabase_flutter`
- `hive_flutter`
- `google_mlkit_pose_detection`
- `flutter_riverpod`
- `freezed`
- `go_router`

### 3. Python Hosting Decision

| Option | Pros | Cons |
|--------|------|------|
| Railway | Free tier, easy deploy | May have limits |
| Render | Free tier, good DX | Cold starts |
| Self-hosted | Full control | Maintenance overhead |
| Dart-only | No external dependency | Complex logic in Dart |

### 4. Flowise Chatbot

- Self-hosted vs cloud?
- Requires Supabase `pgvector` extension
- Sources: verified Shia texts only

### 5. Payment Gateway

For Khums payments and Matrimonial subscriptions:

- **India**: Razorpay
- **International**: Stripe
- **Middle East**: Tap Payments

### 6. Video Call Integration

For Hajj guidance, Qirat correction, Online classes:

- Daily.co (Web-based)
- Agora.io (Native)
- Jitsi Meet (Open source)

### 7. Admin Panel Technology

Current consideration: Flutter Web

Alternatives:

- React Admin
- Retool
- Appsmith

---

## Timeline Summary

| Phase | Duration | Key Deliverables |
|-------|----------|------------------|
| **Phase 1** | Months 1-3 | Core infrastructure, Theme, Localization, Auth, Prayer Times, Quran, Duas, Mohr-e-Ameen, Hijri Calendar |
| **Phase 2** | Months 4-7 | Khums Calculator, Qaza Trackers, Wasiyat, Baby Names, Nifas Calculator, Python Services |
| **Phase 3** | Months 8-12 | Matrimonial, Blood Donation, Shajra, Teachers, Live Classes, Aamaal Marhomeen, Chat, Full Notifications |

---

## Risk Mitigation

| Risk | Mitigation |
|------|------------|
| Complex Arabic text rendering | Use proven fonts (Amiri), test early |
| Offline sync conflicts | Last-write-wins with conflict resolution |
| Quran data integrity | SHA-256 verification on every launch |
| Privacy violations | Strict RLS policies, no admin access to sensitive data |
| Python service downtime | Graceful degradation, show cached results |
| Payment failures | Clear error messages, retry logic, support contact |

---

*This plan is a living document. Update as decisions are made on open questions.*
