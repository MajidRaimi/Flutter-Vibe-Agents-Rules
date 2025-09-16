# Flutter Local State Management with Riverpod

## When to Apply

This rule applies to ALL Flutter projects that need:

- Local state management (UI state, form state, etc.)
- Dependency injection
- State sharing between widgets
- Reactive programming patterns
- Testable state logic
- Any state that doesn't require server synchronization

## Required Package

Always use `flutter_riverpod: ^3.0.0` as the ONLY solution for local state management instead of setState, Provider, Bloc, GetX, or other state management solutions.

## Mandatory Implementation

### 1. App Setup with ProviderScope

Always wrap your app with ProviderScope:

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

void main() {
  runApp(
    ProviderScope(
      child: MyApp(),
    ),
  );
}
```

### 2. ConsumerWidget Usage

Always use ConsumerWidget instead of StatelessWidget when accessing providers:

```dart
import 'package:flutter_riverpod/flutter_riverpod.dart';

// ✅ REQUIRED - ConsumerWidget
class HomePage extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final counter = ref.watch(counterProvider);

    return Scaffold(
      appBar: AppBar(title: Text('Counter: $counter')),
      body: Center(
        child: Column(
          children: [
            Text('Count: $counter'),
            ElevatedButton(
              onPressed: () => ref.read(counterProvider.notifier).increment(),
              child: Text('Increment'),
            ),
          ],
        ),
      ),
    );
  }
}

// ❌ FORBIDDEN - StatelessWidget with Consumer
class BadHomePage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Consumer(
      builder: (context, ref, child) {
        // Don't do this
        return Container();
      },
    );
  }
}
```

### 3. ConsumerStatefulWidget for Complex State

Use ConsumerStatefulWidget when you need lifecycle methods:

```dart
class FormPage extends ConsumerStatefulWidget {
  @override
  ConsumerState<FormPage> createState() => _FormPageState();
}

class _FormPageState extends ConsumerState<FormPage> {
  late final TextEditingController _nameController;
  late final TextEditingController _emailController;

  @override
  void initState() {
    super.initState();
    _nameController = TextEditingController();
    _emailController = TextEditingController();

    // Initialize form with existing data
    final user = ref.read(currentUserProvider);
    _nameController.text = user?.name ?? '';
    _emailController.text = user?.email ?? '';
  }

  @override
  void dispose() {
    _nameController.dispose();
    _emailController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    final isLoading = ref.watch(formSubmissionProvider);

    return Scaffold(
      appBar: AppBar(title: Text('Edit Profile')),
      body: Padding(
        padding: EdgeInsets.all(16),
        child: Column(
          children: [
            TextField(
              controller: _nameController,
              decoration: InputDecoration(labelText: 'Name'),
            ),
            SizedBox(height: 16),
            TextField(
              controller: _emailController,
              decoration: InputDecoration(labelText: 'Email'),
            ),
            SizedBox(height: 24),
            ElevatedButton(
              onPressed: isLoading ? null : _submitForm,
              child: isLoading
                  ? CircularProgressIndicator()
                  : Text('Save'),
            ),
          ],
        ),
      ),
    );
  }

  void _submitForm() {
    ref.read(formSubmissionProvider.notifier).submitForm(
      name: _nameController.text,
      email: _emailController.text,
    );
  }
}
```

### 4. StateNotifierProvider for Complex State

Always use StateNotifierProvider for complex mutable state:

```dart
// State class
@immutable
class CounterState {
  const CounterState({
    required this.count,
    required this.isLoading,
    this.error,
  });

  final int count;
  final bool isLoading;
  final String? error;

  CounterState copyWith({
    int? count,
    bool? isLoading,
    String? error,
  }) {
    return CounterState(
      count: count ?? this.count,
      isLoading: isLoading ?? this.isLoading,
      error: error ?? this.error,
    );
  }
}

// StateNotifier
class CounterNotifier extends StateNotifier<CounterState> {
  CounterNotifier() : super(const CounterState(count: 0, isLoading: false));

  void increment() {
    state = state.copyWith(count: state.count + 1);
  }

  void decrement() {
    state = state.copyWith(count: state.count - 1);
  }

  void reset() {
    state = state.copyWith(count: 0);
  }

  Future<void> incrementAsync() async {
    state = state.copyWith(isLoading: true, error: null);

    try {
      // Simulate API call
      await Future.delayed(Duration(seconds: 1));
      state = state.copyWith(count: state.count + 1, isLoading: false);
    } catch (e) {
      state = state.copyWith(
        isLoading: false,
        error: e.toString(),
      );
    }
  }
}

// Provider
final counterProvider = StateNotifierProvider<CounterNotifier, CounterState>((ref) {
  return CounterNotifier();
});
```

### 5. Simple State with StateProvider

Use StateProvider for simple mutable state:

```dart
// Simple boolean state
final isDarkModeProvider = StateProvider<bool>((ref) => false);

// Simple string state
final searchQueryProvider = StateProvider<String>((ref) => '');

// Simple enum state
enum ViewMode { list, grid }
final viewModeProvider = StateProvider<ViewMode>((ref) => ViewMode.list);

