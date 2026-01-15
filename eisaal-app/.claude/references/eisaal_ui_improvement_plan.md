# Eisaal Foundation - UI Improvement Plan

## Executive Summary

This document provides actionable improvements for the Eisaal Foundation app, focusing on the SliverAppBar aesthetics and overall UI enhancements to achieve a cohesive "Digital Sanctuary" experience.

---

## Part 1: SliverAppBar Improvements

### Current Problem

When the SliverAppBar collapses, it becomes a plain white/scaffold-colored bar with just a title. This breaks the premium "Sanctuary" aesthetic and feels generic.

### Solution: Glassmorphic Persistent Header

Replace the default collapsed state with a premium glassmorphic header that maintains visual continuity.

---

### 1.1 Create a Custom Sliver Persistent Header Delegate

**File:** `lib/core/presentation/widgets/sanctuary_sliver_header.dart`

```dart
import 'dart:ui';
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

/// A premium glassmorphic sliver header that transitions smoothly
/// from expanded to collapsed states while maintaining the Sanctuary aesthetic.
class SanctuarySliverHeader extends SliverPersistentHeaderDelegate {
  SanctuarySliverHeader({
    required this.expandedHeight,
    required this.collapsedHeight,
    required this.expandedContent,
    this.collapsedTitle,
    this.leading,
    this.actions,
    this.gradientColors,
    this.showBorderOnCollapse = true,
  });

  final double expandedHeight;
  final double collapsedHeight;
  final Widget expandedContent;
  final String? collapsedTitle;
  final Widget? leading;
  final List<Widget>? actions;
  final List<Color>? gradientColors;
  final bool showBorderOnCollapse;

  @override
  double get maxExtent => expandedHeight;

  @override
  double get minExtent => collapsedHeight;

  @override
  bool shouldRebuild(covariant SliverPersistentHeaderDelegate oldDelegate) => true;

  @override
  Widget build(BuildContext context, double shrinkOffset, bool overlapsContent) {
    final theme = Theme.of(context);
    final isDark = theme.brightness == Brightness.dark;
    
    // Calculate collapse progress (0.0 = expanded, 1.0 = collapsed)
    final collapseProgress = (shrinkOffset / (maxExtent - minExtent)).clamp(0.0, 1.0);
    
    // Derive colors
    final primaryColor = theme.colorScheme.primary;
    final surfaceColor = theme.colorScheme.surface;
    
    // Glass effect intensifies as we collapse
    final glassOpacity = isDark 
        ? 0.7 + (collapseProgress * 0.2) 
        : 0.85 + (collapseProgress * 0.1);
    
    final blurSigma = 10.0 + (collapseProgress * 14.0); // 10 -> 24
    
    // Border appears as we collapse
    final borderOpacity = collapseProgress * (isDark ? 0.15 : 0.2);
    
    // Shadow appears when collapsed
    final shadowOpacity = collapseProgress * (isDark ? 0.3 : 0.08);

    return AnnotatedRegion<SystemUiOverlayStyle>(
      value: isDark 
          ? SystemUiOverlayStyle.light 
          : SystemUiOverlayStyle.dark,
      child: ClipRect(
        child: Stack(
          fit: StackFit.expand,
          children: [
            // Background gradient (fades out as we collapse)
            Opacity(
              opacity: 1.0 - collapseProgress,
              child: Container(
                decoration: BoxDecoration(
                  gradient: LinearGradient(
                    begin: Alignment.topCenter,
                    end: Alignment.bottomCenter,
                    colors: gradientColors ?? [
                      primaryColor.withOpacity(isDark ? 0.15 : 0.08),
                      surfaceColor,
                    ],
                  ),
                ),
              ),
            ),
            
            // Glassmorphic overlay (fades in as we collapse)
            if (collapseProgress > 0)
              Positioned.fill(
                child: ClipRect(
                  child: BackdropFilter(
                    filter: ImageFilter.blur(
                      sigmaX: blurSigma * collapseProgress,
                      sigmaY: blurSigma * collapseProgress,
                    ),
                    child: Container(
                      decoration: BoxDecoration(
                        color: (isDark ? Colors.black : Colors.white)
                            .withOpacity(glassOpacity * collapseProgress),
                        border: showBorderOnCollapse
                            ? Border(
                                bottom: BorderSide(
                                  color: primaryColor.withOpacity(borderOpacity),
                                  width: 1.0,
                                ),
                              )
                            : null,
                        boxShadow: [
                          BoxShadow(
                            color: Colors.black.withOpacity(shadowOpacity),
                            blurRadius: 8,
                            offset: const Offset(0, 2),
                          ),
                        ],
                      ),
                    ),
                  ),
                ),
              ),

            // Expanded content (fades out)
            Positioned.fill(
              child: Opacity(
                opacity: 1.0 - (collapseProgress * 1.5).clamp(0.0, 1.0),
                child: expandedContent,
              ),
            ),

            // Collapsed header content (fades in)
            Positioned(
              left: 0,
              right: 0,
              bottom: 0,
              height: collapsedHeight,
              child: Opacity(
                opacity: collapseProgress,
                child: _buildCollapsedContent(context, theme, primaryColor),
              ),
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildCollapsedContent(
    BuildContext context, 
    ThemeData theme,
    Color primaryColor,
  ) {
    return SafeArea(
      bottom: false,
      child: Padding(
        padding: const EdgeInsets.symmetric(horizontal: 4),
        child: Row(
          children: [
            // Leading widget
            if (leading != null) leading!,
            
            // Title with gradient effect
            Expanded(
              child: collapsedTitle != null
                  ? ShaderMask(
                      shaderCallback: (bounds) => LinearGradient(
                        colors: [
                          primaryColor,
                          primaryColor.withOpacity(0.7),
                        ],
                      ).createShader(bounds),
                      child: Text(
                        collapsedTitle!,
                        style: theme.textTheme.titleLarge?.copyWith(
                          fontWeight: FontWeight.w600,
                          color: Colors.white,
                        ),
                        textAlign: TextAlign.center,
                      ),
                    )
                  : const SizedBox.shrink(),
            ),
            
            // Actions
            if (actions != null) ...actions!,
          ],
        ),
      ),
    );
  }
}
```

