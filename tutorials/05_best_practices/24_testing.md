# الدرس 24: Testing (اختبار الـ Routes)

## إعداد الاختبارات

```yaml
# pubspec.yaml
dev_dependencies:
  flutter_test:
    sdk: flutter
  mocktail: ^1.0.0  # For mocking
```

---

## اختبار Navigation البسيط

```dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:go_router/go_router.dart';

void main() {
  testWidgets('navigates to details screen', (tester) async {
    final router = GoRouter(
      initialLocation: '/',
      routes: [
        GoRoute(
          path: '/',
          builder: (context, state) => Scaffold(
            body: ElevatedButton(
              onPressed: () => context.go('/details'),
              child: const Text('Go to Details'),
            ),
          ),
        ),
        GoRoute(
          path: '/details',
          builder: (context, state) => const Scaffold(
            body: Text('Details Screen'),
          ),
        ),
      ],
    );

    await tester.pumpWidget(MaterialApp.router(routerConfig: router));

    // Verify initial screen
    expect(find.text('Go to Details'), findsOneWidget);

    // Tap the button
    await tester.tap(find.text('Go to Details'));
    await tester.pumpAndSettle();

    // Verify navigation
    expect(find.text('Details Screen'), findsOneWidget);
  });
}
```

---

## اختبار Path Parameters

```dart
testWidgets('passes path parameters correctly', (tester) async {
  final router = GoRouter(
    initialLocation: '/product/123',
    routes: [
      GoRoute(
        path: '/product/:id',
        builder: (context, state) {
          final id = state.pathParameters['id'];
          return Scaffold(body: Text('Product ID: $id'));
        },
      ),
    ],
  );

  await tester.pumpWidget(MaterialApp.router(routerConfig: router));

  expect(find.text('Product ID: 123'), findsOneWidget);
});
```

---

## اختبار Query Parameters

```dart
testWidgets('handles query parameters', (tester) async {
  final router = GoRouter(
    initialLocation: '/search?q=flutter&page=2',
    routes: [
      GoRoute(
        path: '/search',
        builder: (context, state) {
          final query = state.uri.queryParameters['q'];
          final page = state.uri.queryParameters['page'];
          return Scaffold(
            body: Text('Search: $query, Page: $page'),
          );
        },
      ),
    ],
  );

  await tester.pumpWidget(MaterialApp.router(routerConfig: router));

  expect(find.text('Search: flutter, Page: 2'), findsOneWidget);
});
```

---

## اختبار Redirect

```dart
testWidgets('redirects unauthenticated users to login', (tester) async {
  bool isLoggedIn = false;

  final router = GoRouter(
    initialLocation: '/profile',
    redirect: (context, state) {
      if (!isLoggedIn && state.uri.path != '/login') {
        return '/login';
      }
      return null;
    },
    routes: [
      GoRoute(
        path: '/login',
        builder: (context, state) => const Scaffold(
          body: Text('Login Screen'),
        ),
      ),
      GoRoute(
        path: '/profile',
        builder: (context, state) => const Scaffold(
          body: Text('Profile Screen'),
        ),
      ),
    ],
  );

  await tester.pumpWidget(MaterialApp.router(routerConfig: router));
  await tester.pumpAndSettle();

  // Should be redirected to login
  expect(find.text('Login Screen'), findsOneWidget);
  expect(find.text('Profile Screen'), findsNothing);
});
```

---

## اختبار Error Handling

```dart
testWidgets('shows error screen for unknown routes', (tester) async {
  final router = GoRouter(
    initialLocation: '/unknown-route',
    errorBuilder: (context, state) => const Scaffold(
      body: Text('Error: Page not found'),
    ),
    routes: [
      GoRoute(
        path: '/',
        builder: (context, state) => const Scaffold(
          body: Text('Home'),
        ),
      ),
    ],
  );

  await tester.pumpWidget(MaterialApp.router(routerConfig: router));

  expect(find.text('Error: Page not found'), findsOneWidget);
});
```

---

## اختبار ShellRoute

