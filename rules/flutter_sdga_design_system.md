# Flutter SDGA Design System

## When to Apply

This rule applies to ALL Flutter projects that need to comply with:

- Saudi Digital Government Authority (SDGA) design guidelines
- Government or public sector applications in Saudi Arabia
- Applications requiring SDGA brand compliance
- Consistent government-standard UI components

## Required Package

Always use `sdga_design_system: ^0.0.2` as the PRIMARY design system instead of Material Design components, custom widgets, or other design systems when SDGA compliance is required.

## Mandatory Implementation

### 1. Theme Configuration

Always configure SDGA theme in your MaterialApp:

```dart
import 'package:flutter/material.dart';
import 'package:sdga_design_system/sdga_design_system.dart';

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'SDGA App',

      // Method 1: Using copyWith
      theme: ThemeData.light().copyWith(
        extensions: [SDGATheme.light()],
      ),
      darkTheme: ThemeData.dark().copyWith(
        extensions: [SDGATheme.dark()],
      ),

      // Method 2: Using extension method (Alternative)
      // theme: ThemeData.light().applySDGATheme(),
      // darkTheme: ThemeData.dark().applySDGATheme(),

      home: HomeScreen(),
    );
  }
}
```

### 2. Button Components

Always use SDGA button variants instead of Material buttons:

```dart
class ButtonExamples extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // Primary Brand Button
        SDGAButton(
          style: SDGAButtonStyle.primaryBrand,
          onPressed: () => _handlePrimaryAction(),
          child: Text('Primary Action'),
        ),

        SizedBox(height: SDGANumbers.spacingMD),

        // Secondary Button
        SDGAButton(
          style: SDGAButtonStyle.secondary,
          onPressed: () => _handleSecondaryAction(),
          child: Text('Secondary Action'),
        ),

        SizedBox(height: SDGANumbers.spacingMD),

        // Destructive Button
        SDGAButton.destructive(
          style: SDGADestructiveButtonStyle.primary,
          onPressed: () => _handleDestructiveAction(),
          child: Text('Delete'),
        ),

        SizedBox(height: SDGANumbers.spacingMD),

        // Icon Button
        SDGAButton.icon(
          icon: Icon(Icons.add),
          onPressed: () => _handleIconAction(),
        ),

        SizedBox(height: SDGANumbers.spacingMD),

        // Floating Action Button
        SDGAButton.fab(
          style: SDGAFabButtonStyle.primaryBrand,
          icon: Icon(Icons.camera),
          onPressed: () => _handleFabAction(),
        ),
      ],
    );
  }

  void _handlePrimaryAction() {
    // Primary action logic
  }

  void _handleSecondaryAction() {
    // Secondary action logic
  }

  void _handleDestructiveAction() {
    // Destructive action logic
  }

  void _handleIconAction() {
    // Icon action logic
  }

  void _handleFabAction() {
    // FAB action logic
  }
}
```

### 3. Form Components

Always use SDGA form elements:

