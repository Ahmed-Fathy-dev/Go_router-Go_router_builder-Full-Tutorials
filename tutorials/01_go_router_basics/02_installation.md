# الدرس 2: التثبيت والإعداد

## إضافة المكتبة

### الخطوة 1: أضف الـ dependency

في ملف `pubspec.yaml`:

```yaml
dependencies:
  flutter:
    sdk: flutter
  go_router: ^17.0.1
```

### الخطوة 2: نزل الـ packages

```bash
flutter pub get
```

---

## إعداد الـ Router

### الخطوة 1: أنشئ ملف للـ Router

أنصحك تعمل ملف منفصل للـ router عشان الكود يبقى منظم:

📁 `lib/router/app_router.dart`

```dart
import 'package:go_router/go_router.dart';
import 'package:flutter/material.dart';

// Import your screens
import '../screens/home_screen.dart';
import '../screens/details_screen.dart';

final GoRouter appRouter = GoRouter(
  // First page to open
  initialLocation: '/',

  // Routes list
  routes: [
    GoRoute(
      path: '/',
      builder: (context, state) => const HomeScreen(),
    ),
    GoRoute(
      path: '/details',
      builder: (context, state) => const DetailsScreen(),
    ),
  ],
);
```

### الخطوة 2: استخدم الـ Router في التطبيق

في ملف `main.dart`:

```dart
import 'package:flutter/material.dart';
import 'router/app_router.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      title: 'GoRouter Demo',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.blue),
        useMaterial3: true,
      ),
      // Use the router here
      routerConfig: appRouter,
    );
  }
}
```

> ⚠️ **مهم**: لازم تستخدم `MaterialApp.router()` مش `MaterialApp()` العادي

---

## خصائص GoRouter الأساسية

### 1. initialLocation
الصفحة اللي التطبيق هيفتح عليها أول مرة:

```dart
GoRouter(
  initialLocation: '/',  // Opens on Home
  // Or:
  initialLocation: '/login',  // Opens on Login page
  routes: [...],
)
```

### 2. routes
قائمة الـ routes (الصفحات) المتاحة:

```dart
GoRouter(
  routes: [
    GoRoute(path: '/', builder: ...),
    GoRoute(path: '/profile', builder: ...),
    GoRoute(path: '/settings', builder: ...),
  ],
)
```

### 3. debugLogDiagnostics
لو عايز تشوف log للـ navigation في الـ console:

```dart
GoRouter(
  debugLogDiagnostics: true,  // Very useful in development
  routes: [...],
)
```

الـ output هيبقى زي كده:
```
GoRouter: going to /details
GoRouter: pushing /details
```

### 4. redirect
لو عايز تعمل redirect عام (هنشرحه بالتفصيل بعدين):

```dart
GoRouter(
  redirect: (context, state) {
    // If user is not logged in, redirect to Login page
    if (!isLoggedIn) {
      return '/login';
    }
    return null;  // null means no redirect
  },
  routes: [...],
)
```

### 5. errorBuilder
صفحة الخطأ لما المستخدم يروح لـ route مش موجود:

```dart
GoRouter(
  errorBuilder: (context, state) {
    return Scaffold(
      body: Center(
        child: Text('الصفحة مش موجودة: ${state.uri}'),
      ),
    );
  },
  routes: [...],
)
```

---

## خصائص GoRoute

كل `GoRoute` ليه خصائص مهمة:

```dart
GoRoute(
  // Path of the page (required)
  path: '/profile',

  // Name for the route (optional) - useful for named navigation
  name: 'profile',

  // The builder that returns the Widget
  builder: (BuildContext context, GoRouterState state) {
    return const ProfileScreen();
  },

  // Sub-routes (optional)
  routes: [
    GoRoute(
      path: 'edit',  // Will be /profile/edit
      builder: (context, state) => const EditProfileScreen(),
    ),
  ],

  // Route-specific redirect (optional)
  redirect: (context, state) {
    // ...
  },
)
```

---

## GoRouterState

ده الـ object اللي بتاخده في الـ builder وفيه معلومات عن الـ route:

```dart
GoRoute(
  path: '/user/:id',
  builder: (context, GoRouterState state) {
    // Full URI
    print(state.uri);  // /user/123?tab=posts

    // Path parameters
    print(state.pathParameters);  // {id: 123}

    // Query parameters
    print(state.uri.queryParameters);  // {tab: posts}

    // Extra data (if passed)
    print(state.extra);

    // Full path pattern
    print(state.fullPath);  // /user/:id

    // Route name (if specified)
    print(state.name);

    return UserScreen(id: state.pathParameters['id']!);
  },
)
```

---

## مثال كامل

### هيكلة الملفات

