# الدرس 14: Deep Linking

## يعني إيه Deep Linking؟

الـ Deep Linking هو فتح صفحة معينة في التطبيق مباشرة من link خارجي. مثلاً:
- `myapp://product/123` -> يفتح صفحة المنتج
- `https://myapp.com/user/456` -> يفتح صفحة المستخدم

---

## أنواع الـ Deep Links

| النوع | المثال | الوصف |
|-------|--------|-------|
| **URI Scheme** | `myapp://path` | Custom scheme للتطبيق |
| **Universal Links** (iOS) | `https://myapp.com/path` | HTTPS links |
| **App Links** (Android) | `https://myapp.com/path` | HTTPS links |

---

## GoRouter والـ Deep Linking

الخبر الحلو: **GoRouter بيدعم Deep Linking تلقائياً!**

لو عندك route معرف بـ path معين، GoRouter هيفهمه لما يجيله deep link.

```dart
final router = GoRouter(
  routes: [
    GoRoute(
      path: '/',
      builder: (context, state) => const HomeScreen(),
    ),
    GoRoute(
      path: '/product/:id',
      builder: (context, state) {
        final id = state.pathParameters['id']!;
        return ProductScreen(id: id);
      },
    ),
  ],
);

// Deep link: myapp://product/123
// هيفتح ProductScreen(id: '123')
```

---

## إعداد Deep Linking للـ Android

### 1. AndroidManifest.xml

```xml
<!-- android/app/src/main/AndroidManifest.xml -->
<manifest ...>
  <application ...>
    <activity ...>

      <!-- Deep Links with custom scheme -->
      <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data android:scheme="myapp" />
      </intent-filter>

      <!-- App Links with HTTPS (يحتاج verification) -->
      <intent-filter android:autoVerify="true">
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data
          android:scheme="https"
          android:host="myapp.com"
          android:pathPrefix="/" />
      </intent-filter>

    </activity>
  </application>
</manifest>
```

### 2. اختبار على Android

```bash
# Custom scheme
adb shell am start -a android.intent.action.VIEW \
  -d "myapp://product/123" com.example.myapp

# HTTPS (App Links)
adb shell am start -a android.intent.action.VIEW \
  -d "https://myapp.com/product/123" com.example.myapp
```

---

## إعداد Deep Linking للـ iOS

### 1. Info.plist للـ Custom Scheme

```xml
<!-- ios/Runner/Info.plist -->
<dict>
  ...
  <key>CFBundleURLTypes</key>
  <array>
    <dict>
      <key>CFBundleURLSchemes</key>
      <array>
        <string>myapp</string>
      </array>
      <key>CFBundleURLName</key>
      <string>com.example.myapp</string>
    </dict>
  </array>
  ...
</dict>
```

### 2. Associated Domains للـ Universal Links

في Xcode:
1. اختار الـ Runner target
2. روح لـ Signing & Capabilities
3. أضف Associated Domains
4. أضف `applinks:myapp.com`

### 3. ملف apple-app-site-association