---

### 1.2 Create a Simplified Wrapper Widget

**File:** `lib/core/presentation/widgets/sanctuary_app_bar.dart`

```dart
import 'package:flutter/material.dart';
import 'sanctuary_sliver_header.dart';

/// Easy-to-use wrapper for the Sanctuary-styled app bar.
/// 
/// Usage:
/// ```dart
/// CustomScrollView(
///   slivers: [
///     SanctuaryAppBar(
///       expandedHeight: 200,
///       title: 'Today',
///       expandedContent: MyExpandedHeader(),
///       leading: MyAvatarWidget(),
///       actions: [NotificationButton()],
///     ),
///     // ... other slivers
///   ],
/// )
/// ```
class SanctuaryAppBar extends StatelessWidget {
  const SanctuaryAppBar({
    super.key,
    required this.expandedHeight,
    required this.expandedContent,
    this.title,
    this.leading,
    this.actions,
    this.collapsedHeight,
    this.gradientColors,
    this.floating = false,
    this.pinned = true,
  });

  final double expandedHeight;
  final Widget expandedContent;
  final String? title;
  final Widget? leading;
  final List<Widget>? actions;
  final double? collapsedHeight;
  final List<Color>? gradientColors;
  final bool floating;
  final bool pinned;

  @override
  Widget build(BuildContext context) {
    final topPadding = MediaQuery.of(context).padding.top;
    final effectiveCollapsedHeight = collapsedHeight ?? (kToolbarHeight + topPadding);

    return SliverPersistentHeader(
      pinned: pinned,
      floating: floating,
      delegate: SanctuarySliverHeader(
        expandedHeight: expandedHeight + topPadding,
        collapsedHeight: effectiveCollapsedHeight,
        expandedContent: expandedContent,
        collapsedTitle: title,
        leading: leading,
        actions: actions,
        gradientColors: gradientColors,
      ),
    );
  }
}
```

---

### 1.3 Alternative: Enhanced FlexibleSpaceBar Approach

If you prefer to stick with `SliverAppBar`, enhance it with a custom `FlexibleSpaceBar`:

**File:** `lib/core/presentation/widgets/sanctuary_flexible_space.dart`

```dart
import 'dart:ui';
import 'package:flutter/material.dart';

/// A FlexibleSpaceBar replacement that provides glassmorphic collapse effect.
class SanctuaryFlexibleSpace extends StatelessWidget {
  const SanctuaryFlexibleSpace({
    super.key,
    required this.title,
    this.background,
    this.collapseMode = CollapseMode.parallax,
  });

  final String title;
  final Widget? background;
  final CollapseMode collapseMode;

