# 🚀 Flutter Widgets Master Guide
### A Complete Reference for Beginner to Advanced Flutter Developers

> **Google Developer Expert–Level | Production-Ready | Interview Preparation | Clean Architecture**

---

```
┌─────────────────────────────────────────────────────────────────┐
│           FLUTTER WIDGETS MASTER GUIDE v3.x                     │
│   Complete Handbook · Course Material · Interview Prep          │
│   Null Safety · Material 3 · Clean Architecture · Responsive    │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📋 Table of Contents

| # | Section | Topics |
|---|---------|--------|
| 0 | [Flutter Fundamentals](#0-flutter-fundamentals) | Widget tree, Rendering, Lifecycle |
| 1 | [Basic Structure Widgets](#1-basic-structure-widgets) | MaterialApp, Scaffold, AppBar… |
| 2 | [Layout Widgets](#2-layout-widgets) | Row, Column, Stack, Flex… |
| 3 | [Text & Styling Widgets](#3-text--styling-widgets) | Text, RichText, Icon, Image… |
| 4 | [Input Widgets](#4-input-widgets) | TextField, Form, Checkbox… |
| 5 | [Button Widgets](#5-button-widgets) | ElevatedButton, FAB, ToggleButtons… |
| 6 | [List & Grid Widgets](#6-list--grid-widgets) | ListView, GridView, DataTable… |
| 7 | [Scroll Widgets](#7-scroll-widgets) | CustomScrollView, Slivers… |
| 8 | [Async & State Widgets](#8-async--state-widgets) | FutureBuilder, StreamBuilder… |
| 9 | [Animation Widgets](#9-animation-widgets) | Hero, AnimatedContainer… |
| 10 | [Navigation Widgets](#10-navigation-widgets) | Navigator, Router, GoRouter… |
| 11 | [Material Design Widgets](#11-material-design-widgets) | Card, Dialog, SnackBar… |
| 12 | [Cupertino Widgets](#12-cupertino-widgets) | CupertinoButton, Picker… |
| 13 | [Advanced Rendering](#13-advanced-rendering-widgets) | CustomPaint, ShaderMask… |
| 14 | [Accessibility Widgets](#14-accessibility-widgets) | Semantics, MergeSemantics… |
| 15 | [Sliver Widgets](#15-sliver-widgets) | SliverAppBar, SliverList… |
| 16 | [State Management](#16-state-management-widgets) | Provider, Bloc, GetX… |
| 17 | [Interview Preparation](#17-interview-preparation) | 100+ Q&A |
| 18 | [Clean Architecture](#18-clean-architecture--project-structure) | Folder structure, DDD |
| 19 | [Performance Guide](#19-flutter-performance-optimization) | Rebuild tips, Profiling |

---

# 0. Flutter Fundamentals

## 🌳 The Widget Tree — Everything is a Widget

Flutter's entire UI is composed of **widgets** — lightweight Dart objects that describe how the UI should look. They are **immutable descriptions** that Flutter uses to build the actual rendered interface.

```
                    ┌──────────────────┐
                    │   Widget Tree    │  ← What YOU write (Dart code)
                    │  (Immutable)     │
                    └────────┬─────────┘
                             │ Flutter builds
                    ┌────────▼─────────┐
                    │  Element Tree    │  ← Manages widget lifecycle
                    │  (Mutable)       │    Links widgets to render objects
                    └────────┬─────────┘
                             │ Flutter uses
                    ┌────────▼─────────┐
                    │ RenderObject Tree│  ← Performs layout & painting
                    │  (Mutable)       │
                    └──────────────────┘
```

### Key Rule: Widgets are Cheap

Widgets are **not** the actual rendered UI. They are blueprints. Flutter recreates widget objects on every `build()` call, but the heavy work (layout, paint) only happens when truly necessary.

---

## 🔄 StatelessWidget vs StatefulWidget — Complete Guide

### StatelessWidget

A widget that **never changes** after being built. Ideal for static UI.

```dart
// ✅ Correct StatelessWidget usage
class UserProfileCard extends StatelessWidget {
  const UserProfileCard({
    super.key,
    required this.name,
    required this.avatar,
    required this.role,
  });

  final String name;
  final String avatar;
  final String role;

  @override
  Widget build(BuildContext context) {
    return Card(
      elevation: 4,
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Row(
          children: [
            CircleAvatar(backgroundImage: NetworkImage(avatar), radius: 28),
            const SizedBox(width: 12),
            Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Text(name, style: Theme.of(context).textTheme.titleMedium),
                Text(role, style: Theme.of(context).textTheme.bodySmall),
              ],
            ),
          ],
        ),
      ),
    );
  }
}
```

### StatefulWidget

A widget that **maintains mutable state** and can rebuild when state changes.

```dart
// ✅ Correct StatefulWidget usage
class QuantitySelector extends StatefulWidget {
  const QuantitySelector({super.key, this.initialValue = 1});
  final int initialValue;

  @override
  State<QuantitySelector> createState() => _QuantitySelectorState();
}

class _QuantitySelectorState extends State<QuantitySelector> {
  late int _quantity;

  @override
  void initState() {
    super.initState();
    _quantity = widget.initialValue; // Access widget properties via `widget`
  }

  void _increment() => setState(() => _quantity++);
  void _decrement() => setState(() {
    if (_quantity > 1) _quantity--;
  });

  @override
  Widget build(BuildContext context) {
    return Row(
      mainAxisSize: MainAxisSize.min,
      children: [
        IconButton(icon: const Icon(Icons.remove), onPressed: _decrement),
        Text('$_quantity', style: Theme.of(context).textTheme.titleLarge),
        IconButton(icon: const Icon(Icons.add), onPressed: _increment),
      ],
    );
  }

  @override
  void dispose() {
    // Always clean up resources here
    super.dispose();
  }
}
```

### Comparison Table

| Feature | StatelessWidget | StatefulWidget |
|---------|----------------|----------------|
| State | No mutable state | Has mutable state via `State` |
| Rebuild trigger | Parent rebuild only | `setState()` or parent |
| Performance | Slightly lighter | Slightly heavier |
| Use case | Static UI, pure display | Forms, counters, animations |
| `initState` | ❌ | ✅ |
| `dispose` | ❌ | ✅ |
| `didUpdateWidget` | ❌ | ✅ |

---

## 🏗️ Widget Lifecycle

```
StatefulWidget Lifecycle:
─────────────────────────────────────────────────────────────
createState()          → Called once; creates the State object
     │
initState()            → Called once; init controllers, fetch data
     │
didChangeDependencies()→ Called when InheritedWidget changes
     │
build()                → Called on every rebuild (keep it FAST)
     │
didUpdateWidget()      → Parent rebuilt with new widget config
     │
setState()             → Triggers a rebuild
     │
deactivate()           → Widget removed from tree temporarily
     │
dispose()              → Widget permanently removed; CLEAN UP HERE
─────────────────────────────────────────────────────────────
```

```dart
class LifecycleDemo extends StatefulWidget {
  const LifecycleDemo({super.key});
  @override
  State<LifecycleDemo> createState() => _LifecycleDemoState();
}

class _LifecycleDemoState extends State<LifecycleDemo> {
  late final TextEditingController _controller;
  late final ScrollController _scrollController;

  @override
  void initState() {
    super.initState();
    // ✅ Initialize controllers here
    _controller = TextEditingController();
    _scrollController = ScrollController();
    // ✅ Start async operations
    _loadInitialData();
  }

  Future<void> _loadInitialData() async {
    // fetch from API, etc.
  }

  @override
  void didUpdateWidget(covariant LifecycleDemo oldWidget) {
    super.didUpdateWidget(oldWidget);
    // React to widget property changes
  }

  @override
  void dispose() {
    // ✅ ALWAYS dispose controllers to prevent memory leaks
    _controller.dispose();
    _scrollController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return const Placeholder();
  }
}
```

---

## 🔦 BuildContext — Deep Explanation

`BuildContext` is a **handle to the location of a widget in the widget tree**. It provides access to:

- Theme data (`Theme.of(context)`)
- Media query (`MediaQuery.of(context)`)
- Navigator (`Navigator.of(context)`)
- Inherited widgets (`Provider.of<T>(context)`)

```dart
// ✅ Context from the CORRECT widget
class MyWidget extends StatelessWidget {
  const MyWidget({super.key});

  @override
  Widget build(BuildContext context) {
    // ✅ Safe: context belongs to MyWidget
    final theme = Theme.of(context);
    final size = MediaQuery.sizeOf(context);

    return Builder(
      builder: (innerContext) {
        // ✅ innerContext is a child context — useful for Scaffold.of()
        return ElevatedButton(
          onPressed: () => ScaffoldMessenger.of(innerContext).showSnackBar(
            const SnackBar(content: Text('Hello!')),
          ),
          child: const Text('Show Snackbar'),
        );
      },
    );
  }
}
```

> ⚠️ **Warning:** Never use a `BuildContext` after the widget is disposed — you'll get a `mounted` error. Always check `if (mounted)` before using context in async callbacks.

```dart
// ✅ Safe async context usage
Future<void> _submit() async {
  await someApiCall();
  if (!mounted) return; // Guard against unmounted widget
  Navigator.of(context).pop();
}
```

---

## ⚡ Flutter Rendering Pipeline

```
User Code (Dart)
      │
      ▼
Widget.build() → Widget Tree (your description)
      │
      ▼
Element Tree  (Flutter's internal reconciliation layer)
      │
      ▼
RenderObject  (layout: size/position, paint: pixels)
      │
      ▼
Layer Tree    (composited layers for GPU)
      │
      ▼
Skia / Impeller → GPU → Screen (60/120fps)
```

**Key insight:** Flutter bypasses platform UI entirely — it draws every pixel itself using Skia (legacy) or Impeller (new engine). This is why Flutter looks identical on Android, iOS, web, and desktop.

---

# 1. Basic Structure Widgets

---

## 📱 MaterialApp

### Introduction

`MaterialApp` is the **root widget** for any Material Design Flutter application. It sets up:
- Navigation/routing system
- Theme (colors, typography, shapes)
- Localization
- Overlay entries (dialogs, snackbars)
- Scroll behavior

### Syntax

```dart
MaterialApp(
  title: 'App Name',
  theme: ThemeData(...),
  darkTheme: ThemeData.dark(...),
  themeMode: ThemeMode.system,
  home: const HomeScreen(),
  routes: {'/settings': (_) => const SettingsScreen()},
  onGenerateRoute: (settings) => ...,
  debugShowCheckedModeBanner: false,
  locale: const Locale('en', 'US'),
  supportedLocales: const [Locale('en'), Locale('ar')],
)
```

### Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `title` | `String` | App name shown in task switcher |
| `theme` | `ThemeData?` | Light theme configuration |
| `darkTheme` | `ThemeData?` | Dark theme configuration |
| `themeMode` | `ThemeMode` | Light / Dark / System |
| `home` | `Widget?` | Default route widget |
| `routes` | `Map<String, WidgetBuilder>?` | Named route table |
| `onGenerateRoute` | `RouteFactory?` | Dynamic route generation |
| `navigatorKey` | `GlobalKey<NavigatorState>?` | Access navigator globally |
| `debugShowCheckedModeBanner` | `bool` | Show/hide debug banner |
| `locale` | `Locale?` | Override device locale |
| `localizationsDelegates` | `Iterable<...>?` | Localization setup |

### Production Example — Material 3 App Setup

```dart
// main.dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'ShopEase',
      debugShowCheckedModeBanner: false,

      // ✅ Material 3 with dynamic color seed
      theme: ThemeData(
        useMaterial3: true,
        colorSchemeSeed: const Color(0xFF6750A4),
        brightness: Brightness.light,
        appBarTheme: const AppBarTheme(centerTitle: true, elevation: 0),
        cardTheme: CardTheme(
          elevation: 0,
          shape: RoundedRectangleBorder(
            borderRadius: BorderRadius.circular(16),
          ),
        ),
        elevatedButtonTheme: ElevatedButtonThemeData(
          style: ElevatedButton.styleFrom(
            minimumSize: const Size(double.infinity, 52),
            shape: RoundedRectangleBorder(
              borderRadius: BorderRadius.circular(12),
            ),
          ),
        ),
      ),

      darkTheme: ThemeData(
        useMaterial3: true,
        colorSchemeSeed: const Color(0xFF6750A4),
        brightness: Brightness.dark,
      ),

      themeMode: ThemeMode.system,
      home: const HomeScreen(),
    );
  }
}
```

### Best Practices

- ✅ Always set `useMaterial3: true` for modern apps (Flutter 3.x)
- ✅ Use `colorSchemeSeed` for automatic color scheme generation
- ✅ Set `debugShowCheckedModeBanner: false` before release
- ✅ Prefer `GoRouter` or `auto_route` for complex navigation over manual routes
- ✅ Use `themeMode: ThemeMode.system` for automatic dark mode
- ❌ Don't put business logic inside `MaterialApp` — keep it as a pure configuration widget

### Interview Questions

**Q1: What is the difference between `home` and `initialRoute`?**
> `home` directly sets the widget for the root route (`/`). `initialRoute` specifies the named route string to navigate to first. You cannot use both simultaneously — Flutter throws an assertion error.

**Q2: When would you use `navigatorKey`?**
> When you need to navigate from outside the widget tree — for example, from a service class, a notification handler, or a BLoC without access to `BuildContext`. You'd access it as `navigatorKey.currentState?.push(...)`.

**Q3: What does `onGenerateRoute` do and when is it needed?**
> It's a fallback handler for any route not found in the `routes` map. It's useful for parameterized routes (e.g., `/product/42`) where you extract arguments from `RouteSettings`.

**Q4: How does `ThemeMode.system` work internally?**
> Flutter reads the platform's brightness via `MediaQuery.platformBrightnessOf(context)` or `WidgetsBinding.instance.platformDispatcher.platformBrightness` and switches between `theme` and `darkTheme` accordingly.

**Q5: What is the role of `localizationsDelegates`?**
> They are factories that produce localized resources for the app. You must include `GlobalMaterialLocalizations.delegate`, `GlobalWidgetsLocalizations.delegate`, and your app's own delegate for full i18n support.

---

## 🏛️ Scaffold

### Introduction

`Scaffold` provides the **basic Material Design visual structure** — it's the skeleton every screen in a Material app is built on. It manages slots for AppBar, body, FAB, drawer, bottom navigation, snack bars, and more.

### Syntax

```dart
Scaffold(
  appBar: AppBar(title: const Text('Title')),
  body: const Center(child: Text('Content')),
  floatingActionButton: FloatingActionButton(
    onPressed: () {},
    child: const Icon(Icons.add),
  ),
  floatingActionButtonLocation: FloatingActionButtonLocation.centerDocked,
  bottomNavigationBar: BottomNavigationBar(...),
  drawer: const Drawer(...),
  endDrawer: const Drawer(...),
  backgroundColor: Colors.white,
  resizeToAvoidBottomInset: true,
  extendBodyBehindAppBar: false,
)
```

### Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `appBar` | `PreferredSizeWidget?` | Top app bar |
| `body` | `Widget?` | Main content area |
| `floatingActionButton` | `Widget?` | FAB widget |
| `floatingActionButtonLocation` | `FloatingActionButtonLocation?` | FAB position |
| `bottomNavigationBar` | `Widget?` | Bottom nav |
| `drawer` | `Widget?` | Left-side drawer |
| `endDrawer` | `Widget?` | Right-side drawer |
| `backgroundColor` | `Color?` | Background color |
| `resizeToAvoidBottomInset` | `bool` | Resize when keyboard appears |
| `extendBodyBehindAppBar` | `bool` | Body extends under AppBar |
| `extendBody` | `bool` | Body extends under bottom nav |

### Production Example — E-Commerce Home Screen

```dart
class HomeScreen extends StatefulWidget {
  const HomeScreen({super.key});
  @override
  State<HomeScreen> createState() => _HomeScreenState();
}