```dart
class FormExample extends StatefulWidget {
  @override
  State<FormExample> createState() => _FormExampleState();
}

class _FormExampleState extends State<FormExample> {
  final _formKey = GlobalKey<FormState>();
  final _nameController = TextEditingController();
  final _emailController = TextEditingController();
  bool _acceptTerms = false;
  String _selectedOption = 'option1';

  @override
  Widget build(BuildContext context) {
    return Form(
      key: _formKey,
      child: Padding(
        padding: EdgeInsets.all(SDGANumbers.spacingXL),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.stretch,
          children: [
            // Text Field
            SDGATextField(
              controller: _nameController,
              decoration: SDGAInputDecoration(
                labelText: 'Full Name',
                hintText: 'Enter your full name',
                helperText: 'As it appears on official documents',
                prefixIcon: Icon(Icons.person),
              ),
              validator: (value) {
                if (value == null || value.isEmpty) {
                  return 'Name is required';
                }
                return null;
              },
            ),

            SizedBox(height: SDGANumbers.spacingLG),

            // Email Field
            SDGATextField(
              controller: _emailController,
              decoration: SDGAInputDecoration(
                labelText: 'Email Address',
                hintText: 'user@example.com',
                prefix: SDGAInputAffix.text('Email:'),
                prefixIcon: Icon(Icons.email),
              ),
              keyboardType: TextInputType.emailAddress,
              validator: (value) {
                if (value == null || value.isEmpty) {
                  return 'Email is required';
                }
                if (!value.contains('@')) {
                  return 'Invalid email format';
                }
                return null;
              },
            ),

            SizedBox(height: SDGANumbers.spacingLG),

            // Checkbox
            SDGACheckboxListTile(
              title: Text('I accept the terms and conditions'),
              value: _acceptTerms,
              onChanged: (value) {
                setState(() {
                  _acceptTerms = value ?? false;
                });
              },
            ),

            SizedBox(height: SDGANumbers.spacingLG),

            // Radio Group
            Text('Select an option:', style: SDGATextStyles.textMediumSemiBold),
            SDGARadioListTile<String>(
              title: Text('Option 1'),
              value: 'option1',
              groupValue: _selectedOption,
              onChanged: (value) {
                setState(() {
                  _selectedOption = value!;
                });
              },
            ),
            SDGARadioListTile<String>(
              title: Text('Option 2'),
              value: 'option2',
              groupValue: _selectedOption,
              onChanged: (value) {
                setState(() {
                  _selectedOption = value!;
                });
              },
            ),

            SizedBox(height: SDGANumbers.spacing2XL),

            // Submit Button
            SDGAButton(
              style: SDGAButtonStyle.primaryBrand,
              onPressed: _acceptTerms ? _submitForm : null,
              child: Text('Submit Form'),
            ),
          ],
        ),
      ),
    );
  }

  void _submitForm() {
    if (_formKey.currentState!.validate()) {
      // Process form
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text('Form submitted successfully')),
      );
    }
  }
}
```

### 4. Card and Content Display

Use SDGA cards for content presentation:

```dart
class ContentCards extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return ListView(
      padding: EdgeInsets.all(SDGANumbers.spacingXL),
      children: [
        // Product Card
        SDGACard(
          title: Text('Product Name', style: SDGATextStyles.textLargeSemiBold),
          padChildHorizontally: false,
          child: Column(
            mainAxisSize: MainAxisSize.min,
            children: [
              // Tags Row
              SizedBox(
                height: 24,
                child: ListView.separated(
                  padding: EdgeInsetsDirectional.only(
                    start: SDGANumbers.spacingXL,
                  ),
                  scrollDirection: Axis.horizontal,
                  itemCount: 3,
                  itemBuilder: (context, index) => SDGATag(
                    size: SDGATagSize.small,
                    label: Text('Tag ${index + 1}'),
                  ),
                  separatorBuilder: (context, index) =>
                    SizedBox(width: SDGANumbers.spacingMD),
                ),
              ),

              SizedBox(height: SDGANumbers.spacing3XL),

              // Rating and Reviews
              Padding(
                padding: EdgeInsets.symmetric(
                  horizontal: SDGANumbers.spacingXL,
                ),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.stretch,
                  children: [
                    SDGARatingBar(
                      size: SDGAWidgetSize.small,
                      useBrandColors: false,
                      value: 4.5,
                    ),
                    SizedBox(height: SDGANumbers.spacingXS),
                    Text(
                      '124 reviews',
                      style: SDGATextStyles.textExtraSmallRegular.copyWith(
                        color: SDGAColorScheme.of(context)
                            .links
                            .iconNeutralVisited,
                      ),
                    ),
                  ],
                ),
              ),
            ],
          ),
        ),

        SizedBox(height: SDGANumbers.spacingXL),

        // Information Card
        SDGACard(
          title: Text('Service Information'),
          child: Column(
            children: [
              SDGAListTile(
                leading: SDGAAvatar(
                  child: Icon(Icons.info),
                ),
                title: Text('Service Details'),
                subtitle: Text('Complete information about the service'),
                trailing: Icon(Icons.arrow_forward_ios),
                onTap: () => _navigateToDetails(),
              ),
              SDGAListTile(
                leading: SDGAAvatar(
                  child: Icon(Icons.schedule),
                ),
                title: Text('Operating Hours'),
                subtitle: Text('Sunday to Thursday, 8:00 AM - 4:00 PM'),
              ),
            ],
          ),
        ),
      ],
    );
  }

  void _navigateToDetails() {
    // Navigate to details page
  }
}
```

### 5. Navigation Components

Implement SDGA navigation patterns:

```dart
class NavigationExample extends StatefulWidget {
  @override
  State<NavigationExample> createState() => _NavigationExampleState();
}

class _NavigationExampleState extends State<NavigationExample> {
  int _currentIndex = 0;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('SDGA Navigation'),
        backgroundColor: SDGAColorScheme.of(context).backgrounds.primaryBrand,
      ),
      drawer: SDGADrawer(
        header: SDGADrawerHeader(
          title: Text('Username', maxLines: 1),
          description: Text(
            'user@gov.sa',
            overflow: TextOverflow.ellipsis,
            maxLines: 1,
          ),
        ),
        items: [
          SDGADrawerItem(
            label: 'Home',
            leading: Icon(Icons.home),
            isSelected: _currentIndex == 0,
            onTap: () {
              setState(() => _currentIndex = 0);
              Navigator.pop(context);
            },
          ),
          SDGADrawerItem(
            label: 'Services',
            leading: Icon(Icons.business),
            isSelected: _currentIndex == 1,
            onTap: () {
              setState(() => _currentIndex = 1);
              Navigator.pop(context);
            },
          ),
          SDGADrawerItem(
            label: 'Profile',
            leading: Icon(Icons.person),
            isSelected: _currentIndex == 2,
            onTap: () {
              setState(() => _currentIndex = 2);
              Navigator.pop(context);
            },
          ),
          SDGADrawerItem(
            label: 'Settings',
            leading: Icon(Icons.settings),
            isSelected: _currentIndex == 3,
            onTap: () {
              setState(() => _currentIndex = 3);
              Navigator.pop(context);
            },
          ),
        ],
      ),
      body: _buildCurrentPage(),
    );
  }

  Widget _buildCurrentPage() {
    switch (_currentIndex) {
      case 0:
        return _buildHomePage();
      case 1:
        return _buildServicesPage();
      case 2:
        return _buildProfilePage();
      case 3:
        return _buildSettingsPage();
      default:
        return Container();
    }
  }

  Widget _buildHomePage() {
    return SingleChildScrollView(
      padding: EdgeInsets.all(SDGANumbers.spacingXL),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.stretch,
        children: [
          SDGABanner(
            type: SDGABannerType.info,
            title: 'Welcome to Government Services',
            description: 'Access all your government services in one place.',
          ),
          SizedBox(height: SDGANumbers.spacingXL),
          _buildQuickActions(),
        ],
      ),
    );
  }

  Widget _buildQuickActions() {
    return SDGACard(
      title: Text('Quick Actions'),
      child: Column(
        children: [
          SDGAButton(
            style: SDGAButtonStyle.primaryBrand,
            onPressed: () {},
            child: Text('Apply for Service'),
          ),
          SizedBox(height: SDGANumbers.spacingMD),
          SDGAButton(
            style: SDGAButtonStyle.secondary,
            onPressed: () {},
            child: Text('Track Application'),
          ),
        ],
      ),
    );
  }

  Widget _buildServicesPage() {
    return Center(child: Text('Services Page'));
  }

  Widget _buildProfilePage() {
    return Center(child: Text('Profile Page'));
  }

  Widget _buildSettingsPage() {
    return Center(child: Text('Settings Page'));
  }
}
```

### 6. Feedback and Notification Components

Use SDGA feedback elements:

```dart
class FeedbackComponents extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: EdgeInsets.all(SDGANumbers.spacingXL),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.stretch,
        children: [
          // Alert Banner
          SDGAInlineAlert(
            type: SDGAInlineAlertType.success,
            title: 'Success',
            description: 'Your application has been submitted successfully.',
            onClose: () => _closeAlert(),
          ),

          SizedBox(height: SDGANumbers.spacingLG),

          // Warning Alert
          SDGAInlineAlert(
            type: SDGAInlineAlertType.warning,
            title: 'Warning',
            description: 'Please review your information before submitting.',
          ),

          SizedBox(height: SDGANumbers.spacingLG),

          // Error Alert
          SDGAInlineAlert(
            type: SDGAInlineAlertType.error,
            title: 'Error',
            description: 'There was an error processing your request.',
          ),

          SizedBox(height: SDGANumbers.spacingXL),

          // Show Toast Button
          SDGAButton(
            style: SDGAButtonStyle.primaryBrand,
            onPressed: () => _showToast(context),
            child: Text('Show Notification'),
          ),

          SizedBox(height: SDGANumbers.spacingMD),

          // Show Modal Button
          SDGAButton(
            style: SDGAButtonStyle.secondary,
            onPressed: () => _showModal(context),
            child: Text('Show Modal'),
          ),
        ],
      ),
    );
  }

  void _closeAlert() {
    // Handle alert close
  }

  void _showToast(BuildContext context) {
    SDGANotificationToast.show(
      context,
      type: SDGANotificationToastType.success,
      title: 'Success',
      message: 'Operation completed successfully.',
    );
  }

  void _showModal(BuildContext context) {
    showDialog(
      context: context,
      builder: (context) => SDGAModal(
        title: 'Confirm Action',
        content: Text('Are you sure you want to proceed?'),
        actions: [
          SDGAButton(
            style: SDGAButtonStyle.secondary,
            onPressed: () => Navigator.pop(context),
            child: Text('Cancel'),
          ),
          SDGAButton(
            style: SDGAButtonStyle.primaryBrand,
            onPressed: () {
              Navigator.pop(context);
              // Perform action
            },
            child: Text('Confirm'),
          ),
        ],
      ),
    );
  }
}
```

