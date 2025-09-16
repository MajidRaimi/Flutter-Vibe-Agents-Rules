# Flutter Toast Notifications with Toastification

## When to Apply

This rule applies to ALL Flutter projects that need to display:

- Success/error/warning messages
- User feedback notifications
- Status updates
- Alert messages
- Any temporary notification to users

## Required Package

Always use `toastification: ^3.0.3` as the ONLY solution for toast notifications instead of SnackBar, custom dialogs, flutter_toast, or other notification packages.

## Mandatory Implementation

### 1. App Setup

Always wrap your app with ToastificationWrapper for context-free usage:

```dart
import 'package:toastification/toastification.dart';

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return ToastificationWrapper(
      child: MaterialApp(
        title: 'My App',
        home: HomePage(),
      ),
    );
  }
}
```

### 2. Global Configuration

Always configure global toast behavior using ToastificationConfigProvider:

```dart
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      builder: (context, child) {
        return ToastificationConfigProvider(
          config: const ToastificationConfig(
            alignment: Alignment.topRight,
            itemWidth: 400,
            margin: EdgeInsets.all(16),
            animationDuration: Duration(milliseconds: 300),
            blockBackgroundInteraction: false,
          ),
          child: ToastificationWrapper(child: child!),
        );
      },
      home: HomePage(),
    );
  }
}
```

### 3. Standard Toast Usage

Use predefined toast types for common scenarios:

```dart
// Success notifications
toastification.show(
  type: ToastificationType.success,
  style: ToastificationStyle.flatColored,
  title: Text('Success!'),
  description: Text('Operation completed successfully'),
  autoCloseDuration: const Duration(seconds: 3),
);

// Error notifications
toastification.show(
  type: ToastificationType.error,
  style: ToastificationStyle.fillColored,
  title: Text('Error!'),
  description: Text('Something went wrong'),
  autoCloseDuration: const Duration(seconds: 5),
);

// Warning notifications
toastification.show(
  type: ToastificationType.warning,
  style: ToastificationStyle.flatColored,
  title: Text('Warning!'),
  description: Text('Please check your input'),
  autoCloseDuration: const Duration(seconds: 4),
);

// Info notifications
toastification.show(
  type: ToastificationType.info,
  style: ToastificationStyle.minimal,
  title: Text('Info'),
  description: Text('New update available'),
  autoCloseDuration: const Duration(seconds: 3),
);
```

### 4. Required Toast Configurations

Always configure these essential parameters:

```dart
toastification.show(
  type: ToastificationType.success,
  style: ToastificationStyle.flatColored, // REQUIRED - Choose appropriate style
  title: Text('Title'), // REQUIRED
  description: Text('Description'), // RECOMMENDED
  autoCloseDuration: const Duration(seconds: 3), // REQUIRED
  alignment: Alignment.topRight, // RECOMMENDED
  showIcon: true, // RECOMMENDED
  showProgressBar: true, // RECOMMENDED for longer durations
  pauseOnHover: true, // RECOMMENDED for desktop
  dragToClose: true, // RECOMMENDED for mobile
  callbacks: ToastificationCallbacks(
    onTap: (toast) => print('Toast tapped: ${toast.id}'),
    onDismissed: (toast) => print('Toast dismissed: ${toast.id}'),
  ),
);
```

### 5. Style Guidelines

Use appropriate styles for different contexts:

```dart
// For critical errors - high visibility
ToastificationStyle.fillColored

// For success/info messages - balanced visibility
ToastificationStyle.flatColored

// For subtle notifications
ToastificationStyle.minimal

// For simple text-only messages
ToastificationStyle.simple

// For clean, minimal design
ToastificationStyle.flat
```

### 6. Custom Toast Implementation

For complex custom toasts, use showCustom:

