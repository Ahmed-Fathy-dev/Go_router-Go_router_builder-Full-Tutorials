# الدرس 13: Transition Animations

## الـ Transitions الافتراضية

GoRouter بيستخدم الـ transitions الافتراضية حسب الـ platform:
- **Android**: Material page transition (slide من اليمين)
- **iOS**: Cupertino page transition (slide من اليمين)
- **Web**: No transition

---

## تخصيص الـ Transition باستخدام pageBuilder

بدل `builder`، استخدم `pageBuilder` للتحكم الكامل:

```dart
GoRoute(
  path: '/details',
  pageBuilder: (context, state) {
    return CustomTransitionPage(
      key: state.pageKey,
      child: const DetailsScreen(),
      transitionsBuilder: (context, animation, secondaryAnimation, child) {
        return FadeTransition(
          opacity: animation,
          child: child,
        );
      },
    );
  },
)
```

---

## أنواع الـ Transitions الشائعة

### 1. Fade Transition

```dart
GoRoute(
  path: '/page',
  pageBuilder: (context, state) {
    return CustomTransitionPage(
      key: state.pageKey,
      child: const MyScreen(),
      transitionDuration: const Duration(milliseconds: 300),
      transitionsBuilder: (context, animation, secondaryAnimation, child) {
        return FadeTransition(
          opacity: CurveTween(curve: Curves.easeIn).animate(animation),
          child: child,
        );
      },
    );
  },
)
```

### 2. Slide Transition

```dart
GoRoute(
  path: '/page',
  pageBuilder: (context, state) {
    return CustomTransitionPage(
      key: state.pageKey,
      child: const MyScreen(),
      transitionsBuilder: (context, animation, secondaryAnimation, child) {
        // Slide from bottom
        const begin = Offset(0.0, 1.0);
        const end = Offset.zero;
        final tween = Tween(begin: begin, end: end)
            .chain(CurveTween(curve: Curves.easeInOut));

        return SlideTransition(
          position: animation.drive(tween),
          child: child,
        );
      },
    );
  },
)
```

### 3. Scale Transition

```dart
GoRoute(
  path: '/page',
  pageBuilder: (context, state) {
    return CustomTransitionPage(
      key: state.pageKey,
      child: const MyScreen(),
      transitionsBuilder: (context, animation, secondaryAnimation, child) {
        return ScaleTransition(
          scale: Tween<double>(begin: 0.0, end: 1.0).animate(
            CurvedAnimation(parent: animation, curve: Curves.elasticOut),
          ),
          child: child,
        );
      },
    );
  },
)
```

### 4. Rotation Transition

```dart
GoRoute(
  path: '/page',
  pageBuilder: (context, state) {
    return CustomTransitionPage(
      key: state.pageKey,
      child: const MyScreen(),
      transitionsBuilder: (context, animation, secondaryAnimation, child) {
        return RotationTransition(
          turns: Tween<double>(begin: 0.5, end: 1.0).animate(animation),
          child: FadeTransition(
            opacity: animation,
            child: child,
          ),
        );
      },
    );
  },
)
```

### 5. Combined Transitions

```dart
GoRoute(
  path: '/page',
  pageBuilder: (context, state) {
    return CustomTransitionPage(
      key: state.pageKey,
      child: const MyScreen(),
      transitionsBuilder: (context, animation, secondaryAnimation, child) {
        // Fade + Slide combined
        return FadeTransition(
          opacity: animation,
          child: SlideTransition(
            position: Tween<Offset>(
              begin: const Offset(0.0, 0.25),
              end: Offset.zero,
            ).animate(CurvedAnimation(
              parent: animation,
              curve: Curves.easeOut,
            )),
            child: child,
          ),
        );
      },
    );
  },
)
```

---

## NoTransitionPage

لو مش عايز أي transition:

```dart
GoRoute(
  path: '/instant',
  pageBuilder: (context, state) {
    return NoTransitionPage(
      key: state.pageKey,
      child: const MyScreen(),
    );
  },
)
```

---

## إنشاء Custom Transition Page قابلة لإعادة الاستخدام

### 1. Fade Page

