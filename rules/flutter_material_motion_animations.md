# Flutter Material Motion Animations

## When to Apply

This rule applies to ALL Flutter projects that need:

- Navigation transitions between screens
- Card-to-detail page transitions
- List item expansions
- Modal presentations
- Tab switching animations
- Search expansions
- Any UI element transitions that need professional polish

## Required Package

Always use `animations: ^2.0.11` as the PRIMARY solution for Material Design transitions instead of custom PageRouteBuilder, Hero animations, or other transition packages.

## Mandatory Implementation

### 1. Container Transform Pattern

Always use for card-to-detail and list-to-detail transitions:

```dart
import 'package:animations/animations.dart';

class ProductCard extends StatelessWidget {
  final Product product;

  const ProductCard({Key? key, required this.product}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return OpenContainer<bool>(
      transitionType: ContainerTransitionType.fade,
      openBuilder: (BuildContext context, VoidCallback _) {
        return ProductDetailPage(product: product);
      },
      closedElevation: 6.0,
      closedShape: const RoundedRectangleBorder(
        borderRadius: BorderRadius.all(Radius.circular(16)),
      ),
      closedColor: Theme.of(context).cardColor,
      closedBuilder: (BuildContext context, VoidCallback openContainer) {
        return InkWell(
          onTap: openContainer,
          child: SizedBox(
            height: 300,
            child: Card(
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Expanded(
                    child: Container(
                      decoration: BoxDecoration(
                        image: DecorationImage(
                          image: NetworkImage(product.imageUrl),
                          fit: BoxFit.cover,
                        ),
                        borderRadius: BorderRadius.vertical(top: Radius.circular(16)),
                      ),
                    ),
                  ),
                  Padding(
                    padding: const EdgeInsets.all(16),
                    child: Column(
                      crossAxisAlignment: CrossAxisAlignment.start,
                      children: [
                        Text(
                          product.name,
                          style: Theme.of(context).textTheme.titleLarge,
                          maxLines: 1,
                          overflow: TextOverflow.ellipsis,
                        ),
                        SizedBox(height: 8),
                        Text(
                          '\$${product.price}',
                          style: Theme.of(context).textTheme.headlineSmall?.copyWith(
                            color: Theme.of(context).primaryColor,
                          ),
                        ),
                      ],
                    ),
                  ),
                ],
              ),
            ),
          ),
        );
      },
    );
  }
}
```

### 2. Shared Axis Pattern - Horizontal (X-axis)

Use for onboarding flows and tab navigation:

```dart
class OnboardingFlow extends StatefulWidget {
  @override
  State<OnboardingFlow> createState() => _OnboardingFlowState();
}

class _OnboardingFlowState extends State<OnboardingFlow> {
  int currentIndex = 0;
  bool reverse = false;

  final List<Widget> pages = [
    OnboardingPage1(),
    OnboardingPage2(),
    OnboardingPage3(),
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: PageTransitionSwitcher(
        transitionBuilder: (
          Widget child,
          Animation<double> animation,
          Animation<double> secondaryAnimation,
        ) {
          return SharedAxisTransition(
            child: child,
            animation: animation,
            secondaryAnimation: secondaryAnimation,
            transitionType: SharedAxisTransitionType.horizontal,
          );
        },
        child: pages[currentIndex],
      ),
      bottomNavigationBar: Row(
        mainAxisAlignment: MainAxisAlignment.spaceBetween,
        children: [
          TextButton(
            onPressed: currentIndex > 0 ? _previousPage : null,
            child: Text('Previous'),
          ),
          Row(
            children: List.generate(pages.length, (index) =>
              Container(
                margin: EdgeInsets.symmetric(horizontal: 4),
                width: 8,
                height: 8,
                decoration: BoxDecoration(
                  shape: BoxShape.circle,
                  color: currentIndex == index
                      ? Theme.of(context).primaryColor
                      : Colors.grey,
                ),
              ),
            ),
          ),
          TextButton(
            onPressed: currentIndex < pages.length - 1 ? _nextPage : _finish,
            child: Text(currentIndex < pages.length - 1 ? 'Next' : 'Finish'),
          ),
        ],
      ),
    );
  }

  void _nextPage() {
    setState(() {
      reverse = false;
      currentIndex++;
    });
  }

  void _previousPage() {
    setState(() {
      reverse = true;
      currentIndex--;
    });
  }

  void _finish() {
    // Complete onboarding
    Navigator.pushReplacementNamed(context, '/home');
  }
}
```