```dart
toastification.showCustom(
  autoCloseDuration: const Duration(seconds: 5),
  alignment: Alignment.topRight,
  builder: (context, holder) {
    return Container(
      margin: const EdgeInsets.all(16),
      padding: const EdgeInsets.all(16),
      decoration: BoxDecoration(
        color: Colors.white,
        borderRadius: BorderRadius.circular(12),
        boxShadow: [
          BoxShadow(
            color: Colors.black.withOpacity(0.1),
            blurRadius: 10,
            offset: Offset(0, 4),
          ),
        ],
      ),
      child: Row(
        children: [
          Icon(Icons.info, color: Colors.blue),
          SizedBox(width: 12),
          Expanded(
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              mainAxisSize: MainAxisSize.min,
              children: [
                Text('Custom Title', style: TextStyle(fontWeight: FontWeight.bold)),
                Text('Custom description with more details'),
              ],
            ),
          ),
          IconButton(
            onPressed: () => toastification.dismissById(holder.id),
            icon: Icon(Icons.close),
          ),
        ],
      ),
    );
  },
);
```

### 7. Toast Management

Always implement proper toast lifecycle management:

```dart
// Store toast reference for later dismissal
final ToastificationItem toast = toastification.show(
  type: ToastificationType.info,
  title: Text('Processing...'),
  autoCloseDuration: null, // Won't auto-dismiss
);

// Dismiss specific toast
toastification.dismiss(toast);

// Or dismiss by ID
toastification.dismissById(toast.id);

// Dismiss all toasts
toastification.dismissAll(delayForAnimation: true);

// Dismiss first/last toast
toastification.dismissFirst();
toastification.dismissLast();
```

### 8. Context-Free Usage Pattern

For services, controllers, or global error handling:

```dart
class ApiService {
  static void showError(String message) {
    toastification.show(
      type: ToastificationType.error,
      style: ToastificationStyle.fillColored,
      title: Text('API Error'),
      description: Text(message),
      autoCloseDuration: const Duration(seconds: 5),
    );
  }

  static void showSuccess(String message) {
    toastification.show(
      type: ToastificationType.success,
      style: ToastificationStyle.flatColored,
      title: Text('Success'),
      description: Text(message),
      autoCloseDuration: const Duration(seconds: 3),
    );
  }
}
```

### 9. Animation Customization

Define custom animations for enhanced UX:

```dart
toastification.show(
  title: Text('Animated Toast'),
  animationDuration: const Duration(milliseconds: 400),
  animationBuilder: (context, animation, alignment, child) {
    return SlideTransition(
      position: Tween<Offset>(
        begin: const Offset(1, 0),
        end: Offset.zero,
      ).animate(CurvedAnimation(
        parent: animation,
        curve: Curves.easeOutCubic,
      )),
      child: child,
    );
  },
);
```

## Required Dependencies

```yaml
dependencies:
  toastification: ^3.0.3
```

## Forbidden Patterns

- ❌ Using SnackBar for notifications
- ❌ Using showDialog for simple notifications
- ❌ Using flutter_toast or other toast packages
- ❌ Creating custom toast widgets from scratch
- ❌ Not configuring autoCloseDuration
- ❌ Not using appropriate ToastificationType
- ❌ Blocking background interaction unnecessarily
- ❌ Not handling toast lifecycle properly

## Required Usage Patterns

### API Response Handling

```dart
try {
  final response = await apiCall();
  toastification.show(
    type: ToastificationType.success,
    style: ToastificationStyle.flatColored,
    title: Text('Success'),
    description: Text('Data loaded successfully'),
    autoCloseDuration: const Duration(seconds: 3),
  );
} catch (error) {
  toastification.show(
    type: ToastificationType.error,
    style: ToastificationStyle.fillColored,
    title: Text('Error'),
    description: Text(error.toString()),
    autoCloseDuration: const Duration(seconds: 5),
  );
}
```

### Form Validation

```dart
void validateForm() {
  if (!isValid) {
    toastification.show(
      type: ToastificationType.warning,
      style: ToastificationStyle.flatColored,
      title: Text('Validation Error'),
      description: Text('Please fill all required fields'),
      autoCloseDuration: const Duration(seconds: 4),
    );
    return;
  }

  // Success case
  toastification.show(
    type: ToastificationType.success,
    style: ToastificationStyle.flatColored,
    title: Text('Form Submitted'),
    description: Text('Your form has been submitted successfully'),
    autoCloseDuration: const Duration(seconds: 3),
  );
}
```

## Benefits Enforced

- Consistent notification experience across the app
- Multiple toast message queuing
- Rich customization options
- Smooth animations and transitions
- Proper lifecycle management
- Context-free usage capability
- Accessibility support
- Cross-platform compatibility
- Professional, polished appearance