  @override
  Widget build(BuildContext context) {
    return LayoutBuilder(
      builder: (context, constraints) {
        final theme = Theme.of(context);
        final isDark = theme.brightness == Brightness.dark;
        final settings = context.dependOnInheritedWidgetOfExactType<FlexibleSpaceBarSettings>();
        
        if (settings == null) return const SizedBox.shrink();
        
        final deltaExtent = settings.maxExtent - settings.minExtent;
        final t = (1.0 - (settings.currentExtent - settings.minExtent) / deltaExtent)
            .clamp(0.0, 1.0);
        
        // Glass effect
        final blurSigma = t * 20.0;
        final glassOpacity = t * (isDark ? 0.8 : 0.9);
        
        return Stack(
          fit: StackFit.expand,
          children: [
            // Background
            if (background != null)
              Positioned.fill(
                child: Opacity(
                  opacity: 1.0 - t,
                  child: background,
                ),
              ),
            
            // Glass overlay when collapsing
            if (t > 0)
              Positioned.fill(
                child: ClipRect(
                  child: BackdropFilter(
                    filter: ImageFilter.blur(sigmaX: blurSigma, sigmaY: blurSigma),
                    child: Container(
                      decoration: BoxDecoration(
                        gradient: LinearGradient(
                          begin: Alignment.topCenter,
                          end: Alignment.bottomCenter,
                          colors: [
                            (isDark ? Colors.black : Colors.white)
                                .withOpacity(glassOpacity * 0.9),
                            (isDark ? Colors.black : Colors.white)
                                .withOpacity(glassOpacity),
                          ],
                        ),
                        border: Border(
                          bottom: BorderSide(
                            color: theme.colorScheme.primary.withOpacity(t * 0.2),
                            width: 1,
                          ),
                        ),
                      ),
                    ),
                  ),
                ),
              ),
            
            // Title with gradient (appears on collapse)
            Positioned(
              left: 16,
              right: 16,
              bottom: 16,
              child: Opacity(
                opacity: t,
                child: ShaderMask(
                  shaderCallback: (bounds) => LinearGradient(
                    colors: [
                      theme.colorScheme.primary,
                      theme.colorScheme.primary.withOpacity(0.7),
                    ],
                  ).createShader(bounds),
                  child: Text(
                    title,
                    style: theme.textTheme.titleLarge?.copyWith(
                      fontWeight: FontWeight.w600,
                      color: Colors.white,
                    ),
                  ),
                ),
              ),
            ),
          ],
        );
      },
    );
  }
}
```

---

### 1.4 Implementation in Home Page

Update `lib/modules/home/presentation/pages/home_page.dart`:

```dart
// Replace current SliverAppBar with:
SanctuaryAppBar(
  expandedHeight: 200,
  title: l10n.today,
  gradientColors: [
    theme.colorScheme.primary.withOpacity(isDark ? 0.15 : 0.08),
    theme.scaffoldBackgroundColor,
  ],
  leading: Padding(
    padding: const EdgeInsets.only(left: 16),
    child: _buildAvatarWidget(context, user),
  ),
  actions: [
    _buildActionButton(context, icon: Icons.favorite_outline, onTap: () => {}),
    const SizedBox(width: 8),
    _buildNotificationButton(context, notificationCount: 3, onTap: () => {}),
    const SizedBox(width: 16),
  ],
  expandedContent: _buildExpandedHeader(context, theme),
),

// Extracted expanded header widget
Widget _buildExpandedHeader(BuildContext context, ThemeData theme) {
  return SafeArea(
    bottom: false,
    child: Padding(
      padding: const EdgeInsets.fromLTRB(16, 60, 16, 20),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        mainAxisAlignment: MainAxisAlignment.end,
        children: [
          // Hijri Date
          Text(
            '$hijriDay $hijriMonth $hijriYear',
            style: AppFonts.titleStyle(
              fontSize: 28,
              fontWeight: FontWeight.bold,
              color: theme.colorScheme.onSurface,
            ),
          ),
          const SizedBox(height: 4),
          Text(
            gregorianDateStr,
            style: theme.textTheme.bodyMedium?.copyWith(
              color: theme.colorScheme.onSurface.withOpacity(0.6),
            ),
          ),
          // Upcoming event chip...
        ],
      ),
    ),
  );
}
```

---

## Part 2: General UI Improvements

### 2.1 Card System Refinements

#### Problem
Cards across the app have inconsistent styling and don't fully leverage the Sanctuary aesthetic.

#### Solution: Unified Card Component

**File:** `lib/core/presentation/widgets/sanctuary_card.dart`

```dart
import 'dart:ui';
import 'package:flutter/material.dart';

enum SanctuaryCardVariant {
  elevated,    // Standard elevated card
  glass,       // Glassmorphic
  outlined,    // Subtle border only
  gradient,    // Gradient background
}

