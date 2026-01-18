# الدرس 3: التوجيه الأساسي

## طرق التنقل في GoRouter

GoRouter بيوفرلك كذا طريقة للتنقل بين الصفحات، كل طريقة ليها استخدامها:

| الـ Method | الوظيفة |
|-----------|---------|
| `go()` | الانتقال لصفحة مع مسح الـ stack |
| `push()` | إضافة صفحة فوق الـ stack الحالي |
| `pop()` | الرجوع للصفحة السابقة |
| `replace()` | استبدال الصفحة الحالية |
| `pushReplacement()` | push مع استبدال الصفحة الحالية |

---

## 1. context.go()

### ماذا يفعل؟
بينقلك للصفحة المطلوبة وبيمسح الـ navigation stack القديم.

### متى تستخدمه؟
- لما تعمل navigation رئيسي (من tab لـ tab مثلاً)
- بعد تسجيل الدخول (مش عايز المستخدم يرجع لصفحة Login)
- Navigation يعتمد على الـ URL

```dart
// الانتقال للرئيسية
context.go('/');

// الانتقال لصفحة التفاصيل
context.go('/details');

// مع parameters
context.go('/user/123');

// مع query parameters
context.go('/search?query=flutter&sort=recent');
```

### مثال عملي

```dart
class HomeScreen extends StatelessWidget {
  const HomeScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('الرئيسية')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            ElevatedButton(
              onPressed: () => context.go('/profile'),
              child: const Text('الملف الشخصي'),
            ),
            const SizedBox(height: 16),
            ElevatedButton(
              onPressed: () => context.go('/settings'),
              child: const Text('الإعدادات'),
            ),
          ],
        ),
      ),
    );
  }
}
```

---

## 2. context.push()

### ماذا يفعل؟
بيضيف صفحة جديدة فوق الصفحة الحالية (زي الـ stack).

### متى تستخدمه؟
- لما تفتح صفحة تفاصيل
- لما تفتح popup أو modal
- لما تحتاج المستخدم يقدر يرجع بالـ back button

```dart
// إضافة صفحة للـ stack
context.push('/details');

// مع parameters
context.push('/product/456');
```

### الفرق بين go() و push()

```dart
// السيناريو: أنت في Home وعايز تروح Details

// باستخدام go():
context.go('/details');
// الـ Stack: [Details]
// لو ضغطت back -> هتخرج من التطبيق!

// باستخدام push():
context.push('/details');
// الـ Stack: [Home, Details]
// لو ضغطت back -> هترجع لـ Home ✅
```

### مثال عملي

```dart
class ProductListScreen extends StatelessWidget {
  final List<Product> products;

  const ProductListScreen({super.key, required this.products});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('المنتجات')),
      body: ListView.builder(
        itemCount: products.length,
        itemBuilder: (context, index) {
          final product = products[index];
          return ListTile(
            title: Text(product.name),
            subtitle: Text('${product.price} ج.م'),
            onTap: () {
              // push عشان المستخدم يقدر يرجع للقائمة
              context.push('/product/${product.id}');
            },
          );
        },
      ),
    );
  }
}
```

---

## 3. context.pop()

### ماذا يفعل؟
بيرجع للصفحة السابقة في الـ stack.

```dart
// الرجوع للخلف
context.pop();

// الرجوع مع إرجاع قيمة
context.pop(true);  // مثلاً: هل تم الحفظ؟
context.pop({'status': 'saved', 'id': 123});
```

### التحقق من إمكانية الرجوع

```dart
// هل فيه صفحة قبل دي؟
if (context.canPop()) {
  context.pop();
} else {
  // مفيش صفحة قبلها، روح للرئيسية
  context.go('/');
}
```

### استقبال القيمة المرجعة من pop()

```dart
// في الصفحة الأولى
ElevatedButton(
  onPressed: () async {
    // push وانتظر النتيجة
    final result = await context.push<bool>('/confirm');

    if (result == true) {
      // المستخدم أكد العملية
      print('تم التأكيد!');
    }
  },
  child: const Text('تأكيد'),
)

// في صفحة التأكيد
ElevatedButton(
  onPressed: () {
    context.pop(true);  // أكد
  },
  child: const Text('نعم'),
),
ElevatedButton(
  onPressed: () {
    context.pop(false);  // ألغى
  },
  child: const Text('لا'),
),
```

---

## 4. context.replace()

### ماذا يفعل؟
بيستبدل الصفحة الحالية بصفحة جديدة بدون ما يضيفها للـ stack.

```dart
// استبدال الصفحة الحالية
context.replace('/new-page');
```

### متى تستخدمه؟
- بعد صفحة Splash
- بعد صفحة Onboarding
- أي صفحة مش عايز المستخدم يرجعلها

