# Flutter Responsive Design with ScreenUtil

## When to Apply

This rule applies to ALL Flutter projects that need:

- Responsive design across different screen sizes
- Consistent UI scaling on various devices
- Adaptive font sizing
- Screen density adaptation
- Support for tablets, phones, and different screen ratios

## Required Package

Always use `flutter_screenutil: ^5.9.3` as the ONLY solution for responsive design and screen adaptation instead of MediaQuery calculations, manual responsive logic, or other responsive packages.

## Mandatory Implementation

### 1. App Initialization with ScreenUtilInit

Always wrap your MaterialApp with ScreenUtilInit:

```dart
import 'package:flutter/material.dart';
import 'package:flutter_screenutil/flutter_screenutil.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return ScreenUtilInit(
      // REQUIRED - Design size from your UI/UX design (usually 375x812 or 360x690)
      designSize: const Size(375, 812),

      // REQUIRED - Enable text adaptation
      minTextAdapt: true,

      // REQUIRED - Support split screen mode
      splitScreenMode: true,

      builder: (context, child) {
        return MaterialApp(
          debugShowCheckedModeBanner: false,
          title: 'My App',

          // REQUIRED - Apply responsive font scaling to theme
          theme: ThemeData(
            primarySwatch: Colors.blue,
            textTheme: Typography.englishLike2018.apply(
              fontSizeFactor: 1.sp,
            ),
          ),

          home: child,
        );
      },
      child: HomePage(),
    );
  }
}
```

### 2. Responsive Sizing with Extension Methods

Always use ScreenUtil extension methods for all sizing:

```dart
class ResponsiveWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        // Responsive height
        toolbarHeight: 60.h,
        title: Text(
          'Responsive App',
          // Responsive font size
          style: TextStyle(fontSize: 20.sp),
        ),
      ),
      body: Padding(
        // Responsive padding
        padding: EdgeInsets.all(16.w),
        child: Column(
          children: [
            // Responsive container
            Container(
              // Width adapts to screen width
              width: 300.w,
              // Height adapts to screen width (for consistent aspect ratio)
              height: 200.w,
              decoration: BoxDecoration(
                color: Colors.blue,
                // Responsive border radius
                borderRadius: BorderRadius.circular(12.r),
              ),
              child: Center(
                child: Text(
                  'Responsive Container',
                  style: TextStyle(
                    fontSize: 16.sp,
                    color: Colors.white,
                  ),
                ),
              ),
            ),

            // Responsive vertical spacing
            SizedBox(height: 20.h),

            // Square container (same width and height)
            Container(
              width: 100.w,
              height: 100.w, // Use .w for squares to maintain aspect ratio
              decoration: BoxDecoration(
                color: Colors.green,
                borderRadius: BorderRadius.circular(8.r),
              ),
            ),

            20.verticalSpace, // Alternative spacing method

            // Responsive row with spacing
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceEvenly,
              children: [
                _buildResponsiveButton('Button 1'),
                16.horizontalSpace,
                _buildResponsiveButton('Button 2'),
              ],
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildResponsiveButton(String text) {
    return ElevatedButton(
      style: ElevatedButton.styleFrom(
        // Responsive button padding
        padding: EdgeInsets.symmetric(
          horizontal: 24.w,
          vertical: 12.h,
        ),
        // Responsive border radius
        shape: RoundedRectangleBorder(
          borderRadius: BorderRadius.circular(8.r),
        ),
      ),
      onPressed: () {},
      child: Text(
        text,
        style: TextStyle(fontSize: 14.sp),
      ),
    );
  }
}
```

### 3. Responsive Text and Typography

Always use .sp for font sizes and implement responsive text scaling:

```dart
class ResponsiveText extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Padding(
        padding: EdgeInsets.all(16.w),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // Large heading
            Text(
              'Large Heading',
              style: TextStyle(
                fontSize: 32.sp,
                fontWeight: FontWeight.bold,
              ),
            ),

            16.verticalSpace,

            // Medium heading
            Text(
              'Medium Heading',
              style: TextStyle(
                fontSize: 24.sp,
                fontWeight: FontWeight.w600,
              ),
            ),

            12.verticalSpace,

            // Body text
            Text(
              'This is body text that will scale appropriately across different screen sizes. It maintains readability while adapting to the device.',
              style: TextStyle(
                fontSize: 16.sp,
                height: 1.5, // Line height
              ),
            ),

            8.verticalSpace,

            // Small caption text
            Text(
              'Caption or helper text',
              style: TextStyle(
                fontSize: 12.sp,
                color: Colors.grey[600],
              ),
            ),

            20.verticalSpace,

            // Text that doesn't scale with system font size
            Text(
              'Fixed size text (ignores system font scaling)',
              style: TextStyle(fontSize: 14.sp),
              textScaleFactor: 1.0, // Prevents system font scaling
            ),

            // Minimum text size (chooses smaller of 12sp or 12.sp)
            Text(
              'Minimum size text',
              style: TextStyle(fontSize: 12.sm),
            ),
          ],
        ),
      ),
    );
  }
}
```