class SanctuaryCard extends StatelessWidget {
  const SanctuaryCard({
    super.key,
    required this.child,
    this.variant = SanctuaryCardVariant.elevated,
    this.padding,
    this.margin,
    this.borderRadius,
    this.onTap,
    this.gradientColors,
    this.glowColor,
    this.enableGlow = false,
  });

  final Widget child;
  final SanctuaryCardVariant variant;
  final EdgeInsets? padding;
  final EdgeInsets? margin;
  final BorderRadius? borderRadius;
  final VoidCallback? onTap;
  final List<Color>? gradientColors;
  final Color? glowColor;
  final bool enableGlow;

  @override
  Widget build(BuildContext context) {
    final theme = Theme.of(context);
    final isDark = theme.brightness == Brightness.dark;
    final effectiveRadius = borderRadius ?? BorderRadius.circular(20);
    
    Widget card = _buildCardContent(context, theme, isDark, effectiveRadius);
    
    if (margin != null) {
      card = Padding(padding: margin!, child: card);
    }
    
    if (onTap != null) {
      card = GestureDetector(
        onTap: onTap,
        child: card,
      );
    }
    
    return card;
  }

  Widget _buildCardContent(
    BuildContext context, 
    ThemeData theme, 
    bool isDark,
    BorderRadius radius,
  ) {
    final primary = theme.colorScheme.primary;
    
    switch (variant) {
      case SanctuaryCardVariant.elevated:
        return Container(
          padding: padding ?? const EdgeInsets.all(16),
          decoration: BoxDecoration(
            color: theme.colorScheme.surface,
            borderRadius: radius,
            border: Border.all(
              color: theme.colorScheme.outline.withOpacity(0.1),
            ),
            boxShadow: [
              BoxShadow(
                color: Colors.black.withOpacity(isDark ? 0.3 : 0.06),
                blurRadius: 12,
                offset: const Offset(0, 4),
              ),
              if (enableGlow)
                BoxShadow(
                  color: (glowColor ?? primary).withOpacity(0.15),
                  blurRadius: 20,
                  spreadRadius: -4,
                ),
            ],
          ),
          child: child,
        );
        
      case SanctuaryCardVariant.glass:
        return ClipRRect(
          borderRadius: radius,
          child: BackdropFilter(
            filter: ImageFilter.blur(sigmaX: 12, sigmaY: 12),
            child: Container(
              padding: padding ?? const EdgeInsets.all(16),
              decoration: BoxDecoration(
                color: (isDark ? Colors.white : Colors.black).withOpacity(0.05),
                borderRadius: radius,
                border: Border.all(
                  color: Colors.white.withOpacity(isDark ? 0.1 : 0.2),
                  width: 1.5,
                ),
              ),
              child: child,
            ),
          ),
        );
        
      case SanctuaryCardVariant.outlined:
        return Container(
          padding: padding ?? const EdgeInsets.all(16),
          decoration: BoxDecoration(
            borderRadius: radius,
            border: Border.all(
              color: theme.colorScheme.outline.withOpacity(0.2),
            ),
          ),
          child: child,
        );
        
      case SanctuaryCardVariant.gradient:
        final colors = gradientColors ?? [
          primary.withOpacity(0.9),
          primary,
        ];
        return Container(
          padding: padding ?? const EdgeInsets.all(16),
          decoration: BoxDecoration(
            borderRadius: radius,
            gradient: LinearGradient(
              begin: Alignment.topLeft,
              end: Alignment.bottomRight,
              colors: colors,
            ),
            boxShadow: [
              BoxShadow(
                color: colors.first.withOpacity(0.3),
                blurRadius: 16,
                offset: const Offset(0, 6),
              ),
            ],
          ),
          child: child,
        );
    }
  }
}
```

---

### 2.2 Section Headers with Decorative Elements

#### Problem
Section headers are plain text, missing opportunity for visual hierarchy.

#### Solution: Enhanced Section Header

```dart
class SanctuarySectionHeader extends StatelessWidget {
  const SanctuarySectionHeader({
    super.key,
    required this.title,
    this.subtitle,
    this.trailing,
    this.showDecorator = true,
  });

  final String title;
  final String? subtitle;
  final Widget? trailing;
  final bool showDecorator;

