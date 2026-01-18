# Troubleshooting Guide

## مشاكل شائعة وحلولها

---

## 1. "No GoRouter found in context"

### المشكلة
```
Exception: No GoRouter found in context
```

### السبب
بتحاول تستخدم `context.go()` من مكان مفيهوش GoRouter.

### الحل
```dart
// تأكد إنك بتستخدم MaterialApp.router
MaterialApp.router(
  routerConfig: appRouter,  // ✅
)

// مش MaterialApp العادي
MaterialApp(
  home: HomeScreen(),  // ❌
)
```

---

## 2. "No GoRoute found for location"

### المشكلة
```
Exception: no GoRoute found for location: /unknown
```

### السبب
الـ path مش معرف في الـ routes.

### الحل
```dart
// 1. تأكد من صحة الـ path
context.go('/products');  // تأكد إنه معرف

// 2. أضف errorBuilder
GoRouter(
  errorBuilder: (context, state) => NotFoundScreen(),
  routes: [...],
)

// 3. أو استخدم onException
GoRouter(
  onException: (context, state, router) {
    router.go('/');
  },
  routes: [...],
)
```

---

## 3. الـ redirect بيعمل loop

### المشكلة
```
Exception: Redirect loop detected
```

### السبب
الـ redirect بيروح لصفحة بتعمل redirect تاني.

### الحل
```dart
// ❌ غلط
redirect: (context, state) {
  if (!isLoggedIn) return '/login';
  return null;  // بس مفيش check للـ /login!
}

// ✅ صح
redirect: (context, state) {
  final isLoggingIn = state.uri.path == '/login';

  if (!isLoggedIn && !isLoggingIn) {
    return '/login';
  }

  if (isLoggedIn && isLoggingIn) {
    return '/';
  }

  return null;
}
```

---

## 4. الـ State مش بيتحفظ في الـ Tabs

### المشكلة
لما بتتنقل بين tabs، الـ scroll position والـ state بيضيعوا.

### الحل
```dart
// استخدم StatefulShellRoute بدل ShellRoute
StatefulShellRoute.indexedStack(
  builder: (context, state, navigationShell) => ...,
  branches: [
    StatefulShellBranch(routes: [...]),
  ],
)

// وأضف AutomaticKeepAliveClientMixin
class _MyScreenState extends State<MyScreen>
    with AutomaticKeepAliveClientMixin {

  @override
  bool get wantKeepAlive => true;

  @override
  Widget build(BuildContext context) {
    super.build(context);  // مهم!
    return ...;
  }
}
```

---

## 5. الـ Parameters مش بتوصل

### المشكلة
`state.pathParameters['id']` بترجع `null`.

### الحل
```dart
// تأكد من تطابق الأسماء
GoRoute(
  path: '/product/:productId',  // اسم الـ param
  builder: (context, state) {
    // ❌ غلط
    final id = state.pathParameters['id'];

    // ✅ صح
    final id = state.pathParameters['productId'];
  },
)
```

---

## 6. الـ build_runner مش شغال

### المشكلة
```
Could not find a file named "pubspec.yaml"
```

### الحل
```bash
# تأكد إنك في المجلد الصحيح
cd your_project_folder

# امسح الـ cache
dart run build_runner clean

# شغل تاني
dart run build_runner build --delete-conflicting-outputs
```

---

## 7. "part of" error

### المشكلة
```
Error: The part-of directive must be the first non-comment directive
```

### الحل
```dart
// ❌ غلط
import 'package:flutter/material.dart';
part 'routes.g.dart';  // بعد الـ imports

// ✅ صح
import 'package:flutter/material.dart';
import 'package:go_router/go_router.dart';

part 'routes.g.dart';  // بعد كل الـ imports
```

---

## 8. Deep Link مش بيشتغل

### Android
```xml
<!-- AndroidManifest.xml -->
<intent-filter>
  <action android:name="android.intent.action.VIEW" />
  <category android:name="android.intent.category.DEFAULT" />
  <category android:name="android.intent.category.BROWSABLE" />
  <data android:scheme="myapp" />
</intent-filter>
```

### iOS
```xml
<!-- Info.plist -->
<key>CFBundleURLTypes</key>
<array>
  <dict>
    <key>CFBundleURLSchemes</key>
    <array>
      <string>myapp</string>
    </array>
  </dict>
</array>
```

### اختبار
```bash
# Android
adb shell am start -a android.intent.action.VIEW -d "myapp://path"

# iOS
xcrun simctl openurl booted "myapp://path"
```

---

## 9. الـ Web Back Button مش بيشتغل صح

### الحل
```dart
// استخدم go() للـ main navigation
context.go('/products');

// استخدم push() للـ stack-based
context.push('/product/123');

// للـ web، تأكد من استخدام path-based routing
import 'package:flutter_web_plugins/url_strategy.dart';

void main() {
  usePathUrlStrategy();
  runApp(MyApp());
}
```

---

## 10. "The argument type 'Object?' can't be assigned"

### المشكلة مع Extra
```dart
final product = state.extra;  // Object?
```

### الحل
```dart
// Cast بأمان
final product = state.extra as Product?;

// أو مع null check
if (state.extra is Product) {
  final product = state.extra as Product;
}
```

---

## Debug Tips

### 1. Enable Logging
```dart
GoRouter(
  debugLogDiagnostics: true,
)
```

### 2. Print Current Location
```dart
print(GoRouterState.of(context).uri);
print(GoRouter.of(context).state.uri);
```

### 3. Check Route Stack
```dart
final router = GoRouter.of(context);
print(router.routerDelegate.currentConfiguration);
```