### 3. Shared Axis Pattern - Vertical (Y-axis)

Use for stepper forms and vertical navigation:

```dart
class StepperForm extends StatefulWidget {
  @override
  State<StepperForm> createState() => _StepperFormState();
}

class _StepperFormState extends State<StepperForm> {
  int currentStep = 0;

  final List<Widget> steps = [
    PersonalInfoStep(),
    AddressStep(),
    PaymentStep(),
    ConfirmationStep(),
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Registration'),
        leading: IconButton(
          icon: Icon(Icons.arrow_back),
          onPressed: currentStep > 0 ? _previousStep : () => Navigator.pop(context),
        ),
      ),
      body: Column(
        children: [
          // Progress indicator
          LinearProgressIndicator(
            value: (currentStep + 1) / steps.length,
          ),

          // Step content with vertical transition
          Expanded(
            child: PageTransitionSwitcher(
              transitionBuilder: (
                Widget child,
                Animation<double> animation,
                Animation<double> secondaryAnimation,
              ) {
                return SharedAxisTransition(
                  child: child,
                  animation: animation,
                  secondaryAnimation: secondaryAnimation,
                  transitionType: SharedAxisTransitionType.vertical,
                );
              },
              child: Padding(
                key: ValueKey(currentStep),
                padding: const EdgeInsets.all(16),
                child: steps[currentStep],
              ),
            ),
          ),

          // Navigation buttons
          Padding(
            padding: const EdgeInsets.all(16),
            child: Row(
              children: [
                if (currentStep > 0)
                  Expanded(
                    child: OutlinedButton(
                      onPressed: _previousStep,
                      child: Text('Previous'),
                    ),
                  ),
                if (currentStep > 0) SizedBox(width: 16),
                Expanded(
                  child: ElevatedButton(
                    onPressed: _nextStep,
                    child: Text(currentStep < steps.length - 1 ? 'Next' : 'Submit'),
                  ),
                ),
              ],
            ),
          ),
        ],
      ),
    );
  }

  void _nextStep() {
    if (currentStep < steps.length - 1) {
      setState(() => currentStep++);
    } else {
      _submitForm();
    }
  }

  void _previousStep() {
    if (currentStep > 0) {
      setState(() => currentStep--);
    }
  }

  void _submitForm() {
    // Handle form submission
  }
}
```

### 4. Shared Axis Pattern - Scaled (Z-axis)

Use for parent-child navigation:

```dart
class CategoryNavigation extends StatefulWidget {
  @override
  State<CategoryNavigation> createState() => _CategoryNavigationState();
}

class _CategoryNavigationState extends State<CategoryNavigation> {
  List<String> navigationStack = ['Categories'];
  Widget currentWidget = CategoriesGrid();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(navigationStack.last),
        leading: navigationStack.length > 1
            ? IconButton(
                icon: Icon(Icons.arrow_back),
                onPressed: _navigateBack,
              )
            : null,
      ),
      body: PageTransitionSwitcher(
        transitionBuilder: (
          Widget child,
          Animation<double> animation,
          Animation<double> secondaryAnimation,
        ) {
          return SharedAxisTransition(
            child: child,
            animation: animation,
            secondaryAnimation: secondaryAnimation,
            transitionType: SharedAxisTransitionType.scaled,
          );
        },
        child: currentWidget,
      ),
    );
  }

  void navigateToSubcategory(String title, Widget widget) {
    setState(() {
      navigationStack.add(title);
      currentWidget = widget;
    });
  }

  void _navigateBack() {
    if (navigationStack.length > 1) {
      setState(() {
        navigationStack.removeLast();
        // Determine widget based on navigation stack
        currentWidget = _getWidgetForPath();
      });
    }
  }

  Widget _getWidgetForPath() {
    // Return appropriate widget based on navigation path
    return CategoriesGrid(); // Simplified
  }
}
```

