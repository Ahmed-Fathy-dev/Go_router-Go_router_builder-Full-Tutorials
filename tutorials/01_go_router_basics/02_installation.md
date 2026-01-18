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

// استورد الـ screens بتاعتك
import '../screens/home_screen.dart';
import '../screens/details_screen.dart';

final GoRouter appRouter = GoRouter(
  // الصفحة الأولى اللي هتفتح
  initialLocation: '/',

  // قائمة الـ routes
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
      // هنا بنستخدم الـ router
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
  initialLocation: '/',  // هيفتح على الـ Home
  // أو
  initialLocation: '/login',  // هيفتح على صفحة Login
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
  debugLogDiagnostics: true,  // مفيد جداً في الـ development
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
    // لو المستخدم مش مسجل دخول، وديه لصفحة Login
    if (!isLoggedIn) {
      return '/login';
    }
    return null;  // null يعني متعملش redirect
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
  // الـ path بتاع الصفحة (إجباري)
  path: '/profile',

  // اسم للـ route (اختياري) - مفيد للـ named navigation
  name: 'profile',

  // الـ builder اللي بيرجع الـ Widget
  builder: (BuildContext context, GoRouterState state) {
    return const ProfileScreen();
  },

  // routes فرعية (اختياري)
  routes: [
    GoRoute(
      path: 'edit',  // هيبقى /profile/edit
      builder: (context, state) => const EditProfileScreen(),
    ),
  ],

  // redirect خاص بالـ route ده (اختياري)
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
    // الـ URI الكامل
    print(state.uri);  // /user/123?tab=posts

    // الـ path parameters
    print(state.pathParameters);  // {id: 123}

    // الـ query parameters
    print(state.uri.queryParameters);  // {tab: posts}

    // الـ extra data (لو مررته)
    print(state.extra);

    // الـ full path pattern
    print(state.fullPath);  // /user/:id

    // اسم الـ route (لو حددته)
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

  // صفحة الخطأ
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
  debugLogDiagnostics: kDebugMode,  // هيشتغل بس في الـ debug mode
  routes: [...],
)
```

### 2. دايماً عرّف `errorBuilder`
عشان المستخدم ميشوفش شاشة خطأ قبيحة لو راح لـ route غلط.

### 3. استخدم `initialLocation` بحكمة
```dart
// ممكن تحددها dynamically
GoRouter(
  initialLocation: isLoggedIn ? '/' : '/login',
  routes: [...],
)
```

### 4. الـ path لازم يبدأ بـ `/`
```dart
// صح ✅
GoRoute(path: '/home', ...)

// غلط ❌
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