```dart
testWidgets('shell route displays bottom navigation', (tester) async {
  final router = GoRouter(
    initialLocation: '/home',
    routes: [
      ShellRoute(
        builder: (context, state, child) => Scaffold(
          body: child,
          bottomNavigationBar: BottomNavigationBar(
            items: const [
              BottomNavigationBarItem(icon: Icon(Icons.home), label: 'Home'),
              BottomNavigationBarItem(icon: Icon(Icons.search), label: 'Search'),
            ],
          ),
        ),
        routes: [
          GoRoute(
            path: '/home',
            builder: (context, state) => const Text('Home Screen'),
          ),
          GoRoute(
            path: '/search',
            builder: (context, state) => const Text('Search Screen'),
          ),
        ],
      ),
    ],
  );

  await tester.pumpWidget(MaterialApp.router(routerConfig: router));

  // Check shell is displayed
  expect(find.byType(BottomNavigationBar), findsOneWidget);
  expect(find.text('Home Screen'), findsOneWidget);
});
```

---

## اختبار مع Mocking

```dart
import 'package:mocktail/mocktail.dart';

class MockAuthService extends Mock implements AuthService {}

testWidgets('redirects based on auth state', (tester) async {
  final mockAuth = MockAuthService();

  // Setup mock
  when(() => mockAuth.isLoggedIn).thenReturn(false);

  final router = GoRouter(
    initialLocation: '/profile',
    redirect: (context, state) {
      if (!mockAuth.isLoggedIn && state.uri.path != '/login') {
        return '/login';
      }
      return null;
    },
    routes: [...],
  );

  await tester.pumpWidget(MaterialApp.router(routerConfig: router));
  await tester.pumpAndSettle();

  expect(find.text('Login Screen'), findsOneWidget);

  // Verify mock was called
  verify(() => mockAuth.isLoggedIn).called(greaterThan(0));
});
```

---

## اختبار GoRouter Builder Routes

```dart
testWidgets('typed route navigates correctly', (tester) async {
  final router = GoRouter(
    initialLocation: '/',
    routes: $appRoutes,  // Generated routes
  );

  await tester.pumpWidget(MaterialApp.router(routerConfig: router));

  expect(find.byType(HomeScreen), findsOneWidget);

  // Navigate using typed route
  const ProductRoute(id: 123).go(
    tester.element(find.byType(HomeScreen)),
  );
  await tester.pumpAndSettle();

  expect(find.byType(ProductScreen), findsOneWidget);
});
```

---

## Integration Test

```dart
// test/integration_test.dart
import 'package:flutter_test/flutter_test.dart';
import 'package:integration_test/integration_test.dart';
import 'package:my_app/main.dart' as app;

void main() {
  IntegrationTestWidgetsFlutterBinding.ensureInitialized();

  testWidgets('complete user flow', (tester) async {
    app.main();
    await tester.pumpAndSettle();

    // Login
    await tester.tap(find.text('Login'));
    await tester.pumpAndSettle();

    await tester.enterText(find.byType(TextField).first, 'user@test.com');
    await tester.enterText(find.byType(TextField).last, 'password');
    await tester.tap(find.text('Submit'));
    await tester.pumpAndSettle();

    // Navigate to products
    expect(find.text('Home'), findsOneWidget);

    await tester.tap(find.text('Products'));
    await tester.pumpAndSettle();

    expect(find.byType(ProductsScreen), findsOneWidget);
  });
}
```

---

## Helper Functions

```dart
// test/helpers/router_helper.dart
GoRouter createTestRouter({
  String initialLocation = '/',
  bool isLoggedIn = true,
}) {
  return GoRouter(
    initialLocation: initialLocation,
    redirect: (context, state) {
      if (!isLoggedIn && state.uri.path != '/login') {
        return '/login';
      }
      return null;
    },
    routes: testRoutes,
  );
}

Widget createTestApp(GoRouter router) {
  return MaterialApp.router(routerConfig: router);
}

// Usage
testWidgets('test with helper', (tester) async {
  final router = createTestRouter(
    initialLocation: '/profile',
    isLoggedIn: false,
  );

  await tester.pumpWidget(createTestApp(router));
  await tester.pumpAndSettle();

  expect(find.text('Login'), findsOneWidget);
});
```

---

## ملخص

| نوع الاختبار | ماذا يختبر |
|--------------|-----------|
| **Navigation** | الانتقال بين الصفحات |
| **Parameters** | Path & Query params |
| **Redirect** | حماية الصفحات |
| **Error** | صفحات الخطأ |
| **Shell** | Bottom Nav & layouts |
| **Integration** | User flows كاملة |

---

[⬅️ الدرس السابق: Common Patterns](23_common_patterns.md) | [➡️ الدرس التالي: Performance](25_performance.md)
