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

```dart
// lib/router/app_router.dart
import 'package:go_router/go_router.dart';
import 'package:flutter/material.dart';

final GoRouter appRouter = GoRouter(
  initialLocation: '/',
  debugLogDiagnostics: true,

  // Error page
  errorBuilder: (context, state) => Scaffold(
    appBar: AppBar(title: const Text('خطأ')),
    body: Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          const Icon(Icons.error, size: 64, color: Colors.red),
          const SizedBox(height: 16),
          Text('الصفحة غير موجودة:\n${state.uri}'),
          const SizedBox(height: 16),
          ElevatedButton(
            onPressed: () => context.go('/'),
            child: const Text('الرجوع للرئيسية'),
          ),
        ],
      ),
    ),
  ),

  routes: [
    GoRoute(
      path: '/',
      name: 'home',
      builder: (context, state) => const HomeScreen(),
    ),
    GoRoute(
      path: '/details',
      name: 'details',
      builder: (context, state) => const DetailsScreen(),
    ),
    GoRoute(
      path: '/profile/:userId',
      name: 'profile',
      builder: (context, state) {
        final userId = state.pathParameters['userId']!;
        return ProfileScreen(userId: userId);
      },
    ),
  ],
);

// lib/main.dart
import 'package:flutter/material.dart';
import 'router/app_router.dart';

void main() => runApp(const MyApp());

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      title: 'My App',
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
عشان المستخدم ميشوفش شاشة خطأ قبيحة لو راح لـ route غلط.

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