  @override
  Widget build(BuildContext context) {
    final theme = Theme.of(context);
    final primary = theme.colorScheme.primary;
    
    return Padding(
      padding: const EdgeInsets.symmetric(vertical: 8),
      child: Row(
        children: [
          // Decorative line/dot
          if (showDecorator) ...[
            Container(
              width: 4,
              height: 24,
              decoration: BoxDecoration(
                borderRadius: BorderRadius.circular(2),
                gradient: LinearGradient(
                  begin: Alignment.topCenter,
                  end: Alignment.bottomCenter,
                  colors: [
                    primary,
                    primary.withOpacity(0.3),
                  ],
                ),
              ),
            ),
            const SizedBox(width: 12),
          ],
          
          // Title & Subtitle
          Expanded(
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Text(
                  title,
                  style: theme.textTheme.titleLarge?.copyWith(
                    fontWeight: FontWeight.bold,
                  ),
                ),
                if (subtitle != null) ...[
                  const SizedBox(height: 2),
                  Text(
                    subtitle!,
                    style: theme.textTheme.bodySmall?.copyWith(
                      color: theme.colorScheme.onSurface.withOpacity(0.6),
                    ),
                  ),
                ],
              ],
            ),
          ),
          
          // Trailing action
          if (trailing != null) trailing!,
        ],
      ),
    );
  }
}
```

---

### 2.3 Animated List Items

#### Problem
List items appear statically, missing the "premium app" feel.

#### Solution: Staggered Entrance Animations

```dart
/// Wrap list items with this for staggered entrance
class AnimatedListItem extends StatelessWidget {
  const AnimatedListItem({
    super.key,
    required this.index,
    required this.child,
    this.duration = const Duration(milliseconds: 400),
    this.delayPerItem = const Duration(milliseconds: 50),
  });

  final int index;
  final Widget child;
  final Duration duration;
  final Duration delayPerItem;

  @override
  Widget build(BuildContext context) {
    return TweenAnimationBuilder<double>(
      tween: Tween(begin: 0.0, end: 1.0),
      duration: duration + (delayPerItem * index),
      curve: Curves.easeOutCubic,
      builder: (context, value, child) {
        return Transform.translate(
          offset: Offset(0, 20 * (1 - value)),
          child: Opacity(
            opacity: value,
            child: child,
          ),
        );
      },
      child: child,
    );
  }
}

// Usage in a ListView:
ListView.builder(
  itemCount: items.length,
  itemBuilder: (context, index) => AnimatedListItem(
    index: index,
    child: MyListTile(item: items[index]),
  ),
)
```

---

### 2.4 Empty States & Loading States

#### Problem
Empty and loading states are basic/missing.

#### Solution: Premium State Widgets

```dart
class SanctuaryEmptyState extends StatelessWidget {
  const SanctuaryEmptyState({
    super.key,
    required this.icon,
    required this.title,
    this.subtitle,
    this.action,
  });

  final IconData icon;
  final String title;
  final String? subtitle;
  final Widget? action;

  @override
  Widget build(BuildContext context) {
    final theme = Theme.of(context);
    final primary = theme.colorScheme.primary;
    
    return Center(
      child: Padding(
        padding: const EdgeInsets.all(32),
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            // Animated icon container
            TweenAnimationBuilder<double>(
              tween: Tween(begin: 0.8, end: 1.0),
              duration: const Duration(milliseconds: 600),
              curve: Curves.easeOutBack,
              builder: (context, value, child) {
                return Transform.scale(
                  scale: value,
                  child: Container(
                    width: 100,
                    height: 100,
                    decoration: BoxDecoration(
                      shape: BoxShape.circle,
                      gradient: LinearGradient(
                        begin: Alignment.topLeft,
                        end: Alignment.bottomRight,
                        colors: [
                          primary.withOpacity(0.15),
                          primary.withOpacity(0.05),
                        ],
                      ),
                      border: Border.all(
                        color: primary.withOpacity(0.2),
                      ),
                    ),
                    child: Icon(
                      icon,
                      size: 40,
                      color: primary.withOpacity(0.6),
                    ),
                  ),
                );
              },
            ),
            const SizedBox(height: 24),
            Text(
              title,
              style: theme.textTheme.titleMedium?.copyWith(
                fontWeight: FontWeight.w600,
              ),
              textAlign: TextAlign.center,
            ),
            if (subtitle != null) ...[
              const SizedBox(height: 8),
              Text(
                subtitle!,
                style: theme.textTheme.bodyMedium?.copyWith(
                  color: theme.colorScheme.onSurface.withOpacity(0.6),
                ),
                textAlign: TextAlign.center,
              ),
            ],
            if (action != null) ...[
              const SizedBox(height: 24),
              action!,
            ],
          ],
        ),
      ),
    );
  }
}
```

---

### 2.5 Shimmer Loading with Theme Colors

```dart
class SanctuaryShimmer extends StatelessWidget {
  const SanctuaryShimmer({
    super.key,
    required this.child,
    this.enabled = true,
  });

