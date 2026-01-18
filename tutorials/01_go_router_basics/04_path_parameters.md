# الدرس 4: Path Parameters

## يعني إيه Path Parameters؟

الـ Path Parameters هي قيم ديناميكية بتكون جزء من الـ URL نفسه. بنستخدمها لما نحتاج نمرر بيانات أساسية للصفحة.

### أمثلة:
```
/user/123          -> 123 هو الـ userId
/product/abc-456   -> abc-456 هو الـ productId
/category/electronics/item/789 -> electronics و 789
```

---

## تعريف Path Parameters

### الـ Syntax
بنستخدم `:` قبل اسم الـ parameter:

```dart
GoRoute(
  path: '/user/:id',  // :id is the path parameter
  builder: (context, state) {
    final userId = state.pathParameters['id']!;
    return UserScreen(userId: userId);
  },
)
```

### أمثلة متعددة

```dart
// Single parameter
GoRoute(
  path: '/product/:productId',
  builder: (context, state) {
    final productId = state.pathParameters['productId']!;
    return ProductScreen(id: productId);
  },
)

// Multiple parameters
GoRoute(
  path: '/store/:storeId/product/:productId',
  builder: (context, state) {
    final storeId = state.pathParameters['storeId']!;
    final productId = state.pathParameters['productId']!;
    return ProductScreen(storeId: storeId, productId: productId);
  },
)

// Parameter in the middle of path
GoRoute(
  path: '/users/:userId/orders',
  builder: (context, state) {
    final userId = state.pathParameters['userId']!;
    return UserOrdersScreen(userId: userId);
  },
)
```

---

## الوصول للـ Parameters

### في الـ builder

```dart
GoRoute(
  path: '/article/:slug',
  builder: (context, state) {
    // pathParameters is a Map<String, String>
    final Map<String, String> params = state.pathParameters;

    // Get a specific value
    final String slug = params['slug']!;

    // Or directly
    final String slug2 = state.pathParameters['slug']!;

    return ArticleScreen(slug: slug);
  },
)
```

### التحويل لأنواع أخرى

الـ Path Parameters دايماً بتيجي كـ `String`، لو محتاج نوع تاني لازم تحوله:

```dart
GoRoute(
  path: '/user/:id',
  builder: (context, state) {
    // Convert to int
    final int userId = int.parse(state.pathParameters['id']!);

    return UserScreen(userId: userId);
  },
)

GoRoute(
  path: '/date/:timestamp',
  builder: (context, state) {
    // Convert to DateTime
    final timestamp = int.parse(state.pathParameters['timestamp']!);
    final date = DateTime.fromMillisecondsSinceEpoch(timestamp);

    return DateScreen(date: date);
  },
)
```

### التعامل مع القيم الاختيارية

```dart
GoRoute(
  path: '/product/:id',
  builder: (context, state) {
    // Use tryParse for safety
    final id = int.tryParse(state.pathParameters['id'] ?? '');

    if (id == null) {
      return const ErrorScreen(message: 'Invalid product ID');
    }

    return ProductScreen(id: id);
  },
)
```

---

## التنقل مع Path Parameters

### باستخدام go()

```dart
// Go to specific user page
context.go('/user/123');

// With variable
final userId = '456';
context.go('/user/$userId');

// Multiple parameters
context.go('/store/main/product/789');
```

### باستخدام push()

```dart
// Open product page
context.push('/product/${product.id}');

// With String interpolation
final storeId = 'cairo';
final productId = 'laptop-001';
context.push('/store/$storeId/product/$productId');
```

---

## مثال عملي كامل

### تعريف الـ Routes

```dart
final appRouter = GoRouter(
  initialLocation: '/',
  routes: [
    // Home page
    GoRoute(
      path: '/',
      builder: (context, state) => const HomeScreen(),
    ),

    // Users list
    GoRoute(
      path: '/users',
      builder: (context, state) => const UsersListScreen(),
    ),

    // Specific user details
    GoRoute(
      path: '/user/:userId',
      builder: (context, state) {
        final userId = state.pathParameters['userId']!;
        return UserDetailsScreen(userId: userId);
      },
    ),

    // Specific user orders
    GoRoute(
      path: '/user/:userId/orders',
      builder: (context, state) {
        final userId = state.pathParameters['userId']!;
        return UserOrdersScreen(userId: userId);
      },
    ),

    // Specific order details
    GoRoute(
      path: '/user/:userId/order/:orderId',
      builder: (context, state) {
        final userId = state.pathParameters['userId']!;
        final orderId = state.pathParameters['orderId']!;
        return OrderDetailsScreen(userId: userId, orderId: orderId);
      },
    ),
  ],
);
```

### شاشة قائمة المستخدمين