```dart
class SplashScreen extends StatefulWidget {
  @override
  _SplashScreenState createState() => _SplashScreenState();
}

class _SplashScreenState extends State<SplashScreen> {
  @override
  void initState() {
    super.initState();
    _checkAuth();
  }

  Future<void> _checkAuth() async {
    await Future.delayed(const Duration(seconds: 2));

    if (mounted) {
      // استبدال الـ Splash بالـ Home أو Login
      if (isLoggedIn) {
        context.replace('/home');
      } else {
        context.replace('/login');
      }
    }
  }

  @override
  Widget build(BuildContext context) {
    return const Scaffold(
      body: Center(child: CircularProgressIndicator()),
    );
  }
}
```

---

## 5. context.pushReplacement()

### ماذا يفعل؟
بيعمل push لصفحة جديدة وفي نفس الوقت بيمسح الصفحة الحالية.

```dart
context.pushReplacement('/new-page');
```

### الفرق عن replace()

```dart
// Stack قبل: [A, B, C]

// باستخدام replace():
context.replace('/D');
// Stack بعد: [A, B, D]

// باستخدام pushReplacement():
context.pushReplacement('/D');
// Stack بعد: [A, B, D]
// نفس النتيجة لكن مع transition animation
```

---

## الوصول للـ Router من أي مكان

### طريقة 1: باستخدام context

```dart
// كل الـ methods دي بتحتاج context
context.go('/path');
context.push('/path');
context.pop();
```

### طريقة 2: باستخدام GoRouter.of()

```dart
// الحصول على instance من الـ router
final router = GoRouter.of(context);

router.go('/path');
router.push('/path');
router.pop();
```

### طريقة 3: باستخدام ref من الـ Router مباشرة

```dart
// لو عندك الـ router كـ global variable
appRouter.go('/path');
appRouter.push('/path');
```

---

## معلومات عن الـ Location الحالي

```dart
// الحصول على الـ location الحالي
final currentLocation = GoRouterState.of(context).uri.toString();
print(currentLocation);  // /products/123?sort=price

// أو
final router = GoRouter.of(context);
print(router.state.uri);
```

---

## مثال شامل

```dart
// تعريف الـ Router
final appRouter = GoRouter(
  initialLocation: '/',
  routes: [
    GoRoute(
      path: '/',
      builder: (context, state) => const HomeScreen(),
    ),
    GoRoute(
      path: '/products',
      builder: (context, state) => const ProductsScreen(),
    ),
    GoRoute(
      path: '/product/:id',
      builder: (context, state) {
        final id = state.pathParameters['id']!;
        return ProductDetailsScreen(productId: id);
      },
    ),
    GoRoute(
      path: '/cart',
      builder: (context, state) => const CartScreen(),
    ),
    GoRoute(
      path: '/checkout',
      builder: (context, state) => const CheckoutScreen(),
    ),
  ],
);

// الـ Home Screen
class HomeScreen extends StatelessWidget {
  const HomeScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('المتجر')),
      body: Column(
        children: [
          // go - للانتقال الرئيسي
          ListTile(
            title: const Text('المنتجات'),
            onTap: () => context.go('/products'),
          ),
          // go - للانتقال الرئيسي
          ListTile(
            title: const Text('السلة'),
            onTap: () => context.go('/cart'),
          ),
        ],
      ),
    );
  }
}

// Products Screen
class ProductsScreen extends StatelessWidget {
  const ProductsScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('المنتجات'),
        leading: IconButton(
          icon: const Icon(Icons.arrow_back),
          onPressed: () {
            // التحقق قبل الرجوع
            if (context.canPop()) {
              context.pop();
            } else {
              context.go('/');
            }
          },
        ),
      ),
      body: ListView.builder(
        itemCount: 10,
        itemBuilder: (context, index) {
          return ListTile(
            title: Text('منتج $index'),
            onTap: () {
              // push - عشان المستخدم يقدر يرجع للقائمة
              context.push('/product/$index');
            },
          );
        },
      ),
    );
  }
}

// Checkout Flow
class CartScreen extends StatelessWidget {
  const CartScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('السلة')),
      body: Center(
        child: ElevatedButton(
          onPressed: () {
            // pushReplacement - بعد الـ checkout مش عايز يرجع للسلة
            context.pushReplacement('/checkout');
          },
          child: const Text('إتمام الشراء'),
        ),
      ),
    );
  }
}
```

---

## ملخص

| الـ Method | الاستخدام | الـ Stack |
|-----------|-----------|-----------|
| `go()` | Navigation رئيسي | يمسح الـ stack |
| `push()` | فتح صفحة تفاصيل | يضيف للـ stack |
| `pop()` | الرجوع للخلف | يشيل من الـ stack |
| `replace()` | استبدال (Splash) | يستبدل آخر عنصر |
| `pushReplacement()` | استبدال مع animation | يستبدل آخر عنصر |

---

[⬅️ الدرس السابق: التثبيت](02_installation.md) | [➡️ الدرس التالي: Path Parameters](04_path_parameters.md)