  final Widget child;
  final bool enabled;

  @override
  Widget build(BuildContext context) {
    if (!enabled) return child;
    
    final theme = Theme.of(context);
    final isDark = theme.brightness == Brightness.dark;
    final primary = theme.colorScheme.primary;
    
    return Shimmer.fromColors(
      baseColor: isDark 
          ? theme.colorScheme.surface
          : theme.colorScheme.surfaceVariant,
      highlightColor: isDark
          ? theme.colorScheme.surfaceVariant
          : theme.colorScheme.surface,
      // Add subtle gold tint
      child: ShaderMask(
        shaderCallback: (bounds) => LinearGradient(
          colors: [
            Colors.white,
            primary.withOpacity(0.05),
            Colors.white,
          ],
          stops: const [0.0, 0.5, 1.0],
        ).createShader(bounds),
        child: child,
      ),
    );
  }
}
```

---

### 2.6 Button Enhancements

#### Primary Button with Gradient & Glow

```dart
class SanctuaryButton extends StatefulWidget {
  const SanctuaryButton({
    super.key,
    required this.label,
    required this.onPressed,
    this.icon,
    this.variant = SanctuaryButtonVariant.primary,
    this.isLoading = false,
    this.isExpanded = false,
  });

  final String label;
  final VoidCallback? onPressed;
  final IconData? icon;
  final SanctuaryButtonVariant variant;
  final bool isLoading;
  final bool isExpanded;

  @override
  State<SanctuaryButton> createState() => _SanctuaryButtonState();
}

enum SanctuaryButtonVariant { primary, secondary, ghost }

class _SanctuaryButtonState extends State<SanctuaryButton> 
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  late Animation<double> _scaleAnimation;
  bool _isPressed = false;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(milliseconds: 100),
    );
    _scaleAnimation = Tween<double>(begin: 1.0, end: 0.96).animate(
      CurvedAnimation(parent: _controller, curve: Curves.easeInOut),
    );
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    final theme = Theme.of(context);
    final primary = theme.colorScheme.primary;
    final isDark = theme.brightness == Brightness.dark;
    
    return GestureDetector(
      onTapDown: (_) {
        if (widget.onPressed != null) {
          setState(() => _isPressed = true);
          _controller.forward();
          HapticFeedback.lightImpact();
        }
      },
      onTapUp: (_) {
        setState(() => _isPressed = false);
        _controller.reverse();
      },
      onTapCancel: () {
        setState(() => _isPressed = false);
        _controller.reverse();
      },
      onTap: widget.isLoading ? null : widget.onPressed,
      child: AnimatedBuilder(
        animation: _scaleAnimation,
        builder: (context, child) {
          return Transform.scale(
            scale: _scaleAnimation.value,
            child: _buildButton(theme, primary, isDark),
          );
        },
      ),
    );
  }

  Widget _buildButton(ThemeData theme, Color primary, bool isDark) {
    final content = Row(
      mainAxisSize: widget.isExpanded ? MainAxisSize.max : MainAxisSize.min,
      mainAxisAlignment: MainAxisAlignment.center,
      children: [
        if (widget.isLoading)
          SizedBox(
            width: 20,
            height: 20,
            child: CircularProgressIndicator(
              strokeWidth: 2,
              color: widget.variant == SanctuaryButtonVariant.primary
                  ? Colors.white
                  : primary,
            ),
          )
        else ...[
          if (widget.icon != null) ...[
            Icon(widget.icon, size: 20),
            const SizedBox(width: 8),
          ],
          Text(
            widget.label,
            style: theme.textTheme.labelLarge?.copyWith(
              fontWeight: FontWeight.w600,
            ),
          ),
        ],
      ],
    );
    
    switch (widget.variant) {
      case SanctuaryButtonVariant.primary:
        return Container(
          padding: const EdgeInsets.symmetric(horizontal: 24, vertical: 14),
          decoration: BoxDecoration(
            borderRadius: BorderRadius.circular(14),
            gradient: LinearGradient(
              begin: Alignment.topLeft,
              end: Alignment.bottomRight,
              colors: [
                primary,
                primary.withOpacity(0.85),
              ],
            ),
            boxShadow: [
              BoxShadow(
                color: primary.withOpacity(_isPressed ? 0.2 : 0.4),
                blurRadius: _isPressed ? 8 : 16,
                offset: Offset(0, _isPressed ? 2 : 6),
              ),
            ],
          ),
          child: DefaultTextStyle(
            style: const TextStyle(color: Colors.white),
            child: IconTheme(
              data: const IconThemeData(color: Colors.white),
              child: content,
            ),
          ),
        );
        
      case SanctuaryButtonVariant.secondary:
        return Container(
          padding: const EdgeInsets.symmetric(horizontal: 24, vertical: 14),
          decoration: BoxDecoration(
            borderRadius: BorderRadius.circular(14),
            border: Border.all(color: primary.withOpacity(0.5), width: 1.5),
            color: primary.withOpacity(0.05),
          ),
          child: DefaultTextStyle(
            style: TextStyle(color: primary),
            child: IconTheme(
              data: IconThemeData(color: primary),
              child: content,
            ),
          ),
        );
        
      case SanctuaryButtonVariant.ghost:
        return Container(
          padding: const EdgeInsets.symmetric(horizontal: 24, vertical: 14),
          child: DefaultTextStyle(
            style: TextStyle(color: primary),
            child: IconTheme(
              data: IconThemeData(color: primary),
              child: content,
            ),
          ),
        );
    }
  }
}
```

---

### 2.7 Tab Bar Enhancement

Replace standard TabBar with a more premium version:

```dart
class SanctuaryTabBar extends StatelessWidget {
  const SanctuaryTabBar({
    super.key,
    required this.tabs,
    required this.selectedIndex,
    required this.onTap,
  });