### 4. Responsive Form Components

Create adaptive form layouts:

```dart
class ResponsiveForm extends StatefulWidget {
  @override
  State<ResponsiveForm> createState() => _ResponsiveFormState();
}

class _ResponsiveFormState extends State<ResponsiveForm> {
  final _nameController = TextEditingController();
  final _emailController = TextEditingController();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Responsive Form'),
        toolbarHeight: 56.h,
      ),
      body: SingleChildScrollView(
        padding: EdgeInsets.all(20.w),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.stretch,
          children: [
            // Form title
            Text(
              'User Information',
              style: TextStyle(
                fontSize: 24.sp,
                fontWeight: FontWeight.bold,
              ),
            ),

            24.verticalSpace,

            // Name field
            TextField(
              controller: _nameController,
              style: TextStyle(fontSize: 16.sp),
              decoration: InputDecoration(
                labelText: 'Full Name',
                labelStyle: TextStyle(fontSize: 14.sp),
                hintText: 'Enter your name',
                hintStyle: TextStyle(fontSize: 14.sp),
                contentPadding: EdgeInsets.symmetric(
                  horizontal: 16.w,
                  vertical: 12.h,
                ),
                border: OutlineInputBorder(
                  borderRadius: BorderRadius.circular(8.r),
                ),
              ),
            ),

            16.verticalSpace,

            // Email field
            TextField(
              controller: _emailController,
              style: TextStyle(fontSize: 16.sp),
              keyboardType: TextInputType.emailAddress,
              decoration: InputDecoration(
                labelText: 'Email Address',
                labelStyle: TextStyle(fontSize: 14.sp),
                hintText: 'user@example.com',
                hintStyle: TextStyle(fontSize: 14.sp),
                prefixIcon: Icon(Icons.email, size: 20.sp),
                contentPadding: EdgeInsets.symmetric(
                  horizontal: 16.w,
                  vertical: 12.h,
                ),
                border: OutlineInputBorder(
                  borderRadius: BorderRadius.circular(8.r),
                ),
              ),
            ),

            32.verticalSpace,

            // Submit button
            ElevatedButton(
              style: ElevatedButton.styleFrom(
                padding: EdgeInsets.symmetric(vertical: 16.h),
                shape: RoundedRectangleBorder(
                  borderRadius: BorderRadius.circular(8.r),
                ),
              ),
              onPressed: _submitForm,
              child: Text(
                'Submit',
                style: TextStyle(
                  fontSize: 16.sp,
                  fontWeight: FontWeight.w600,
                ),
              ),
            ),

            // Safe area bottom spacing
            SizedBox(height: ScreenUtil().bottomBarHeight),
          ],
        ),
      ),
    );
  }

  void _submitForm() {
    // Handle form submission
  }
}
```

### 5. Responsive List Items

Create adaptive list layouts:

```dart
class ResponsiveListView extends StatelessWidget {
  final List<User> users = [
    User(name: 'John Doe', email: 'john@example.com', avatar: 'avatar1.jpg'),
    User(name: 'Jane Smith', email: 'jane@example.com', avatar: 'avatar2.jpg'),
    // More users...
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Responsive List'),
        toolbarHeight: 56.h,
      ),
      body: ListView.separated(
        padding: EdgeInsets.all(16.w),
        itemCount: users.length,
        separatorBuilder: (context, index) => SizedBox(height: 8.h),
        itemBuilder: (context, index) {
          final user = users[index];
          return _buildUserTile(user);
        },
      ),
    );
  }

  Widget _buildUserTile(User user) {
    return Container(
      padding: EdgeInsets.all(16.w),
      decoration: BoxDecoration(
        color: Colors.white,
        borderRadius: BorderRadius.circular(12.r),
        boxShadow: [
          BoxShadow(
            color: Colors.black.withOpacity(0.1),
            blurRadius: 8.r,
            offset: Offset(0, 2.h),
          ),
        ],
      ),
      child: Row(
        children: [
          // Avatar
          CircleAvatar(
            radius: 24.r,
            backgroundColor: Colors.blue,
            child: Text(
              user.name[0],
              style: TextStyle(
                fontSize: 16.sp,
                color: Colors.white,
                fontWeight: FontWeight.bold,
              ),
            ),
          ),

          12.horizontalSpace,

          // User info
          Expanded(
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Text(
                  user.name,
                  style: TextStyle(
                    fontSize: 16.sp,
                    fontWeight: FontWeight.w600,
                  ),
                ),
                4.verticalSpace,
                Text(
                  user.email,
                  style: TextStyle(
                    fontSize: 14.sp,
                    color: Colors.grey[600],
                  ),
                ),
              ],
            ),
          ),

          // Action button
          IconButton(
            iconSize: 20.sp,
            onPressed: () {},
            icon: Icon(Icons.more_vert),
          ),
        ],
      ),
    );
  }
}

class User {
  final String name;
  final String email;
  final String avatar;

  User({required this.name, required this.email, required this.avatar});
}
```

