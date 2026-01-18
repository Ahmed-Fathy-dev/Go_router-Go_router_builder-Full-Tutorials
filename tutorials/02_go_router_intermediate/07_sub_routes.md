# الدرس 7: Sub-Routes (الـ Routes المتداخلة)

## يعني إيه Sub-Routes؟

الـ Sub-Routes هي routes بتتعرف جوه route تاني. بتستخدمها لما يكون عندك صفحات مرتبطة ببعض أو عايز تعمل هيكل هرمي للـ navigation.

### مثال على الهيكل:
```
/settings                  -> SettingsScreen
/settings/profile          -> ProfileSettingsScreen
/settings/notifications    -> NotificationSettingsScreen
/settings/privacy          -> PrivacySettingsScreen
```

---

## تعريف Sub-Routes

### الـ Syntax

```dart
GoRoute(
  path: '/settings',
  builder: (context, state) => const SettingsScreen(),
  routes: [  // 👈 Here are the sub-routes
    GoRoute(
      path: 'profile',  // ⚠️ Note: no / at the beginning
      builder: (context, state) => const ProfileSettingsScreen(),
    ),
    GoRoute(
      path: 'notifications',
      builder: (context, state) => const NotificationSettingsScreen(),
    ),
  ],
)
```

> ⚠️ **مهم جداً**: الـ sub-routes مش بتبدأ بـ `/`. لو كتبت `/profile` هيبقى route مستقل مش sub-route!

---

## السلوك الافتراضي

لما تروح لـ sub-route، الـ GoRouter بيعرض الصفحة الجديدة **فوق** الصفحة الأب:

```dart
// When you go to /settings/profile
// The stack will be: [SettingsScreen, ProfileSettingsScreen]
context.go('/settings/profile');

// When you pop
context.pop();
// You'll return to /settings
```

---

## مثال عملي كامل

### هيكل تطبيق متجر

```dart
final appRouter = GoRouter(
  initialLocation: '/',
  routes: [
    // Home page
    GoRoute(
      path: '/',
      name: 'home',
      builder: (context, state) => const HomeScreen(),
    ),

    // Products with sub-routes
    GoRoute(
      path: '/products',
      name: 'products',
      builder: (context, state) => const ProductsScreen(),
      routes: [
        // Product details
        GoRoute(
          path: ':productId',  // /products/123
          name: 'product-details',
          builder: (context, state) {
            final id = state.pathParameters['productId']!;
            return ProductDetailsScreen(id: id);
          },
          routes: [
            // Product reviews
            GoRoute(
              path: 'reviews',  // /products/123/reviews
              name: 'product-reviews',
              builder: (context, state) {
                final id = state.pathParameters['productId']!;
                return ProductReviewsScreen(productId: id);
              },
            ),
            // Product questions
            GoRoute(
              path: 'questions',  // /products/123/questions
              name: 'product-questions',
              builder: (context, state) {
                final id = state.pathParameters['productId']!;
                return ProductQuestionsScreen(productId: id);
              },
            ),
          ],
        ),
      ],
    ),

    // Settings with sub-routes
    GoRoute(
      path: '/settings',
      name: 'settings',
      builder: (context, state) => const SettingsScreen(),
      routes: [
        GoRoute(
          path: 'profile',  // /settings/profile
          name: 'settings-profile',
          builder: (context, state) => const ProfileSettingsScreen(),
        ),
        GoRoute(
          path: 'notifications',  // /settings/notifications
          name: 'settings-notifications',
          builder: (context, state) => const NotificationSettingsScreen(),
        ),
        GoRoute(
          path: 'security',  // /settings/security
          name: 'settings-security',
          builder: (context, state) => const SecuritySettingsScreen(),
          routes: [
            GoRoute(
              path: 'change-password',  // /settings/security/change-password
              name: 'change-password',
              builder: (context, state) => const ChangePasswordScreen(),
            ),
            GoRoute(
              path: 'two-factor',  // /settings/security/two-factor
              name: 'two-factor',
              builder: (context, state) => const TwoFactorScreen(),
            ),
          ],
        ),
      ],
    ),
  ],
);
```

### شاشة الإعدادات

```dart
class SettingsScreen extends StatelessWidget {
  const SettingsScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('الإعدادات')),
      body: ListView(
        children: [
          ListTile(
            leading: const Icon(Icons.person),
            title: const Text('الملف الشخصي'),
            trailing: const Icon(Icons.chevron_right),
            onTap: () => context.go('/settings/profile'),
          ),
          ListTile(
            leading: const Icon(Icons.notifications),
            title: const Text('الإشعارات'),
            trailing: const Icon(Icons.chevron_right),
            onTap: () => context.go('/settings/notifications'),
          ),
          ListTile(
            leading: const Icon(Icons.security),
            title: const Text('الأمان'),
            trailing: const Icon(Icons.chevron_right),
            onTap: () => context.go('/settings/security'),
          ),
        ],
      ),
    );
  }
}
```

### شاشة إعدادات الأمان

```dart
class SecuritySettingsScreen extends StatelessWidget {
  const SecuritySettingsScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('الأمان')),
      body: ListView(
        children: [
          ListTile(
            leading: const Icon(Icons.lock),
            title: const Text('تغيير كلمة المرور'),
            trailing: const Icon(Icons.chevron_right),
            onTap: () => context.go('/settings/security/change-password'),
          ),
          ListTile(
            leading: const Icon(Icons.phone_android),
            title: const Text('التحقق بخطوتين'),
            trailing: const Icon(Icons.chevron_right),
            onTap: () => context.go('/settings/security/two-factor'),
          ),
        ],
      ),
    );
  }
}
```