```dart
class UsersListScreen extends StatelessWidget {
  const UsersListScreen({super.key});

  // Sample data
  final List<Map<String, dynamic>> users = const [
    {'id': '1', 'name': 'أحمد محمد'},
    {'id': '2', 'name': 'سارة علي'},
    {'id': '3', 'name': 'محمود حسن'},
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('المستخدمين')),
      body: ListView.builder(
        itemCount: users.length,
        itemBuilder: (context, index) {
          final user = users[index];
          return ListTile(
            leading: CircleAvatar(child: Text(user['name'][0])),
            title: Text(user['name']),
            subtitle: Text('ID: ${user['id']}'),
            trailing: const Icon(Icons.arrow_forward_ios),
            onTap: () {
              // Navigate with userId
              context.push('/user/${user['id']}');
            },
          );
        },
      ),
    );
  }
}
```

### شاشة تفاصيل المستخدم

```dart
class UserDetailsScreen extends StatelessWidget {
  final String userId;

  const UserDetailsScreen({super.key, required this.userId});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('المستخدم #$userId')),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // User info
            Card(
              child: Padding(
                padding: const EdgeInsets.all(16),
                child: Column(
                  children: [
                    const CircleAvatar(
                      radius: 40,
                      child: Icon(Icons.person, size: 40),
                    ),
                    const SizedBox(height: 16),
                    Text(
                      'User ID: $userId',
                      style: Theme.of(context).textTheme.headlineSmall,
                    ),
                  ],
                ),
              ),
            ),

            const SizedBox(height: 24),

            // Navigation buttons
            SizedBox(
              width: double.infinity,
              child: ElevatedButton.icon(
                onPressed: () {
                  // Navigate to user orders
                  context.push('/user/$userId/orders');
                },
                icon: const Icon(Icons.shopping_bag),
                label: const Text('عرض الطلبات'),
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

### شاشة طلبات المستخدم

```dart
class UserOrdersScreen extends StatelessWidget {
  final String userId;

  const UserOrdersScreen({super.key, required this.userId});

  @override
  Widget build(BuildContext context) {
    // Sample data
    final orders = [
      {'id': 'ORD-001', 'total': 150.0, 'status': 'delivered'},
      {'id': 'ORD-002', 'total': 280.0, 'status': 'pending'},
      {'id': 'ORD-003', 'total': 95.0, 'status': 'shipped'},
    ];

    return Scaffold(
      appBar: AppBar(title: Text('طلبات المستخدم #$userId')),
      body: ListView.builder(
        itemCount: orders.length,
        itemBuilder: (context, index) {
          final order = orders[index];
          return ListTile(
            title: Text(order['id'] as String),
            subtitle: Text('الإجمالي: ${order['total']} ج.م'),
            trailing: Chip(label: Text(order['status'] as String)),
            onTap: () {
              // Navigate to order details with userId and orderId
              context.push('/user/$userId/order/${order['id']}');
            },
          );
        },
      ),
    );
  }
}
```

---

## نصائح مهمة

### 1. التحقق من صحة الـ Parameters

```dart
GoRoute(
  path: '/product/:id',
  builder: (context, state) {
    final idString = state.pathParameters['id'];

    // Check if value exists
    if (idString == null || idString.isEmpty) {
      return const ErrorScreen(message: 'Product ID is required');
    }

    // Check if conversion is valid
    final id = int.tryParse(idString);
    if (id == null) {
      return const ErrorScreen(message: 'Invalid Product ID');
    }

    return ProductScreen(id: id);
  },
)
```

### 2. استخدام redirect للتحقق

```dart
GoRoute(
  path: '/user/:id',
  redirect: (context, state) {
    final id = state.pathParameters['id'];

    // If ID is not valid, redirect to home
    if (id == null || int.tryParse(id) == null) {
      return '/';
    }

    return null; // Don't redirect
  },
  builder: (context, state) {
    final id = int.parse(state.pathParameters['id']!);
    return UserScreen(id: id);
  },
)
```

### 3. Case Sensitivity

بشكل افتراضي، الـ GoRouter **case-sensitive** في الـ paths:

```dart
// These are different routes!
/user/ABC
/user/abc

// To disable case sensitivity
GoRouter(
  caseSensitive: false, // Will treat them as the same route
  routes: [...],
)
```

### 4. تجنب الـ Parameters الكتيرة

```dart
// ❌ Hard to read and maintain
'/a/:x/b/:y/c/:z/d/:w'

// ✅ Better - use query parameters for optional values
'/item/:id?filter=value&sort=asc'
```

---

## ملخص

| العنصر | الوصف |
|--------|-------|
| **الـ Syntax** | `:parameterName` في الـ path |
| **الوصول** | `state.pathParameters['name']` |
| **النوع** | دايماً `String` - لازم تحوله |
| **الـ Scope** | متاح في الـ builder و redirect |

---

[⬅️ الدرس السابق: التوجيه الأساسي](03_basic_routing.md) | [➡️ الدرس التالي: Query Parameters](05_query_parameters.md)
