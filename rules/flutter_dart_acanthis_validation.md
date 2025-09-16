# Dart/Flutter Validation with Acanthis

## When to Apply

This rule applies to ALL Dart/Flutter projects that need:

- Form validation
- Data validation
- Input sanitization
- Type-safe validation schemas
- API request/response validation
- Configuration validation

## Required Package

Always use `acanthis: ^1.4.3` as the PRIMARY validation solution instead of manual validation, custom validators, or other validation packages.

## Mandatory Implementation

### 1. Import and Usage

Always import acanthis with a prefix for clarity:

```dart
import 'package:acanthis/acanthis.dart' as v;
```

### 2. Basic Validation Patterns

Use Acanthis validators for all data validation:

```dart
// String validation
final nameValidator = v.string().min(2).max(50).trim();
final emailValidator = v.string().email().min(5).max(100);
final passwordValidator = v.string().min(8).max(128).regex(
  RegExp(r'^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]'),
);

// Number validation
final ageValidator = v.integer().min(0).max(120);
final priceValidator = v.double().positive();

// Boolean validation
final acceptTermsValidator = v.boolean().isTrue();
```

### 3. Form Validation Schema

Always create validation schemas for forms:

```dart
class RegisterFormValidator {
  static final schema = v.object({
    'firstName': v.string().min(2).max(50).trim(),
    'lastName': v.string().min(2).max(50).trim(),
    'email': v.string().email().toLowerCase(),
    'password': v.string().min(8).regex(
      RegExp(r'^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]'),
    ),
    'confirmPassword': v.string().min(8),
    'age': v.integer().min(13).max(120),
    'acceptTerms': v.boolean().isTrue(),
  }).refine((data) {
    // Custom validation for password confirmation
    if (data['password'] != data['confirmPassword']) {
      throw v.ValidationException('Passwords do not match');
    }
    return data;
  });

  static ValidationResult<Map<String, dynamic>> validate(Map<String, dynamic> data) {
    return schema.tryParse(data);
  }
}
```

### 4. Model Validation

Always validate model data with schemas:

```dart
class User {
  final String id;
  final String email;
  final String name;
  final int age;

  const User({
    required this.id,
    required this.email,
    required this.name,
    required this.age,
  });

  // Validation schema
  static final validator = v.object({
    'id': v.string().uuid(),
    'email': v.string().email(),
    'name': v.string().min(1).max(100).trim(),
    'age': v.integer().min(0).max(150),
  });

  // Factory constructor with validation
  factory User.fromJson(Map<String, dynamic> json) {
    final result = validator.tryParse(json);

    if (!result.success) {
      throw ArgumentError('Invalid user data: ${result.errors}');
    }

    final data = result.data!;
    return User(
      id: data['id'],
      email: data['email'],
      name: data['name'],
      age: data['age'],
    );
  }

  // Validation method
  static bool isValid(Map<String, dynamic> data) {
    return validator.tryParse(data).success;
  }
}
```

### 5. API Request Validation

Always validate API requests and responses:

```dart
class ApiService {
  // Request validation
  static final createUserRequest = v.object({
    'email': v.string().email(),
    'name': v.string().min(1).max(100).trim(),
    'age': v.integer().min(0).max(150).optional(),
  });

  // Response validation
  static final userResponse = v.object({
    'id': v.string().uuid(),
    'email': v.string().email(),
    'name': v.string(),
    'createdAt': v.string().datetime(),
  });

  static Future<User> createUser(Map<String, dynamic> userData) async {
    // Validate request data
    final requestResult = createUserRequest.tryParse(userData);
    if (!requestResult.success) {
      throw ArgumentError('Invalid request data: ${requestResult.errors}');
    }

    final response = await httpClient.post('/users', body: requestResult.data);

    // Validate response data
    final responseResult = userResponse.tryParse(response.data);
    if (!responseResult.success) {
      throw Exception('Invalid response data: ${responseResult.errors}');
    }

    return User.fromJson(responseResult.data!);
  }
}
```

### 6. Complex Validation Schemas

Use composition for complex validations:

```dart
// Address validation
final addressValidator = v.object({
  'street': v.string().min(1).max(100),
  'city': v.string().min(1).max(50),
  'state': v.string().length(2), // US state code
  'zipCode': v.string().regex(RegExp(r'^\d{5}(-\d{4})?$')),
  'country': v.string().length(2), // ISO country code
});

// Profile validation with nested objects
final profileValidator = v.object({
  'personalInfo': v.object({
    'firstName': v.string().min(1).max(50),
    'lastName': v.string().min(1).max(50),
    'dateOfBirth': v.string().datetime(),
  }),
  'contact': v.object({
    'email': v.string().email(),
    'phone': v.string().regex(RegExp(r'^\+?[\d\s\-\(\)]{10,}$')),
  }),
  'address': addressValidator,
  'preferences': v.object({
    'newsletter': v.boolean(),
    'notifications': v.boolean(),
    'theme': v.string().oneOf(['light', 'dark', 'auto']),
  }).optional(),
});
```