---

## الوصول للـ Path Parameters من الـ Parent

الـ sub-routes بتقدر توصل للـ parameters من الـ parent routes:

```dart
GoRoute(
  path: '/user/:userId',
  builder: (context, state) {
    final userId = state.pathParameters['userId']!;
    return UserScreen(userId: userId);
  },
  routes: [
    GoRoute(
      path: 'posts',  // /user/123/posts
      builder: (context, state) {
        // You can access the userId from the parent
        final userId = state.pathParameters['userId']!;
        return UserPostsScreen(userId: userId);
      },
      routes: [
        GoRoute(
          path: ':postId',  // /user/123/posts/456
          builder: (context, state) {
            // You can access both parameters
            final userId = state.pathParameters['userId']!;
            final postId = state.pathParameters['postId']!;
            return PostDetailsScreen(userId: userId, postId: postId);
          },
        ),
      ],
    ),
  ],
)
```

---

## استخدام go() vs push() مع Sub-Routes

### go()
```dart
// From anywhere, go to the sub-route
context.go('/settings/profile');
// The Stack: [SettingsScreen, ProfileSettingsScreen]

// Even if you're on a completely different page
context.go('/settings/security/change-password');
// The Stack: [SettingsScreen, SecuritySettingsScreen, ChangePasswordScreen]
```

### push()
```dart
// If you're in SettingsScreen
context.push('/settings/profile');
// Adds ProfileSettingsScreen on top of the current stack
```

---

## Sub-Routes مع redirect

تقدر تستخدم redirect في الـ sub-routes:

```dart
GoRoute(
  path: '/admin',
  builder: (context, state) => const AdminDashboard(),
  redirect: (context, state) {
    if (!isAdmin) return '/';
    return null;
  },
  routes: [
    GoRoute(
      path: 'users',  // /admin/users
      builder: (context, state) => const AdminUsersScreen(),
      redirect: (context, state) {
        // Additional redirect for the sub-route
        if (!hasUserManagementPermission) return '/admin';
        return null;
      },
    ),
    GoRoute(
      path: 'settings',  // /admin/settings
      builder: (context, state) => const AdminSettingsScreen(),
    ),
  ],
)
```

---

## Nesting متعدد المستويات

```dart
GoRoute(
  path: '/shop',
  builder: (context, state) => const ShopScreen(),
  routes: [
    GoRoute(
      path: 'category/:categoryId',
      builder: (context, state) => CategoryScreen(...),
      routes: [
        GoRoute(
          path: 'subcategory/:subId',
          builder: (context, state) => SubcategoryScreen(...),
          routes: [
            GoRoute(
              path: 'product/:productId',
              builder: (context, state) {
                // All parameters are available
                final categoryId = state.pathParameters['categoryId']!;
                final subId = state.pathParameters['subId']!;
                final productId = state.pathParameters['productId']!;

                return ProductScreen(
                  categoryId: categoryId,
                  subcategoryId: subId,
                  productId: productId,
                );
              },
            ),
          ],
        ),
      ],
    ),
  ],
)

// Result:
// /shop
// /shop/category/electronics
// /shop/category/electronics/subcategory/phones
// /shop/category/electronics/subcategory/phones/product/iphone-15
```

---

## fullPath في GoRouterState

الـ `fullPath` بيديك الـ pattern الكامل للـ route:

```dart
GoRoute(
  path: '/user/:userId/order/:orderId',
  builder: (context, state) {
    print(state.uri.path);  // /user/123/order/456 (actual values)
    print(state.fullPath);  // /user/:userId/order/:orderId (the pattern)

    return OrderScreen(...);
  },
)
```

---

## نصائح مهمة

### 1. الـ sub-routes مش بتبدأ بـ `/`

```dart
// Correct ✅
routes: [
  GoRoute(path: 'details', ...),  // Will be /parent/details
]

// Wrong ❌
routes: [
  GoRoute(path: '/details', ...),  // Will be an independent route /details
]
```

### 2. استخدم أسماء واضحة

```dart
// Correct ✅ Names that clarify the relationship
GoRoute(
  path: '/settings',
  name: 'settings',
  routes: [
    GoRoute(path: 'profile', name: 'settings-profile', ...),
    GoRoute(path: 'notifications', name: 'settings-notifications', ...),
  ],
)
```

### 3. متعمقش كتير

```dart
// ⚠️ Avoid excessive complexity
/a/b/c/d/e/f/g  // Hard to maintain

// Correct ✅ Try to stay at 2-3 levels
/shop/category/product
```

### 4. استخدم Sub-Routes لما يكون فيه علاقة

```dart
// Correct ✅ Makes sense - reviews belong to product
/products/:id/reviews

// Wrong ❌ Doesn't make sense - cart doesn't belong to product
/products/:id/cart  // Better to use /cart
```

---

## ملخص

| العنصر | الوصف |
|--------|-------|
| **التعريف** | جوه `routes: []` في الـ parent |
| **الـ Path** | مش بيبدأ بـ `/` |
| **الـ Parameters** | متاحة من كل الـ parents |
| **الـ Navigation Stack** | بيضيف الصفحات فوق بعض |
| **الاستخدام** | صفحات مرتبطة ببعض |

---

[⬅️ الدرس السابق: Named Routes](../01_go_router_basics/06_named_routes.md) | [➡️ الدرس التالي: Redirection](08_redirection.md)
