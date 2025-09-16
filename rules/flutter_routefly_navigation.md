# Flutter Routing with Routefly

## When to Apply

This rule applies to ALL Flutter projects that require:

- Navigation between screens
- URL-based routing (especially for web)
- Nested routing/layouts
- Dynamic routes
- Route organization and management

## Required Package

Always use `routefly: ^3.1.3` as the primary and ONLY routing solution instead of Navigator 1.0, go_router, auto_route, or manual route management.

## Mandatory Implementation

### 1. Project Structure

Organize all pages in the `lib/app` directory with file-based routing:

```
lib/
└── app/
    ├── home/
    │   └── home_page.dart
    ├── product/
    │   ├── [id]/
    │   │   └── product_details_page.dart
    │   └── product_list_page.dart
    ├── dashboard/
    │   ├── users/
    │   │   └── users_page.dart
    │   ├── settings/
    │   │   └── settings_page.dart
    │   └── dashboard_layout.dart
    └── (auth)/
        ├── login/
        │   └── login_page.dart
        └── register/
            └── register_page.dart
```

### 2. App Configuration

Always configure the main app with Routefly:

```dart
import 'package:routefly/routefly.dart';
import 'my_app.route.dart'; // GENERATED

part 'my_app.g.dart'; // GENERATED

@Main('lib/app')
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      routerConfig: Routefly.routerConfig(
        routes: routes, // GENERATED
        middlewares: [authGuard], // Optional
        notFoundPath: '/404', // Optional
      ),
    );
  }
}
```

### 3. Navigation Methods

Use ONLY Routefly navigation methods:

```dart
// Replace entire route stack
Routefly.navigate('/dashboard/users');

// Push new route on stack
Routefly.pushNavigate('/product/123');

// Add route to stack
Routefly.push('/settings');

// Remove top route
Routefly.pop();

// Replace current route
Routefly.replace('/login');
```

### 4. Type-Safe Navigation (Object Notation)

Always prefer object notation over string paths:

```dart
// ✅ DO - Type-safe object notation
Routefly.navigate(routePaths.dashboard.users);
Routefly.navigate(routePaths.product.changes({'id': '123'}));

// ❌ DON'T - String notation (error-prone)
Routefly.navigate('/dashboard/users');
Routefly.navigate('/product/123');
```

### 5. Dynamic Routes

Create dynamic routes using bracket notation:

```dart
// File: lib/app/product/[id]/product_page.dart
class ProductPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final productId = Routefly.query['id'];
    final searchTerm = Routefly.query.params['search'];

    return Scaffold(
      appBar: AppBar(title: Text('Product $productId')),
      body: Text('Search: $searchTerm'),
    );
  }
}
```

### 6. Route Groups

Use parentheses for route groups that shouldn't affect URL structure:

```dart
// Directory: lib/app/(auth)/login/login_page.dart
// Generates route: /login (not /(auth)/login)

// Directory: lib/app/(dashboard)/users/users_page.dart
// Generates route: /users (not /(dashboard)/users)
```

### 7. Layouts with Nested Navigation

Create layouts using `*_layout.dart` files:

```dart
// File: lib/app/dashboard/dashboard_layout.dart
class DashboardLayout extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Row(
        children: [
          NavigationRail(
            destinations: [
              NavigationRailDestination(
                icon: Icon(Icons.people),
                label: Text('Users'),
              ),
              NavigationRailDestination(
                icon: Icon(Icons.settings),
                label: Text('Settings'),
              ),
            ],
            onDestinationSelected: (index) {
              switch (index) {
                case 0:
                  Routefly.navigate(routePaths.dashboard.users);
                  break;
                case 1:
                  Routefly.navigate(routePaths.dashboard.settings);
                  break;
              }
            },
          ),
          Expanded(child: RouterOutlet()), // REQUIRED for nested routes
        ],
      ),
    );
  }
}
```

### 8. Custom Transitions

Define custom transitions per page:

```dart
// In your page file
Route routeBuilder(BuildContext context, RouteSettings settings) {
  return PageRouteBuilder(
    settings: settings, // REQUIRED
    pageBuilder: (_, a1, a2) => const MyPage(),
    transitionsBuilder: (_, a1, a2, child) {
      return FadeTransition(opacity: a1, child: child);
    },
  );
}
```

### 9. Middleware Implementation

Always implement route guards and middleware:

```dart
FutureOr<RouteInformation> authGuard(RouteInformation routeInformation) {
  final isProtectedRoute = routeInformation.uri.path.startsWith('/dashboard');
  final isAuthenticated = AuthService.isLoggedIn();

  if (isProtectedRoute && !isAuthenticated) {
    return routeInformation.redirect(Uri.parse('/login'));
  }

  return routeInformation;
}

FutureOr<RouteInformation> adminGuard(RouteInformation routeInformation) {
  final isAdminRoute = routeInformation.uri.path.startsWith('/admin');
  final isAdmin = AuthService.isAdmin();

  if (isAdminRoute && !isAdmin) {
    return routeInformation.redirect(Uri.parse('/unauthorized'));
  }

  return routeInformation;
}
```

### 10. Code Generation Command

Always run after creating/modifying pages:

```bash
# Generate routes
dart run routefly

# Auto-generate on file changes (recommended during development)
dart run routefly --watch
```

## Required Directory Structure

```
lib/
├── app/                    # REQUIRED - All pages go here
│   ├── home/
│   │   └── home_page.dart  # Creates route: /home
│   ├── 404/
│   │   └── 404_page.dart   # Not found page
│   └── (auth)/             # Route group - doesn't affect URL
│       └── login/
│           └── login_page.dart  # Creates route: /login
└── main.dart
```

## Required Dependencies

```yaml
dependencies:
  routefly: ^3.1.3
```

## Forbidden Patterns

- ❌ Using Navigator.push/pop directly
- ❌ Using go_router, auto_route, or other routing packages
- ❌ Manual route configuration in MaterialApp
- ❌ String-based navigation without type safety
- ❌ Creating routes outside the lib/app directory
- ❌ Forgetting to run dart run routefly after changes
- ❌ Not using @Main() annotation
- ❌ Missing RouterOutlet() in layout files

## Required File Naming Conventions

- Pages: `*_page.dart` (e.g., `home_page.dart`)
- Layouts: `*_layout.dart` (e.g., `dashboard_layout.dart`)
- Dynamic routes: `[param]/page_name.dart` (e.g., `[id]/product_page.dart`)
- Route groups: `(group_name)/` (e.g., `(auth)/`)

## Benefits Enforced

- File-system based routing (like Next.js)
- Type-safe navigation with object notation
- Automatic route generation
- Built-in middleware support
- Nested routing with layouts
- Dynamic route parameters
- Route groups for organization
- Custom transitions per route
- Hot-reload friendly development
- Web-friendly URL structure