```dart
class FadePage<T> extends CustomTransitionPage<T> {
  FadePage({
    required super.child,
    super.key,
    Duration duration = const Duration(milliseconds: 300),
  }) : super(
          transitionDuration: duration,
          transitionsBuilder: (context, animation, secondaryAnimation, child) {
            return FadeTransition(
              opacity: CurveTween(curve: Curves.easeIn).animate(animation),
              child: child,
            );
          },
        );
}

// Usage
GoRoute(
  path: '/page',
  pageBuilder: (context, state) {
    return FadePage(
      key: state.pageKey,
      child: const MyScreen(),
    );
  },
)
```

### 2. Slide Up Page

```dart
class SlideUpPage<T> extends CustomTransitionPage<T> {
  SlideUpPage({
    required super.child,
    super.key,
  }) : super(
          transitionDuration: const Duration(milliseconds: 300),
          transitionsBuilder: (context, animation, secondaryAnimation, child) {
            return SlideTransition(
              position: Tween<Offset>(
                begin: const Offset(0, 1),
                end: Offset.zero,
              ).animate(CurvedAnimation(
                parent: animation,
                curve: Curves.easeOutCubic,
              )),
              child: child,
            );
          },
        );
}
```

### 3. Modal Page (iOS style)

```dart
class ModalPage<T> extends CustomTransitionPage<T> {
  ModalPage({
    required super.child,
    super.key,
  }) : super(
          fullscreenDialog: true,
          transitionDuration: const Duration(milliseconds: 400),
          transitionsBuilder: (context, animation, secondaryAnimation, child) {
            final curvedAnimation = CurvedAnimation(
              parent: animation,
              curve: Curves.easeOut,
              reverseCurve: Curves.easeIn,
            );

            return SlideTransition(
              position: Tween<Offset>(
                begin: const Offset(0, 1),
                end: Offset.zero,
              ).animate(curvedAnimation),
              child: child,
            );
          },
        );
}
```

---

## Transition مختلف حسب الـ Direction

```dart
class DirectionalSlidePage<T> extends CustomTransitionPage<T> {
  DirectionalSlidePage({
    required super.child,
    super.key,
    this.direction = AxisDirection.right,
  }) : super(
          transitionsBuilder: (context, animation, secondaryAnimation, child) {
            Offset begin;
            switch (direction) {
              case AxisDirection.up:
                begin = const Offset(0, 1);
                break;
              case AxisDirection.down:
                begin = const Offset(0, -1);
                break;
              case AxisDirection.left:
                begin = const Offset(1, 0);
                break;
              case AxisDirection.right:
                begin = const Offset(-1, 0);
                break;
            }

            return SlideTransition(
              position: Tween<Offset>(
                begin: begin,
                end: Offset.zero,
              ).animate(CurvedAnimation(
                parent: animation,
                curve: Curves.easeOutCubic,
              )),
              child: child,
            );
          },
        );

  final AxisDirection direction;
}
```

---

## secondaryAnimation

الـ `secondaryAnimation` بيشتغل لما الصفحة الحالية بتخرج عشان صفحة جديدة تظهر:

```dart
transitionsBuilder: (context, animation, secondaryAnimation, child) {
  // animation: incoming page
  // secondaryAnimation: outgoing page

  return SlideTransition(
    // Incoming page slides from right
    position: Tween<Offset>(
      begin: const Offset(1, 0),
      end: Offset.zero,
    ).animate(animation),
    child: SlideTransition(
      // Current page moves slightly to the left
      position: Tween<Offset>(
        begin: Offset.zero,
        end: const Offset(-0.3, 0),
      ).animate(secondaryAnimation),
      child: child,
    ),
  );
}
```

---

## مثال شامل