// Usage
class SettingsPage extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final isDarkMode = ref.watch(isDarkModeProvider);
    final viewMode = ref.watch(viewModeProvider);

    return Scaffold(
      body: Column(
        children: [
          SwitchListTile(
            title: Text('Dark Mode'),
            value: isDarkMode,
            onChanged: (value) {
              ref.read(isDarkModeProvider.notifier).state = value;
            },
          ),
          ListTile(
            title: Text('View Mode'),
            trailing: DropdownButton<ViewMode>(
              value: viewMode,
              onChanged: (mode) {
                if (mode != null) {
                  ref.read(viewModeProvider.notifier).state = mode;
                }
              },
              items: ViewMode.values.map((mode) =>
                DropdownMenuItem(
                  value: mode,
                  child: Text(mode.name),
                ),
              ).toList(),
            ),
          ),
        ],
      ),
    );
  }
}
```

### 6. Computed Values with Provider

Use regular Provider for derived/computed state:

```dart
// Base providers
final firstNameProvider = StateProvider<String>((ref) => '');
final lastNameProvider = StateProvider<String>((ref) => '');

// Computed provider
final fullNameProvider = Provider<String>((ref) {
  final firstName = ref.watch(firstNameProvider);
  final lastName = ref.watch(lastNameProvider);
  return '$firstName $lastName'.trim();
});

// Shopping cart example
final cartItemsProvider = StateProvider<List<CartItem>>((ref) => []);

final cartTotalProvider = Provider<double>((ref) {
  final items = ref.watch(cartItemsProvider);
  return items.fold(0.0, (total, item) => total + (item.price * item.quantity));
});

final cartItemCountProvider = Provider<int>((ref) {
  final items = ref.watch(cartItemsProvider);
  return items.fold(0, (total, item) => total + item.quantity);
});
```

### 7. Family Providers for Parameterized State

Use family modifiers for providers that need parameters:

```dart
// Todo item provider
final todoProvider = StateProvider.family<Todo, String>((ref, id) {
  // Return todo by ID
  return Todo(id: id, title: '', completed: false);
});

// Filtered todos
final filteredTodosProvider = Provider.family<List<Todo>, TodoFilter>((ref, filter) {
  final todos = ref.watch(todosProvider);

  switch (filter) {
    case TodoFilter.all:
      return todos;
    case TodoFilter.completed:
      return todos.where((todo) => todo.completed).toList();
    case TodoFilter.pending:
      return todos.where((todo) => !todo.completed).toList();
  }
});

// Usage
class TodoList extends ConsumerWidget {
  final TodoFilter filter;

  const TodoList({Key? key, required this.filter}) : super(key: key);

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final todos = ref.watch(filteredTodosProvider(filter));

    return ListView.builder(
      itemCount: todos.length,
      itemBuilder: (context, index) {
        final todo = todos[index];
        return TodoItem(todoId: todo.id);
      },
    );
  }
}
```

### 8. Form Management with Riverpod

Create comprehensive form state management:

```dart
// Form state
@immutable
class LoginFormState {
  const LoginFormState({
    required this.email,
    required this.password,
    required this.isLoading,
    this.emailError,
    this.passwordError,
    this.generalError,
  });

  final String email;
  final String password;
  final bool isLoading;
  final String? emailError;
  final String? passwordError;
  final String? generalError;

  bool get isValid =>
      email.isNotEmpty &&
      password.isNotEmpty &&
      emailError == null &&
      passwordError == null;

  LoginFormState copyWith({
    String? email,
    String? password,
    bool? isLoading,
    String? emailError,
    String? passwordError,
    String? generalError,
  }) {
    return LoginFormState(
      email: email ?? this.email,
      password: password ?? this.password,
      isLoading: isLoading ?? this.isLoading,
      emailError: emailError ?? this.emailError,
      passwordError: passwordError ?? this.passwordError,
      generalError: generalError ?? this.generalError,
    );
  }
}

// Form notifier
class LoginFormNotifier extends StateNotifier<LoginFormState> {
  LoginFormNotifier() : super(const LoginFormState(
    email: '',
    password: '',
    isLoading: false,
  ));

  void updateEmail(String email) {
    final emailError = _validateEmail(email);
    state = state.copyWith(
      email: email,
      emailError: emailError,
      generalError: null,
    );
  }

  void updatePassword(String password) {
    final passwordError = _validatePassword(password);
    state = state.copyWith(
      password: password,
      passwordError: passwordError,
      generalError: null,
    );
  }

  Future<bool> submitForm() async {
    if (!state.isValid) return false;

    state = state.copyWith(isLoading: true, generalError: null);

    try {
      // Simulate login API call
      await Future.delayed(Duration(seconds: 2));

      if (state.email == 'test@test.com' && state.password == 'password') {
        state = state.copyWith(isLoading: false);
        return true;
      } else {
        state = state.copyWith(
          isLoading: false,
          generalError: 'Invalid credentials',
        );
        return false;
      }
    } catch (e) {
      state = state.copyWith(
        isLoading: false,
        generalError: e.toString(),
      );
      return false;
    }
  }

