# Flutter Hooks - Stateful Logic Reuse

## When to Apply

This rule applies to ALL Flutter projects that have:

- Repeated stateful logic across widgets
- Animation controllers
- Text editing controllers
- Focus nodes
- Stream/Future handling
- Lifecycle management
- Custom stateful logic that could be reused

## Required Package

Always use `flutter_hooks: ^0.21.3+1` instead of StatefulWidget when stateful logic can be reused.

## Mandatory Implementation

### 1. Widget Structure

Replace StatefulWidget with HookWidget for any widget that uses hooks:

```dart
// ❌ DON'T - Traditional StatefulWidget
class MyWidget extends StatefulWidget {
  @override
  _MyWidgetState createState() => _MyWidgetState();
}

class _MyWidgetState extends State<MyWidget> {
  // stateful logic here
}

// ✅ DO - Use HookWidget
class MyWidget extends HookWidget {
  @override
  Widget build(BuildContext context) {
    // hooks here
    return Container();
  }
}
```

### 2. Common Hook Patterns

**State Management:**

```dart
// Instead of setState
final counter = useState(0);
// Usage: counter.value++; or counter.value = newValue;
```

**Controllers:**

```dart
// Animation Controller
final animationController = useAnimationController(
  duration: Duration(milliseconds: 300),
);

// Text Editing Controller
final textController = useTextEditingController();

// Focus Node
final focusNode = useFocusNode();

// Scroll Controller
final scrollController = useScrollController();
```

**Async Operations:**

```dart
// Future handling
final futureSnapshot = useFuture(fetchData());

// Stream handling
final streamSnapshot = useStream(dataStream);

// Stream Controller
final streamController = useStreamController<String>();
```

**Effects and Lifecycle:**

```dart
// Side effects (replaces initState, didUpdateWidget, dispose)
useEffect(() {
  // Setup code here
  print('Widget mounted or dependency changed');

  return () {
    // Cleanup code here (replaces dispose)
    print('Cleanup before unmount or dependency change');
  };
}, [dependency]); // Dependencies array - when empty [], runs once like initState

// Listen to value changes
useValueChanged(someValue, (oldValue, newValue) {
  print('Value changed from $oldValue to $newValue');
});
```

**Memoization:**

```dart
// Cache expensive computations
final expensiveValue = useMemoized(() {
  return performExpensiveCalculation();
}, [dependency]);

// Cache function instances
final onPressed = useCallback(() {
  doSomething();
}, [dependency]);
```

### 3. Custom Hook Creation

**Function-based Hooks (Preferred):**

```dart
// Custom hook for form validation
ValueNotifier<String?> useFormField(String initialValue, String? Function(String) validator) {
  final value = useState(initialValue);
  final error = useState<String?>(null);

  useValueChanged(value.value, (_, newValue) {
    error.value = validator(newValue);
  });

  return value;
}

// Usage
final emailField = useFormField('', (value) =>
  value.contains('@') ? null : 'Invalid email');
```

**Class-based Hooks (For Complex Logic):**

```dart
Timer? useTimer(Duration duration, VoidCallback callback) {
  return use(_TimerHook(duration, callback));
}

class _TimerHook extends Hook<Timer?> {
  final Duration duration;
  final VoidCallback callback;

  const _TimerHook(this.duration, this.callback);

  @override
  _TimerHookState createState() => _TimerHookState();
}

class _TimerHookState extends HookState<Timer?, _TimerHook> {
  Timer? timer;

  @override
  void initHook() {
    super.initHook();
    timer = Timer.periodic(hook.duration, (_) => hook.callback());
  }

  @override
  Timer? build(BuildContext context) => timer;

  @override
  void dispose() {
    timer?.cancel();
    super.dispose();
  }
}
```

## Hook Naming Rules

### DO: Always prefix hooks with "use"

```dart
// ✅ Correct
useMyCustomHook();
useFormValidation();
useApiCall();

// ❌ Incorrect
myCustomHook();
formValidation();
apiCall();
```

### DO: Call hooks unconditionally

```dart
// ✅ Correct
Widget build(BuildContext context) {
  final counter = useState(0);
  final controller = useAnimationController();

  if (someCondition) {
    return SomeWidget();
  }
  return AnotherWidget();
}

// ❌ Incorrect - conditional hook calls
Widget build(BuildContext context) {
  if (someCondition) {
    final counter = useState(0); // This will break!
  }
  return Container();
}
```

## Required Dependencies

```yaml
dependencies:
  flutter_hooks: ^0.21.3+1
```

## Migration Guidelines

### From StatefulWidget to HookWidget:

1. Change `extends StatefulWidget` to `extends HookWidget`
2. Remove the State class
3. Move `initState` logic to `useEffect(() {}, [])`
4. Move `dispose` logic to `useEffect` return function
5. Replace `setState()` calls with hook state setters
6. Replace controllers with corresponding `use*Controller` hooks

### Example Migration:

```dart
// Before - StatefulWidget
class Counter extends StatefulWidget {
  @override
  _CounterState createState() => _CounterState();
}

class _CounterState extends State<Counter> {
  int count = 0;
  late Timer timer;

  @override
  void initState() {
    super.initState();
    timer = Timer.periodic(Duration(seconds: 1), (_) {
      setState(() => count++);
    });
  }

  @override
  void dispose() {
    timer.cancel();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Text('$count');
  }
}

// After - HookWidget
class Counter extends HookWidget {
  @override
  Widget build(BuildContext context) {
    final count = useState(0);

    useEffect(() {
      final timer = Timer.periodic(Duration(seconds: 1), (_) {
        count.value++;
      });

      return timer.cancel; // Cleanup
    }, []); // Empty deps = run once

    return Text('${count.value}');
  }
}
```

## Forbidden Patterns

- ❌ Using StatefulWidget when logic could be a reusable hook
- ❌ Calling hooks conditionally
- ❌ Not prefixing custom hooks with "use"
- ❌ Creating controllers manually in build method
- ❌ Manual dispose logic when hooks handle it automatically

## Benefits Enforced

- Code reusability across widgets
- Automatic resource disposal
- Cleaner, more functional code style
- Better testing capabilities
- Reduced boilerplate
- Composable stateful logic
- Hot-reload friendly
