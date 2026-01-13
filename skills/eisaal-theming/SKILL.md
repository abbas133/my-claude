---
name: eisaal-theming
description: Eisaal Foundation app theme system. Use when creating UI components, styling widgets, applying colors, or working with any visual elements. ALWAYS use this skill for UI work in Eisaal app.
---

# Eisaal Theme System

## CRITICAL: Always Use Theme, Never Hardcode

```dart
// ❌ NEVER DO THIS
Color(0xFF6B4226)
Colors.amber
TextStyle(fontSize: 16)

// ✅ ALWAYS DO THIS
Theme.of(context).colorScheme.primary
Theme.of(context).textTheme.bodyLarge
AppStyles.cardPadding
```

## Available Themes

| Theme | Use Case | Primary Color |
|-------|----------|---------------|
| Divine Reflection | Default | Gold/Silver |
| Crimson Tear | Muharram/Safar | Deep Red |
| Inner Sanctum | Reading mode | Amber |
| Grounded Earth | Neutral | Earthy tones |
| Persian Sky | Modern | Blue/Gold |
| Mirror of Paradise | Festive | - |
| Royal Lattice | Premium | - |
| Velvet Sorrow | Mourning | - |
| Verdant Garden | Spring | Green |
| Midnight Dome | Dark mode | Dark blue |

## Color Access

```dart
final colors = Theme.of(context).colorScheme;

colors.primary          // Main brand color
colors.onPrimary        // Text on primary
colors.secondary        // Accent color
colors.surface          // Card backgrounds
colors.onSurface        // Text on surface
colors.error            // Error states
colors.outline          // Borders
```

## Extension Colors (Sanctuary System)

```dart
final sanctuary = Theme.of(context).extension<SanctuaryColors>()!;

sanctuary.gold          // Decorative gold
sanctuary.calligraphy   // Arabic text
sanctuary.glass         // Glassmorphism overlay
sanctuary.shimmer       // Loading shimmer
```

## Typography

```dart
final text = Theme.of(context).textTheme;

text.displayLarge       // Hero text
text.headlineMedium     // Section headers
text.titleLarge         // Card titles
text.bodyLarge          // Main content
text.bodyMedium         // Secondary content
text.labelLarge         // Buttons
text.labelSmall         // Captions
```

## Arabic Text

```dart
// Use ArabicText widget for Quran/Duas
ArabicText(
  text: arabicString,
  style: Theme.of(context).textTheme.headlineMedium,
)

// Or apply Arabic font manually
TextStyle(
  fontFamily: 'Amiri',  // or 'ScheherazadeNew'
  fontSize: 24,
)
```

## Spacing (AppStyles)

```dart
AppStyles.paddingXS     // 4.0
AppStyles.paddingSM     // 8.0
AppStyles.paddingMD     // 16.0
AppStyles.paddingLG     // 24.0
AppStyles.paddingXL     // 32.0

AppStyles.radiusSM      // 8.0
AppStyles.radiusMD      // 12.0
AppStyles.radiusLG      // 16.0
AppStyles.radiusXL      // 24.0
```

## Glassmorphism Card

```dart
GlassCard(
  child: content,
  // Uses sanctuary.glass automatically
)

// Or manual:
Container(
  decoration: BoxDecoration(
    color: sanctuary.glass.withOpacity(0.1),
    borderRadius: BorderRadius.circular(AppStyles.radiusMD),
    border: Border.all(
      color: sanctuary.glass.withOpacity(0.2),
    ),
  ),
)
```

## Standard Card

```dart
Card(
  elevation: 0,
  color: Theme.of(context).colorScheme.surface,
  shape: RoundedRectangleBorder(
    borderRadius: BorderRadius.circular(AppStyles.radiusMD),
  ),
  child: Padding(
    padding: EdgeInsets.all(AppStyles.paddingMD),
    child: content,
  ),
)
```

## Buttons

```dart
// Primary action
ElevatedButton(
  onPressed: onTap,
  child: Text('Action'),
)

// Secondary action
OutlinedButton(
  onPressed: onTap,
  child: Text('Action'),
)

// Text action
TextButton(
  onPressed: onTap,
  child: Text('Action'),
)
```

## Icons

```dart
Icon(
  Icons.mosque,
  color: Theme.of(context).colorScheme.primary,
  size: 24,
)

// Font Awesome icons
FaIcon(
  FontAwesomeIcons.quran,
  color: Theme.of(context).colorScheme.primary,
)
```

## Dark Mode Check

```dart
final isDark = Theme.of(context).brightness == Brightness.dark;
```

## Full Widget Example

```dart
class PrayerTimeCard extends StatelessWidget {
  const PrayerTimeCard({super.key, required this.prayer});
  
  final Prayer prayer;

  @override
  Widget build(BuildContext context) {
    final colors = Theme.of(context).colorScheme;
    final text = Theme.of(context).textTheme;
    final sanctuary = Theme.of(context).extension<SanctuaryColors>()!;

    return Card(
      elevation: 0,
      color: colors.surface,
      shape: RoundedRectangleBorder(
        borderRadius: BorderRadius.circular(AppStyles.radiusMD),
      ),
      child: Padding(
        padding: EdgeInsets.all(AppStyles.paddingMD),
        child: Row(
          children: [
            Icon(Icons.access_time, color: colors.primary),
            SizedBox(width: AppStyles.paddingSM),
            Text(prayer.name, style: text.titleMedium),
            const Spacer(),
            Text(
              prayer.time,
              style: text.bodyLarge?.copyWith(
                color: sanctuary.gold,
                fontWeight: FontWeight.bold,
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```