### 6. Responsive Grid Layout

Implement adaptive grid layouts:

```dart
class ResponsiveGrid extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    // Calculate number of columns based on screen width
    final screenWidth = ScreenUtil().screenWidth;
    final crossAxisCount = screenWidth > 600 ? 3 : 2;

    return Scaffold(
      appBar: AppBar(
        title: Text('Responsive Grid'),
        toolbarHeight: 56.h,
      ),
      body: Padding(
        padding: EdgeInsets.all(16.w),
        child: GridView.builder(
          gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
            crossAxisCount: crossAxisCount,
            crossAxisSpacing: 12.w,
            mainAxisSpacing: 12.h,
            childAspectRatio: 0.8,
          ),
          itemCount: 20,
          itemBuilder: (context, index) {
            return _buildGridItem(index);
          },
        ),
      ),
    );
  }

  Widget _buildGridItem(int index) {
    return Container(
      decoration: BoxDecoration(
        color: Colors.white,
        borderRadius: BorderRadius.circular(12.r),
        boxShadow: [
          BoxShadow(
            color: Colors.black.withOpacity(0.1),
            blurRadius: 8.r,
            offset: Offset(0, 2.h),
          ),
        ],
      ),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          // Image placeholder
          Container(
            height: 120.h,
            decoration: BoxDecoration(
              color: Colors.blue[100],
              borderRadius: BorderRadius.vertical(
                top: Radius.circular(12.r),
              ),
            ),
            child: Center(
              child: Icon(
                Icons.image,
                size: 40.sp,
                color: Colors.blue,
              ),
            ),
          ),

          // Content
          Padding(
            padding: EdgeInsets.all(12.w),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Text(
                  'Item ${index + 1}',
                  style: TextStyle(
                    fontSize: 16.sp,
                    fontWeight: FontWeight.w600,
                  ),
                ),
                4.verticalSpace,
                Text(
                  'Description for item ${index + 1}',
                  style: TextStyle(
                    fontSize: 12.sp,
                    color: Colors.grey[600],
                  ),
                  maxLines: 2,
                  overflow: TextOverflow.ellipsis,
                ),
                8.verticalSpace,
                Text(
                  '\$${(index + 1) * 10}',
                  style: TextStyle(
                    fontSize: 14.sp,
                    fontWeight: FontWeight.bold,
                    color: Colors.green,
                  ),
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

### 7. Responsive EdgeInsets and Constraints

Use responsive padding, margins, and constraints:

```dart
class ResponsivePaddingExample extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Container(
        // Responsive padding using extension
        padding: EdgeInsets.all(16.w),

        // Alternative responsive padding
        // padding: REdgeInsets.all(16),

        child: Column(
          children: [
            Container(
              // Responsive constraints
              constraints: BoxConstraints(
                maxWidth: 400.w,
                minHeight: 100.h,
              ).w,

              // Alternative: RBoxConstraints
              // constraints: RBoxConstraints(maxWidth: 400, minHeight: 100),

              padding: EdgeInsets.symmetric(
                horizontal: 20.w,
                vertical: 16.h,
              ),

              // Responsive margin
              margin: EdgeInsets.only(
                top: 10.h,
                bottom: 20.h,
                left: 8.w,
                right: 8.w,
              ),

              decoration: BoxDecoration(
                color: Colors.blue[50],
                borderRadius: BorderRadius.circular(12.r),
                border: Border.all(
                  color: Colors.blue,
                  width: 2.w,
                ),
              ),

              child: Text(
                'Responsive Container',
                style: TextStyle(fontSize: 16.sp),
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

### 8. Screen Size Breakpoints and Adaptive Layouts

Handle different screen sizes:

```dart
class AdaptiveLayout extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final screenWidth = ScreenUtil().screenWidth;
    final isTablet = screenWidth > 600;
    final isDesktop = screenWidth > 900;

    return Scaffold(
      body: SafeArea(
        child: Padding(
          padding: EdgeInsets.all(16.w),
          child: isDesktop
              ? _buildDesktopLayout()
              : isTablet
                  ? _buildTabletLayout()
                  : _buildMobileLayout(),
        ),
      ),
    );
  }

  Widget _buildMobileLayout() {
    return Column(
      children: [
        _buildHeader(),
        16.verticalSpace,
        Expanded(child: _buildContent()),
        16.verticalSpace,
        _buildActions(),
      ],
    );
  }

  Widget _buildTabletLayout() {
    return Column(
      children: [
        _buildHeader(),
        24.verticalSpace,
        Expanded(
          child: Row(
            children: [
              Expanded(flex: 2, child: _buildContent()),
              16.horizontalSpace,
              Expanded(flex: 1, child: _buildSidebar()),
            ],
          ),
        ),
        24.verticalSpace,
        _buildActions(),
      ],
    );
  }

  Widget _buildDesktopLayout() {
    return Row(
      children: [
        SizedBox(
          width: 250.w,
          child: _buildSidebar(),
        ),
        24.horizontalSpace,
        Expanded(
          child: Column(
            children: [
              _buildHeader(),
              24.verticalSpace,
              Expanded(child: _buildContent()),
              24.verticalSpace,
              _buildActions(),
            ],
          ),
        ),
      ],
    );
  }

  Widget _buildHeader() {
    return Container(
      padding: EdgeInsets.all(16.w),
      decoration: BoxDecoration(
        color: Colors.blue,
        borderRadius: BorderRadius.circular(8.r),
      ),
      child: Text(
        'Adaptive Header',
        style: TextStyle(
          fontSize: 20.sp,
          color: Colors.white,
          fontWeight: FontWeight.bold,
        ),
      ),
    );
  }

  Widget _buildContent() {
    return Container(
      padding: EdgeInsets.all(16.w),
      decoration: BoxDecoration(
        color: Colors.grey[100],
        borderRadius: BorderRadius.circular(8.r),
      ),
      child: Center(
        child: Text(
          'Main Content Area',
          style: TextStyle(fontSize: 16.sp),
        ),
      ),
    );
  }

  Widget _buildSidebar() {
    return Container(
      padding: EdgeInsets.all(16.w),
      decoration: BoxDecoration(
        color: Colors.green[100],
        borderRadius: BorderRadius.circular(8.r),
      ),
      child: Center(
        child: Text(
          'Sidebar',
          style: TextStyle(fontSize: 14.sp),
        ),
      ),
    );
  }

  Widget _buildActions() {
    return Row(
      mainAxisAlignment: MainAxisAlignment.spaceEvenly,
      children: [
        ElevatedButton(
          onPressed: () {},
          child: Text('Action 1', style: TextStyle(fontSize: 14.sp)),
        ),
        ElevatedButton(
          onPressed: () {},
          child: Text('Action 2', style: TextStyle(fontSize: 14.sp)),
        ),
      ],
    );
  }
}
```

## Required Dependencies

```yaml
dependencies:
  flutter_screenutil: ^5.9.3
```

## Forbidden Patterns

- ❌ Using hardcoded pixel values for sizing
- ❌ Using MediaQuery calculations for responsive design
- ❌ Not wrapping app with ScreenUtilInit
- ❌ Using double values instead of ScreenUtil extensions
- ❌ Not using .sp for font sizes
- ❌ Not considering different screen sizes in layouts
- ❌ Using fixed aspect ratios without responsive consideration

## Required Extension Usage

- **Width**: Use `.w` for horizontal dimensions
- **Height**: Use `.h` for vertical dimensions (sparingly)
- **Radius**: Use `.r` for border radius and circular dimensions
- **Font Size**: Use `.sp` for all font sizes
- **Minimum Size**: Use `.sm` for minimum text scaling
- **Screen Percentage**: Use `.sw` (screen width %) and `.sh` (screen height %)
- **Spacing**: Use `.verticalSpace` and `.horizontalSpace`

## Required Design Size Configuration

```dart
// REQUIRED - Set your design size (common sizes)
ScreenUtilInit(
  designSize: const Size(375, 812), // iPhone X/11/12 design
  // OR
  // designSize: const Size(360, 690), // Android design
  // OR
  // designSize: const Size(414, 896), // iPhone XR/11 design
)
```

## Benefits Enforced

- Consistent UI scaling across all device sizes
- Automatic font size adaptation based on screen density
- Support for tablets, phones, and different aspect ratios
- Reduced manual responsive design calculations
- Better accessibility with proper text scaling
- Split screen mode compatibility
- Simplified responsive layout implementation
