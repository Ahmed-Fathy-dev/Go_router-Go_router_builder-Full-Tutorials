# الدرس 25: Performance (الأداء)

## 1. استخدم StatefulShellRoute للـ Tabs

```dart
// ❌ غلط - كل tab بيتبني من الأول
ShellRoute(
  builder: (context, state, child) => TabScaffold(child: child),
  routes: [...],
)

// ✅ صح - الـ tabs بتتحفظ
StatefulShellRoute.indexedStack(
  builder: (context, state, navigationShell) => TabScaffold(
    navigationShell: navigationShell,
  ),
  branches: [...],
)
```

---

## 2. استخدم AutomaticKeepAliveClientMixin

```dart
class MyTabScreen extends StatefulWidget {
  @override
  State<MyTabScreen> createState() => _MyTabScreenState();
}

class _MyTabScreenState extends State<MyTabScreen>
    with AutomaticKeepAliveClientMixin {

  @override
  bool get wantKeepAlive => true;  // 👈 مهم

  @override
  Widget build(BuildContext context) {
    super.build(context);  // 👈 مهم
    return ExpensiveWidget();
  }
}
```

---

## 3. تجنب إعادة البناء غير الضرورية

```dart
// ❌ غلط - الـ router بيتبني كل مرة
Widget build(BuildContext context) {
  final router = GoRouter(routes: [...]);  // جديد كل مرة!
  return MaterialApp.router(routerConfig: router);
}

// ✅ صح - الـ router ثابت
final appRouter = GoRouter(routes: [...]);

Widget build(BuildContext context) {
  return MaterialApp.router(routerConfig: appRouter);
}
```

---

## 4. Lazy Loading للـ Screens

```dart
GoRoute(
  path: '/heavy-screen',
  builder: (context, state) {
    // الـ screen بتتحمل بس لما تحتاجها
    return const HeavyScreen();
  },
)

// أو باستخدام FutureBuilder
GoRoute(
  path: '/data-screen',
  builder: (context, state) {
    return FutureBuilder<Data>(
      future: DataService.loadData(),
      builder: (context, snapshot) {
        if (snapshot.connectionState == ConnectionState.waiting) {
          return const LoadingScreen();
        }
        return DataScreen(data: snapshot.data!);
      },
    );
  },
)
```

---

## 5. تجنب Redirect المعقد

```dart
// ❌ غلط - redirect بطيء
redirect: (context, state) async {
  await Future.delayed(Duration(seconds: 1));  // ❌
  final user = await AuthService.getUser();    // ❌
  // ...
}

// ✅ صح - redirect سريع
redirect: (context, state) {
  // استخدم cached data
  final isLoggedIn = AuthService.cachedLoginState;
  if (!isLoggedIn) return '/login';
  return null;
}
```

---

## 6. استخدم const للـ Routes

```dart
// ✅ صح - const routes
const HomeRoute().go(context);
const ProductRoute(id: 123).push(context);

// Routes classes
@TypedGoRoute<HomeRoute>(path: '/')
class HomeRoute extends GoRouteData {
  const HomeRoute();  // 👈 const constructor
  // ...
}
```

---

## 7. NoTransitionPage للـ Tabs

```dart
StatefulShellBranch(
  routes: [
    GoRoute(
      path: '/home',
      pageBuilder: (context, state) {
        // بدون transition للـ tabs
        return NoTransitionPage(
          key: state.pageKey,
          child: const HomeScreen(),
        );
      },
    ),
  ],
)
```

---

## 8. Extra بدل API Calls

```dart
// ❌ غلط - API call كل مرة
GoRoute(
  path: '/product/:id',
  builder: (context, state) {
    return FutureBuilder(
      future: ProductService.getProduct(state.pathParameters['id']!),
      builder: ...
    );
  },
)

// ✅ صح - استخدم extra لو عندك الـ data
context.go('/product/123', extra: productObject);

GoRoute(
  path: '/product/:id',
  builder: (context, state) {
    final product = state.extra as Product?;
    if (product != null) {
      return ProductScreen(product: product);  // مباشرة!
    }
    // Fallback لو مفيش extra
    return ProductScreen.fromId(state.pathParameters['id']!);
  },
)
```

---

## 9. تحسين الـ Rebuild

```dart
class TabScaffold extends StatelessWidget {
  final StatefulNavigationShell navigationShell;

  const TabScaffold({super.key, required this.navigationShell});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      // ❌ الـ body بيتبني كل مرة
      // body: navigationShell,

      // ✅ استخدم RepaintBoundary
      body: RepaintBoundary(
        child: navigationShell,
      ),
      bottomNavigationBar: const BottomNav(),  // const!
    );
  }
}
```

---

## 10. Deferred Loading

```dart
// للـ screens الكبيرة
import 'heavy_screen.dart' deferred as heavy;

GoRoute(
  path: '/heavy',
  builder: (context, state) {
    return FutureBuilder(
      future: heavy.loadLibrary(),
      builder: (context, snapshot) {
        if (snapshot.connectionState == ConnectionState.done) {
          return heavy.HeavyScreen();
        }
        return const LoadingScreen();
      },
    );
  },
)
```

---

## ملخص

| النصيحة | الفائدة |
|---------|--------|
| StatefulShellRoute | حفظ حالة الـ tabs |
| AutomaticKeepAlive | منع إعادة البناء |
| const Router | تجنب rebuilds |
| Sync Redirect | سرعة الـ navigation |
| Extra Data | تجنب API calls |
| NoTransitionPage | سرعة الـ tabs |

---

[⬅️ الدرس السابق: Testing](24_testing.md) | [➡️ الدرس التالي: Migration Guide](26_migration_guide.md)