### 5. Fade Through Pattern

Use for bottom navigation and unrelated content switching:

```dart
class MainNavigationScreen extends StatefulWidget {
  @override
  State<MainNavigationScreen> createState() => _MainNavigationScreenState();
}

class _MainNavigationScreenState extends State<MainNavigationScreen> {
  int selectedIndex = 0;

  final List<Widget> pages = [
    HomePage(),
    SearchPage(),
    FavoritesPage(),
    ProfilePage(),
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: PageTransitionSwitcher(
        transitionBuilder: (
          Widget child,
          Animation<double> animation,
          Animation<double> secondaryAnimation,
        ) {
          return FadeThroughTransition(
            animation: animation,
            secondaryAnimation: secondaryAnimation,
            child: child,
          );
        },
        child: pages[selectedIndex],
      ),
      bottomNavigationBar: BottomNavigationBar(
        type: BottomNavigationBarType.fixed,
        currentIndex: selectedIndex,
        onTap: (index) {
          setState(() => selectedIndex = index);
        },
        items: [
          BottomNavigationBarItem(
            icon: Icon(Icons.home),
            label: 'Home',
          ),
          BottomNavigationBarItem(
            icon: Icon(Icons.search),
            label: 'Search',
          ),
          BottomNavigationBarItem(
            icon: Icon(Icons.favorite),
            label: 'Favorites',
          ),
          BottomNavigationBarItem(
            icon: Icon(Icons.person),
            label: 'Profile',
          ),
        ],
      ),
    );
  }
}
```

### 6. Fade Pattern

Use for modals, dialogs, and overlays:

```dart
class AnimatedModal extends StatelessWidget {
  static Future<T?> show<T>(
    BuildContext context,
    Widget child, {
    bool barrierDismissible = true,
  }) {
    return showModal<T>(
      context: context,
      configuration: FadeScaleTransitionConfiguration(
        barrierDismissible: barrierDismissible,
      ),
      builder: (context) => child,
    );
  }

  @override
  Widget build(BuildContext context) {
    return Center(
      child: Container(
        margin: EdgeInsets.all(16),
        padding: EdgeInsets.all(24),
        decoration: BoxDecoration(
          color: Theme.of(context).cardColor,
          borderRadius: BorderRadius.circular(16),
          boxShadow: [
            BoxShadow(
              color: Colors.black.withOpacity(0.1),
              blurRadius: 10,
              offset: Offset(0, 4),
            ),
          ],
        ),
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            Icon(
              Icons.check_circle,
              size: 64,
              color: Colors.green,
            ),
            SizedBox(height: 16),
            Text(
              'Success!',
              style: Theme.of(context).textTheme.headlineSmall,
            ),
            SizedBox(height: 8),
            Text(
              'Your action was completed successfully.',
              textAlign: TextAlign.center,
            ),
            SizedBox(height: 24),
            ElevatedButton(
              onPressed: () => Navigator.pop(context),
              child: Text('OK'),
            ),
          ],
        ),
      ),
    );
  }
}

// Usage
void _showSuccessDialog() {
  AnimatedModal.show(
    context,
    AnimatedModal(),
  );
}
```

### 7. FAB Transformation

Use for floating action button expansions:

```dart
class AnimatedFAB extends StatelessWidget {
  final VoidCallback? onPressed;
  final Widget child;
  final Widget destinationPage;

  const AnimatedFAB({
    Key? key,
    this.onPressed,
    required this.child,
    required this.destinationPage,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return OpenContainer(
      transitionType: ContainerTransitionType.fadeThrough,
      openBuilder: (BuildContext context, VoidCallback _) {
        return destinationPage;
      },
      closedElevation: 6.0,
      closedShape: const CircleBorder(),
      closedColor: Theme.of(context).primaryColor,
      closedBuilder: (BuildContext context, VoidCallback openContainer) {
        return FloatingActionButton(
          onPressed: openContainer,
          child: child,
        );
      },
    );
  }
}

// Usage
AnimatedFAB(
  child: Icon(Icons.add),
  destinationPage: CreateItemPage(),
)
```