  final List<String> tabs;
  final int selectedIndex;
  final ValueChanged<int> onTap;

  @override
  Widget build(BuildContext context) {
    final theme = Theme.of(context);
    final primary = theme.colorScheme.primary;
    final isDark = theme.brightness == Brightness.dark;
    
    return Container(
      height: 48,
      margin: const EdgeInsets.symmetric(horizontal: 16),
      decoration: BoxDecoration(
        color: theme.colorScheme.surfaceVariant.withOpacity(0.5),
        borderRadius: BorderRadius.circular(12),
      ),
      child: Row(
        children: List.generate(tabs.length, (index) {
          final isSelected = index == selectedIndex;
          return Expanded(
            child: GestureDetector(
              onTap: () => onTap(index),
              child: AnimatedContainer(
                duration: const Duration(milliseconds: 200),
                margin: const EdgeInsets.all(4),
                decoration: BoxDecoration(
                  borderRadius: BorderRadius.circular(10),
                  color: isSelected 
                      ? theme.colorScheme.surface 
                      : Colors.transparent,
                  boxShadow: isSelected
                      ? [
                          BoxShadow(
                            color: Colors.black.withOpacity(isDark ? 0.3 : 0.08),
                            blurRadius: 8,
                            offset: const Offset(0, 2),
                          ),
                        ]
                      : null,
                ),
                child: Center(
                  child: Text(
                    tabs[index],
                    style: theme.textTheme.labelLarge?.copyWith(
                      color: isSelected 
                          ? primary 
                          : theme.colorScheme.onSurface.withOpacity(0.6),
                      fontWeight: isSelected ? FontWeight.w600 : FontWeight.w500,
                    ),
                  ),
                ),
              ),
            ),
          );
        }),
      ),
    );
  }
}
```

---

### 2.8 Input Fields Enhancement

```dart
class SanctuaryTextField extends StatefulWidget {
  const SanctuaryTextField({
    super.key,
    this.controller,
    this.label,
    this.hint,
    this.prefixIcon,
    this.suffixIcon,
    this.obscureText = false,
    this.keyboardType,
    this.validator,
    this.onChanged,
  });

  final TextEditingController? controller;
  final String? label;
  final String? hint;
  final IconData? prefixIcon;
  final Widget? suffixIcon;
  final bool obscureText;
  final TextInputType? keyboardType;
  final String? Function(String?)? validator;
  final ValueChanged<String>? onChanged;

  @override
  State<SanctuaryTextField> createState() => _SanctuaryTextFieldState();
}

class _SanctuaryTextFieldState extends State<SanctuaryTextField> {
  bool _isFocused = false;
  final _focusNode = FocusNode();

  @override
  void initState() {
    super.initState();
    _focusNode.addListener(() {
      setState(() => _isFocused = _focusNode.hasFocus);
    });
  }

