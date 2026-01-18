# الدرس 1: مقدمة عن GoRouter

## يعني إيه GoRouter؟

مكتبة **GoRouter** هي مكتبة routing رسمية من فريق Flutter، بتوفرلك طريقة declarative للتنقل بين الصفحات في تطبيقك. المكتبة دي مبنية على **Navigator 2.0 API** بس بتخلي استخدامه أسهل بكتير.

> 💡 المكتبة دي من نوع **Flutter Favorite** يعني معتمدة رسمياً من Flutter Team

---

## ليه نستخدم GoRouter بدل Navigator العادي؟

### المشكلة مع Navigator 1.0
دة شكل التنقل الافتراضي فى Flutter عشوائي وطبقات فوق بعض ومفيش مخطط واضح للصفحات وممكن يعمل مشاكل كتير فى التطبيقات المتوسطة والكبيرة 

```dart
Navigator.push(
  context,
  MaterialPageRoute(
    builder: (context) => DetailsScreen(id: '123'),
  ),
);
```

### المميزات اللي بيوفرها GoRouter

| الميزة | الشرح |
|--------|-------|
| **URL-based routing** | كل صفحة ليها URL واضح زي `/users/123` |
| **Deep Linking** | المستخدم يقدر يفتح أي صفحة من link مباشرة |
| **Web Support** | الـ URL بيتغير في المتصفح تلقائياً |
| **Path Parameters** | تقدر تمرر parameters في الـ URL زي `:id` |
| **Query Parameters** | تقدر تستخدم `?sort=asc&filter=active` |
| **Redirection** | تقدر تعمل redirect للمستخدم لو مش مسجل دخول مثلاً |
| **Nested Navigation** | تقدر تعمل Bottom Navigation مع حفظ حالة كل tab |
| **Type-safe routing** | مع GoRouter Builder بتتجنب أخطاء الـ runtime |

---

## مقارنة سريعة

### Navigator 1.0 (الطريقة القديمة)
```dart
// Navigate to a page
Navigator.push(
  context,
  MaterialPageRoute(builder: (context) => DetailsScreen()),
);

// Go back
Navigator.pop(context);

// Problems:
// - No clear URL
// - Deep Linking is difficult
// - Web doesn't work correctly
```

### GoRouter (الطريقة الجديدة)
```dart
// Navigate to a page
context.go('/details/123');

// Or
context.push('/details/123');

// Go back
context.pop();

// Advantages:
// - Clear and understandable URL
// - Deep Linking works automatically
// - Web works perfectly
```

---

## المفاهيم الأساسية

### 1. GoRouter
ده الـ class الرئيسي اللي بتعرف فيه كل الـ routes بتاعت التطبيق.

```dart
final router = GoRouter(
  routes: [
    // Define your routes here
  ],
);
```

### 2. GoRoute
ده الـ class اللي بيمثل route واحد (صفحة واحدة).

```dart
GoRoute(
  path: '/home',
  builder: (context, state) => const HomeScreen(),
)
```

### 3. GoRouterState
ده الـ object اللي بيحتوي على معلومات الـ route الحالي زي الـ parameters.

```dart
GoRoute(
  path: '/user/:id',
  builder: (context, state) {
    final userId = state.pathParameters['id'];
    return UserScreen(id: userId!);
  },
)
```

---

## حالة المكتبة

فريق Flutter بيعتبر المكتبة **feature-complete** يعني:
- كل الـ features الأساسية موجودة
- التركيز دلوقتي على الـ bug fixes والـ stability
- الـ community contributions مرحب بيها

### آخر إصدار
- **go_router**: `17.0.1`
- **go_router_builder**: `4.1.3`
- **Minimum Flutter**: `3.32`
- **Minimum Dart**: `3.9`

---

## متى تستخدم GoRouter؟

### استخدم GoRouter لو:
- ✅ بتعمل تطبيق Web أو تطبيق هيشتغل على الويب
- ✅ محتاج Deep Linking
- ✅ عندك authentication flow معقد
- ✅ محتاج Bottom Navigation مع حفظ الحالة
- ✅ عايز URL واضح لكل صفحة
- ✅ المشروع متوسط أو كبير

### ممكن تستخدم Navigator العادي لو:
- التطبيق بسيط جداً (2-3 صفحات)
- مش محتاج Deep Linking
- مش هتنشر على الويب

---

## الخطوة الجاية

في الدرس الجاي هنتعلم إزاي نثبت المكتبة ونعمل الإعداد الأولي.

---

## ملخص

| النقطة | الشرح |
|--------|-------|
| GoRouter | مكتبة routing رسمية من Flutter |
| الميزة الرئيسية | URL-based routing + Deep Linking |
| الـ API | سهل وبسيط (`go`, `push`, `pop`) |
| الويب | مدعوم بالكامل |
| الحالة | Feature-complete ومستقرة |

---

[➡️ الدرس التالي: التثبيت والإعداد](02_installation.md)