### 8. Search Bar Expansion

Create animated search functionality:

```dart
class AnimatedSearchBar extends StatefulWidget {
  @override
  State<AnimatedSearchBar> createState() => _AnimatedSearchBarState();
}

class _AnimatedSearchBarState extends State<AnimatedSearchBar> {
  bool isSearching = false;

  @override
  Widget build(BuildContext context) {
    return OpenContainer<bool>(
      transitionType: ContainerTransitionType.fade,
      openBuilder: (BuildContext context, VoidCallback _) {
        return SearchPage();
      },
      tappable: false,
      closedBuilder: (BuildContext context, VoidCallback openContainer) {
        return AppBar(
          title: Text('My App'),
          actions: [
            IconButton(
              icon: Icon(Icons.search),
              onPressed: openContainer,
            ),
          ],
        );
      },
    );
  }
}
```

### 9. List Item Detail Expansion

Animate list items to detail views:

```dart
class AnimatedListItem extends StatelessWidget {
  final User user;

  const AnimatedListItem({Key? key, required this.user}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return OpenContainer<bool>(
      transitionType: ContainerTransitionType.fade,
      openBuilder: (BuildContext context, VoidCallback _) {
        return UserDetailPage(user: user);
      },
      closedElevation: 0,
      closedBuilder: (BuildContext context, VoidCallback openContainer) {
        return ListTile(
          leading: CircleAvatar(
            backgroundImage: NetworkImage(user.avatarUrl),
          ),
          title: Text(user.name),
          subtitle: Text(user.email),
          trailing: Icon(Icons.arrow_forward_ios),
          onTap: openContainer,
        );
      },
    );
  }
}
```

### 10. Custom Transition Configuration

Create app-wide transition themes:

```dart
class AppTransitions {
  static const Duration defaultDuration = Duration(milliseconds: 300);

  static Widget containerTransform({
    required Widget closedChild,
    required Widget openChild,
    VoidCallback? onClosed,
  }) {
    return OpenContainer<bool>(
      transitionDuration: defaultDuration,
      transitionType: ContainerTransitionType.fade,
      openBuilder: (context, _) => openChild,
      closedBuilder: (context, openContainer) => InkWell(
        onTap: openContainer,
        child: closedChild,
      ),
      onClosed: (result) => onClosed?.call(),
    );
  }

  static Widget sharedAxisHorizontal({
    required Widget child,
    required Animation<double> animation,
    required Animation<double> secondaryAnimation,
  }) {
    return SharedAxisTransition(
      child: child,
      animation: animation,
      secondaryAnimation: secondaryAnimation,
      transitionType: SharedAxisTransitionType.horizontal,
    );
  }

  static Widget fadeThrough({
    required Widget child,
    required Animation<double> animation,
    required Animation<double> secondaryAnimation,
  }) {
    return FadeThroughTransition(
      animation: animation,
      secondaryAnimation: secondaryAnimation,
      child: child,
    );
  }
}
```

## Required Dependencies

```yaml
dependencies:
  animations: ^2.0.11
```

## Forbidden Patterns

- ❌ Custom PageRouteBuilder for standard transitions
- ❌ Manual Hero animations for card-to-detail transitions
- ❌ Using SlideTransition directly for navigation
- ❌ Immediate page switches without transition animations
- ❌ Custom animation controllers for standard Material patterns
- ❌ Not using OpenContainer for expandable content
- ❌ Static modals and dialogs without fade animations

## Required Pattern Usage

- **Container Transform**: Card to detail, list to detail, FAB to page
- **Shared Axis Horizontal**: Onboarding, tab switching, carousel
- **Shared Axis Vertical**: Stepper forms, vertical navigation
- **Shared Axis Scaled**: Parent-child navigation, drill-down
- **Fade Through**: Bottom nav, unrelated content switching
- **Fade**: Modals, dialogs, overlays, tooltips

## Benefits Enforced

- Material Design compliant motion patterns
- Improved user navigation understanding
- Professional animation quality without custom coding
- Consistent transition behavior across the app
- Better perceived performance and user engagement
- Reduced development time for complex transitions
- Automatic accessibility support for reduced motion preferences
