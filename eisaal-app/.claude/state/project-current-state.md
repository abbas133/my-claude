# Eisaal Foundation App - Project Current State

**Last Updated:** 2026-01-16
**Current Branch:** feat-phase1-validate
**Flutter Version:** 3.x
**Phase:** Phase 1 - The Foundation

---

## Project Overview

Eisaal Foundation is a comprehensive Islamic lifestyle platform built with Flutter, targeting the Shia Muslim community. The app serves as a "Digital Sanctuary" supporting users from daily practice to lifecycle events.

---

## Implementation Status

### Phase 1: The Foundation (Current Phase)

| Feature | Status | Notes |
|---------|--------|-------|
| Prayer Times | **Working** | GPS + profile location fallback via geocoding |
| Hijri Calendar | **Working** | Basic calendar with events |
| Quran Reader | **UI Ready** | Stub data, needs Tanzil integration |
| Duas & Ziyarat | **UI Ready** | Stub data |
| Mohr-e-Ameen | **UI Ready** | Proximity sensor logic pending |
| User Authentication | **Working** | Google OAuth, Email login, Guest mode |
| Profile Setup | **Working** | City/State/Country, Gender, Marja selection |

### Phase 2: Lifecycle Utilities (Planned)

| Feature | Status | Notes |
|---------|--------|-------|
| Khums Calculator | **UI Ready** | Calculator page exists, needs Marja-specific logic |
| Qaza Namaz Tracker | **UI Ready** | Page exists, needs persistence |
| Qaza Roza Tracker | **UI Ready** | Page exists, needs persistence |
| Nifas Calculator | **Not Started** | - |
| Wasiyat Generator | **UI Ready** | Basic page, PDF generation pending |
| Baby Names | **UI Ready** | Page exists, needs database |

### Phase 3: Community Ecosystem (Planned)

| Feature | Status | Notes |
|---------|--------|-------|
| Matrimonial | **UI Ready** | Page exists, needs Supabase integration |
| Blood Donation | **UI Ready** | Page exists, needs PostGIS |
| Shajra (Genealogy) | **UI Ready** | Page exists, needs graph database |
| Teachers Directory | **UI Ready** | Page exists |
| Live Classes | **UI Ready** | Page exists |
| Aamaal for Marhomeen | **UI Ready** | Page exists |

---

## Technical Architecture

### Stack
- **Frontend:** Flutter 3.x with Dart
- **State Management:** Riverpod 3.1 (with code generation)
- **Routing:** GoRouter 17
- **Backend:** Supabase (PostgreSQL)
- **Local Storage:** Hive
- **Architecture:** Clean Architecture (Domain-Driven)

### Module Structure
```
lib/
├── core/              # Shared utilities, theme, DI
├── modules/           # Feature modules
│   ├── auth/          # Authentication (Working)
│   ├── home/          # Home dashboard (Working)
│   ├── prayer_times/  # Prayer times (Working)
│   ├── hijri_calendar/# Hijri calendar (Working)
│   ├── quran/         # Quran reader (UI Ready)
│   ├── duas/          # Duas & Ziyarat (UI Ready)
│   ├── mohr_e_ameen/  # Sajda counter (UI Ready)
│   ├── khums/         # Khums calculator (UI Ready)
│   ├── qaza/          # Qaza tracker (UI Ready)
│   ├── qibla/         # Qibla compass (UI Ready)
│   ├── fiqh/          # Fiqhi Masail (UI Ready)
│   ├── wasiyat/       # Will generator (UI Ready)
│   ├── baby_names/    # Name suggestions (UI Ready)
│   ├── matrimonial/   # Matrimonial (UI Ready)
│   ├── blood_donation/# Blood donors (UI Ready)
│   ├── teachers/      # Teacher directory (UI Ready)
│   ├── live_classes/  # Live classes (UI Ready)
│   ├── shajra/        # Genealogy (UI Ready)
│   ├── aamaal_marhomeen/ # Marhomeen services (UI Ready)
│   ├── settings/      # App settings (Working)
│   ├── theme/         # Theme selection (Working)
│   ├── favorites/     # Favorites (UI Ready)
│   ├── notifications/ # Notifications (UI Ready)
│   ├── localization/  # Language selection (Working)
│   ├── resources/     # Resources hub (Working)
│   └── community/     # Community hub (Working)
├── services/          # Shared services
│   ├── geocoding_service.dart  # Address to coordinates
│   └── python_api_service.dart # Python backend integration
└── router/            # Navigation
```

---

## Recent Changes (2026-01-16)

### Prayer Times Location Enhancement
- **Added:** Geocoding service using Nominatim (OpenStreetMap) API
- **Updated:** `UserLocationController` with location fallback logic
- **Flow:**
  1. Try saved location from cache
  2. Try GPS location (with permission request)
  3. Fall back to user profile location (city/state/country → geocoding)

### Files Modified
- `lib/services/geocoding_service.dart` (NEW)
- `lib/modules/prayer_times/presentation/providers/prayer_times_provider.dart`

---

## Key Integrations

### Working Integrations
| Service | Purpose | Status |
|---------|---------|--------|
| Aladhan API | Prayer times calculation | Working |
| Nominatim API | Geocoding (address → coords) | Working |
| Supabase Auth | User authentication | Working |
| Geolocator | GPS location | Working |
| Hive | Local caching | Working |

### Pending Integrations
| Service | Purpose | Status |
|---------|---------|--------|
| Tanzil Project | Quran text database | Not Started |
| Duas.org | Duas/Ziyarat content | Not Started |
| Supabase Storage | User avatars, documents | Not Started |
| Firebase FCM | Push notifications | Not Started |
| PostGIS | Geospatial queries | Not Started |
| Python FastAPI | Complex calculations | Stub Ready |

---

## Theme System

Using Karbala shrine-inspired design system with 5 themes:
1. **Divine Reflection** - Default (Gold + Silver)
2. **Crimson Tear** - Muharram mode (Red + Black)
3. **Inner Sanctum** - Reading mode (Amber + Cream)
4. **Grounded Earth** - Neutral (Taupe + Cream)
5. **Persian Sky** - Alternative (Slate Blue + Gold)

---

## Known Issues / TODO

### High Priority
- [ ] Prayer times not showing when location unavailable (FIXED)
- [ ] Quran content needs Tanzil database integration
- [ ] Push notifications not implemented

### Medium Priority
- [ ] Reverse geocoding for GPS coordinates (show city name)
- [ ] Offline mode for prayer times
- [ ] Prayer notification scheduling

### Low Priority
- [ ] Monthly prayer timetable view
- [ ] Widget for home screen
- [ ] Dark mode refinement

---

## Commands

```bash
# Build runner (regenerate providers)
dart run build_runner build --delete-conflicting-outputs

# Run tests
flutter test --coverage

# Analyze code
flutter analyze

# Run app
flutter run
```

---

## Git Status (Current)

**Modified:**
- `.claude/agents/` - Multiple agent configs updated
- `CLAUDE.md` - Project instructions
- `lib/modules/prayer_times/presentation/providers/` - Location fallback

**Untracked:**
- `lib/services/geocoding_service.dart` - New geocoding service