```dart
// lib/router/page_transitions.dart

class AppPageTransitions {
  // Fade
  static CustomTransitionPage fade<T>({
    required Widget child,
    required LocalKey key,
    Duration duration = const Duration(milliseconds: 300),
  }) {
    return CustomTransitionPage<T>(
      key: key,
      child: child,
      transitionDuration: duration,
      transitionsBuilder: (context, animation, secondaryAnimation, child) {
        return FadeTransition(opacity: animation, child: child);
      },
    );
  }

  // Slide from bottom
  static CustomTransitionPage slideUp<T>({
    required Widget child,
    required LocalKey key,
  }) {
    return CustomTransitionPage<T>(
      key: key,
      child: child,
      transitionsBuilder: (context, animation, secondaryAnimation, child) {
        return SlideTransition(
          position: Tween(
            begin: const Offset(0, 1),
            end: Offset.zero,
          ).animate(CurvedAnimation(
            parent: animation,
            curve: Curves.easeOutCubic,
          )),
          child: child,
        );
      },
    );
  }

  // Scale with fade
  static CustomTransitionPage scaleWithFade<T>({
    required Widget child,
    required LocalKey key,
  }) {
    return CustomTransitionPage<T>(
      key: key,
      child: child,
      transitionsBuilder: (context, animation, secondaryAnimation, child) {
        return FadeTransition(
          opacity: animation,
          child: ScaleTransition(
            scale: Tween(begin: 0.9, end: 1.0).animate(
              CurvedAnimation(parent: animation, curve: Curves.easeOut),
            ),
            child: child,
          ),
        );
      },
    );
  }

  // No transition
  static NoTransitionPage none<T>({
    required Widget child,
    required LocalKey key,
  }) {
    return NoTransitionPage<T>(key: key, child: child);
  }
}

// Usage in the Router
final appRouter = GoRouter(
  routes: [
    GoRoute(
      path: '/',
      pageBuilder: (context, state) => AppPageTransitions.none(
        key: state.pageKey,
        child: const HomeScreen(),
      ),
    ),
    GoRoute(
      path: '/details/:id',
      pageBuilder: (context, state) => AppPageTransitions.slideUp(
        key: state.pageKey,
        child: DetailsScreen(id: state.pathParameters['id']!),
      ),
    ),
    GoRoute(
      path: '/settings',
      pageBuilder: (context, state) => AppPageTransitions.fade(
        key: state.pageKey,
        child: const SettingsScreen(),
      ),
    ),
    GoRoute(
      path: '/modal',
      pageBuilder: (context, state) => AppPageTransitions.scaleWithFade(
        key: state.pageKey,
        child: const ModalScreen(),
      ),
    ),
  ],
);
```

---

## Platform-specific Transitions

```dart
CustomTransitionPage platformPage<T>({
  required Widget child,
  required LocalKey key,
}) {
  if (Platform.isIOS) {
    // iOS: Cupertino style slide transition
    return CustomTransitionPage<T>(
      key: key,
      child: child,
      transitionsBuilder: (context, animation, secondaryAnimation, child) {
        return CupertinoPageTransition(
          primaryRouteAnimation: animation,
          secondaryRouteAnimation: secondaryAnimation,
          linearTransition: false,
          child: child,
        );
      },
    );
  } else {
    // Android/Web: Fade transition
    return CustomTransitionPage<T>(
      key: key,
      child: child,
      transitionsBuilder: (context, animation, secondaryAnimation, child) {
        return FadeTransition(opacity: animation, child: child);
      },
    );
  }
}
```

---

## نصائح

### 1. استخدم state.pageKey

```dart
pageBuilder: (context, state) {
  return CustomTransitionPage(
    key: state.pageKey,  // 👈 Important for animations
    child: MyScreen(),
    ...
  );
}
```

### 2. مدة مناسبة

```dart
// Too short - not noticeable
transitionDuration: const Duration(milliseconds: 100),

// Appropriate duration
transitionDuration: const Duration(milliseconds: 300),

// Too long - feels slow
transitionDuration: const Duration(milliseconds: 600),
```

### 3. Curves مناسبة

```dart
// For entering
Curves.easeOut
Curves.easeOutCubic

// For exiting
Curves.easeIn

// For bouncing effect
Curves.elasticOut
Curves.bounceOut
```

---

## ملخص

| العنصر | الوصف |
|--------|-------|
| **pageBuilder** | للتحكم في الـ transition |
| **CustomTransitionPage** | لتخصيص الـ animation |
| **NoTransitionPage** | بدون transition |
| **transitionsBuilder** | الـ function اللي بتبني الـ animation |
| **animation** | الصفحة الداخلة |
| **secondaryAnimation** | الصفحة الخارجة |

---

[⬅️ الدرس السابق: StatefulShellRoute](12_stateful_shell_route.md) | [➡️ الدرس التالي: Deep Linking](14_deep_linking.md)