  @override
  void dispose() {
    _focusNode.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    final theme = Theme.of(context);
    final primary = theme.colorScheme.primary;
    final isDark = theme.brightness == Brightness.dark;

    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        if (widget.label != null) ...[
          Text(
            widget.label!,
            style: theme.textTheme.labelMedium?.copyWith(
              fontWeight: FontWeight.w600,
              color: _isFocused 
                  ? primary 
                  : theme.colorScheme.onSurface.withOpacity(0.7),
            ),
          ),
          const SizedBox(height: 8),
        ],
        AnimatedContainer(
          duration: const Duration(milliseconds: 200),
          decoration: BoxDecoration(
            borderRadius: BorderRadius.circular(14),
            border: Border.all(
              color: _isFocused 
                  ? primary 
                  : theme.colorScheme.outline.withOpacity(0.2),
              width: _isFocused ? 2 : 1,
            ),
            color: isDark
                ? theme.colorScheme.surface
                : theme.colorScheme.surfaceVariant.withOpacity(0.3),
            boxShadow: _isFocused
                ? [
                    BoxShadow(
                      color: primary.withOpacity(0.1),
                      blurRadius: 8,
                      spreadRadius: -2,
                    ),
                  ]
                : null,
          ),
          child: TextFormField(
            controller: widget.controller,
            focusNode: _focusNode,
            obscureText: widget.obscureText,
            keyboardType: widget.keyboardType,
            validator: widget.validator,
            onChanged: widget.onChanged,
            style: theme.textTheme.bodyLarge,
            decoration: InputDecoration(
              hintText: widget.hint,
              hintStyle: theme.textTheme.bodyLarge?.copyWith(
                color: theme.colorScheme.onSurface.withOpacity(0.4),
              ),
              prefixIcon: widget.prefixIcon != null
                  ? Icon(
                      widget.prefixIcon,
                      color: _isFocused 
                          ? primary 
                          : theme.colorScheme.onSurface.withOpacity(0.5),
                    )
                  : null,
              suffixIcon: widget.suffixIcon,
              border: InputBorder.none,
              contentPadding: const EdgeInsets.symmetric(
                horizontal: 16,
                vertical: 14,
              ),
            ),
          ),
        ),
      ],
    );
  }
}
```

---

## Part 3: Implementation Checklist

### Phase 1: Critical (Week 1)

- [ ] Implement `SanctuarySliverHeader` for glassmorphic app bars
- [ ] Update Home Page to use new header
- [ ] Update Community Page to use new header  
- [ ] Update Resources Page to use new header

### Phase 2: High Priority (Week 2)

- [ ] Create `SanctuaryCard` component
- [ ] Replace all card usages across app
- [ ] Implement `SanctuarySectionHeader`
- [ ] Add entrance animations to list views

### Phase 3: Enhancement (Week 3)

- [ ] Create `SanctuaryButton` component
- [ ] Implement `SanctuaryTextField`
- [ ] Add `SanctuaryEmptyState` widgets
- [ ] Add `SanctuaryShimmer` loading states

### Phase 4: Polish (Week 4)

- [ ] Fine-tune animation timings
- [ ] Test across all themes (especially Muharram themes)
- [ ] Ensure dark mode consistency
- [ ] Performance optimization for animations

---

## Part 4: Design Tokens Reference

### Spacing
```dart
const double spacingXs = 4;
const double spacingSm = 8;
const double spacingMd = 16;
const double spacingLg = 24;
const double spacingXl = 32;
```

### Border Radius
```dart
const double radiusSm = 8;
const double radiusMd = 12;
const double radiusLg = 16;
const double radiusXl = 20;
const double radiusFull = 100;
```

### Animation Durations
```dart
const Duration durationFast = Duration(milliseconds: 150);
const Duration durationNormal = Duration(milliseconds: 250);
const Duration durationSlow = Duration(milliseconds: 400);
```

### Shadow Elevation Levels
```dart
// Level 1 - Subtle (cards at rest)
BoxShadow(color: Colors.black.withOpacity(0.04), blurRadius: 8, offset: Offset(0, 2))

// Level 2 - Medium (cards on hover/focus)
BoxShadow(color: Colors.black.withOpacity(0.08), blurRadius: 16, offset: Offset(0, 6))

// Level 3 - Elevated (floating elements)
BoxShadow(color: Colors.black.withOpacity(0.12), blurRadius: 24, offset: Offset(0, 10))
```

---

## Summary

This improvement plan addresses:

1. **SliverAppBar** - Transform from plain white to glassmorphic with smooth transitions
2. **Cards** - Unified component with multiple variants (elevated, glass, gradient)
3. **Animations** - Staggered entrances, press feedback, smooth transitions
4. **States** - Premium empty states, themed shimmer loading
5. **Inputs** - Enhanced text fields with focus animations
6. **Buttons** - Gradient primary buttons with glow effects
7. **Consistency** - Design tokens for spacing, radius, timing

The key principle: **Every interaction should feel intentional and premium**, matching the sacred "Digital Sanctuary" vision.
