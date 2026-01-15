# CLAUDE.md - Eisaal Foundation App

## Project

**Eisaal Foundation** - Islamic Lifestyle Platform (Flutter)
**Phase:** UI Development with Dummy Data
**Stack:** Flutter 3.x, Riverpod 3.1, GoRouter 17, Supabase, Hive

## Structure

```
lib/
├── core/              # Shared: config, DI, errors, styles, widgets
├── modules/           # Feature modules (Clean Architecture)
│   ├── today/         # Home, prayer times, calendar, aamaal
│   ├── resources/     # Quran, duas, knowledge
│   ├── community/     # Matrimonial, blood donation, shajra
│   ├── calculators/   # Khums, qaza namaz/roza, nifas
│   ├── tools/         # Mohr-e-ameen, wasiyat, baby names
│   ├── auth/
│   ├── profile/
│   └── notifications/
├── services/
└── router/
```

## Conventions

- **Files:** `lowercase_with_underscores.dart`
- **Classes:** `UpperCamelCase`
- **Providers:** `featureNameProvider`
- **Always:** `const` constructors, extract widgets
- **Always:** Use `Theme.of(context)` - NEVER hardcode colors
- **Always:** Use `AppStyles` for spacing - NEVER hardcode values

## Commands

```bash
dart run build_runner build --delete-conflicting-outputs
flutter test --coverage
flutter analyze
```

## Theme System

**CRITICAL:** Always use theme system. See `.claude/skills/eisaal-theming/SKILL.md`

## Skills (ALWAYS invoke when applicable)

Skills provide specialized knowledge. Invoke proactively when task matches.

| Skill              | Trigger Phrases                            | Auto-Invoke When                           |
|--------------------|--------------------------------------------|--------------------------------------------|
| `eisaal-theming`   | colors, theme, styling, fonts, spacing     | Creating/editing ANY UI component          |
| `flutter-riverpod` | provider, state, notifier, AsyncValue      | Working with providers or state management |
| `flutter-supabase` | database, auth, storage, realtime, RLS     | Any Supabase-related work                  |
| `flutter-firebase` | FCM, push notifications, analytics         | Any Firebase-related work                  |

## Agents (Spawn for complex/multi-file tasks)

Use Task tool with appropriate `subagent_type` for specialized work.

### UI Work

| Agent                     | When to Use                          | Trigger Examples                                       |
|---------------------------|--------------------------------------|--------------------------------------------------------|
| `ui-creator`              | Building NEW screens, pages, widgets | "create a new page", "add a screen", "build widget"    |
| `ui-fixer`                | Fixing visual bugs, overflow, layout | "overflow error", "doesn't look right", "layout issue" |
| `accessibility-validator` | Checking WCAG compliance             | "accessibility audit", "screen reader", "contrast"     |

### Bug Fixing

| Agent             | When to Use                              | Trigger Examples                                          |
|-------------------|------------------------------------------|-----------------------------------------------------------|
| `bug-hunter`      | Logic errors, runtime errors, async      | "not working", "error", "bug", "crash", "null exception"  |
| `riverpod-expert` | Provider issues, state not updating      | "provider not working", "state not updating", "notifier"  |

### Backend Integration

| Agent               | When to Use                              | Trigger Examples                                  |
|---------------------|------------------------------------------|---------------------------------------------------|
| `supabase-expert`   | Supabase auth, queries, RLS, storage     | "supabase", "database query", "RLS", "realtime"   |
| `firebase-expert`   | Firebase auth, Firestore, FCM, analytics | "firebase", "push notification", "FCM"            |
| `python-api-expert` | Python FastAPI integration, calculations | "python API", "khums calculation", "qaza"         |

### Architecture & Quality

| Agent                    | When to Use                            | Trigger Examples                              |
|--------------------------|----------------------------------------|-----------------------------------------------|
| `module-scaffolder`      | Creating new feature module structure  | "new module", "add feature module", "scaffold"|
| `architecture-validator` | Validating Clean Architecture          | "check architecture", "validate structure"    |
| `code-reviewer`          | Code quality review                    | "review code", "check quality", "code review" |
| `test-generator`         | Writing unit/widget/integration tests  | "write tests", "add coverage", "test this"    |
| `cicd-automator`         | CI/CD pipelines, GitHub Actions        | "setup CI", "automate deployment"             |

### Exploration

| Agent     | When to Use                             | Trigger Examples                                  |
|-----------|-----------------------------------------|---------------------------------------------------|
| `Explore` | Codebase exploration, finding patterns  | "where is", "how does X work", "find all"         |

## Agent Selection Rules

1. **UI creation** → `ui-creator` + invoke `eisaal-theming` skill
2. **UI bug/layout** → `ui-fixer`
3. **Logic bug/error** → `bug-hunter`
4. **State management** → `riverpod-expert` + invoke `flutter-riverpod` skill
5. **Database/auth** → `supabase-expert` or `firebase-expert`
6. **New feature module** → `module-scaffolder`
7. **Before major refactor** → `architecture-validator`
8. **After implementation** → `test-generator`

## Reference Docs

- `Eisaal Foundation - Requirement Landscape.md`
- `Eisaal Foundation - Theme Configurations.md`