class _HomeScreenState extends State<HomeScreen> {
  int _selectedIndex = 0;

  final List<Widget> _pages = const [
    ProductListPage(),
    CartPage(),
    WishlistPage(),
    ProfilePage(),
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      extendBody: true, // Body flows under the bottom nav for edge-to-edge look

      appBar: AppBar(
        title: const Text('ShopEase'),
        actions: [
          IconButton(
            icon: const Icon(Icons.search),
            onPressed: () => showSearch(context: context, delegate: ProductSearchDelegate()),
          ),
          IconButton(
            icon: const Badge(label: Text('3'), child: Icon(Icons.notifications_outlined)),
            onPressed: () {},
          ),
        ],
      ),

      drawer: const AppDrawer(),

      body: IndexedStack(
        index: _selectedIndex,
        children: _pages,
      ),

      bottomNavigationBar: NavigationBar(
        selectedIndex: _selectedIndex,
        onDestinationSelected: (i) => setState(() => _selectedIndex = i),
        destinations: const [
          NavigationDestination(icon: Icon(Icons.home_outlined), selectedIcon: Icon(Icons.home), label: 'Home'),
          NavigationDestination(icon: Icon(Icons.shopping_cart_outlined), selectedIcon: Icon(Icons.shopping_cart), label: 'Cart'),
          NavigationDestination(icon: Icon(Icons.favorite_outline), selectedIcon: Icon(Icons.favorite), label: 'Wishlist'),
          NavigationDestination(icon: Icon(Icons.person_outline), selectedIcon: Icon(Icons.person), label: 'Profile'),
        ],
      ),

      floatingActionButton: _selectedIndex == 0
          ? FloatingActionButton.extended(
              onPressed: () {},
              icon: const Icon(Icons.filter_list),
              label: const Text('Filter'),
            )
          : null,
    );
  }
}
```

### Common Mistakes

- ❌ **Nesting Scaffolds** — Never put a `Scaffold` inside another `Scaffold`. Each screen should have exactly one.
- ❌ **Using `Scaffold.of(context)` with wrong context** — Use `Builder` or `ScaffoldMessenger.of(context)` to access the scaffold from within the body.
- ❌ **Forgetting `resizeToAvoidBottomInset: false`** on screens with custom keyboard handling (e.g., full-screen maps).

### Performance Considerations

- Use `IndexedStack` (not `PageView`) for bottom nav if you want to preserve page state between tab switches.
- `extendBody: true` paired with a transparent bottom nav creates an immersive edge-to-edge experience without extra padding hacks.

---

## 📊 AppBar

### Introduction

`AppBar` is the Material Design top navigation component. In Material 3, it supports four sizes: `AppBar` (small), `MediumAppBar`, `LargeAppBar`, and `SliverAppBar`.

### Syntax

```dart
AppBar(
  leading: const BackButton(),
  automaticallyImplyLeading: true,
  title: const Text('Screen Title'),
  centerTitle: true,
  actions: [
    IconButton(icon: const Icon(Icons.share), onPressed: () {}),
  ],
  bottom: const TabBar(tabs: [Tab(text: 'Tab 1'), Tab(text: 'Tab 2')]),
  elevation: 0,
  scrolledUnderElevation: 4,
  backgroundColor: Colors.transparent,
  foregroundColor: Colors.black,
  flexibleSpace: ...,
  toolbarHeight: kToolbarHeight, // 56.0
  shape: const RoundedRectangleBorder(
    borderRadius: BorderRadius.vertical(bottom: Radius.circular(16)),
  ),
)
```

### Production Example — Product Detail AppBar

```dart
class ProductDetailScreen extends StatelessWidget {
  const ProductDetailScreen({super.key, required this.product});
  final Product product;

  @override
  Widget build(BuildContext context) {
    final colorScheme = Theme.of(context).colorScheme;

    return Scaffold(
      body: CustomScrollView(
        slivers: [
          SliverAppBar.large(
            expandedHeight: 300,
            pinned: true,
            stretch: true,
            flexibleSpace: FlexibleSpaceBar(
              title: Text(
                product.name,
                style: const TextStyle(fontWeight: FontWeight.bold),
              ),
              background: Hero(
                tag: 'product_${product.id}',
                child: Image.network(
                  product.imageUrl,
                  fit: BoxFit.cover,
                ),
              ),
              stretchModes: const [
                StretchMode.zoomBackground,
                StretchMode.blurBackground,
              ],
            ),
            actions: [
              IconButton(
                icon: const Icon(Icons.favorite_border),
                onPressed: () {},
              ),
              IconButton(
                icon: const Icon(Icons.share),
                onPressed: () {},
              ),
            ],
          ),
          // ... rest of body
        ],
      ),
    );
  }
}
```

### Interview Questions

**Q1: What is the difference between `AppBar` and `SliverAppBar`?**
> `AppBar` is a fixed-height bar always visible at the top. `SliverAppBar` is part of a `CustomScrollView` and can expand, collapse, float, or pin as the user scrolls.

**Q2: What is `scrolledUnderElevation`?**
> It's a Material 3 property that adds a surface tint/shadow to the AppBar when content scrolls underneath it. Set to `0` for a flat look.

**Q3: How do you add a `TabBar` to an `AppBar`?**
> Use the `bottom` property of `AppBar`, which accepts a `PreferredSizeWidget`. `TabBar` implements this interface.

---

## 🔲 SafeArea

### Introduction

`SafeArea` insets its child to avoid OS intrusions: the status bar, notch, home indicator on iPhone, and navigation bar on Android. **Always wrap screen body content in `SafeArea`** when not using `AppBar`.

### Syntax

```dart
SafeArea(
  top: true,       // Avoid top status bar / notch
  bottom: true,    // Avoid bottom home indicator
  left: true,
  right: true,
  minimum: const EdgeInsets.all(8),
  child: YourWidget(),
)
```

### Production Example

```dart
// ✅ Full-screen custom screen without AppBar
class OnboardingScreen extends StatelessWidget {
  const OnboardingScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: SafeArea(
        child: Padding(
          padding: const EdgeInsets.symmetric(horizontal: 24),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              const Spacer(),
              Image.asset('assets/onboard_1.png'),
              const SizedBox(height: 32),
              Text(
                'Discover\nAmazing Products',
                style: Theme.of(context).textTheme.displaySmall?.copyWith(
                  fontWeight: FontWeight.bold,
                ),
              ),
              const SizedBox(height: 16),
              Text(
                'Shop from thousands of brands with fast delivery to your door.',
                style: Theme.of(context).textTheme.bodyLarge,
              ),
              const Spacer(),
              ElevatedButton(
                onPressed: () {},
                child: const Text('Get Started'),
              ),
              const SizedBox(height: 24),
            ],
          ),
        ),
      ),
    );
  }
}
```

### Responsive Design Tips

Use `MediaQuery.paddingOf(context)` to read safe area insets manually when you need more control — e.g., on screens with custom bottom sheets that should respect the home indicator.

---

## 🗂️ Drawer

### Introduction

`Drawer` slides in from the left edge (or right for `endDrawer`) to display navigation options. In Material 3 it becomes a `NavigationDrawer` with built-in selection support.

### Production Example — Navigation Drawer

```dart
class AppDrawer extends StatelessWidget {
  const AppDrawer({super.key});

  @override
  Widget build(BuildContext context) {
    final user = context.read<AuthProvider>().currentUser;

    return NavigationDrawer(
      onDestinationSelected: (index) {
        Navigator.pop(context); // Close drawer
        switch (index) {
          case 0: context.go('/home');
          case 1: context.go('/orders');
          case 2: context.go('/profile');
          case 3: context.go('/settings');
        }
      },
      children: [
        // Header
        Padding(
          padding: const EdgeInsets.fromLTRB(28, 16, 16, 10),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              CircleAvatar(
                radius: 32,
                backgroundImage: NetworkImage(user?.avatarUrl ?? ''),
              ),
              const SizedBox(height: 8),
              Text(user?.name ?? 'Guest', style: Theme.of(context).textTheme.titleMedium),
              Text(user?.email ?? '', style: Theme.of(context).textTheme.bodySmall),
              const SizedBox(height: 8),
            ],
          ),
        ),
        const Divider(),
        const NavigationDrawerDestination(icon: Icon(Icons.home_outlined), selectedIcon: Icon(Icons.home), label: Text('Home')),
        const NavigationDrawerDestination(icon: Icon(Icons.receipt_long_outlined), selectedIcon: Icon(Icons.receipt_long), label: Text('Orders')),
        const NavigationDrawerDestination(icon: Icon(Icons.person_outline), selectedIcon: Icon(Icons.person), label: Text('Profile')),
        const NavigationDrawerDestination(icon: Icon(Icons.settings_outlined), selectedIcon: Icon(Icons.settings), label: Text('Settings')),
        const Divider(),
        ListTile(
          leading: const Icon(Icons.logout),
          title: const Text('Sign Out'),
          onTap: () => context.read<AuthProvider>().signOut(),
        ),
      ],
    );
  }
}
```

---

# 2. Layout Widgets

> 💡 **Key Mental Model:** Layout in Flutter is a **top-down constraint, bottom-up sizing** system.
> - Parent tells child: *"You can be at most this big"* (constraints flow down)
> - Child tells parent: *"I will be this big"* (sizes flow up)
> - Parent positions child based on its own logic

---

## ↔️ Row

### Introduction

`Row` arranges its children **horizontally** in a single line. It applies loose height constraints and tight width constraints by default.

### Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `mainAxisAlignment` | `MainAxisAlignment` | Horizontal distribution |
| `crossAxisAlignment` | `CrossAxisAlignment` | Vertical alignment |
| `mainAxisSize` | `MainAxisSize` | `max` (fill) or `min` (shrink) |
| `children` | `List<Widget>` | Child widgets |
| `textDirection` | `TextDirection?` | LTR or RTL |

### MainAxisAlignment Visual Guide

```
MainAxisAlignment.start:     [A][B][C]____________
MainAxisAlignment.end:       ____________[A][B][C]
MainAxisAlignment.center:    _____[A][B][C]_______
MainAxisAlignment.spaceBetween: [A]_____[B]_____[C]
MainAxisAlignment.spaceAround:  _[A]___[B]___[C]_
MainAxisAlignment.spaceEvenly:  __[A]__[B]__[C]__
```

### Production Example — Product Price Row

```dart
class ProductPriceRow extends StatelessWidget {
  const ProductPriceRow({
    super.key,
    required this.price,
    required this.originalPrice,
    required this.discount,
  });

  final double price;
  final double originalPrice;
  final int discount;