### 7. Array and List Validation

Always validate collections:

```dart
// Array of strings
final tagsValidator = v.array(v.string().min(1).max(20)).min(1).max(10);

// Array of objects
final itemsValidator = v.array(
  v.object({
    'id': v.string().uuid(),
    'name': v.string().min(1),
    'quantity': v.integer().positive(),
    'price': v.double().positive(),
  })
).min(1);

// Order validation
final orderValidator = v.object({
  'id': v.string().uuid(),
  'customerId': v.string().uuid(),
  'items': itemsValidator,
  'total': v.double().positive(),
  'status': v.string().oneOf(['pending', 'processing', 'shipped', 'delivered']),
  'tags': tagsValidator.optional(),
});
```

### 8. Form Field Validation Integration

Integrate with Flutter form validation:

```dart
class FormValidator {
  static String? validateEmail(String? value) {
    if (value == null || value.isEmpty) {
      return 'Email is required';
    }

    final result = v.string().email().tryParse(value);
    return result.success ? null : 'Please enter a valid email';
  }

  static String? validatePassword(String? value) {
    if (value == null || value.isEmpty) {
      return 'Password is required';
    }

    final result = v.string()
        .min(8, 'Password must be at least 8 characters')
        .regex(
          RegExp(r'^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]'),
          'Password must contain uppercase, lowercase, number and special character',
        )
        .tryParse(value);

    return result.success ? null : result.errors.first;
  }

  static String? validateAge(String? value) {
    if (value == null || value.isEmpty) {
      return 'Age is required';
    }

    final ageResult = v.string().transform(int.tryParse).tryParse(value);
    if (!ageResult.success) {
      return 'Please enter a valid number';
    }

    final validationResult = v.integer().min(13).max(120).tryParse(ageResult.data!);
    return validationResult.success ? null : 'Age must be between 13 and 120';
  }
}
```

### 9. Error Handling Pattern

Always handle validation errors consistently:

```dart
class ValidationHelper {
  static void handleValidationResult<T>(ValidationResult<T> result, {
    required Function(T data) onSuccess,
    required Function(List<String> errors) onError,
  }) {
    if (result.success) {
      onSuccess(result.data!);
    } else {
      onError(result.errors);
    }
  }

  static Map<String, String> mapFieldErrors(List<String> errors) {
    final fieldErrors = <String, String>{};

    for (final error in errors) {
      // Parse error message to extract field name
      final parts = error.split(':');
      if (parts.length >= 2) {
        final fieldName = parts[0].trim();
        final message = parts[1].trim();
        fieldErrors[fieldName] = message;
      }
    }

    return fieldErrors;
  }
}
```

### 10. Configuration Validation

Validate app configuration:

```dart
class AppConfig {
  static final configValidator = v.object({
    'apiBaseUrl': v.string().url(),
    'apiTimeout': v.integer().positive(),
    'enableLogging': v.boolean(),
    'theme': v.string().oneOf(['light', 'dark']),
    'supportedLanguages': v.array(v.string().length(2)).min(1),
    'features': v.object({
      'push_notifications': v.boolean(),
      'analytics': v.boolean(),
      'offline_mode': v.boolean(),
    }),
  });

  static AppConfig fromMap(Map<String, dynamic> config) {
    final result = configValidator.tryParse(config);

    if (!result.success) {
      throw ConfigurationException('Invalid app configuration: ${result.errors}');
    }

    return AppConfig._internal(result.data!);
  }
}
```

## Required Dependencies

```yaml
dependencies:
  acanthis: ^1.4.3
```

## Forbidden Patterns

- ❌ Manual validation with if/else statements
- ❌ Using RegExp directly without Acanthis wrapper
- ❌ Returning bool from validation functions (use ValidationResult)
- ❌ Not validating API request/response data
- ❌ Custom validation libraries or solutions
- ❌ Ignoring validation results
- ❌ Not using type-safe schemas
- ❌ Validating data in multiple places without schemas

## Required Naming Conventions

- Validators: `*Validator` (e.g., `userValidator`, `emailValidator`)
- Schemas: `*Schema` for complex objects
- Form validators: Static methods in `*FormValidator` classes
- Results: Always use `ValidationResult<T>` type

## Benefits Enforced

- Type-safe validation with compile-time checking
- Composable and reusable validation schemas
- Consistent error handling across the application
- Reduced boilerplate code
- Better maintainability
- Clear separation of validation logic
- Automatic data transformation and sanitization
- Enhanced debugging with detailed error messages
