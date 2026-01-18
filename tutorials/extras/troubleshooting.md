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
// Make sure you're using MaterialApp.router
MaterialApp.router(
  routerConfig: appRouter,  // ✅
)

// Not the regular MaterialApp
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
// 1. Make sure the path is correct
context.go('/products');  // Make sure it's defined

// 2. Add errorBuilder
GoRouter(
  errorBuilder: (context, state) => NotFoundScreen(),
  routes: [...],
)

// 3. Or use onException
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
// Wrong ❌
redirect: (context, state) {
  if (!isLoggedIn) return '/login';
  return null;  // But there's no check for /login!
}

// Correct ✅
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
// Use StatefulShellRoute instead of ShellRoute
StatefulShellRoute.indexedStack(
  builder: (context, state, navigationShell) => ...,
  branches: [
    StatefulShellBranch(routes: [...]),
  ],
)

// And add AutomaticKeepAliveClientMixin
class _MyScreenState extends State<MyScreen>
    with AutomaticKeepAliveClientMixin {

  @override
  bool get wantKeepAlive => true;

  @override
  Widget build(BuildContext context) {
    super.build(context);  // Important!
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
// Make sure names match
GoRoute(
  path: '/product/:productId',  // The param name
  builder: (context, state) {
    // Wrong ❌
    final id = state.pathParameters['id'];

    // Correct ✅
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
# Make sure you're in the correct folder
cd your_project_folder

# Clear the cache
dart run build_runner clean

# Run again
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
// Wrong ❌
import 'package:flutter/material.dart';
part 'routes.g.dart';  // After the imports

// Correct ✅
import 'package:flutter/material.dart';
import 'package:go_router/go_router.dart';

part 'routes.g.dart';  // After all imports
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
// Use go() for main navigation
context.go('/products');

// Use push() for stack-based
context.push('/product/123');

// For web, make sure to use path-based routing
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
// Cast safely
final product = state.extra as Product?;

// Or with null check
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