على السيرفر بتاعك (https://myapp.com/.well-known/apple-app-site-association):

```json
{
  "applinks": {
    "apps": [],
    "details": [
      {
        "appID": "TEAM_ID.com.example.myapp",
        "paths": ["*"]
      }
    ]
  }
}
```

### 4. اختبار على iOS

```bash
# فتح Safari واكتب الـ link
myapp://product/123

# أو من الـ Terminal
xcrun simctl openurl booted "myapp://product/123"
```

---

## إعداد Deep Linking للـ Web

### الطريقة 1: Hash-based routing (الأسهل)

```dart
GoRouter(
  // مفيش إعداد إضافي للـ web
  routes: [...],
);
```

الـ URLs هتبقى زي: `https://myapp.com/#/product/123`

### الطريقة 2: Path-based routing

```dart
// في web/index.html
<script>
  // Configure the URL strategy
  window.flutterWebRenderer = "html";
</script>

// في main.dart
import 'package:flutter_web_plugins/url_strategy.dart';

void main() {
  usePathUrlStrategy();  // بدون #
  runApp(MyApp());
}
```

الـ URLs هتبقى زي: `https://myapp.com/product/123`

> ⚠️ الـ path-based routing محتاج server configuration عشان الـ routes تشتغل

---

## initialLocation vs Deep Link

```dart
GoRouter(
  initialLocation: '/home',  // الـ default location
  routes: [...],
);

// لو التطبيق اتفتح من deep link:
// myapp://product/123
// -> هيروح للـ /product/123 مش /home

// لو التطبيق اتفتح عادي:
// -> هيروح للـ /home
```

---

## معالجة الـ Deep Link

### الحصول على الـ Initial Link

```dart
import 'package:app_links/app_links.dart';

class DeepLinkHandler {
  final AppLinks _appLinks = AppLinks();

  Future<void> init() async {
    // الـ link اللي فتح التطبيق
    final initialLink = await _appLinks.getInitialLink();
    if (initialLink != null) {
      _handleDeepLink(initialLink);
    }

    // الـ links اللي بتيجي والتطبيق مفتوح
    _appLinks.uriLinkStream.listen((uri) {
      _handleDeepLink(uri);
    });
  }

  void _handleDeepLink(Uri uri) {
    // GoRouter بيتعامل معاه تلقائي
    // بس لو محتاج تعمل حاجة إضافية:
    print('Deep link received: $uri');

    // مثلاً: Track analytics
    Analytics.logDeepLink(uri.toString());
  }
}
```

---

## Redirect بناءً على الـ Deep Link

```dart
GoRouter(
  redirect: (context, state) {
    // لو جه من deep link لصفحة محمية
    if (state.uri.path.startsWith('/admin') && !isAdmin) {
      return '/login?redirect=${state.uri}';
    }

    // لو جه من deep link قديم
    if (state.uri.path.startsWith('/old-path')) {
      return state.uri.path.replaceFirst('/old-path', '/new-path');
    }

    return null;
  },
  routes: [...],
);
```

---

## Query Parameters في Deep Links

```dart
// Deep link: myapp://search?q=flutter&category=books

GoRoute(
  path: '/search',
  builder: (context, state) {
    final query = state.uri.queryParameters['q'];
    final category = state.uri.queryParameters['category'];

    return SearchScreen(
      query: query,
      category: category,
    );
  },
)
```

---

## مثال شامل

```dart
final appRouter = GoRouter(
  initialLocation: '/',
  debugLogDiagnostics: true,

  redirect: (context, state) {
    final isLoggedIn = AuthService.instance.isLoggedIn;
    final isProtectedPath = state.uri.path.startsWith('/profile') ||
                            state.uri.path.startsWith('/orders');

    // Deep link لصفحة محمية
    if (!isLoggedIn && isProtectedPath) {
      // احفظ الـ deep link عشان نرجعله بعد Login
      return '/login?redirect=${Uri.encodeComponent(state.uri.toString())}';
    }

    return null;
  },

  routes: [
    GoRoute(
      path: '/',
      builder: (context, state) => const HomeScreen(),
    ),

    // Product deep link
    GoRoute(
      path: '/product/:id',
      builder: (context, state) {
        final id = state.pathParameters['id']!;
        // ممكن يجي من deep link مع extra query params
        final ref = state.uri.queryParameters['ref'];

        if (ref != null) {
          Analytics.logReferral(ref);
        }

        return ProductScreen(id: id);
      },
    ),

    // User profile deep link
    GoRoute(
      path: '/user/:username',
      builder: (context, state) {
        final username = state.pathParameters['username']!;
        return UserProfileScreen(username: username);
      },
    ),

    // Search with filters
    GoRoute(
      path: '/search',
      builder: (context, state) {
        return SearchScreen(
          query: state.uri.queryParameters['q'],
          category: state.uri.queryParameters['category'],
          sort: state.uri.queryParameters['sort'],
        );
      },
    ),

    // Share link
    GoRoute(
      path: '/share/:type/:id',
      builder: (context, state) {
        final type = state.pathParameters['type']!;
        final id = state.pathParameters['id']!;

        // Redirect to the actual content
        switch (type) {
          case 'product':
            return ProductScreen(id: id);
          case 'article':
            return ArticleScreen(id: id);
          default:
            return const NotFoundScreen();
        }
      },
    ),

    GoRoute(
      path: '/login',
      builder: (context, state) {
        final redirect = state.uri.queryParameters['redirect'];
        return LoginScreen(redirectTo: redirect);
      },
    ),
  ],
);
```

---

## اختبار الـ Deep Links

### 1. من الـ Terminal

```bash
# Android
adb shell am start -a android.intent.action.VIEW \
  -d "myapp://product/123?ref=facebook" \
  com.example.myapp

# iOS Simulator
xcrun simctl openurl booted "myapp://product/123"
```

### 2. Integration Test

```dart
testWidgets('deep link opens product screen', (tester) async {
  final router = GoRouter(
    initialLocation: '/product/123',
    routes: [...],
  );

  await tester.pumpWidget(MaterialApp.router(routerConfig: router));

  expect(find.byType(ProductScreen), findsOneWidget);
});
```

---

## نصائح

### 1. استخدم debugLogDiagnostics

```dart
GoRouter(
  debugLogDiagnostics: true,  // هيطبع الـ deep links في الـ console
  routes: [...],
)
```

### 2. Handle Invalid Deep Links

```dart
GoRouter(
  onException: (context, state, router) {
    // Deep link لصفحة مش موجودة
    Analytics.logInvalidDeepLink(state.uri.toString());
    router.go('/');
  },
  routes: [...],
)
```

### 3. Share Links

```dart
// إنشاء link للمشاركة
String createShareLink(String productId) {
  return 'https://myapp.com/product/$productId';
  // أو
  return 'myapp://product/$productId';
}

// استخدامه
Share.share(createShareLink(product.id));
```

---

## ملخص

| العنصر | الوصف |
|--------|-------|
| **GoRouter** | بيدعم deep linking تلقائي |
| **Android** | AndroidManifest.xml |
| **iOS** | Info.plist + Associated Domains |
| **Web** | Hash أو Path-based |
| **Redirect** | للـ protected routes |

---

[⬅️ الدرس السابق: Transition Animations](13_transition_animations.md) | [➡️ الدرس التالي: Extra Data & Codec](15_extra_codec.md)