  String? _validateEmail(String email) {
    if (email.isEmpty) return 'Email is required';
    if (!email.contains('@')) return 'Invalid email format';
    return null;
  }

  String? _validatePassword(String password) {
    if (password.isEmpty) return 'Password is required';
    if (password.length < 6) return 'Password must be at least 6 characters';
    return null;
  }
}

// Provider
final loginFormProvider = StateNotifierProvider<LoginFormNotifier, LoginFormState>((ref) {
  return LoginFormNotifier();
});
```

### 9. Listening to Provider Changes

Handle side effects with ref.listen:

```dart
class LoginPage extends ConsumerStatefulWidget {
  @override
  ConsumerState<LoginPage> createState() => _LoginPageState();
}

class _LoginPageState extends ConsumerState<LoginPage> {
  @override
  void initState() {
    super.initState();

    // Listen to authentication changes
    ref.listenManual(authenticationProvider, (previous, next) {
      if (next.isAuthenticated) {
        Navigator.pushReplacementNamed(context, '/home');
      }
    });

    // Listen to form submission results
    ref.listenManual(loginFormProvider, (previous, next) {
      if (previous?.generalError != next.generalError && next.generalError != null) {
        ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(content: Text(next.generalError!)),
        );
      }
    });
  }

  @override
  Widget build(BuildContext context) {
    final formState = ref.watch(loginFormProvider);

    return Scaffold(
      appBar: AppBar(title: Text('Login')),
      body: Padding(
        padding: EdgeInsets.all(16),
        child: Column(
          children: [
            TextField(
              decoration: InputDecoration(
                labelText: 'Email',
                errorText: formState.emailError,
              ),
              onChanged: (value) {
                ref.read(loginFormProvider.notifier).updateEmail(value);
              },
            ),
            SizedBox(height: 16),
            TextField(
              obscureText: true,
              decoration: InputDecoration(
                labelText: 'Password',
                errorText: formState.passwordError,
              ),
              onChanged: (value) {
                ref.read(loginFormProvider.notifier).updatePassword(value);
              },
            ),
            SizedBox(height: 24),
            ElevatedButton(
              onPressed: formState.isValid && !formState.isLoading
                  ? () async {
                      final success = await ref.read(loginFormProvider.notifier).submitForm();
                      if (success) {
                        // Handle success (navigation handled by listener)
                      }
                    }
                  : null,
              child: formState.isLoading
                  ? CircularProgressIndicator()
                  : Text('Login'),
            ),
          ],
        ),
      ),
    );
  }
}
```

### 10. Testing Providers

Always write tests for your providers:

```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

void main() {
  group('CounterNotifier', () {
    test('should start with initial state', () {
      final container = ProviderContainer();
      addTearDown(container.dispose);

      final counterState = container.read(counterProvider);
      expect(counterState.count, 0);
      expect(counterState.isLoading, false);
      expect(counterState.error, null);
    });

    test('should increment count', () {
      final container = ProviderContainer();
      addTearDown(container.dispose);

      container.read(counterProvider.notifier).increment();

      final counterState = container.read(counterProvider);
      expect(counterState.count, 1);
    });

    test('should handle async operations', () async {
      final container = ProviderContainer();
      addTearDown(container.dispose);

      // Start async operation
      final future = container.read(counterProvider.notifier).incrementAsync();

      // Check loading state
      expect(container.read(counterProvider).isLoading, true);

      // Wait for completion
      await future;

      // Check final state
      final finalState = container.read(counterProvider);
      expect(finalState.count, 1);
      expect(finalState.isLoading, false);
      expect(finalState.error, null);
    });
  });
}
```

## Required Dependencies

```yaml
dependencies:
  flutter_riverpod: ^3.0.0

dev_dependencies:
  flutter_test:
    sdk: flutter
```

## Forbidden Patterns

- ❌ Using setState() for state management
- ❌ Using Provider package instead of Riverpod
- ❌ Using Bloc/Cubit for local state
- ❌ Using GetX for state management
- ❌ Creating StatefulWidget for simple state
- ❌ Not wrapping app with ProviderScope
- ❌ Using Consumer widget instead of ConsumerWidget
- ❌ Manual state management without providers

## Required Provider Types

- **StateProvider**: Simple mutable values (bool, String, int, enum)
- **Provider**: Computed/derived values, immutable data
- **StateNotifierProvider**: Complex mutable state with logic
- **FutureProvider**: Async operations (use with server state libraries)
- **StreamProvider**: Real-time data streams

## Required Widget Types

- **ConsumerWidget**: Instead of StatelessWidget when using providers
- **ConsumerStatefulWidget**: Instead of StatefulWidget when using providers
- **Consumer**: Only for small portions of widgets that need providers

## Benefits Enforced

- Reactive programming with automatic UI updates
- Testable state logic separated from UI
- Type-safe state management
- Dependency injection built-in
- No boilerplate compared to other solutions
- Excellent performance with automatic optimization
- Easy state sharing between widgets
- Built-in error handling and loading states