  @override
  Widget build(BuildContext context) {
    final theme = Theme.of(context);

    return Row(
      crossAxisAlignment: CrossAxisAlignment.baseline,
      textBaseline: TextBaseline.alphabetic,
      children: [
        Text(
          '₹${price.toStringAsFixed(0)}',
          style: theme.textTheme.headlineSmall?.copyWith(
            fontWeight: FontWeight.bold,
            color: theme.colorScheme.primary,
          ),
        ),
        const SizedBox(width: 8),
        Text(
          '₹${originalPrice.toStringAsFixed(0)}',
          style: theme.textTheme.bodyMedium?.copyWith(
            decoration: TextDecoration.lineThrough,
            color: theme.colorScheme.onSurface.withOpacity(0.5),
          ),
        ),
        const SizedBox(width: 8),
        Container(
          padding: const EdgeInsets.symmetric(horizontal: 6, vertical: 2),
          decoration: BoxDecoration(
            color: Colors.green.shade100,
            borderRadius: BorderRadius.circular(4),
          ),
          child: Text(
            '$discount% OFF',
            style: theme.textTheme.labelSmall?.copyWith(
              color: Colors.green.shade700,
              fontWeight: FontWeight.bold,
            ),
          ),
        ),
        const Spacer(), // Push next item to the right
        const Icon(Icons.star, size: 16, color: Colors.amber),
        Text('4.5 (2.1k)', style: theme.textTheme.bodySmall),
      ],
    );
  }
}
```

### Common Mistakes

- ❌ **Overflow error** — Don't put a `Column` or full-width widget inside a `Row` without wrapping in `Expanded` or `Flexible`.
- ❌ **Using `Row` for long lists** — Use `ListView` with `scrollDirection: Axis.horizontal` instead.
- ❌ **Forgetting `textBaseline`** when using `CrossAxisAlignment.baseline` — You must specify `textBaseline` or you'll get an assertion error.

---

## ↕️ Column

### Introduction

`Column` arranges children **vertically**. It's the vertical counterpart to `Row`. The same alignment properties apply, but on opposite axes.

### Production Example — Login Screen Layout

```dart
class LoginScreen extends StatelessWidget {
  const LoginScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: SafeArea(
        child: SingleChildScrollView( // ✅ Prevents overflow when keyboard appears
          padding: const EdgeInsets.all(24),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.stretch, // ✅ Stretch children to full width
            children: [
              const SizedBox(height: 40),
              // Logo
              Center(
                child: Image.asset('assets/logo.png', height: 80),
              ),
              const SizedBox(height: 32),
              // Title
              Text(
                'Welcome Back!',
                style: Theme.of(context).textTheme.headlineMedium?.copyWith(
                  fontWeight: FontWeight.bold,
                ),
              ),
              const SizedBox(height: 8),
              Text(
                'Sign in to continue shopping',
                style: Theme.of(context).textTheme.bodyLarge?.copyWith(
                  color: Colors.grey,
                ),
              ),
              const SizedBox(height: 32),
              // Form fields
              const EmailTextField(),
              const SizedBox(height: 16),
              const PasswordTextField(),
              const SizedBox(height: 8),
              // Forgot password
              Align(
                alignment: Alignment.centerRight,
                child: TextButton(
                  onPressed: () {},
                  child: const Text('Forgot Password?'),
                ),
              ),
              const SizedBox(height: 24),
              ElevatedButton(
                onPressed: () {},
                child: const Text('Sign In'),
              ),
              const SizedBox(height: 24),
              // Social login
              const _SocialLoginRow(),
              const SizedBox(height: 24),
              // Register link
              Row(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  const Text("Don't have an account? "),
                  TextButton(
                    onPressed: () {},
                    child: const Text('Sign Up'),
                  ),
                ],
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

---

## 🃏 Stack

### Introduction

`Stack` layers its children **on top of each other** — like CSS absolute positioning. Children are positioned relative to the Stack's box, using `Positioned` widgets or alignment.

### Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `alignment` | `AlignmentGeometry` | Default alignment for non-positioned children |
| `fit` | `StackFit` | `loose`, `expand`, `passthrough` |
| `clipBehavior` | `Clip` | Whether to clip overflowing children |
| `children` | `List<Widget>` | Bottom-to-top order |

### Production Example — Product Card with Badge

```dart
class ProductCard extends StatelessWidget {
  const ProductCard({super.key, required this.product});
  final Product product;

  @override
  Widget build(BuildContext context) {
    return Card(
      clipBehavior: Clip.antiAlias,
      shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(16)),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          // Image with overlaid badge
          Stack(
            children: [
              // Background image
              AspectRatio(
                aspectRatio: 1,
                child: Image.network(
                  product.imageUrl,
                  fit: BoxFit.cover,
                ),
              ),
              // Discount badge (top-left)
              Positioned(
                top: 8,
                left: 8,
                child: Container(
                  padding: const EdgeInsets.symmetric(horizontal: 8, vertical: 4),
                  decoration: BoxDecoration(
                    color: Colors.red,
                    borderRadius: BorderRadius.circular(8),
                  ),
                  child: Text(
                    '${product.discount}% OFF',
                    style: const TextStyle(
                      color: Colors.white,
                      fontSize: 12,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                ),
              ),
              // Wishlist button (top-right)
              Positioned(
                top: 4,
                right: 4,
                child: IconButton(
                  style: IconButton.styleFrom(
                    backgroundColor: Colors.white.withOpacity(0.9),
                  ),
                  icon: Icon(
                    product.isWishlisted ? Icons.favorite : Icons.favorite_border,
                    color: product.isWishlisted ? Colors.red : Colors.grey,
                    size: 20,
                  ),
                  onPressed: () {},
                ),
              ),
              // Out of stock overlay
              if (!product.inStock)
                Positioned.fill(
                  child: Container(
                    color: Colors.black.withOpacity(0.5),
                    child: const Center(
                      child: Text(
                        'OUT OF STOCK',
                        style: TextStyle(
                          color: Colors.white,
                          fontWeight: FontWeight.bold,
                          letterSpacing: 2,
                        ),
                      ),
                    ),
                  ),
                ),
            ],
          ),
          // Product info
          Padding(
            padding: const EdgeInsets.all(12),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Text(product.name, maxLines: 2, overflow: TextOverflow.ellipsis),
                const SizedBox(height: 4),
                Text('₹${product.price}', style: const TextStyle(fontWeight: FontWeight.bold)),
              ],
            ),
          ),
        ],
      ),
    );
  }
}
```

---

## 📐 Expanded & Flexible

### Introduction

Both `Expanded` and `Flexible` are used inside `Row`, `Column`, or `Flex` to distribute available space among children.

| | `Expanded` | `Flexible` |
|---|---|---|
| **Space behavior** | Forces child to fill allocated space | Child CAN be smaller than allocated space |
| `FlexFit` | Always `FlexFit.tight` | Default `FlexFit.loose` |
| Use case | Equal / proportional fill | Child sizes itself, extra space absorbed |

```dart
// ✅ Expanded — forces equal width
Row(
  children: [
    Expanded(child: ElevatedButton(onPressed: () {}, child: const Text('Cancel'))),
    const SizedBox(width: 12),
    Expanded(child: ElevatedButton(onPressed: () {}, child: const Text('Confirm'))),
  ],
)

// ✅ Flexible with flex — 1:2 ratio
Row(
  children: [
    Flexible(flex: 1, child: Container(color: Colors.blue)),
    Flexible(flex: 2, child: Container(color: Colors.green)),
  ],
)
```

### Production Example — Search Bar Layout

```dart
Row(
  children: [
    Expanded(
      child: TextField(
        decoration: InputDecoration(
          hintText: 'Search products...',
          prefixIcon: const Icon(Icons.search),
          border: OutlineInputBorder(
            borderRadius: BorderRadius.circular(12),
          ),
        ),
      ),
    ),
    const SizedBox(width: 8),
    // Fixed-size filter button — NOT Expanded
    FilledButton.icon(
      onPressed: () {},
      icon: const Icon(Icons.tune),
      label: const Text('Filter'),
    ),
  ],
)
```

---

## 📦 Container

### Introduction

`Container` is one of Flutter's most used widgets. It combines padding, margin, decoration, sizing, and transformation in a single widget. Internally it composes `ColoredBox`, `DecoratedBox`, `ConstrainedBox`, `Transform`, and `Padding`.

### Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `width` | `double?` | Fixed width |
| `height` | `double?` | Fixed height |
| `padding` | `EdgeInsetsGeometry?` | Inner spacing |
| `margin` | `EdgeInsetsGeometry?` | Outer spacing |
| `color` | `Color?` | Background color (cannot use with `decoration`) |
| `decoration` | `Decoration?` | BoxDecoration for borders, gradients, shadows |
| `alignment` | `AlignmentGeometry?` | Align child within container |
| `constraints` | `BoxConstraints?` | Width/height constraints |
| `transform` | `Matrix4?` | Transformation matrix |
| `clipBehavior` | `Clip` | How to clip the content |

### Production Example — Gradient Banner Card

```dart
class PromoBanner extends StatelessWidget {
  const PromoBanner({super.key});

  @override
  Widget build(BuildContext context) {
    return Container(
      width: double.infinity,
      height: 160,
      margin: const EdgeInsets.symmetric(horizontal: 16),
      decoration: BoxDecoration(
        borderRadius: BorderRadius.circular(20),
        gradient: const LinearGradient(
          begin: Alignment.topLeft,
          end: Alignment.bottomRight,
          colors: [Color(0xFF6750A4), Color(0xFFB7A4DF)],
        ),
        boxShadow: [
          BoxShadow(
            color: const Color(0xFF6750A4).withOpacity(0.4),
            blurRadius: 20,
            offset: const Offset(0, 8),
          ),
        ],
      ),
      child: Stack(
        children: [
          // Decorative circle
          Positioned(
            right: -20,
            top: -20,
            child: Container(
              width: 140,
              height: 140,
              decoration: BoxDecoration(
                shape: BoxShape.circle,
                color: Colors.white.withOpacity(0.1),
              ),
            ),
          ),
          // Content
          Padding(
            padding: const EdgeInsets.all(20),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                const Text(
                  'Flash Sale! 🔥',
                  style: TextStyle(
                    color: Colors.white70,
                    fontSize: 12,
                    letterSpacing: 1,
                  ),
                ),
                const SizedBox(height: 4),
                const Text(
                  'Up to 60% Off',
                  style: TextStyle(
                    color: Colors.white,
                    fontSize: 26,
                    fontWeight: FontWeight.bold,
                  ),
                ),
                const SizedBox(height: 12),
                ElevatedButton(
                  style: ElevatedButton.styleFrom(
                    backgroundColor: Colors.white,
                    foregroundColor: const Color(0xFF6750A4),
                    minimumSize: const Size(120, 36),
                  ),
                  onPressed: () {},
                  child: const Text('Shop Now'),
                ),
              ],
            ),
          ),
        ],
      ),
    );
  }
}
```

### Performance Tip

> 💡 If you only need a background color, prefer `ColoredBox` over `Container(color: ...)`. `ColoredBox` is a `LeafRenderObjectWidget` — much cheaper than `Container` which uses a full `DecoratedBox`.

```dart
// ✅ Prefer ColoredBox for simple colors
const ColoredBox(color: Colors.blue, child: SizedBox(width: 50, height: 50))

// vs (heavier)
Container(color: Colors.blue, width: 50, height: 50)
```

---

## 📏 SizedBox

### Introduction

`SizedBox` forces its child to a specific size, or adds empty space in a layout. It's simpler and cheaper than `Container` for pure sizing.

```dart
// Fixed space between widgets (preferred over SizedBox.fromSize or Container)
const SizedBox(height: 16)
const SizedBox(width: 8)

// Force a specific size on a child
SizedBox(
  width: 200,
  height: 48,
  child: ElevatedButton(onPressed: () {}, child: const Text('Buy Now')),
)

// Shrink to nothing
const SizedBox.shrink()  // width: 0, height: 0

// Fill available space
SizedBox.expand(child: Container(color: Colors.blue))
```

> 💡 **Best Practice:** Use `const SizedBox(height: n)` for vertical spacing instead of `Padding`. It's the idiomatic Flutter way.

---

## 📐 LayoutBuilder

### Introduction

`LayoutBuilder` builds its widget tree based on the **parent's constraints**. Essential for responsive layouts.

### Production Example — Responsive Product Grid

```dart
class ResponsiveProductGrid extends StatelessWidget {
  const ResponsiveProductGrid({super.key, required this.products});
  final List<Product> products;

  @override
  Widget build(BuildContext context) {
    return LayoutBuilder(
      builder: (context, constraints) {
        // Calculate columns based on available width
        final crossAxisCount = switch (constraints.maxWidth) {
          < 400 => 1,
          < 700 => 2,
          < 1100 => 3,
          _ => 4,
        };

        return GridView.builder(
          padding: const EdgeInsets.all(16),
          gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
            crossAxisCount: crossAxisCount,
            crossAxisSpacing: 12,
            mainAxisSpacing: 12,
            childAspectRatio: 0.75,
          ),
          itemCount: products.length,
          itemBuilder: (context, index) => ProductCard(product: products[index]),
        );
      },
    );
  }
}
```

---

## 🌯 Wrap

### Introduction

`Wrap` places its children in a horizontal (or vertical) run and automatically moves to the next line when there's no room. Ideal for tags, chips, filter badges.

```dart
// ✅ Filter chips that wrap to next line
Wrap(
  spacing: 8,          // horizontal gap between chips
  runSpacing: 8,       // vertical gap between lines
  alignment: WrapAlignment.start,
  children: categories.map((cat) => FilterChip(
    label: Text(cat.name),
    selected: cat.isSelected,
    onSelected: (v) => onCategoryToggle(cat, v),
  )).toList(),
)
```

---

## ↔️ Padding

### Introduction

`Padding` adds empty space around its child. Prefer it over Container when you only need padding — it's cheaper.

```dart
// ✅ Preferred
Padding(
  padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 8),
  child: Text('Hello'),
)

// ✅ Common patterns
EdgeInsets.all(16)                           // all sides
EdgeInsets.symmetric(horizontal: 24)        // left + right
EdgeInsets.only(top: 8, bottom: 16)        // specific sides
EdgeInsets.fromLTRB(16, 8, 16, 24)        // explicit
```

---

## 🎯 Center & Align

```dart
// Center — equivalent to Align(alignment: Alignment.center)
Center(child: Text('Centered'))

// Align — precise positioning
Align(
  alignment: Alignment.bottomRight,
  child: FloatingActionButton(onPressed: () {}, child: const Icon(Icons.add)),
)

// FractionalOffset-style alignment
Align(
  alignment: const Alignment(0.5, -0.3), // custom x,y from -1 to 1
  child: const Text('Custom Position'),
)
```

---

## 📐 AspectRatio

Forces child to a specific width-to-height ratio, regardless of parent constraints.

```dart
// ✅ Product image always 1:1
AspectRatio(
  aspectRatio: 1.0,
  child: Image.network(imageUrl, fit: BoxFit.cover),
)

// ✅ Video thumbnail 16:9
AspectRatio(
  aspectRatio: 16 / 9,
  child: VideoThumbnail(url: videoUrl),
)
```

---

## 🔲 FractionallySizedBox

Sizes child as a fraction of available space.

```dart
// Child takes 80% of parent width
FractionallySizedBox(
  widthFactor: 0.8,
  child: ElevatedButton(onPressed: () {}, child: const Text('Continue')),
)
```

---

## 🔄 Transform

Applies a `Matrix4` transformation without triggering layout recalculation.

```dart
// Rotate
Transform.rotate(
  angle: math.pi / 4, // 45 degrees
  child: const Icon(Icons.star),
)

// Scale
Transform.scale(
  scale: 1.2,
  child: const Icon(Icons.favorite),
)

// Translate (offset without affecting layout)
Transform.translate(
  offset: const Offset(10, 0),
  child: const Text('Shifted text'),
)
```

> ⚠️ `Transform` does **not** affect hit testing by default. Use `GestureDetector` carefully with transformed widgets.

---

## 🗂️ IndexedStack

Shows only one child at a time but **keeps all children in the widget tree** (preserving state).

```dart
// ✅ Perfect for bottom navigation — preserves scroll position, form state, etc.
IndexedStack(
  index: _selectedTab,
  children: const [
    HomeTab(),
    SearchTab(),
    CartTab(),
    ProfileTab(),
  ],
)
```

---

## 🙈 Offstage

Hides a widget (makes it invisible and non-interactive) but keeps it in the tree.

```dart
Offstage(
  offstage: !_showBanner, // true = hidden, false = visible
  child: const PromoBanner(),
)
```

> Use `Visibility` if you also need to control whether the widget takes up space.

---

# 3. Text & Styling Widgets

---

## 📝 Text

### Introduction

The most fundamental widget for displaying text. Automatically respects theme, locale, and text direction.

### Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `data` | `String` | The text to display |
| `style` | `TextStyle?` | Font, color, size, weight |
| `textAlign` | `TextAlign?` | Left, right, center, justify |
| `overflow` | `TextOverflow?` | ellipsis, clip, fade |
| `maxLines` | `int?` | Maximum line count |
| `softWrap` | `bool?` | Whether to wrap text |
| `textScaler` | `TextScaler?` | Accessibility scaling |

### Production Example — Typography System

```dart
class TypographyDemo extends StatelessWidget {
  const TypographyDemo({super.key});

  @override
  Widget build(BuildContext context) {
    final theme = Theme.of(context).textTheme;
    final colorScheme = Theme.of(context).colorScheme;

    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        // Display (largest)
        Text('Display Large', style: theme.displayLarge),
        Text('Display Medium', style: theme.displayMedium),
        Text('Display Small', style: theme.displaySmall),
        // Headlines
        Text('Headline Large', style: theme.headlineLarge),
        Text('Headline Medium', style: theme.headlineMedium),
        Text('Headline Small', style: theme.headlineSmall),
        // Titles
        Text('Title Large', style: theme.titleLarge),
        Text('Title Medium', style: theme.titleMedium),
        Text('Title Small', style: theme.titleSmall),
        // Body
        Text('Body Large', style: theme.bodyLarge),
        Text('Body Medium', style: theme.bodyMedium),
        Text('Body Small', style: theme.bodySmall),
        // Labels
        Text('Label Large', style: theme.labelLarge),
        Text('Label Medium', style: theme.labelMedium),
        Text('Label Small', style: theme.labelSmall),

        // ✅ Custom styled text
        Text(
          'Sale ends in 2:30:00',
          style: theme.titleMedium?.copyWith(
            color: colorScheme.error,
            fontWeight: FontWeight.bold,
            fontFeatures: const [FontFeature.tabularFigures()], // monospace numbers
          ),
        ),

        // ✅ Ellipsis overflow
        Text(
          'This is a very long product description that will not fit on one line',
          maxLines: 2,
          overflow: TextOverflow.ellipsis,
          style: theme.bodyMedium,
        ),
      ],
    );
  }
}
```

---

## 🎨 RichText

### Introduction

`RichText` displays text with **multiple styles** in a single widget using `TextSpan` trees. Use it when you need mixed colors, fonts, or sizes within one paragraph.

### Production Example — Formatted Product Description

```dart
RichText(
  text: TextSpan(
    style: Theme.of(context).textTheme.bodyMedium,
    children: [
      const TextSpan(text: 'Limited offer: '),
      TextSpan(
        text: 'Buy 2, Get 1 FREE',
        style: TextStyle(
          color: Theme.of(context).colorScheme.primary,
          fontWeight: FontWeight.bold,
        ),
      ),
      const TextSpan(text: ' on all '),
      TextSpan(
        text: 'electronics',
        style: TextStyle(
          color: Theme.of(context).colorScheme.tertiary,
          decoration: TextDecoration.underline,
        ),
        recognizer: TapGestureRecognizer()..onTap = () => navigateToElectronics(),
      ),
      const TextSpan(text: '. Offer valid until '),
      const TextSpan(
        text: '31st December 2024.',
        style: TextStyle(fontWeight: FontWeight.w600),
      ),
    ],
  ),
)
```

---

## 🖼️ Image

### Introduction

Flutter provides `Image` widget for loading images from various sources.

```dart
// Network image with loading & error states
Image.network(
  'https://example.com/product.jpg',
  fit: BoxFit.cover,
  loadingBuilder: (context, child, loadingProgress) {
    if (loadingProgress == null) return child;
    return Center(
      child: CircularProgressIndicator(
        value: loadingProgress.expectedTotalBytes != null
            ? loadingProgress.cumulativeBytesLoaded / loadingProgress.expectedTotalBytes!
            : null,
      ),
    );
  },
  errorBuilder: (context, error, stackTrace) => const Icon(Icons.broken_image, size: 48),
)

// Asset image
Image.asset('assets/images/logo.png', width: 120)

// FadeInImage — placeholder + fade-in effect
FadeInImage.assetNetwork(
  placeholder: 'assets/placeholder.png',
  image: product.imageUrl,
  fit: BoxFit.cover,
  imageErrorBuilder: (_, __, ___) => const Icon(Icons.error),
)
```

### Production-Grade Network Image Widget

```dart
// ✅ Use cached_network_image package for production
import 'package:cached_network_image/cached_network_image.dart';

class AppNetworkImage extends StatelessWidget {
  const AppNetworkImage({
    super.key,
    required this.url,
    this.width,
    this.height,
    this.fit = BoxFit.cover,
    this.borderRadius,
  });

  final String url;
  final double? width;
  final double? height;
  final BoxFit fit;
  final BorderRadius? borderRadius;

  @override
  Widget build(BuildContext context) {
    Widget image = CachedNetworkImage(
      imageUrl: url,
      width: width,
      height: height,
      fit: fit,
      placeholder: (_, __) => Container(
        color: Theme.of(context).colorScheme.surfaceVariant,
        child: const Center(child: CircularProgressIndicator()),
      ),
      errorWidget: (_, __, ___) => Container(
        color: Theme.of(context).colorScheme.surfaceVariant,
        child: const Icon(Icons.image_not_supported),
      ),
    );

    if (borderRadius != null) {
      image = ClipRRect(borderRadius: borderRadius!, child: image);
    }

    return image;
  }
}
```

---

## 🎨 Theme & ThemeData

### Introduction

Flutter's theming system lets you define colors, typography, shapes, and component defaults once and apply them everywhere.

### Material 3 Complete Theme Setup

```dart
ThemeData buildLightTheme() {
  final colorScheme = ColorScheme.fromSeed(
    seedColor: const Color(0xFF6750A4),
    brightness: Brightness.light,
  );

  return ThemeData(
    useMaterial3: true,
    colorScheme: colorScheme,

    // Typography
    textTheme: GoogleFonts.poppinsTextTheme().copyWith(
      displayLarge: GoogleFonts.playfairDisplay(
        fontSize: 57,
        fontWeight: FontWeight.w400,
      ),
    ),

    // AppBar
    appBarTheme: AppBarTheme(
      backgroundColor: colorScheme.surface,
      foregroundColor: colorScheme.onSurface,
      elevation: 0,
      scrolledUnderElevation: 1,
      centerTitle: true,
    ),

    // Cards
    cardTheme: CardTheme(
      elevation: 0,
      color: colorScheme.surfaceContainer,
      shape: RoundedRectangleBorder(
        borderRadius: BorderRadius.circular(16),
      ),
    ),

    // Buttons
    elevatedButtonTheme: ElevatedButtonThemeData(
      style: ElevatedButton.styleFrom(
        elevation: 0,
        minimumSize: const Size(64, 48),
        shape: RoundedRectangleBorder(
          borderRadius: BorderRadius.circular(12),
        ),
      ),
    ),

    // Input fields
    inputDecorationTheme: InputDecorationTheme(
      filled: true,
      fillColor: colorScheme.surfaceContainer,
      border: OutlineInputBorder(
        borderRadius: BorderRadius.circular(12),
        borderSide: BorderSide.none,
      ),
      enabledBorder: OutlineInputBorder(
        borderRadius: BorderRadius.circular(12),
        borderSide: BorderSide.none,
      ),
      focusedBorder: OutlineInputBorder(
        borderRadius: BorderRadius.circular(12),
        borderSide: BorderSide(color: colorScheme.primary, width: 2),
      ),
    ),

    // Chips
    chipTheme: ChipThemeData(
      shape: RoundedRectangleBorder(
        borderRadius: BorderRadius.circular(8),
      ),
    ),
  );
}
```

### Dark Mode Implementation

```dart
class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    // Using Provider for theme mode persistence
    return Consumer<ThemeProvider>(
      builder: (context, themeProvider, _) {
        return MaterialApp(
          theme: buildLightTheme(),
          darkTheme: buildDarkTheme(),
          themeMode: themeProvider.themeMode,
          home: const HomeScreen(),
        );
      },
    );
  }
}

class ThemeProvider extends ChangeNotifier {
  ThemeMode _themeMode = ThemeMode.system;

  ThemeMode get themeMode => _themeMode;

  void setThemeMode(ThemeMode mode) {
    _themeMode = mode;
    notifyListeners();
    // Save to shared_preferences
    SharedPreferences.getInstance().then(
      (prefs) => prefs.setString('themeMode', mode.name),
    );
  }
}
```

---

## 🔲 ClipRRect & ClipOval

```dart
// Rounded corners clip
ClipRRect(
  borderRadius: BorderRadius.circular(16),
  child: Image.network(url, width: 200, height: 200, fit: BoxFit.cover),
)

// Circular clip (for avatars)
ClipOval(
  child: Image.network(avatarUrl, width: 60, height: 60, fit: BoxFit.cover),
)
```

---

# 4. Input Widgets

---

## 📝 TextField & TextFormField

### Introduction

`TextField` is Flutter's core text input widget. `TextFormField` extends it with `Form` validation support.

### Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `controller` | `TextEditingController?` | Manage text programmatically |
| `focusNode` | `FocusNode?` | Control keyboard focus |
| `decoration` | `InputDecoration?` | Label, hint, border, icons |
| `keyboardType` | `TextInputType?` | email, number, phone, etc. |
| `textInputAction` | `TextInputAction?` | next, done, search |
| `obscureText` | `bool` | Password masking |
| `maxLines` | `int?` | Multi-line input |
| `onChanged` | `ValueChanged<String>?` | Real-time change callback |
| `onSubmitted` | `ValueChanged<String>?` | On keyboard action |
| `validator` | `String? Function(String?)?` | Validation (TextFormField only) |
| `inputFormatters` | `List<TextInputFormatter>?` | Input masking/restriction |
| `enabled` | `bool?` | Enable/disable input |
| `readOnly` | `bool` | Non-editable display |
| `autofillHints` | `Iterable<String>?` | AutoFill support |

### Production Example — Complete Registration Form

```dart
class RegistrationForm extends StatefulWidget {
  const RegistrationForm({super.key});

  @override
  State<RegistrationForm> createState() => _RegistrationFormState();
}

class _RegistrationFormState extends State<RegistrationForm> {
  final _formKey = GlobalKey<FormState>();
  final _nameController = TextEditingController();
  final _emailController = TextEditingController();
  final _phoneController = TextEditingController();
  final _passwordController = TextEditingController();

  final _nameFocus = FocusNode();
  final _emailFocus = FocusNode();
  final _phoneFocus = FocusNode();
  final _passwordFocus = FocusNode();

  bool _obscurePassword = true;
  bool _isLoading = false;

  @override
  void dispose() {
    // ✅ Always dispose controllers and focus nodes
    _nameController.dispose();
    _emailController.dispose();
    _phoneController.dispose();
    _passwordController.dispose();
    _nameFocus.dispose();
    _emailFocus.dispose();
    _phoneFocus.dispose();
    _passwordFocus.dispose();
    super.dispose();
  }

  Future<void> _submit() async {
    if (!_formKey.currentState!.validate()) return;
    setState(() => _isLoading = true);

    try {
      await AuthService.register(
        name: _nameController.text.trim(),
        email: _emailController.text.trim(),
        phone: _phoneController.text.trim(),
        password: _passwordController.text,
      );
      if (!mounted) return;
      Navigator.of(context).pushReplacementNamed('/home');
    } on AuthException catch (e) {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text(e.message), backgroundColor: Colors.red),
      );
    } finally {
      if (mounted) setState(() => _isLoading = false);
    }
  }

  @override
  Widget build(BuildContext context) {
    return Form(
      key: _formKey,
      child: Column(
        children: [
          // Name field
          TextFormField(
            controller: _nameController,
            focusNode: _nameFocus,
            textInputAction: TextInputAction.next,
            textCapitalization: TextCapitalization.words,
            autofillHints: const [AutofillHints.name],
            decoration: const InputDecoration(
              labelText: 'Full Name',
              prefixIcon: Icon(Icons.person_outline),
            ),
            onFieldSubmitted: (_) => _emailFocus.requestFocus(),
            validator: (v) {
              if (v == null || v.trim().isEmpty) return 'Name is required';
              if (v.trim().length < 2) return 'Name too short';
              return null;
            },
          ),
          const SizedBox(height: 16),

          // Email field
          TextFormField(
            controller: _emailController,
            focusNode: _emailFocus,
            textInputAction: TextInputAction.next,
            keyboardType: TextInputType.emailAddress,
            autofillHints: const [AutofillHints.email],
            decoration: const InputDecoration(
              labelText: 'Email Address',
              prefixIcon: Icon(Icons.email_outlined),
            ),
            onFieldSubmitted: (_) => _phoneFocus.requestFocus(),
            validator: (v) {
              if (v == null || v.isEmpty) return 'Email is required';
              if (!RegExp(r'^[\w-.]+@([\w-]+\.)+[\w-]{2,4}$').hasMatch(v)) {
                return 'Enter a valid email';
              }
              return null;
            },
          ),
          const SizedBox(height: 16),

          // Phone field
          TextFormField(
            controller: _phoneController,
            focusNode: _phoneFocus,
            textInputAction: TextInputAction.next,
            keyboardType: TextInputType.phone,
            autofillHints: const [AutofillHints.telephoneNumber],
            inputFormatters: [
              FilteringTextInputFormatter.digitsOnly,
              LengthLimitingTextInputFormatter(10),
            ],
            decoration: const InputDecoration(
              labelText: 'Phone Number',
              prefixIcon: Icon(Icons.phone_outlined),
              prefix: Text('+91 '),
            ),
            onFieldSubmitted: (_) => _passwordFocus.requestFocus(),
            validator: (v) {
              if (v == null || v.isEmpty) return 'Phone is required';
              if (v.length != 10) return 'Enter a valid 10-digit number';
              return null;
            },
          ),
          const SizedBox(height: 16),

          // Password field
          TextFormField(
            controller: _passwordController,
            focusNode: _passwordFocus,
            textInputAction: TextInputAction.done,
            obscureText: _obscurePassword,
            autofillHints: const [AutofillHints.newPassword],
            decoration: InputDecoration(
              labelText: 'Password',
              prefixIcon: const Icon(Icons.lock_outline),
              suffixIcon: IconButton(
                icon: Icon(_obscurePassword
                    ? Icons.visibility_outlined
                    : Icons.visibility_off_outlined),
                onPressed: () => setState(() => _obscurePassword = !_obscurePassword),
              ),
            ),
            onFieldSubmitted: (_) => _submit(),
            validator: (v) {
              if (v == null || v.isEmpty) return 'Password is required';
              if (v.length < 8) return 'Minimum 8 characters';
              if (!RegExp(r'(?=.*[A-Z])').hasMatch(v)) return 'Must contain uppercase letter';
              if (!RegExp(r'(?=.*\d)').hasMatch(v)) return 'Must contain a number';
              return null;
            },
          ),
          const SizedBox(height: 32),

          // Submit button
          FilledButton(
            onPressed: _isLoading ? null : _submit,
            child: _isLoading
                ? const SizedBox(
                    height: 20,
                    width: 20,
                    child: CircularProgressIndicator(strokeWidth: 2, color: Colors.white),
                  )
                : const Text('Create Account'),
          ),
        ],
      ),
    );
  }
}
```

---

## ✅ Checkbox, Radio & Switch

```dart
// Checkbox
CheckboxListTile(
  title: const Text('I agree to Terms & Conditions'),
  subtitle: const Text('Please read before agreeing'),
  value: _agreeToTerms,
  onChanged: (v) => setState(() => _agreeToTerms = v!),
  controlAffinity: ListTileControlAffinity.leading,
)

// Radio group
Column(
  children: PaymentMethod.values.map((method) => RadioListTile<PaymentMethod>(
    title: Text(method.label),
    value: method,
    groupValue: _selectedPayment,
    onChanged: (v) => setState(() => _selectedPayment = v!),
  )).toList(),
)

// Switch
SwitchListTile(
  title: const Text('Push Notifications'),
  subtitle: const Text('Receive order updates'),
  value: _notificationsEnabled,
  onChanged: (v) => setState(() => _notificationsEnabled = v),
)
```

---

## 🎚️ Slider & RangeSlider

```dart
// Price range filter
RangeSlider(
  values: _priceRange,
  min: 0,
  max: 100000,
  divisions: 100,
  labels: RangeLabels(
    '₹${_priceRange.start.round()}',
    '₹${_priceRange.end.round()}',
  ),
  onChanged: (range) => setState(() => _priceRange = range),
)
```

---

## 🎯 GestureDetector & InkWell

```dart
// GestureDetector — raw gestures (no ripple)
GestureDetector(
  onTap: () => print('tapped'),
  onLongPress: () => print('long pressed'),
  onDoubleTap: () => print('double tapped'),
  onPanUpdate: (details) => print('dragging: ${details.delta}'),
  child: Container(color: Colors.blue, width: 100, height: 100),
)

// InkWell — Material ripple effect
InkWell(
  onTap: () => Navigator.push(...),
  borderRadius: BorderRadius.circular(12),
  splashColor: Theme.of(context).colorScheme.primary.withOpacity(0.2),
  child: Padding(
    padding: const EdgeInsets.all(12),
    child: Row(
      children: [
        Icon(Icons.chevron_right),
        Text('View Details'),
      ],
    ),
  ),
)

// ✅ Prefer InkWell inside Material widget for correct ripple rendering
Material(
  color: Colors.transparent,
  child: InkWell(
    onTap: () {},
    borderRadius: BorderRadius.circular(16),
    child: const Padding(
      padding: EdgeInsets.all(12),
      child: Text('Tappable item'),
    ),
  ),
)
```

---

# 5. Button Widgets

---

## 🔘 Button Hierarchy in Material 3

```
Emphasis level:
───────────────────────────────────────────────────────
High     │ FilledButton        (primary actions, CTA)
         │ FilledButton.tonal  (secondary primary actions)
Medium   │ ElevatedButton      (slightly raised, moderate emphasis)
         │ OutlinedButton      (medium, border emphasis)
Low      │ TextButton          (inline, low emphasis)
Minimal  │ IconButton           (icon-only actions)
Special  │ FloatingActionButton (primary screen action)
───────────────────────────────────────────────────────
```

### Production Example — Button Showcase

```dart
class ButtonShowcase extends StatelessWidget {
  const ButtonShowcase({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // Primary CTA
        FilledButton.icon(
          onPressed: () {},
          icon: const Icon(Icons.shopping_cart),
          label: const Text('Add to Cart'),
        ),

        // Secondary
        FilledButton.tonal(
          onPressed: () {},
          child: const Text('Save to Wishlist'),
        ),

        // Elevated
        ElevatedButton(
          onPressed: () {},
          child: const Text('View Details'),
        ),

        // Outline
        OutlinedButton.icon(
          onPressed: () {},
          icon: const Icon(Icons.compare),
          label: const Text('Compare'),
        ),

        // Text
        TextButton(
          onPressed: () {},
          child: const Text('Read Reviews'),
        ),

        // Custom styled
        ElevatedButton(
          style: ElevatedButton.styleFrom(
            backgroundColor: const Color(0xFFE91E63),
            foregroundColor: Colors.white,
            minimumSize: const Size(double.infinity, 56),
            shape: RoundedRectangleBorder(
              borderRadius: BorderRadius.circular(28),
            ),
            elevation: 8,
            shadowColor: const Color(0xFFE91E63).withOpacity(0.5),
          ),
          onPressed: () {},
          child: const Row(
            mainAxisSize: MainAxisSize.min,
            children: [
              Icon(Icons.flash_on),
              SizedBox(width: 8),
              Text('Buy Now', style: TextStyle(fontSize: 16, fontWeight: FontWeight.bold)),
            ],
          ),
        ),
      ],
    );
  }
}
```

---

## 🔵 FloatingActionButton

```dart
// Standard FAB
FloatingActionButton(
  onPressed: () {},
  child: const Icon(Icons.add),
)

// Extended FAB
FloatingActionButton.extended(
  onPressed: () {},
  icon: const Icon(Icons.add_shopping_cart),
  label: const Text('Add to Cart'),
  backgroundColor: Theme.of(context).colorScheme.primary,
)

// Small FAB
FloatingActionButton.small(
  onPressed: () {},
  child: const Icon(Icons.edit),
)

// Large FAB
FloatingActionButton.large(
  onPressed: () {},
  child: const Icon(Icons.navigation),
)
```

---

## 🎛️ ToggleButtons & SegmentedButton

```dart
// SegmentedButton — Material 3 (preferred)
SegmentedButton<SortOrder>(
  segments: const [
    ButtonSegment(value: SortOrder.popularity, label: Text('Popular'), icon: Icon(Icons.trending_up)),
    ButtonSegment(value: SortOrder.price, label: Text('Price'), icon: Icon(Icons.attach_money)),
    ButtonSegment(value: SortOrder.newest, label: Text('Newest'), icon: Icon(Icons.new_releases)),
  ],
  selected: {_sortOrder},
  onSelectionChanged: (Set<SortOrder> selection) {
    setState(() => _sortOrder = selection.first);
  },
)
```

---

# 6. List & Grid Widgets

---

## 📋 ListView

### Introduction

`ListView` is the primary scrollable list widget. It comes in four constructors for different use cases.

```dart
// ListView — for short static lists (all rendered at once — avoid for long lists)
ListView(
  padding: const EdgeInsets.all(16),
  children: [
    const ListTile(title: Text('Item 1')),
    const Divider(),
    const ListTile(title: Text('Item 2')),
  ],
)

// ✅ ListView.builder — lazy loading, efficient for long lists
ListView.builder(
  itemCount: products.length,
  itemExtent: 80, // ✅ Fixed height improves scroll performance
  itemBuilder: (context, index) {
    final product = products[index];
    return ProductListItem(product: product);
  },
)

// ListView.separated — with dividers
ListView.separated(
  itemCount: orders.length,
  separatorBuilder: (_, __) => const Divider(height: 1),
  itemBuilder: (context, index) => OrderTile(order: orders[index]),
)

// ListView.custom — custom SliverChildDelegate
ListView.custom(
  childrenDelegate: SliverChildBuilderDelegate(
    (context, index) => ProductListItem(product: products[index]),
    itemCount: products.length,
    findChildIndexCallback: (key) {
      final productKey = key as ValueKey<String>;
      return products.indexWhere((p) => p.id == productKey.value);
    },
  ),
)
```

### Performance Considerations

- ✅ Use `ListView.builder` (not `ListView`) for lists with more than ~20 items
- ✅ Use `itemExtent` or `prototypeItem` for fixed-height lists — dramatically improves scrolling performance
- ✅ Use `const` constructors for list items that don't change
- ❌ Don't put `ListView` inside a `Column` without wrapping in `Expanded` or setting `shrinkWrap: true` (shrinkWrap is expensive!)

---

## 🔲 GridView

```dart
// GridView.builder — efficient
GridView.builder(
  padding: const EdgeInsets.all(16),
  gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
    crossAxisCount: 2,
    crossAxisSpacing: 12,
    mainAxisSpacing: 12,
    childAspectRatio: 0.75, // height:width ratio
  ),
  itemCount: products.length,
  itemBuilder: (context, index) => ProductCard(product: products[index]),
)

// Masonry grid (use flutter_staggered_grid_view package)
MasonryGridView.builder(
  gridDelegate: const SliverSimpleGridDelegateWithFixedCrossAxisCount(crossAxisCount: 2),
  itemCount: items.length,
  itemBuilder: (context, index) => ImageCard(item: items[index]),
)
```

---

## 📄 ListTile

### Introduction

`ListTile` is a pre-built Material list item with slots for leading icon, title, subtitle, and trailing widget.

```dart
// ✅ Rich ListTile
ListTile(
  leading: const CircleAvatar(
    backgroundImage: NetworkImage('https://...'),
    radius: 24,
  ),
  title: const Text('John Doe'),
  subtitle: const Text('Order #12345 placed'),
  trailing: Column(
    mainAxisAlignment: MainAxisAlignment.center,
    crossAxisAlignment: CrossAxisAlignment.end,
    children: [
      const Text('₹2,499', style: TextStyle(fontWeight: FontWeight.bold)),
      Text('2 items', style: Theme.of(context).textTheme.bodySmall),
    ],
  ),
  onTap: () => navigateToOrderDetail(),
  isThreeLine: false,
  dense: false,
)
```

---

## 📊 DataTable & PaginatedDataTable

```dart
// DataTable for small, fixed datasets
DataTable(
  sortColumnIndex: _sortColumnIndex,
  sortAscending: _sortAscending,
  columns: const [
    DataColumn(label: Text('Product')),
    DataColumn(label: Text('Price'), numeric: true),
    DataColumn(label: Text('Stock'), numeric: true),
    DataColumn(label: Text('Actions')),
  ],
  rows: products.map((product) => DataRow(
    cells: [
      DataCell(Text(product.name)),
      DataCell(Text('₹${product.price}')),
      DataCell(Text('${product.stock}')),
      DataCell(Row(
        children: [
          IconButton(icon: const Icon(Icons.edit, size: 18), onPressed: () => editProduct(product)),
          IconButton(icon: const Icon(Icons.delete, size: 18), onPressed: () => deleteProduct(product)),
        ],
      )),
    ],
  )).toList(),
)
```

---

## 📑 ExpansionTile

```dart
ExpansionTile(
  leading: const Icon(Icons.local_shipping_outlined),
  title: const Text('Delivery Information'),
  subtitle: const Text('Free delivery in 3-5 days'),
  children: [
    Padding(
      padding: const EdgeInsets.all(16),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          _InfoRow(icon: Icons.timer, text: 'Standard: 3-5 business days'),
          _InfoRow(icon: Icons.bolt, text: 'Express: Next day delivery'),
          _InfoRow(icon: Icons.assignment_return, text: '30-day free returns'),
        ],
      ),
    ),
  ],
)
```

---

# 7. Scroll Widgets

---

## 📜 SingleChildScrollView

Use for **screens that may overflow when the keyboard appears**, or simple scrollable content.

```dart
// ✅ Scrollable form that doesn't overflow
SingleChildScrollView(
  padding: const EdgeInsets.all(16),
  keyboardDismissBehavior: ScrollViewKeyboardDismissBehavior.onDrag,
  child: Column(
    children: [...formFields],
  ),
)
```

> ⚠️ Don't use `SingleChildScrollView` for long lists — use `ListView.builder` instead.

---

## 📜 CustomScrollView

`CustomScrollView` is a scroll view that takes `slivers` (scroll-aware building blocks). It's the foundation for advanced scroll effects.

```dart
// ✅ Combined sliver layout
CustomScrollView(
  slivers: [
    SliverAppBar.large(
      title: const Text('Products'),
      pinned: true,
      actions: [IconButton(icon: const Icon(Icons.search), onPressed: () {})],
    ),
    // Horizontal category chips
    SliverToBoxAdapter(
      child: SizedBox(
        height: 48,
        child: ListView.builder(
          scrollDirection: Axis.horizontal,
          padding: const EdgeInsets.symmetric(horizontal: 16),
          itemCount: categories.length,
          itemBuilder: (_, i) => CategoryChip(category: categories[i]),
        ),
      ),
    ),
    const SliverToBoxAdapter(child: SizedBox(height: 8)),
    // Product grid
    SliverPadding(
      padding: const EdgeInsets.all(16),
      sliver: SliverGrid.builder(
        gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
          crossAxisCount: 2,
          crossAxisSpacing: 12,
          mainAxisSpacing: 12,
          childAspectRatio: 0.75,
        ),
        itemCount: products.length,
        itemBuilder: (context, index) => ProductCard(product: products[index]),
      ),
    ),
  ],
)
```

---

## 🔄 RefreshIndicator

```dart
RefreshIndicator(
  onRefresh: () async {
    await context.read<ProductProvider>().fetchProducts();
  },
  color: Theme.of(context).colorScheme.primary,
  child: ListView.builder(
    itemCount: products.length,
    itemBuilder: (_, i) => ProductTile(product: products[i]),
  ),
)
```

---

# 8. Async & State Widgets

---

## ⏳ FutureBuilder

### Introduction

`FutureBuilder` builds UI based on the state of a `Future`. It tracks `ConnectionState`: `none`, `waiting`, `active`, `done`.

### Production Example — Product Detail Loader

```dart
class ProductDetailPage extends StatefulWidget {
  const ProductDetailPage({super.key, required this.productId});
  final String productId;

  @override
  State<ProductDetailPage> createState() => _ProductDetailPageState();
}

class _ProductDetailPageState extends State<ProductDetailPage> {
  late Future<Product> _productFuture;

  @override
  void initState() {
    super.initState();
    // ✅ Create the Future in initState, NOT inside build()
    _productFuture = ProductRepository.fetchById(widget.productId);
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Product Detail')),
      body: FutureBuilder<Product>(
        future: _productFuture,
        builder: (context, snapshot) {
          return switch (snapshot.connectionState) {
            ConnectionState.waiting => const Center(child: CircularProgressIndicator()),
            ConnectionState.done when snapshot.hasError => Center(
                child: Column(
                  mainAxisSize: MainAxisSize.min,
                  children: [
                    const Icon(Icons.error_outline, size: 48, color: Colors.red),
                    const SizedBox(height: 16),
                    Text('Error: ${snapshot.error}'),
                    const SizedBox(height: 16),
                    ElevatedButton(
                      onPressed: () => setState(() {
                        _productFuture = ProductRepository.fetchById(widget.productId);
                      }),
                      child: const Text('Retry'),
                    ),
                  ],
                ),
              ),
            ConnectionState.done => ProductDetailContent(product: snapshot.data!),
            _ => const SizedBox.shrink(),
          };
        },
      ),
    );
  }
}
```

> ⚠️ **Critical Mistake:** Never put a `Future` directly in `FutureBuilder`'s `future` parameter if it's created inline — it will be recreated on every `build()` call, causing infinite loading loops. Always create the `Future` in `initState`.

---

## 🌊 StreamBuilder

### Introduction

`StreamBuilder` builds UI reactively based on a stream of data — perfect for real-time features.

### Production Example — Real-time Order Tracking

```dart
class OrderTrackingScreen extends StatelessWidget {
  const OrderTrackingScreen({super.key, required this.orderId});
  final String orderId;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Track Order')),
      body: StreamBuilder<OrderStatus>(
        stream: OrderTrackingService.watchOrder(orderId),
        builder: (context, snapshot) {
          if (snapshot.connectionState == ConnectionState.waiting) {
            return const Center(child: CircularProgressIndicator());
          }

          if (snapshot.hasError) {
            return ErrorView(error: snapshot.error.toString());
          }

          if (!snapshot.hasData) {
            return const Center(child: Text('No tracking data'));
          }

          final status = snapshot.data!;
          return OrderStatusTimeline(status: status);
        },
      ),
    );
  }
}
```

---

## 🔔 ValueListenableBuilder

`ValueListenableBuilder` rebuilds when a `ValueNotifier` changes — more efficient than `setState` for isolated state.

```dart
// ✅ Counter example with ValueNotifier
class CartIconBadge extends StatelessWidget {
  const CartIconBadge({super.key, required this.cartItemCount});
  final ValueNotifier<int> cartItemCount;

  @override
  Widget build(BuildContext context) {
    return ValueListenableBuilder<int>(
      valueListenable: cartItemCount,
      builder: (context, count, child) {
        return Badge(
          isLabelVisible: count > 0,
          label: Text('$count'),
          child: child!, // ✅ Pass non-reactive child for optimization
        );
      },
      child: const Icon(Icons.shopping_cart_outlined), // ✅ Not rebuilt
    );
  }
}
```

---

## 🏗️ StatefulBuilder

Allows `setState` in a stateless context — useful inside dialogs, bottom sheets, and `showDialog`.

```dart
showDialog(
  context: context,
  builder: (context) {
    int quantity = 1; // Local state

    return StatefulBuilder(
      builder: (context, setState) {
        return AlertDialog(
          title: const Text('Add to Cart'),
          content: Row(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              IconButton(
                onPressed: () => setState(() { if (quantity > 1) quantity--; }),
                icon: const Icon(Icons.remove),
              ),
              Text('$quantity', style: Theme.of(context).textTheme.titleLarge),
              IconButton(
                onPressed: () => setState(() => quantity++),
                icon: const Icon(Icons.add),
              ),
            ],
          ),
          actions: [
            TextButton(onPressed: () => Navigator.pop(context), child: const Text('Cancel')),
            FilledButton(
              onPressed: () {
                cartProvider.add(product, quantity);
                Navigator.pop(context);
              },
              child: const Text('Add'),
            ),
          ],
        );
      },
    );
  },
);
```

---

# 9. Animation Widgets

---

## 🎬 Flutter Animations Master Guide

### Animation Types

```
┌─────────────────────────────────────────────────────────────┐
│                    FLUTTER ANIMATION TYPES                   │
├──────────────────┬──────────────────────────────────────────┤
│ Implicit         │ AnimatedContainer, AnimatedOpacity,       │
│ (automatic)      │ AnimatedPositioned, TweenAnimationBuilder  │
├──────────────────┼──────────────────────────────────────────┤
│ Explicit         │ AnimationController + AnimatedBuilder,    │
│ (manual control) │ FadeTransition, SlideTransition, Hero      │
├──────────────────┼──────────────────────────────────────────┤
│ Physics-based    │ SpringSimulation, GravitySimulation        │
└──────────────────┴──────────────────────────────────────────┘
```

---

## ✨ AnimatedContainer

Automatically animates property changes. The simplest form of animation.

```dart
class AnimatedCardDemo extends StatefulWidget {
  const AnimatedCardDemo({super.key});

  @override
  State<AnimatedCardDemo> createState() => _AnimatedCardDemoState();
}

class _AnimatedCardDemoState extends State<AnimatedCardDemo> {
  bool _isExpanded = false;

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: () => setState(() => _isExpanded = !_isExpanded),
      child: AnimatedContainer(
        duration: const Duration(milliseconds: 300),
        curve: Curves.easeInOut,
        width: double.infinity,
        height: _isExpanded ? 200 : 80,
        decoration: BoxDecoration(
          color: _isExpanded
              ? Theme.of(context).colorScheme.primaryContainer
              : Theme.of(context).colorScheme.surfaceContainer,
          borderRadius: BorderRadius.circular(_isExpanded ? 20 : 12),
          boxShadow: _isExpanded
              ? [BoxShadow(blurRadius: 16, color: Colors.black.withOpacity(0.2))]
              : [],
        ),
        child: Center(
          child: Text(_isExpanded ? 'Tap to collapse' : 'Tap to expand'),
        ),
      ),
    );
  }
}
```

---

## 🌟 Hero Animation

`Hero` creates a shared element transition between screens.

```dart
// ✅ Source screen
Hero(
  tag: 'product_image_${product.id}', // Must be unique
  child: ClipRRect(
    borderRadius: BorderRadius.circular(12),
    child: Image.network(product.imageUrl, width: 100, height: 100, fit: BoxFit.cover),
  ),
)

// ✅ Destination screen
Hero(
  tag: 'product_image_${product.id}',
  child: Image.network(
    product.imageUrl,
    width: double.infinity,
    height: 300,
    fit: BoxFit.cover,
  ),
)
```

---

## 🔀 AnimatedSwitcher

Animates the transition when switching between different child widgets.

```dart
AnimatedSwitcher(
  duration: const Duration(milliseconds: 300),
  transitionBuilder: (child, animation) => FadeTransition(
    opacity: animation,
    child: ScaleTransition(scale: animation, child: child),
  ),
  child: _isLoading
      ? const CircularProgressIndicator(key: ValueKey('loading'))
      : Text(
          '$_cartCount',
          key: ValueKey(_cartCount), // ✅ Key is critical for AnimatedSwitcher
          style: const TextStyle(fontSize: 24, fontWeight: FontWeight.bold),
        ),
)
```

---

## 🎭 Explicit Animations — AnimationController Pattern

```dart
class PulseAnimation extends StatefulWidget {
  const PulseAnimation({super.key, required this.child});
  final Widget child;

  @override
  State<PulseAnimation> createState() => _PulseAnimationState();
}

class _PulseAnimationState extends State<PulseAnimation>
    with SingleTickerProviderStateMixin {
  late final AnimationController _controller;
  late final Animation<double> _scaleAnimation;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this, // ✅ SingleTickerProviderStateMixin
      duration: const Duration(milliseconds: 800),
    )..repeat(reverse: true);

    _scaleAnimation = Tween<double>(begin: 1.0, end: 1.1).animate(
      CurvedAnimation(parent: _controller, curve: Curves.easeInOut),
    );
  }

  @override
  void dispose() {
    _controller.dispose(); // ✅ Always dispose
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return AnimatedBuilder(
      animation: _scaleAnimation,
      builder: (context, child) => Transform.scale(
        scale: _scaleAnimation.value,
        child: child,
      ),
      child: widget.child, // ✅ Pass as child to avoid rebuilding
    );
  }
}
```

---

## 🎞️ TweenAnimationBuilder

Animate from one value to another without managing `AnimationController`.

```dart
TweenAnimationBuilder<double>(
  tween: Tween(begin: 0, end: _progress),
  duration: const Duration(seconds: 1),
  curve: Curves.easeOut,
  builder: (context, value, child) {
    return Column(
      children: [
        LinearProgressIndicator(value: value),
        Text('${(value * 100).round()}%'),
      ],
    );
  },
)
```

---

# 10. Navigation Widgets

---

## 🧭 Navigator

### Push & Pop Pattern

```dart
// Push named route
Navigator.pushNamed(context, '/product', arguments: {'id': product.id});

// Push with custom transition
Navigator.push(
  context,
  PageRouteBuilder(
    pageBuilder: (_, animation, __) => ProductDetailScreen(product: product),
    transitionsBuilder: (_, animation, __, child) {
      return SlideTransition(
        position: Tween<Offset>(
          begin: const Offset(1, 0),
          end: Offset.zero,
        ).animate(CurvedAnimation(parent: animation, curve: Curves.easeOut)),
        child: child,
      );
    },
    transitionDuration: const Duration(milliseconds: 300),
  ),
);

// Pop with result
Navigator.pop(context, selectedItem);

// Push and remove all previous routes (after login)
Navigator.pushNamedAndRemoveUntil(context, '/home', (_) => false);
```

### GoRouter — Production Navigation (Preferred)

```dart
// router.dart
final appRouter = GoRouter(
  initialLocation: '/home',
  debugLogDiagnostics: true,
  redirect: (context, state) {
    final isLoggedIn = context.read<AuthProvider>().isLoggedIn;
    if (!isLoggedIn && !state.matchedLocation.startsWith('/auth')) {
      return '/auth/login';
    }
    return null;
  },
  routes: [
    GoRoute(
      path: '/auth/login',
      builder: (_, __) => const LoginScreen(),
    ),
    ShellRoute(
      builder: (context, state, child) => AppShell(child: child),
      routes: [
        GoRoute(
          path: '/home',
          builder: (_, __) => const HomeScreen(),
        ),
        GoRoute(
          path: '/product/:id',
          builder: (context, state) => ProductDetailScreen(
            productId: state.pathParameters['id']!,
          ),
        ),
        GoRoute(
          path: '/cart',
          builder: (_, __) => const CartScreen(),
        ),
        GoRoute(
          path: '/profile',
          builder: (_, __) => const ProfileScreen(),
        ),
      ],
    ),
  ],
);

// main.dart
MaterialApp.router(routerConfig: appRouter)

// Navigation
context.go('/product/42');
context.push('/cart');
context.pop();
```

---

# 11. Material Design Widgets

---

## 🃏 Card

```dart
// ✅ Material 3 Card variants
Card(child: ListTile(title: Text('Elevated Card')))           // Default
Card.outlined(child: ListTile(title: Text('Outlined Card'))) // Outlined
Card.filled(child: ListTile(title: Text('Filled Card')))     // Filled (tonal)
```

---

## 💬 Dialog & AlertDialog

```dart
// ✅ Production alert dialog
Future<bool?> showDeleteConfirmDialog(BuildContext context) {
  return showDialog<bool>(
    context: context,
    builder: (context) => AlertDialog(
      icon: Icon(Icons.warning_amber_rounded,
          color: Theme.of(context).colorScheme.error, size: 48),
      title: const Text('Delete Item?'),
      content: const Text(
          'This item will be permanently removed from your cart. This action cannot be undone.'),
      actions: [
        TextButton(
          onPressed: () => Navigator.pop(context, false),
          child: const Text('Cancel'),
        ),
        FilledButton(
          style: FilledButton.styleFrom(
            backgroundColor: Theme.of(context).colorScheme.error,
          ),
          onPressed: () => Navigator.pop(context, true),
          child: const Text('Delete'),
        ),
      ],
    ),
  );
}
```

---

## 📢 SnackBar

```dart
// ✅ Production snackbar patterns
void showSuccessSnackBar(BuildContext context, String message) {
  ScaffoldMessenger.of(context)
    ..hideCurrentSnackBar()
    ..showSnackBar(
      SnackBar(
        content: Row(
          children: [
            const Icon(Icons.check_circle, color: Colors.white),
            const SizedBox(width: 8),
            Expanded(child: Text(message)),
          ],
        ),
        backgroundColor: Colors.green,
        behavior: SnackBarBehavior.floating,
        shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(12)),
        margin: const EdgeInsets.all(16),
        duration: const Duration(seconds: 3),
        action: SnackBarAction(
          label: 'Undo',
          textColor: Colors.white,
          onPressed: () => undoAction(),
        ),
      ),
    );
}
```

---

## ↗️ BottomSheet

```dart
// Modal bottom sheet
showModalBottomSheet(
  context: context,
  isScrollControlled: true, // ✅ Allows full-screen height
  useSafeArea: true,
  shape: const RoundedRectangleBorder(
    borderRadius: BorderRadius.vertical(top: Radius.circular(24)),
  ),
  builder: (context) => DraggableScrollableSheet(
    initialChildSize: 0.6,
    minChildSize: 0.4,
    maxChildSize: 0.95,
    expand: false,
    builder: (context, scrollController) {
      return ProductFilterSheet(scrollController: scrollController);
    },
  ),
);
```

---

## ⏳ Progress Indicators

```dart
// Circular (indefinite loading)
const CircularProgressIndicator()

// Circular (determinate — showing progress)
CircularProgressIndicator(value: uploadProgress) // 0.0 to 1.0

// Linear
LinearProgressIndicator(value: downloadProgress)

// ✅ Loading button pattern
FilledButton(
  onPressed: _isLoading ? null : _submit,
  child: _isLoading
      ? const SizedBox(
          width: 20, height: 20,
          child: CircularProgressIndicator(strokeWidth: 2, color: Colors.white),
        )
      : const Text('Submit'),
)
```

---

# 12. Cupertino Widgets

---

## 🍎 Cupertino Widget Overview

```dart
// CupertinoApp — iOS-style root
CupertinoApp(
  theme: const CupertinoThemeData(
    primaryColor: CupertinoColors.activeBlue,
  ),
  home: const CupertinoHomeScreen(),
)

// CupertinoNavigationBar
CupertinoNavigationBar(
  middle: const Text('Products'),
  trailing: CupertinoButton(
    padding: EdgeInsets.zero,
    child: const Icon(CupertinoIcons.add),
    onPressed: () {},
  ),
)

// CupertinoAlertDialog
showCupertinoDialog(
  context: context,
  builder: (_) => CupertinoAlertDialog(
    title: const Text('Delete Item'),
    content: const Text('Are you sure?'),
    actions: [
      CupertinoDialogAction(child: const Text('Cancel'), onPressed: () => Navigator.pop(context)),
      CupertinoDialogAction(
        isDestructiveAction: true,
        onPressed: () => Navigator.pop(context),
        child: const Text('Delete'),
      ),
    ],
  ),
)

// CupertinoSwitch
CupertinoSwitch(
  value: _isEnabled,
  onChanged: (v) => setState(() => _isEnabled = v),
)

// CupertinoPicker
CupertinoPicker(
  itemExtent: 40,
  onSelectedItemChanged: (index) => setState(() => _selectedSize = sizes[index]),
  children: sizes.map((s) => Center(child: Text(s))).toList(),
)
```

---

# 13. Advanced Rendering Widgets

---

## 🎨 CustomPaint

Allows you to draw anything using a `Canvas`.

```dart
class StarRatingPainter extends CustomPainter {
  final double rating;
  final Color filled;
  final Color empty;

  StarRatingPainter({required this.rating, required this.filled, required this.empty});

  @override
  void paint(Canvas canvas, Size size) {
    final paint = Paint()..style = PaintingStyle.fill;
    // Draw 5 stars with filled/empty based on rating
    for (int i = 0; i < 5; i++) {
      paint.color = i < rating.floor() ? filled : empty;
      _drawStar(canvas, Offset(i * 24.0 + 12, size.height / 2), 10, paint);
    }
  }

  void _drawStar(Canvas canvas, Offset center, double radius, Paint paint) {
    final path = Path();
    for (int i = 0; i < 5; i++) {
      final angle = (i * 144 - 90) * 3.14159 / 180;
      final innerAngle = angle + 72 * 3.14159 / 180;
      final outer = Offset(
        center.dx + radius * math.cos(angle),
        center.dy + radius * math.sin(angle),
      );
      final inner = Offset(
        center.dx + (radius * 0.4) * math.cos(innerAngle),
        center.dy + (radius * 0.4) * math.sin(innerAngle),
      );
      i == 0 ? path.moveTo(outer.dx, outer.dy) : path.lineTo(outer.dx, outer.dy);
      path.lineTo(inner.dx, inner.dy);
    }
    path.close();
    canvas.drawPath(path, paint);
  }

  @override
  bool shouldRepaint(StarRatingPainter oldDelegate) => rating != oldDelegate.rating;
}

// Usage
CustomPaint(
  size: const Size(120, 24),
  painter: StarRatingPainter(
    rating: product.rating,
    filled: Colors.amber,
    empty: Colors.grey.shade300,
  ),
)
```

---

## ⚡ RepaintBoundary

Isolates a subtree from parent repaints — critical for performance with animations.

```dart
// ✅ Wrap expensive or frequently-animating widgets
RepaintBoundary(
  child: AnimatedCounter(value: cartCount), // Only this repaints
)

// ✅ Wrap list items that contain animations
ListView.builder(
  itemBuilder: (context, index) => RepaintBoundary(
    child: ProductCard(product: products[index]),
  ),
)
```

---

## 🌫️ BackdropFilter

Applies a filter (blur, color matrix) to widgets behind it.

```dart
Stack(
  children: [
    BackgroundImage(),
    // Glassmorphism effect
    ClipRRect(
      borderRadius: BorderRadius.circular(20),
      child: BackdropFilter(
        filter: ImageFilter.blur(sigmaX: 10, sigmaY: 10),
        child: Container(
          decoration: BoxDecoration(
            color: Colors.white.withOpacity(0.15),
            borderRadius: BorderRadius.circular(20),
            border: Border.all(color: Colors.white.withOpacity(0.3)),
          ),
          padding: const EdgeInsets.all(20),
          child: const Text('Glass Card', style: TextStyle(color: Colors.white)),
        ),
      ),
    ),
  ],
)
```

---

# 14. Accessibility Widgets

---

## ♿ Semantics

Add accessibility metadata for screen readers.

```dart
Semantics(
  label: 'Product image: ${product.name}',
  child: Image.network(product.imageUrl),
)

// Button semantics
Semantics(
  button: true,
  label: 'Add ${product.name} to cart',
  hint: 'Double tap to add',
  child: InkWell(
    onTap: () => addToCart(product),
    child: const Icon(Icons.add_shopping_cart),
  ),
)

// Exclude from semantics (decorative elements)
ExcludeSemantics(
  child: const Icon(Icons.star, color: Colors.amber),
)

// Merge semantics of children
MergeSemantics(
  child: Row(
    children: [
      const Icon(Icons.star, color: Colors.amber),
      Text('4.5 (${review.count} reviews)'),
    ],
  ),
)
```

### Accessibility Best Practices

- ✅ All interactive elements must have a semantic label
- ✅ Minimum tap target size: **48x48 logical pixels**
- ✅ Color is not the only differentiator (add icons/text)
- ✅ Support dynamic text scaling (`textScaler`)
- ✅ Test with TalkBack (Android) and VoiceOver (iOS)
- ✅ Use `Tooltip` for icon buttons

```dart
// ✅ Ensure minimum touch target
SizedBox(
  width: 48,
  height: 48,
  child: IconButton(
    icon: const Icon(Icons.close, size: 20),
    tooltip: 'Close dialog',
    onPressed: () => Navigator.pop(context),
  ),
)
```

---

# 15. Sliver Widgets

---

## 🏔️ SliverAppBar

```dart
CustomScrollView(
  slivers: [
    // ✅ Collapsing app bar with parallax
    SliverAppBar(
      expandedHeight: 250,
      floating: false,  // stays hidden after scroll up
      pinned: true,     // always shows collapsed bar
      snap: false,
      stretch: true,
      flexibleSpace: FlexibleSpaceBar(
        title: const Text('Category: Electronics'),
        background: Stack(
          fit: StackFit.expand,
          children: [
            Image.network(categoryBannerUrl, fit: BoxFit.cover),
            // Gradient overlay so title is readable
            const DecoratedBox(
              decoration: BoxDecoration(
                gradient: LinearGradient(
                  begin: Alignment.topCenter,
                  end: Alignment.bottomCenter,
                  colors: [Colors.transparent, Colors.black54],
                ),
              ),
            ),
          ],
        ),
      ),
    ),
    // Content...
  ],
)
```

---

## 📋 SliverList & SliverGrid

```dart
// SliverList
SliverList.builder(
  itemCount: items.length,
  itemBuilder: (context, index) => ItemTile(item: items[index]),
)

// SliverGrid
SliverGrid.builder(
  gridDelegate: const SliverGridDelegateWithMaxCrossAxisExtent(
    maxCrossAxisExtent: 200, // Each cell max 200px wide
    crossAxisSpacing: 12,
    mainAxisSpacing: 12,
    childAspectRatio: 0.75,
  ),
  itemCount: products.length,
  itemBuilder: (context, index) => ProductCard(product: products[index]),
)
```

---

# 16. State Management Widgets

---

## 🏗️ State Management Comparison

```
┌──────────────────────────────────────────────────────────────────┐
│              STATE MANAGEMENT COMPARISON                         │
├────────────────┬────────────┬────────────┬────────────┬─────────┤
│ Feature        │ Provider   │ Riverpod   │ BLoC       │ GetX    │
├────────────────┼────────────┼────────────┼────────────┼─────────┤
│ Learning curve │ Low        │ Medium     │ High       │ Low     │
│ Boilerplate    │ Medium     │ Low-Medium │ High       │ Low     │
│ Testability    │ Good       │ Excellent  │ Excellent  │ Medium  │
│ Scalability    │ Good       │ Excellent  │ Excellent  │ Good    │
│ Performance    │ Good       │ Excellent  │ Good       │ Good    │
│ Community      │ Large      │ Growing    │ Large      │ Large   │
│ Flutter team   │ Recommended│ Recommended│ Supported  │ Popular │
└────────────────┴────────────┴────────────┴────────────┴─────────┘
```

---

## 🔌 Provider Pattern

```dart
// ✅ ChangeNotifier-based provider
class CartProvider extends ChangeNotifier {
  final List<CartItem> _items = [];

  List<CartItem> get items => List.unmodifiable(_items);
  int get itemCount => _items.fold(0, (sum, item) => sum + item.quantity);
  double get total => _items.fold(0, (sum, item) => sum + item.totalPrice);

  void addItem(Product product, {int quantity = 1}) {
    final existingIndex = _items.indexWhere((i) => i.productId == product.id);
    if (existingIndex >= 0) {
      _items[existingIndex].increaseQuantity(quantity);
    } else {
      _items.add(CartItem(product: product, quantity: quantity));
    }
    notifyListeners();
  }

  void removeItem(String productId) {
    _items.removeWhere((i) => i.productId == productId);
    notifyListeners();
  }

  void clear() {
    _items.clear();
    notifyListeners();
  }
}

// Setup at app level
MultiProvider(
  providers: [
    ChangeNotifierProvider(create: (_) => CartProvider()),
    ChangeNotifierProvider(create: (_) => AuthProvider()),
    ChangeNotifierProxyProvider<AuthProvider, OrderProvider>(
      create: (_) => OrderProvider(),
      update: (_, auth, orders) => orders!..updateUser(auth.user),
    ),
  ],
  child: const MyApp(),
)

// Consume in UI
Consumer<CartProvider>(
  builder: (context, cart, child) {
    return Column(
      children: [
        child!, // Non-reactive header
        Text('Items: ${cart.itemCount}'),
        Text('Total: ₹${cart.total.toStringAsFixed(2)}'),
        ...cart.items.map((item) => CartItemTile(item: item)),
      ],
    );
  },
  child: const CartHeader(), // ✅ Does not rebuild
)

// Read without listening (for actions)
context.read<CartProvider>().addItem(product);

// Watch (listen for rebuilds)
final cart = context.watch<CartProvider>();
```

---

## 🧊 BLoC Pattern

```dart
// Events
abstract class CartEvent {}
class AddToCart extends CartEvent { final Product product; AddToCart(this.product); }
class RemoveFromCart extends CartEvent { final String productId; RemoveFromCart(this.productId); }
class ClearCart extends CartEvent {}

// State
class CartState {
  final List<CartItem> items;
  final bool isLoading;
  final String? error;

  const CartState({this.items = const [], this.isLoading = false, this.error});

  CartState copyWith({List<CartItem>? items, bool? isLoading, String? error}) {
    return CartState(
      items: items ?? this.items,
      isLoading: isLoading ?? this.isLoading,
      error: error,
    );
  }

  double get total => items.fold(0, (sum, item) => sum + item.totalPrice);
}

// BLoC
class CartBloc extends Bloc<CartEvent, CartState> {
  CartBloc() : super(const CartState()) {
    on<AddToCart>(_onAddToCart);
    on<RemoveFromCart>(_onRemoveFromCart);
    on<ClearCart>(_onClearCart);
  }

  void _onAddToCart(AddToCart event, Emitter<CartState> emit) {
    final updatedItems = List<CartItem>.from(state.items);
    final idx = updatedItems.indexWhere((i) => i.productId == event.product.id);
    if (idx >= 0) {
      updatedItems[idx] = updatedItems[idx].copyWith(quantity: updatedItems[idx].quantity + 1);
    } else {
      updatedItems.add(CartItem(product: event.product, quantity: 1));
    }
    emit(state.copyWith(items: updatedItems));
  }

  void _onRemoveFromCart(RemoveFromCart event, Emitter<CartState> emit) {
    emit(state.copyWith(
      items: state.items.where((i) => i.productId != event.productId).toList(),
    ));
  }

  void _onClearCart(ClearCart event, Emitter<CartState> emit) {
    emit(const CartState());
  }
}

// UI
BlocBuilder<CartBloc, CartState>(
  builder: (context, state) {
    if (state.isLoading) return const CircularProgressIndicator();
    if (state.items.isEmpty) return const EmptyCartView();
    return CartItemList(items: state.items, total: state.total);
  },
)

// Dispatch event
context.read<CartBloc>().add(AddToCart(product));
```

---

# 17. Interview Preparation

---

## 🎯 Flutter Widget Interview Questions — 100+ Q&A

### Category 1: Widget Fundamentals

**Q1: What is a widget in Flutter?**
> A widget is an **immutable description** of a part of the user interface. Flutter renders the UI by building a tree of widgets. Widgets are lightweight Dart objects — they are not the rendered UI but describe what the UI should look like. Flutter uses the widget tree to create and manage the element and render object trees.

**Q2: What is the difference between StatelessWidget and StatefulWidget?**
> A `StatelessWidget` has no mutable state — once built, it only changes if its parent passes different configuration. A `StatefulWidget` maintains a separate `State` object that can call `setState()` to trigger rebuilds. Use `StatelessWidget` for static/display content and `StatefulWidget` when the widget needs to change in response to user interaction or data.

**Q3: Explain the three trees in Flutter.**
> Flutter maintains three parallel trees:
> 1. **Widget Tree** — Your Dart code (immutable blueprints)
> 2. **Element Tree** — Flutter's internal mutable instances that map widgets to render objects; manages lifecycle
> 3. **RenderObject Tree** — Handles layout (sizes/positions) and painting (pixels on screen)

**Q4: Why are widgets immutable in Flutter?**
> Immutability enables Flutter to efficiently determine when the UI needs to change. Since widgets are cheap to create (just data objects), Flutter rebuilds them on every `build()` call. The element tree's reconciliation algorithm (similar to React's virtual DOM diffing) determines which changes actually require layout/paint work.

**Q5: What is `BuildContext` and what does it represent?**
> `BuildContext` is a handle to a widget's location in the widget tree. It's implemented by `Element`. It provides access to the widget's ancestors — used for `Theme.of(context)`, `Navigator.of(context)`, `MediaQuery.of(context)`, etc. It should never be used after the widget is disposed or across async gaps without checking `mounted`.

**Q6: What is the `key` property in Flutter and when should you use it?**
> Keys help Flutter's reconciliation algorithm match widget instances across rebuilds. They are critical when:
> - Reordering list items (`ValueKey`, `ObjectKey`)
> - Preserving state when a widget moves in the tree
> - Triggering a full widget reset (`UniqueKey`)
> - Using `GlobalKey` to access widget state from outside
>
> Without keys, Flutter may match the wrong element to a widget, causing state to be lost or transferred incorrectly.

**Q7: What is `const` constructor optimization in Flutter?**
> Using `const` constructors creates compile-time constants. Flutter can skip rebuilding `const` subtrees even when a parent rebuilds because it knows the subtree is identical. This is a significant performance optimization for widgets with fixed properties.

---

### Category 2: Layout Questions

**Q8: Explain Flutter's layout algorithm.**
> Flutter uses a "constraint down, size up" system:
> 1. Parent passes `BoxConstraints` (min/max width/height) to child
> 2. Child determines its own size within those constraints
> 3. Parent positions child within itself
> 
> Constraints are **tight** (min==max, forces exact size), **loose** (min==0, child picks size), or **bounded/unbounded**.

**Q9: What causes "RenderBox was not laid out" error?**
> This occurs when a widget with unbounded constraints is given to a widget that requires bounded constraints. Common cases: `ListView` inside `Column` without `Expanded`, `Row` inside `Row` without `Expanded`.

**Q10: When would you use `Expanded` vs `Flexible`?**
> - `Expanded` forces the child to fill all allocated space (tight constraint)
> - `Flexible` gives the child a share of space but allows it to be smaller (loose constraint)
> Use `Expanded` for buttons/fields that should fill available width. Use `Flexible` when the child should size itself naturally but not overflow.

**Q11: What is the difference between `Padding` and `Container(padding: ...)`?**
> Both add inner spacing, but `Padding` is lighter — it's a single-purpose `RenderObjectWidget`. `Container` composes multiple widgets internally (Padding, DecoratedBox, ConstrainedBox, etc.), adding overhead. Prefer `Padding` when you only need padding.

**Q12: How does `LayoutBuilder` differ from `MediaQuery`?**
> `MediaQuery` gives screen/device dimensions (viewport size). `LayoutBuilder` gives the **available constraints from the immediate parent** — which may be smaller than the screen. `LayoutBuilder` is preferred for reusable responsive widgets that should adapt to their parent container, not just the screen.

---

### Category 3: Performance Questions

**Q13: How do you prevent unnecessary widget rebuilds?**
> - Use `const` constructors wherever possible
> - Use `RepaintBoundary` around frequently-animating/repainting subtrees
> - Use `ValueListenableBuilder` or `Selector` instead of `Consumer` for fine-grained rebuilds
> - Use `AnimatedBuilder`'s `child` parameter for the non-animating portion
> - Split large widgets into smaller ones so only the changed part rebuilds

**Q14: What is `shouldRepaint` in `CustomPainter`?**
> It's called before repainting to check if a repaint is necessary. Return `true` if properties changed, `false` to skip. Optimization: Compare old and new delegate properties rather than always returning `true`.

**Q15: What is `RepaintBoundary` and when should you use it?**
> `RepaintBoundary` creates a new layer in the layer tree, isolating repaint of its subtree from the rest of the UI. Use it around:
> - Animated widgets
> - Frequently-changing widgets in otherwise-static layouts
> - Expensive custom paint operations
> Don't overuse it — each `RepaintBoundary` has memory cost (stores its own GPU texture).

**Q16: What is the difference between `setState`, `notifyListeners`, and `emit` in state management?**
> - `setState`: triggers a rebuild of the `StatefulWidget`'s `build` method only
> - `notifyListeners`: (ChangeNotifier) notifies all `Consumer`/`watch` listeners to rebuild
> - `emit`: (BLoC) emits a new state to all `BlocBuilder` instances subscribed to that BLoC

**Q17: Why should you avoid creating objects inside `build()`?**
> The `build()` method can be called many times per second. Creating heavy objects (like `TextStyle`, `BoxDecoration`, `BorderRadius`) inside `build()` causes garbage collection pressure. Prefer `const` or define them as `static const` outside the build method.

---

### Category 4: Navigation & Routing

**Q18: What is the difference between `Navigator.push` and `Navigator.pushReplacement`?**
> `push` adds the new route on top of the stack (can go back). `pushReplacement` replaces the current route (cannot go back to it). Use `pushReplacement` after login (replacing the login screen with home) or between onboarding screens.

**Q19: What is `WillPopScope` and why is it deprecated?**
> `WillPopScope` intercepted the back button press to allow custom behavior or preventing navigation. It was deprecated in Flutter 3.12 because it didn't work correctly with Android's predictive back gesture. The replacement is `PopScope` with `canPop` and `onPopInvokedWithResult`.

**Q20: Explain deep linking in Flutter.**
> Deep linking allows URLs to navigate directly to specific screens within the app. On mobile, this uses platform-level URL schemes (iOS: `CFBundleURLTypes`, Android: `intent-filter`). `GoRouter` has built-in deep link support via `GoRouter.redirect` and URL-pattern routes.

---

### Category 5: Async & State

**Q21: Why must a `Future` not be created inline in `FutureBuilder`?**
> If you write `FutureBuilder(future: fetchData(), ...)`, a new `Future` is created on every `build()` call, which causes an infinite loop of loading states. Create the `Future` in `initState` and store it in a field.

**Q22: What is the difference between `context.read` and `context.watch` in Provider?**
> - `context.watch<T>()` subscribes to changes — the widget rebuilds when T notifies listeners
> - `context.read<T>()` gets the current value without subscribing — use in callbacks/event handlers
> - `context.select<T, R>((t) => t.value)` subscribes to only a specific property of T

**Q23: How does `InheritedWidget` work?**
> `InheritedWidget` is Flutter's built-in mechanism for passing data down the widget tree without explicitly threading it through constructors. Widgets that call `MyInherited.of(context)` subscribe to it — they rebuild when the `InheritedWidget`'s `updateShouldNotify` returns `true`. All state management packages (Provider, Riverpod, etc.) are built on top of `InheritedWidget`.

---

### Category 6: Architecture Questions

**Q24: Describe Clean Architecture in a Flutter app.**
> Clean Architecture separates concerns into layers:
> - **Presentation**: Widgets, ViewModels/BLoCs, UI state
> - **Domain**: Use Cases (business rules), Entities, Repository interfaces
> - **Data**: Repository implementations, Remote/Local data sources, DTOs
>
> Dependencies only point inward — Presentation depends on Domain, Data depends on Domain, Domain knows nothing about Presentation or Data.

**Q25: How would you structure a Flutter project for large teams?**
> Use feature-first folder structure:
```
lib/
├── core/                # Shared across features
│   ├── error/
│   ├── network/
│   ├── theme/
│   └── utils/
├── features/
│   ├── auth/
│   │   ├── data/        # Repository impl, API, models
│   │   ├── domain/      # Entities, use cases, repo interfaces
│   │   └── presentation/# Widgets, BLoC, state
│   ├── products/
│   ├── cart/
│   └── orders/
└── main.dart
```

---

### Scenario-Based Questions

**Q26: A user reports that a list with 10,000 items is slow. How do you optimize it?**
> 1. Switch from `ListView` to `ListView.builder` if not already done
> 2. Add `itemExtent` (fixed height) — eliminates layout computation for each item
> 3. Wrap complex list items with `RepaintBoundary`
> 4. Make list items `const` where possible
> 5. Implement pagination — load only first 20-50 items
> 6. Use `cacheExtent` to control how many off-screen items are retained
> 7. Avoid heavy widgets (images) without caching (use `cached_network_image`)

**Q27: How do you handle network errors gracefully in Flutter?**
> Use a Result type or Either monad:
```dart
sealed class Result<T> {
  const Result();
}
class Success<T> extends Result<T> { final T data; const Success(this.data); }
class Failure<T> extends Result<T> { final String message; final Exception? error; const Failure(this.message, {this.error}); }

// In repository
Future<Result<List<Product>>> getProducts() async {
  try {
    final response = await api.fetchProducts();
    return Success(response.map(Product.fromDto).toList());
  } on NetworkException catch (e) {
    return Failure('No internet connection', error: e);
  } on ServerException catch (e) {
    return Failure('Server error: ${e.message}', error: e);
  }
}
```

**Q28: How would you implement offline-first functionality?**
> 1. Use a local database (Hive, Drift/Moor, SQLite via sqflite)
> 2. Repository pattern: always read from local DB, sync with server in background
> 3. Use `connectivity_plus` to detect network state
> 4. Implement a sync queue for failed write operations
> 5. Show "offline" indicator in the UI using a network status widget

---

# 18. Clean Architecture & Project Structure

---

## 📁 Complete Folder Structure

```
my_flutter_app/
├── lib/
│   ├── core/
│   │   ├── constants/
│   │   │   ├── api_constants.dart
│   │   │   └── app_constants.dart
│   │   ├── error/
│   │   │   ├── exceptions.dart
│   │   │   └── failures.dart
│   │   ├── network/
│   │   │   ├── api_client.dart          # Dio/http setup
│   │   │   ├── network_info.dart
│   │   │   └── interceptors/
│   │   ├── router/
│   │   │   └── app_router.dart          # GoRouter config
│   │   ├── theme/
│   │   │   ├── app_theme.dart
│   │   │   ├── app_colors.dart
│   │   │   └── app_text_styles.dart
│   │   ├── utils/
│   │   │   ├── date_utils.dart
│   │   │   ├── currency_utils.dart
│   │   │   └── validators.dart
│   │   └── widgets/                     # Shared reusable widgets
│   │       ├── app_button.dart
│   │       ├── app_text_field.dart
│   │       ├── loading_indicator.dart
│   │       └── error_view.dart
│   │
│   ├── features/
│   │   ├── auth/
│   │   │   ├── data/
│   │   │   │   ├── datasources/
│   │   │   │   │   ├── auth_remote_datasource.dart
│   │   │   │   │   └── auth_local_datasource.dart
│   │   │   │   ├── models/
│   │   │   │   │   └── user_model.dart  # DTO with fromJson/toJson
│   │   │   │   └── repositories/
│   │   │   │       └── auth_repository_impl.dart
│   │   │   ├── domain/
│   │   │   │   ├── entities/
│   │   │   │   │   └── user.dart        # Pure Dart entity
│   │   │   │   ├── repositories/
│   │   │   │   │   └── auth_repository.dart  # Abstract interface
│   │   │   │   └── usecases/
│   │   │   │       ├── login_usecase.dart
│   │   │   │       ├── register_usecase.dart
│   │   │   │       └── logout_usecase.dart
│   │   │   └── presentation/
│   │   │       ├── bloc/
│   │   │       │   ├── auth_bloc.dart
│   │   │       │   ├── auth_event.dart
│   │   │       │   └── auth_state.dart
│   │   │       ├── pages/
│   │   │       │   ├── login_page.dart
│   │   │       │   └── register_page.dart
│   │   │       └── widgets/
│   │   │           ├── login_form.dart
│   │   │           └── social_login_buttons.dart
│   │   │
│   │   ├── products/
│   │   │   └── ... (same structure)
│   │   │
│   │   └── cart/
│   │       └── ... (same structure)
│   │
│   ├── injection_container.dart          # GetIt DI setup
│   └── main.dart
│
├── test/
│   ├── core/
│   ├── features/
│   │   ├── auth/
│   │   │   ├── data/
│   │   │   ├── domain/
│   │   │   └── presentation/
│   │   └── products/
│   └── helpers/
│
├── integration_test/
├── assets/
│   ├── images/
│   ├── fonts/
│   └── animations/
└── pubspec.yaml
```

---

## 💉 Dependency Injection with GetIt

```dart
// injection_container.dart
import 'package:get_it/get_it.dart';

final sl = GetIt.instance;

Future<void> initDependencies() async {
  // External
  final sharedPrefs = await SharedPreferences.getInstance();
  sl.registerLazySingleton(() => sharedPrefs);
  sl.registerLazySingleton(() => Dio()..interceptors.addAll([
    AuthInterceptor(sl()),
    LoggingInterceptor(),
  ]));

  // Core
  sl.registerLazySingleton<NetworkInfo>(() => NetworkInfoImpl(sl()));

  // Data sources
  sl.registerLazySingleton<ProductRemoteDataSource>(
    () => ProductRemoteDataSourceImpl(dio: sl()),
  );
  sl.registerLazySingleton<ProductLocalDataSource>(
    () => ProductLocalDataSourceImpl(prefs: sl()),
  );

  // Repositories
  sl.registerLazySingleton<ProductRepository>(
    () => ProductRepositoryImpl(
      remote: sl(),
      local: sl(),
      networkInfo: sl(),
    ),
  );

  // Use cases
  sl.registerLazySingleton(() => GetProductsUseCase(sl()));
  sl.registerLazySingleton(() => GetProductByIdUseCase(sl()));

  // BLoCs (factory — new instance per usage)
  sl.registerFactory(() => ProductBloc(
    getProducts: sl(),
    getProductById: sl(),
  ));
}
```

---

# 19. Flutter Performance Optimization

---

## ⚡ Performance Checklist

```
REBUILD OPTIMIZATION
─────────────────────────────────────────────────────────
✅ Use const constructors for immutable widgets
✅ Use keys correctly for list items
✅ Use Selector<T,R> instead of Consumer<T> for partial rebuilds
✅ Use AnimatedBuilder's child parameter for static children
✅ Avoid setState in frequently-called callbacks (throttle/debounce)
✅ Extract widgets to classes rather than inline builder functions

RENDERING OPTIMIZATION
─────────────────────────────────────────────────────────
✅ Use RepaintBoundary for isolated repainting areas
✅ Use cacheExtent on scrollable lists
✅ Add itemExtent to ListView.builder when height is fixed
✅ Use opaque: true on PageRoute when possible
✅ Prefer ColoredBox over Container for solid colors

IMAGE OPTIMIZATION
─────────────────────────────────────────────────────────
✅ Use cached_network_image for automatic disk/memory cache
✅ Resize images server-side to match display size
✅ Use WebP format (smaller file, same quality)
✅ Use cacheWidth/cacheHeight on Image widget
✅ Lazy-load images (only visible items fetch)

COMPILATION OPTIMIZATION
─────────────────────────────────────────────────────────
✅ Use flutter build --release for production
✅ Enable tree shaking (removes unused code)
✅ Use dart:isolate for heavy computation
✅ Use compute() for background tasks

PROFILING TOOLS
─────────────────────────────────────────────────────────
• Flutter DevTools → Performance tab (flame chart)
• Flutter DevTools → Widget Rebuild Stats
• flutter run --profile (production-like profiling)
• Repaint Rainbow (debugRepaintRainbowEnabled = true)
• Widget Inspector for rebuild detection
```

### Code Example — Optimized List Widget

```dart
// ❌ Bad — rebuilds entire list on any change
Consumer<ProductProvider>(
  builder: (context, provider, _) {
    return ListView.builder(
      itemCount: provider.products.length,
      itemBuilder: (_, i) => ProductCard(product: provider.products[i]),
    );
  },
)

// ✅ Good — only rebuilds when product count changes
Selector<ProductProvider, int>(
  selector: (_, provider) => provider.products.length,
  builder: (context, count, _) {
    return ListView.builder(
      itemCount: count,
      itemExtent: 120, // ✅ Fixed height = faster layout
      itemBuilder: (context, i) {
        // ✅ Read product here — won't trigger rebuild of list
        return Selector<ProductProvider, Product>(
          selector: (_, p) => p.products[i],
          builder: (_, product, __) => RepaintBoundary( // ✅ Isolate repaint
            child: ProductCard(product: product),
          ),
        );
      },
    );
  },
)
```

---

## 🧪 Custom Widget Creation — Production Reusable Components

```dart
// ✅ Production-grade reusable button with loading state
class AppButton extends StatelessWidget {
  const AppButton({
    super.key,
    required this.label,
    this.onPressed,
    this.isLoading = false,
    this.variant = AppButtonVariant.filled,
    this.icon,
    this.size = AppButtonSize.medium,
    this.width,
  });

  final String label;
  final VoidCallback? onPressed;
  final bool isLoading;
  final AppButtonVariant variant;
  final IconData? icon;
  final AppButtonSize size;
  final double? width;

  @override
  Widget build(BuildContext context) {
    final theme = Theme.of(context);

    final double height = switch (size) {
      AppButtonSize.small => 36,
      AppButtonSize.medium => 48,
      AppButtonSize.large => 56,
    };

    Widget buttonChild = isLoading
        ? SizedBox(
            width: 20,
            height: 20,
            child: CircularProgressIndicator(
              strokeWidth: 2,
              color: variant == AppButtonVariant.filled
                  ? theme.colorScheme.onPrimary
                  : theme.colorScheme.primary,
            ),
          )
        : icon != null
            ? Row(
                mainAxisSize: MainAxisSize.min,
                children: [
                  Icon(icon, size: 18),
                  const SizedBox(width: 8),
                  Text(label),
                ],
              )
            : Text(label);

    final style = ButtonStyle(
      minimumSize: WidgetStateProperty.all(Size(width ?? 64, height)),
      shape: WidgetStateProperty.all(
        RoundedRectangleBorder(borderRadius: BorderRadius.circular(12)),
      ),
    );

    return switch (variant) {
      AppButtonVariant.filled => FilledButton(
          onPressed: isLoading ? null : onPressed,
          style: style,
          child: buttonChild,
        ),
      AppButtonVariant.outlined => OutlinedButton(
          onPressed: isLoading ? null : onPressed,
          style: style,
          child: buttonChild,
        ),
      AppButtonVariant.text => TextButton(
          onPressed: isLoading ? null : onPressed,
          style: style,
          child: buttonChild,
        ),
    };
  }
}

enum AppButtonVariant { filled, outlined, text }
enum AppButtonSize { small, medium, large }
```

---

## 📱 Responsive UI Architecture

```dart
// ✅ Breakpoint-based responsive system
class Responsive {
  static const double mobileMax = 599;
  static const double tabletMax = 1199;

  static bool isMobile(BuildContext context) =>
      MediaQuery.sizeOf(context).width <= mobileMax;

  static bool isTablet(BuildContext context) {
    final w = MediaQuery.sizeOf(context).width;
    return w > mobileMax && w <= tabletMax;
  }

  static bool isDesktop(BuildContext context) =>
      MediaQuery.sizeOf(context).width > tabletMax;

  static T value<T>(
    BuildContext context, {
    required T mobile,
    T? tablet,
    T? desktop,
  }) {
    if (isDesktop(context)) return desktop ?? tablet ?? mobile;
    if (isTablet(context)) return tablet ?? mobile;
    return mobile;
  }
}

// ✅ Usage
class ResponsiveLayout extends StatelessWidget {
  const ResponsiveLayout({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Row(
        children: [
          // Sidebar only on tablet/desktop
          if (!Responsive.isMobile(context))
            NavigationRail(
              extended: Responsive.isDesktop(context),
              selectedIndex: 0,
              onDestinationSelected: (_) {},
              destinations: const [
                NavigationRailDestination(icon: Icon(Icons.home), label: Text('Home')),
                NavigationRailDestination(icon: Icon(Icons.search), label: Text('Search')),
              ],
            ),
          Expanded(
            child: LayoutBuilder(
              builder: (context, constraints) {
                final columns = Responsive.value(
                  context,
                  mobile: 1,
                  tablet: 2,
                  desktop: 3,
                );
                return GridView.builder(
                  gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
                    crossAxisCount: columns,
                    childAspectRatio: 0.75,
                    crossAxisSpacing: 12,
                    mainAxisSpacing: 12,
                  ),
                  padding: EdgeInsets.all(Responsive.value(context, mobile: 12, tablet: 16, desktop: 24)),
                  itemCount: products.length,
                  itemBuilder: (_, i) => ProductCard(product: products[i]),
                );
              },
            ),
          ),
        ],
      ),
    );
  }
}
```

---

## 🎯 Mini Projects Summary

Throughout this guide, the following production-ready mini-projects were demonstrated:

| # | Project | Widgets Covered |
|---|---------|----------------|
| 1 | **ShopEase E-Commerce Home** | MaterialApp, Scaffold, NavigationBar, IndexedStack |
| 2 | **Product Detail with Parallax** | SliverAppBar, FlexibleSpaceBar, Hero |
| 3 | **User Registration Form** | TextFormField, Form, GlobalKey, Validators |
| 4 | **Product Card with Badge** | Stack, Positioned, ClipRRect, InkWell |
| 5 | **Animated Cart Counter** | AnimatedSwitcher, AnimationController, TweenAnimationBuilder |
| 6 | **Real-time Order Tracking** | StreamBuilder, CustomScrollView, SliverList |
| 7 | **Product Filter Sheet** | DraggableScrollableSheet, Wrap, FilterChip, RangeSlider |
| 8 | **Clean Architecture Cart** | BLoC, Provider, GetIt, Repository pattern |
| 9 | **Responsive Product Grid** | LayoutBuilder, GridView, Responsive utility class |
| 10 | **Glassmorphism Promo Card** | BackdropFilter, Stack, Container gradient |

---

## 📚 Recommended Packages Reference

```yaml
dependencies:
  # State Management
  flutter_bloc: ^8.1.6
  provider: ^6.1.2
  get_it: ^7.7.0
  injectable: ^2.4.4

  # Navigation
  go_router: ^14.2.7

  # Networking
  dio: ^5.6.0
  retrofit: ^4.4.1

  # Local Storage
  shared_preferences: ^2.3.2
  hive_flutter: ^1.1.0
  drift: ^2.22.1           # SQLite ORM

  # Images
  cached_network_image: ^3.4.1
  flutter_svg: ^2.0.10

  # UI
  shimmer: ^3.0.0
  lottie: ^3.1.2
  flutter_animate: ^4.5.0

  # Utils
  freezed_annotation: ^2.4.4
  json_annotation: ^4.9.0
  intl: ^0.19.0
  equatable: ^2.0.5

dev_dependencies:
  build_runner: ^2.4.12
  freezed: ^2.5.7
  json_serializable: ^6.8.0
  injectable_generator: ^2.6.2
  flutter_lints: ^4.0.0
  mocktail: ^1.0.4
  bloc_test: ^9.1.7
```

---

## 🏆 Production-Ready Flutter App Checklist

```
PRE-DEVELOPMENT
─────────────────────────────────────────────────────────
□ Set up lint rules (flutter_lints or very_good_analysis)
□ Configure code generation (build_runner)
□ Set up flavor environments (dev/staging/prod)
□ Configure CI/CD (GitHub Actions / Codemagic)

DEVELOPMENT
─────────────────────────────────────────────────────────
□ Follow Clean Architecture (data/domain/presentation)
□ Use dependency injection (GetIt + Injectable)
□ Implement proper error handling (Result type)
□ Write unit tests for use cases and BLoCs
□ Write widget tests for complex UI
□ Add integration tests for critical flows

PERFORMANCE
─────────────────────────────────────────────────────────
□ Profile with DevTools in profile mode
□ Fix jank (drops below 60fps)
□ Minimize widget rebuilds
□ Implement image caching
□ Lazy-load lists and images
□ Implement pagination

ACCESSIBILITY
─────────────────────────────────────────────────────────
□ Add Semantics labels to all interactive elements
□ Test with TalkBack/VoiceOver
□ Ensure 48x48 minimum touch targets
□ Support text scaling up to 200%
□ Check color contrast ratios (WCAG AA: 4.5:1)

RELEASE
─────────────────────────────────────────────────────────
□ Remove debug banners and print statements
□ Configure ProGuard/R8 rules (Android)
□ Set up crash reporting (Firebase Crashlytics)
□ Set up analytics (Firebase Analytics)
□ Configure app signing
□ Test on physical devices (both platforms)
□ Submit for review with accurate metadata
```

---

*End of Flutter Widgets Master Guide*

---

> **📖 This guide covers Flutter 3.x | Dart 3.x | Material 3 | Null Safety**
> 
> *Built for developers who want to go from zero to production-ready Flutter engineering.*
