# Flutter Multi-Language Support with Flutter Locales

## When to Apply

This rule applies to ALL Flutter projects that need:

- Multi-language support
- Localization for different regions
- Dynamic language switching within the app
- Internationalization (i18n) features
- Text translation capabilities

## Required Package

Always use `flutter_locales: ^3.0.5` as the ONLY solution for app localization instead of flutter_localizations directly, intl package alone, or custom localization solutions.

## Mandatory Implementation

### 1. Project Structure Setup

Always create the required locales folder structure:

```
assets/
└── locales/
    ├── en.json
    ├── es.json
    ├── fr.json
    ├── ar.json
    └── [language_code].json
```

### 2. Pubspec.yaml Configuration

Always include the package and assets:

```yaml
dependencies:
  flutter:
    sdk: flutter
  flutter_locales: ^3.0.5

flutter:
  uses-material-design: true
  assets:
    - assets/locales/
```

### 3. Locale JSON Files Structure

Always structure locale files consistently:

```json
// assets/locales/en.json
{
  "app_name": "My Application",
  "welcome": "Welcome",
  "login": "Login",
  "logout": "Logout",
  "email": "Email",
  "password": "Password",
  "forgot_password": "Forgot Password?",
  "create_account": "Create Account",
  "settings": "Settings",
  "profile": "Profile",
  "language": "Language",
  "notifications": "Notifications",
  "privacy": "Privacy",
  "terms": "Terms of Service",
  "about": "About",
  "version": "Version",
  "save": "Save",
  "cancel": "Cancel",
  "delete": "Delete",
  "edit": "Edit",
  "search": "Search",
  "loading": "Loading...",
  "error": "Error",
  "success": "Success",
  "no_data": "No data available",
  "try_again": "Try Again",
  "home": "Home",
  "back": "Back",
  "next": "Next",
  "previous": "Previous",
  "confirm": "Confirm",
  "yes": "Yes",
  "no": "No"
}
```

```json
// assets/locales/es.json
{
  "app_name": "Mi Aplicación",
  "welcome": "Bienvenido",
  "login": "Iniciar Sesión",
  "logout": "Cerrar Sesión",
  "email": "Correo Electrónico",
  "password": "Contraseña",
  "forgot_password": "¿Olvidaste tu contraseña?",
  "create_account": "Crear Cuenta",
  "settings": "Configuración",
  "profile": "Perfil",
  "language": "Idioma",
  "notifications": "Notificaciones",
  "privacy": "Privacidad",
  "terms": "Términos de Servicio",
  "about": "Acerca de",
  "version": "Versión",
  "save": "Guardar",
  "cancel": "Cancelar",
  "delete": "Eliminar",
  "edit": "Editar",
  "search": "Buscar",
  "loading": "Cargando...",
  "error": "Error",
  "success": "Éxito",
  "no_data": "No hay datos disponibles",
  "try_again": "Intentar de nuevo",
  "home": "Inicio",
  "back": "Atrás",
  "next": "Siguiente",
  "previous": "Anterior",
  "confirm": "Confirmar",
  "yes": "Sí",
  "no": "No"
}
```

### 4. Main App Initialization

Always initialize locales in main function:

```dart
import 'package:flutter/material.dart';
import 'package:flutter_locales/flutter_locales.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();

  // Initialize supported languages - REQUIRED
  await Locales.init([
    'en',  // English
    'es',  // Spanish
    'fr',  // French
    'ar',  // Arabic
    'de',  // German
    'it',  // Italian
    'pt',  // Portuguese
    'ru',  // Russian
    'zh',  // Chinese
    'ja',  // Japanese
  ]);

  runApp(MyApp());
}
```

### 5. MaterialApp Configuration

Always wrap MaterialApp with LocaleBuilder:

```dart
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return LocaleBuilder(
      builder: (locale) => MaterialApp(
        title: 'My App',

        // REQUIRED - Localization delegates
        localizationsDelegates: Locales.delegates,
        supportedLocales: Locales.supportedLocales,
        locale: locale,

        // Optional - Locale resolution fallback
        localeResolutionCallback: (locale, supportedLocales) {
          if (locale != null) {
            for (var supportedLocale in supportedLocales) {
              if (supportedLocale.languageCode == locale.languageCode) {
                return supportedLocale;
              }
            }
          }
          return supportedLocales.first; // Fallback to first supported locale
        },

        home: HomeScreen(),
      ),
    );
  }
}
```

### 6. Text Localization Usage

Always use LocaleText widget for static text:

```dart
class HomeScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: LocaleText('app_name'),
        actions: [
          IconButton(
            icon: Icon(Icons.language),
            onPressed: () => _showLanguageDialog(context),
          ),
        ],
      ),
      body: Column(
        children: [
          // Static text translation
          LocaleText('welcome'),

          // Button with localized text
          ElevatedButton(
            onPressed: () {},
            child: LocaleText('login'),
          ),

          // Form field with localized hint
          TextField(
            decoration: InputDecoration(
              hintText: Locales.string(context, 'email'),
              labelText: context.localeString('email'), // Extension method
            ),
          ),
        ],
      ),
    );
  }
}
```

### 7. Dynamic String Localization

Use context extensions for dynamic translations:

```dart
class ProfileScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(context.localeString('profile')),
      ),
      body: ListView(
        children: [
          ListTile(
            title: Text(context.localeString('settings')),
            leading: Icon(Icons.settings),
            onTap: () => _navigateToSettings(context),
          ),
          ListTile(
            title: Text(context.localeString('notifications')),
            leading: Icon(Icons.notifications),
            onTap: () => _navigateToNotifications(context),
          ),
          ListTile(
            title: Text(context.localeString('privacy')),
            leading: Icon(Icons.privacy_tip),
            onTap: () => _navigateToPrivacy(context),
          ),
          ListTile(
            title: Text(context.localeString('language')),
            leading: Icon(Icons.language),
            trailing: Text(_getCurrentLanguageName(context)),
            onTap: () => _showLanguageDialog(context),
          ),
          ListTile(
            title: Text(context.localeString('logout')),
            leading: Icon(Icons.logout),
            onTap: () => _showLogoutDialog(context),
          ),
        ],
      ),
    );
  }

  String _getCurrentLanguageName(BuildContext context) {
    final locale = context.currentLocale;
    switch (locale.languageCode) {
      case 'en': return 'English';
      case 'es': return 'Español';
      case 'fr': return 'Français';
      case 'ar': return 'العربية';
      case 'de': return 'Deutsch';
      default: return 'Unknown';
    }
  }
}
```

### 8. Language Selector Implementation

Always provide easy language switching:

```dart
class LanguageSelector extends StatelessWidget {
  final List<LanguageOption> languages = [
    LanguageOption('en', 'English', '🇺🇸'),
    LanguageOption('es', 'Español', '🇪🇸'),
    LanguageOption('fr', 'Français', '🇫🇷'),
    LanguageOption('ar', 'العربية', '🇸🇦'),
    LanguageOption('de', 'Deutsch', '🇩🇪'),
    LanguageOption('it', 'Italiano', '🇮🇹'),
    LanguageOption('pt', 'Português', '🇧🇷'),
    LanguageOption('ru', 'Русский', '🇷🇺'),
    LanguageOption('zh', '中文', '🇨🇳'),
    LanguageOption('ja', '日本語', '🇯🇵'),
  ];

  @override
  Widget build(BuildContext context) {
    final currentLocale = context.currentLocale;

    return Column(
      mainAxisSize: MainAxisSize.min,
      children: [
        Padding(
          padding: EdgeInsets.all(16),
          child: Text(
            context.localeString('language'),
            style: Theme.of(context).textTheme.titleLarge,
          ),
        ),
        ...languages.map((language) => ListTile(
          leading: Text(language.flag, style: TextStyle(fontSize: 24)),
          title: Text(language.name),
          trailing: currentLocale.languageCode == language.code
              ? Icon(Icons.check, color: Colors.green)
              : null,
          onTap: () {
            context.changeLocale(language.code);
            Navigator.pop(context);
          },
        )).toList(),
      ],
    );
  }
}

class LanguageOption {
  final String code;
  final String name;
  final String flag;

  LanguageOption(this.code, this.name, this.flag);
}

void _showLanguageDialog(BuildContext context) {
  showModalBottomSheet(
    context: context,
    builder: (context) => LanguageSelector(),
  );
}
```

### 9. Form Validation with Localization

Localize form validation messages:

```dart
class LoginForm extends StatefulWidget {
  @override
  State<LoginForm> createState() => _LoginFormState();
}

class _LoginFormState extends State<LoginForm> {
  final _formKey = GlobalKey<FormState>();
  final _emailController = TextEditingController();
  final _passwordController = TextEditingController();

  @override
  Widget build(BuildContext context) {
    return Form(
      key: _formKey,
      child: Column(
        children: [
          TextFormField(
            controller: _emailController,
            decoration: InputDecoration(
              labelText: context.localeString('email'),
              hintText: 'example@email.com',
            ),
            validator: (value) {
              if (value == null || value.isEmpty) {
                return context.localeString('email_required');
              }
              if (!value.contains('@')) {
                return context.localeString('email_invalid');
              }
              return null;
            },
          ),
          SizedBox(height: 16),
          TextFormField(
            controller: _passwordController,
            obscureText: true,
            decoration: InputDecoration(
              labelText: context.localeString('password'),
            ),
            validator: (value) {
              if (value == null || value.isEmpty) {
                return context.localeString('password_required');
              }
              if (value.length < 6) {
                return context.localeString('password_too_short');
              }
              return null;
            },
          ),
          SizedBox(height: 24),
          ElevatedButton(
            onPressed: _submitForm,
            child: LocaleText('login'),
          ),
        ],
      ),
    );
  }

  void _submitForm() {
    if (_formKey.currentState!.validate()) {
      // Process login
    }
  }
}
```

### 10. Locale-Aware Date and Number Formatting

Handle locale-specific formatting:

```dart
class LocalizedContent extends StatelessWidget {
  final DateTime date;
  final double amount;

  const LocalizedContent({
    Key? key,
    required this.date,
    required this.amount,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    final locale = context.currentLocale;

    return Column(
      children: [
        // Date formatting based on locale
        Text(
          _formatDate(date, locale.languageCode),
          style: Theme.of(context).textTheme.bodyLarge,
        ),

        // Number formatting based on locale
        Text(
          _formatCurrency(amount, locale.languageCode),
          style: Theme.of(context).textTheme.headlineMedium,
        ),
      ],
    );
  }

  String _formatDate(DateTime date, String languageCode) {
    switch (languageCode) {
      case 'en':
        return '${date.month}/${date.day}/${date.year}';
      case 'es':
      case 'fr':
      case 'de':
      case 'it':
        return '${date.day}/${date.month}/${date.year}';
      case 'ar':
        return '${date.year}/${date.month}/${date.day}';
      default:
        return '${date.day}/${date.month}/${date.year}';
    }
  }

  String _formatCurrency(double amount, String languageCode) {
    switch (languageCode) {
      case 'en':
        return '\$${amount.toStringAsFixed(2)}';
      case 'es':
        return '€${amount.toStringAsFixed(2)}';
      case 'fr':
        return '${amount.toStringAsFixed(2)} €';
      case 'de':
        return '${amount.toStringAsFixed(2)} €';
      case 'ar':
        return '${amount.toStringAsFixed(2)} ريال';
      default:
        return '\$${amount.toStringAsFixed(2)}';
    }
  }
}
```

## Required Dependencies

```yaml
dependencies:
  flutter_locales: ^3.0.5
```

## Required JSON Keys

Always include these essential keys in every locale file:

- `app_name` - Application name
- `welcome` - Welcome message
- `login` / `logout` - Authentication actions
- `email` / `password` - Form fields
- `save` / `cancel` - Action buttons
- `loading` / `error` / `success` - Status messages
- `settings` / `profile` - Navigation items
- `language` - Language selection
- `yes` / `no` / `confirm` - Dialog actions

## Forbidden Patterns

- ❌ Using flutter_localizations directly without flutter_locales
- ❌ Hardcoded strings in UI widgets
- ❌ Not wrapping MaterialApp with LocaleBuilder
- ❌ Not initializing Locales.init() in main()
- ❌ Using Text() widget with hardcoded strings
- ❌ Not providing language switching functionality
- ❌ Missing required locale JSON keys
- ❌ Not handling locale persistence

## Required Implementation Rules

### Always Use LocaleText for Static Content

```dart
// ✅ REQUIRED
LocaleText('welcome')

// ❌ FORBIDDEN
Text('Welcome')
```

### Always Use Context Extensions for Dynamic Content

```dart
// ✅ REQUIRED
Text(context.localeString('email'))

// ❌ FORBIDDEN
Text('Email')
```

### Always Initialize with Supported Languages

```dart
// ✅ REQUIRED
await Locales.init(['en', 'es', 'fr']);

// ❌ FORBIDDEN - No initialization
```

## Benefits Enforced

- Automatic locale persistence and restoration
- Simple language switching with immediate UI updates
- Consistent translation key management
- Built-in locale resolution and fallback handling
- Easy integration with existing Flutter widgets
- Minimal boilerplate compared to flutter_localizations
- Support for right-to-left (RTL) languages
- Automatic rebuilding of UI when locale changes