### 7. Color and Typography Usage

Always use SDGA colors and typography:

```dart
class DesignTokensExample extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final colorScheme = SDGAColorScheme.of(context);

    return Scaffold(
      backgroundColor: colorScheme.backgrounds.neutral,
      body: Padding(
        padding: EdgeInsets.all(SDGANumbers.spacingXL),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // Typography Examples
            Text(
              'Heading Large',
              style: SDGATextStyles.headingLarge,
            ),
            SizedBox(height: SDGANumbers.spacingMD),

            Text(
              'Heading Medium',
              style: SDGATextStyles.headingMedium,
            ),
            SizedBox(height: SDGANumbers.spacingMD),

            Text(
              'Body text with regular weight for general content.',
              style: SDGATextStyles.textMediumRegular,
            ),
            SizedBox(height: SDGANumbers.spacingMD),

            Text(
              'Small text for captions and helper text.',
              style: SDGATextStyles.textSmallRegular.copyWith(
                color: colorScheme.content.contentSecondary,
              ),
            ),

            SizedBox(height: SDGANumbers.spacing2XL),

            // Color Boxes
            Row(
              children: [
                _buildColorBox(
                  'Primary Brand',
                  colorScheme.backgrounds.primaryBrand,
                ),
                SizedBox(width: SDGANumbers.spacingMD),
                _buildColorBox(
                  'Secondary',
                  colorScheme.backgrounds.secondary,
                ),
                SizedBox(width: SDGANumbers.spacingMD),
                _buildColorBox(
                  'Success',
                  colorScheme.status.success,
                ),
              ],
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildColorBox(String label, Color color) {
    return Column(
      children: [
        Container(
          width: 60,
          height: 60,
          decoration: BoxDecoration(
            color: color,
            borderRadius: BorderRadius.circular(8),
            boxShadow: SDGAShadows.medium,
          ),
        ),
        SizedBox(height: SDGANumbers.spacingXS),
        Text(
          label,
          style: SDGATextStyles.textExtraSmallRegular,
          textAlign: TextAlign.center,
        ),
      ],
    );
  }
}
```

## Required Dependencies

```yaml
dependencies:
  sdga_design_system: ^0.0.2
```

## Forbidden Patterns

- ❌ Using Material Design components directly (Button, Card, TextField, etc.)
- ❌ Custom UI components that don't follow SDGA guidelines
- ❌ Non-SDGA colors, typography, or spacing values
- ❌ Material theme without SDGA extensions
- ❌ Custom design tokens instead of SDGA design tokens
- ❌ Non-compliant navigation patterns
- ❌ Using other design systems alongside SDGA

## Required Component Migration

- **Button** → **SDGAButton**
- **Card** → **SDGACard**
- **TextField** → **SDGATextField**
- **ListTile** → **SDGAListTile**
- **Checkbox** → **SDGACheckbox**
- **Radio** → **SDGARadio**
- **Switch** → **SDGASwitch**
- **Drawer** → **SDGADrawer**
- **AppBar** → Custom implementation with SDGA colors

## Required Design Tokens

- **Colors**: Use `SDGAColorScheme.of(context)` or `SDGAColors`
- **Typography**: Use `SDGATextStyles` for all text styling
- **Spacing**: Use `SDGANumbers` for consistent spacing
- **Shadows**: Use `SDGAShadows` for elevation effects

## Benefits Enforced

- Full compliance with Saudi Digital Government Authority guidelines
- Consistent government-standard user interface
- Accessibility compliance built into components
- Professional appearance suitable for government applications
- Standardized interaction patterns across government services
- Arabic and RTL language support
- Brand consistency across all government digital services