```
lib/
├── main.dart
├── core/
│   └── router/
│       ├── app_router.dart
│       └── routes.dart
├── features/
│   ├── home/
│   │   └── presentation/
│   │       └── home_screen.dart
│   ├── profile/
│   │   └── presentation/
│   │       └── profile_screen.dart
│   └── error/
│       └── presentation/
│           └── error_screen.dart
```

### ملف الـ Router

```dart
// lib/core/router/app_router.dart
import 'package:flutter/foundation.dart';
import 'package:flutter/material.dart';
import 'package:go_router/go_router.dart';

import 'routes.dart';
import '../../features/error/presentation/error_screen.dart';

/// Main app router configuration
final GoRouter appRouter = GoRouter(
  initialLocation: Routes.home,
  debugLogDiagnostics: kDebugMode,
  errorBuilder: (context, state) => ErrorScreen(
    error: state.error,
    uri: state.uri,
  ),
  routes: $appRoutes,
);
```

### ملف الـ Routes

```dart
// lib/core/router/routes.dart
import 'package:go_router/go_router.dart';

import '../../features/home/presentation/home_screen.dart';
import '../../features/profile/presentation/profile_screen.dart';

/// Route paths as constants
abstract final class Routes {
  static const home = '/';
  static const profile = '/profile/:userId';

  // Helper to build profile path
  static String profilePath(String userId) => '/profile/$userId';
}

/// All app routes
final List<RouteBase> $appRoutes = [
  GoRoute(
    path: Routes.home,
    name: 'home',
    builder: (context, state) => const HomeScreen(),
  ),
  GoRoute(
    path: Routes.profile,
    name: 'profile',
    builder: (context, state) {
      final userId = state.pathParameters['userId']!;
      return ProfileScreen(userId: userId);
    },
  ),
];
```

### صفحة الخطأ

```dart
// lib/features/error/presentation/error_screen.dart
import 'package:flutter/material.dart';
import 'package:go_router/go_router.dart';

class ErrorScreen extends StatelessWidget {
  const ErrorScreen({
    super.key,
    this.error,
    required this.uri,
  });

  final Exception? error;
  final Uri uri;

  @override
  Widget build(BuildContext context) {
    final theme = Theme.of(context);

    return Scaffold(
      body: SafeArea(
        child: Center(
          child: Padding(
            padding: const EdgeInsets.all(24),
            child: Column(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                Icon(
                  Icons.error_outline,
                  size: 80,
                  color: theme.colorScheme.error,
                ),
                const SizedBox(height: 24),
                Text(
                  'الصفحة غير موجودة',
                  style: theme.textTheme.headlineSmall,
                ),
                const SizedBox(height: 8),
                Text(
                  uri.toString(),
                  style: theme.textTheme.bodyMedium?.copyWith(
                    color: theme.colorScheme.onSurfaceVariant,
                  ),
                  textAlign: TextAlign.center,
                ),
                const SizedBox(height: 32),
                FilledButton.icon(
                  onPressed: () => context.go('/'),
                  icon: const Icon(Icons.home),
                  label: const Text('الرئيسية'),
                ),
              ],
            ),
          ),
        ),
      ),
    );
  }
}
```

### main.dart

```dart
// lib/main.dart
import 'package:flutter/material.dart';

import 'core/router/app_router.dart';

void main() => runApp(const MyApp());

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      title: 'My App',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.blue),
        useMaterial3: true,
      ),
      routerConfig: appRouter,
    );
  }
}
```

---

## نصائح مهمة

### 1. استخدم `debugLogDiagnostics` في الـ Development
```dart
GoRouter(
  debugLogDiagnostics: kDebugMode,  // Only works in debug mode
  routes: [...],
)
```

### 2. دايماً عرّف `errorBuilder`
عشان المستخدم ميشوفش شاشة خطأ عشوائية لو راح لـ route غلط.

### 3. استخدم `initialLocation` بحكمة
```dart
// Can be set dynamically
GoRouter(
  initialLocation: isLoggedIn ? '/' : '/login',
  routes: [...],
)
```

### 4. الـ path لازم يبدأ بـ `/`
```dart
// Correct ✅
GoRoute(path: '/home', ...)

// Wrong ❌
GoRoute(path: 'home', ...)
```

> استثناء: الـ sub-routes مش لازم تبدأ بـ `/` (هنشرح ده بعدين)

---

## ملخص

| الخطوة | الوصف |
|--------|-------|
| 1 | أضف `go_router` في `pubspec.yaml` |
| 2 | اعمل `flutter pub get` |
| 3 | أنشئ `GoRouter` instance |
| 4 | استخدم `MaterialApp.router()` |
| 5 | حط `routerConfig: appRouter` |

---

[⬅️ الدرس السابق: المقدمة](01_introduction.md) | [➡️ الدرس التالي: التوجيه الأساسي](03_basic_routing.md)
