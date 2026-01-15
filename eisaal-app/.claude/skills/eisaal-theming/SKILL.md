---
name: eisaal-theming
description: Complete Eisaal Sanctuary design system including Karbala shrine-inspired color palette, typography, spacing tokens, glassmorphic effects, and animation patterns. Use for any UI work.
---

# Eisaal Sanctuary Design System

## Color Usage Rules

**CRITICAL:** Never hardcode colors. Always use theme:
```dart
final theme = Theme.of(context);
final colors = theme.colorScheme;

// ✅ Correct
Container(color: colors.primary)

// ❌ Wrong
Container(color: Color(0xFFD4AF37))
```

## Color Palette

### Core Theme Colors
```dart
// Access via theme.colorScheme
primary         // Gold (zareeh/dome)
secondary       // Crimson (Muharram chandeliers)
tertiary        // Green (Hussaini)
surface         // Card background
surfaceVariant  // Alternative surface
background      // Screen background
```

### Extended Colors (Direct Import)
```dart
import 'package:eisaal_app/core/theme/sanctuary_colors.dart';

// Golds
SanctuaryColors.zareehGold       // #D4AF37 - Bright shrine gold
SanctuaryColors.antiqueGold      // #C9A227 - Aged patina
SanctuaryColors.champagneGold    // #F7E7CE - Light wash

// Crimsons (Muharram themes)
SanctuaryColors.muharramCrimson  // #8B0000 - Deep blood red
SanctuaryColors.ashuraCrimson    // #C41E3A - Vivid mourning red

// Greens (Hussaini themes)  
SanctuaryColors.hussainiGreen    // #228B22 - Forest green
SanctuaryColors.emeraldGlow      // #50C878 - Bright emerald
```

## Typography
```dart
final text = Theme.of(context).textTheme;

// Headlines
text.displayLarge    // 57px, feature headers
text.displayMedium   // 45px, section headers
text.displaySmall    // 36px, page titles

// Titles
text.titleLarge      // 22px, card titles
text.titleMedium     // 16px, list tiles
text.titleSmall      // 14px, small headers

// Body
text.bodyLarge       // 16px, primary body
text.bodyMedium      // 14px, secondary body
text.bodySmall       // 12px, captions

// Labels
text.labelLarge      // 14px, buttons
text.labelMedium     // 12px, chips
text.labelSmall      // 11px, badges
```

## Spacing System

**Import AppStyles:**
```dart
import 'package:eisaal_app/core/styles/app_styles.dart';

AppStyles.paddingXS   // 4px
AppStyles.paddingSM   // 8px
AppStyles.paddingMD   // 16px
AppStyles.paddingLG   // 24px
AppStyles.paddingXL   // 32px
```

**Never use:**
```dart
// ❌ Wrong
padding: EdgeInsets.all(16)

// ✅ Correct
padding: EdgeInsets.all(AppStyles.paddingMD)
```

## Border Radius
```dart
AppStyles.radiusSM   // 8px
AppStyles.radiusMD   // 12px
AppStyles.radiusLG   // 16px
AppStyles.radiusXL   // 20px
AppStyles.radiusFull // 100px (circular)
```

## Shadow Levels
```dart
// Level 1 - Cards at rest
BoxShadow(
  color: Colors.black.withOpacity(isDark ? 0.3 : 0.04),
  blurRadius: 8,
  offset: const Offset(0, 2),
)

// Level 2 - Interactive elements
BoxShadow(
  color: Colors.black.withOpacity(isDark ? 0.4 : 0.08),
  blurRadius: 16,
  offset: const Offset(0, 6),
)

// Level 3 - Elevated overlays
BoxShadow(
  color: Colors.black.withOpacity(isDark ? 0.5 : 0.12),
  blurRadius: 24,
  offset: const Offset(0, 10),
)
```

## Glassmorphic Effect
```dart
Container(
  decoration: BoxDecoration(
    borderRadius: BorderRadius.circular(AppStyles.radiusLG),
    color: colors.surface.withOpacity(isDark ? 0.7 : 0.85),
    border: Border.all(
      color: colors.primary.withOpacity(0.2),
      width: 1,
    ),
  ),
  child: ClipRRect(
    borderRadius: BorderRadius.circular(AppStyles.radiusLG),
    child: BackdropFilter(
      filter: ImageFilter.blur(sigmaX: 10, sigmaY: 10),
      child: content,
    ),
  ),
)
```

## Animation Durations
```dart
const durationFast = Duration(milliseconds: 150);    // Micro-interactions
const durationNormal = Duration(milliseconds: 250);  // Standard
const durationSlow = Duration(milliseconds: 400);    // Page transitions
```

## Component Naming

All Sanctuary components use prefix:
- `SanctuaryCard`
- `SanctuaryButton`
- `SanctuaryTextField`
- `SanctuaryAppBar`
- etc.

## Dark Mode Considerations

Always check theme brightness:
```dart
final isDark = theme.brightness == Brightness.dark;

final glassOpacity = isDark ? 0.7 : 0.85;
final shadowOpacity = isDark ? 0.3 : 0.04;
```

## Accessibility

- Color contrast ratio: Minimum 4.5:1 for normal text
- Touch targets: Minimum 48x48 dp
- Semantic labels: Add to all interactive elements