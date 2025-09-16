# Flutter Staggered Animations

## When to Apply

This rule applies to ALL Flutter projects that display:

- ListView items that need entrance animations
- GridView items that need entrance animations
- Column/Row children that need staggered reveals
- Any list of widgets that should animate in sequence
- Content that appears dynamically and needs polished presentation

## Required Package

Always use `flutter_staggered_animations: ^1.1.1` as the ONLY solution for list and grid entrance animations instead of custom animation controllers, manually timed animations, or other animation libraries.

## Mandatory Implementation

### 1. ListView Staggered Animation Pattern

Always animate ListView items with staggered entrance:

```dart
import 'package:flutter_staggered_animations/flutter_staggered_animations.dart';

@override
Widget build(BuildContext context) {
  return Scaffold(
    body: AnimationLimiter(
      child: ListView.builder(
        itemCount: items.length,
        itemBuilder: (BuildContext context, int index) {
          return AnimationConfiguration.staggeredList(
            position: index,
            duration: const Duration(milliseconds: 375),
            child: SlideAnimation(
              verticalOffset: 50.0,
              child: FadeInAnimation(
                child: YourListItem(item: items[index]),
              ),
            ),
          );
        },
      ),
    ),
  );
}
```

### 2. GridView Staggered Animation Pattern

Always animate GridView items with dual-axis staggered entrance:

```dart
@override
Widget build(BuildContext context) {
  const int columnCount = 2;

  return Scaffold(
    body: AnimationLimiter(
      child: GridView.builder(
        gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
          crossAxisCount: columnCount,
          crossAxisSpacing: 8,
          mainAxisSpacing: 8,
        ),
        itemCount: items.length,
        itemBuilder: (BuildContext context, int index) {
          return AnimationConfiguration.staggeredGrid(
            position: index,
            duration: const Duration(milliseconds: 375),
            columnCount: columnCount,
            child: ScaleAnimation(
              scale: 0.5,
              child: FadeInAnimation(
                child: YourGridItem(item: items[index]),
              ),
            ),
          );
        },
      ),
    ),
  );
}
```

### 3. Column/Row Children Animation Pattern

Always animate Column/Row children with staggered reveals:

```dart
@override
Widget build(BuildContext context) {
  return Scaffold(
    body: SingleChildScrollView(
      child: AnimationLimiter(
        child: Column(
          children: AnimationConfiguration.toStaggeredList(
            duration: const Duration(milliseconds: 375),
            childAnimationBuilder: (widget) => SlideAnimation(
              horizontalOffset: 50.0,
              child: FadeInAnimation(
                child: widget,
              ),
            ),
            children: [
              ProfileHeader(),
              UserStats(),
              ActionButtons(),
              RecentActivity(),
            ],
          ),
        ),
      ),
    ),
  );
}
```

### 4. Custom List Widget Implementation

Create reusable animated list components:

```dart
class AnimatedListView<T> extends StatelessWidget {
  final List<T> items;
  final Widget Function(T item, int index) itemBuilder;
  final Duration duration;
  final double verticalOffset;
  final ScrollController? scrollController;

  const AnimatedListView({
    Key? key,
    required this.items,
    required this.itemBuilder,
    this.duration = const Duration(milliseconds: 375),
    this.verticalOffset = 50.0,
    this.scrollController,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return AnimationLimiter(
      child: ListView.builder(
        controller: scrollController,
        itemCount: items.length,
        itemBuilder: (BuildContext context, int index) {
          return AnimationConfiguration.staggeredList(
            position: index,
            duration: duration,
            child: SlideAnimation(
              verticalOffset: verticalOffset,
              child: FadeInAnimation(
                child: itemBuilder(items[index], index),
              ),
            ),
          );
        },
      ),
    );
  }
}

// Usage
AnimatedListView<Product>(
  items: products,
  itemBuilder: (product, index) => ProductCard(product: product),
  duration: const Duration(milliseconds: 300),
)
```

### 5. Custom Grid Widget Implementation

Create reusable animated grid components:

```dart
class AnimatedGridView<T> extends StatelessWidget {
  final List<T> items;
  final Widget Function(T item, int index) itemBuilder;
  final int crossAxisCount;
  final Duration duration;
  final double scale;
  final double crossAxisSpacing;
  final double mainAxisSpacing;

  const AnimatedGridView({
    Key? key,
    required this.items,
    required this.itemBuilder,
    required this.crossAxisCount,
    this.duration = const Duration(milliseconds: 375),
    this.scale = 0.5,
    this.crossAxisSpacing = 8.0,
    this.mainAxisSpacing = 8.0,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return AnimationLimiter(
      child: GridView.builder(
        gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
          crossAxisCount: crossAxisCount,
          crossAxisSpacing: crossAxisSpacing,
          mainAxisSpacing: mainAxisSpacing,
        ),
        itemCount: items.length,
        itemBuilder: (BuildContext context, int index) {
          return AnimationConfiguration.staggeredGrid(
            position: index,
            duration: duration,
            columnCount: crossAxisCount,
            child: ScaleAnimation(
              scale: scale,
              child: FadeInAnimation(
                child: itemBuilder(items[index], index),
              ),
            ),
          );
        },
      ),
    );
  }
}

// Usage
AnimatedGridView<Photo>(
  items: photos,
  crossAxisCount: 3,
  itemBuilder: (photo, index) => PhotoTile(photo: photo),
)
```

### 6. Card List Animation Implementation

Always animate card-based lists:

```dart
class AnimatedCardList extends StatelessWidget {
  final List<CardData> cards;

  const AnimatedCardList({Key? key, required this.cards}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return AnimationLimiter(
      child: ListView.builder(
        padding: const EdgeInsets.all(16),
        itemCount: cards.length,
        itemBuilder: (context, index) {
          return AnimationConfiguration.staggeredList(
            position: index,
            duration: const Duration(milliseconds: 375),
            child: SlideAnimation(
              verticalOffset: 50.0,
              child: FadeInAnimation(
                child: Card(
                  margin: const EdgeInsets.only(bottom: 16),
                  child: ListTile(
                    leading: CircleAvatar(
                      backgroundImage: NetworkImage(cards[index].avatarUrl),
                    ),
                    title: Text(cards[index].title),
                    subtitle: Text(cards[index].subtitle),
                    trailing: Icon(Icons.arrow_forward_ios),
                  ),
                ),
              ),
            ),
          );
        },
      ),
    );
  }
}
```

### 7. Dashboard Widget Animation

Animate dashboard components in sequence:

```dart
class DashboardPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Dashboard')),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: AnimationLimiter(
          child: Column(
            children: AnimationConfiguration.toStaggeredList(
              duration: const Duration(milliseconds: 375),
              childAnimationBuilder: (widget) => SlideAnimation(
                verticalOffset: 50.0,
                child: FadeInAnimation(
                  child: widget,
                ),
              ),
              children: [
                // Header Stats
                Row(
                  children: [
                    Expanded(child: StatCard(title: 'Revenue', value: '\$12.4K')),
                    SizedBox(width: 16),
                    Expanded(child: StatCard(title: 'Orders', value: '1,234')),
                  ],
                ),
                SizedBox(height: 16),

                // Chart Section
                Card(
                  child: Padding(
                    padding: EdgeInsets.all(16),
                    child: Column(
                      crossAxisAlignment: CrossAxisAlignment.start,
                      children: [
                        Text('Sales Chart', style: Theme.of(context).textTheme.titleLarge),
                        SizedBox(height: 16),
                        Container(height: 200, child: SalesChart()),
                      ],
                    ),
                  ),
                ),
                SizedBox(height: 16),

                // Recent Orders
                Card(
                  child: Padding(
                    padding: EdgeInsets.all(16),
                    child: Column(
                      crossAxisAlignment: CrossAxisAlignment.start,
                      children: [
                        Text('Recent Orders', style: Theme.of(context).textTheme.titleLarge),
                        SizedBox(height: 8),
                        RecentOrdersList(),
                      ],
                    ),
                  ),
                ),
              ],
            ),
          ),
        ),
      ),
    );
  }
}
```

### 8. Complex Animation Combinations

Combine multiple animation types for advanced effects:

```dart
class AdvancedAnimatedList extends StatelessWidget {
  final List<Item> items;

  @override
  Widget build(BuildContext context) {
    return AnimationLimiter(
      child: ListView.builder(
        itemCount: items.length,
        itemBuilder: (context, index) {
          return AnimationConfiguration.staggeredList(
            position: index,
            duration: const Duration(milliseconds: 500),
            delay: Duration(milliseconds: 50),
            child: FlipAnimation(
              axis: FlipAxis.x,
              child: SlideAnimation(
                verticalOffset: 100.0,
                child: ScaleAnimation(
                  scale: 0.8,
                  child: FadeInAnimation(
                    child: Container(
                      margin: EdgeInsets.symmetric(horizontal: 16, vertical: 8),
                      child: ItemCard(item: items[index]),
                    ),
                  ),
                ),
              ),
            ),
          );
        },
      ),
    );
  }
}
```

### 9. Tab View Animation Implementation

Animate tab content changes:

```dart
class AnimatedTabView extends StatefulWidget {
  @override
  State<AnimatedTabView> createState() => _AnimatedTabViewState();
}

class _AnimatedTabViewState extends State<AnimatedTabView>
    with SingleTickerProviderStateMixin {
  late TabController _tabController;

  @override
  void initState() {
    super.initState();
    _tabController = TabController(length: 3, vsync: this);
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        bottom: TabBar(
          controller: _tabController,
          tabs: [
            Tab(text: 'Featured'),
            Tab(text: 'Popular'),
            Tab(text: 'Recent'),
          ],
        ),
      ),
      body: TabBarView(
        controller: _tabController,
        children: [
          _buildTabContent('Featured', featuredItems),
          _buildTabContent('Popular', popularItems),
          _buildTabContent('Recent', recentItems),
        ],
      ),
    );
  }

  Widget _buildTabContent(String title, List<Item> items) {
    return AnimationLimiter(
      child: ListView.builder(
        key: ValueKey(title), // Important for proper animation reset
        itemCount: items.length,
        itemBuilder: (context, index) {
          return AnimationConfiguration.staggeredList(
            position: index,
            duration: const Duration(milliseconds: 375),
            child: SlideAnimation(
              horizontalOffset: 50.0,
              child: FadeInAnimation(
                child: ItemTile(item: items[index]),
              ),
            ),
          );
        },
      ),
    );
  }
}
```

### 10. Performance Optimization

Always implement proper animation limiting:

```dart
class OptimizedAnimatedList extends StatelessWidget {
  final List<Item> items;
  final bool useAnimationLimiter;

  const OptimizedAnimatedList({
    Key? key,
    required this.items,
    this.useAnimationLimiter = true,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    Widget listView = ListView.builder(
      itemCount: items.length,
      itemBuilder: (context, index) {
        return AnimationConfiguration.staggeredList(
          position: index,
          duration: const Duration(milliseconds: 375),
          child: SlideAnimation(
            verticalOffset: 50.0,
            child: FadeInAnimation(
              child: ItemCard(
                item: items[index],
                key: ValueKey(items[index].id), // Important for performance
              ),
            ),
          ),
        );
      },
    );

    // Use AnimationLimiter only for scrollable content
    // to prevent animations during scrolling
    return useAnimationLimiter
        ? AnimationLimiter(child: listView)
        : listView;
  }
}
```

## Required Dependencies

```yaml
dependencies:
  flutter_staggered_animations: ^1.1.1
```

## Forbidden Patterns

- ❌ Using custom AnimationController for list entrance animations
- ❌ Manual timing of sequential animations
- ❌ Not using AnimationLimiter for scrollable content
- ❌ Static lists without entrance animations
- ❌ Immediate appearance of list items without animation
- ❌ Using other animation libraries for staggered effects
- ❌ Not combining animations (always use at least Slide + Fade)

## Required Animation Combinations

### Standard Combinations:

```dart
// ✅ REQUIRED - Slide + Fade for lists
SlideAnimation(
  verticalOffset: 50.0,
  child: FadeInAnimation(
    child: widget,
  ),
)

// ✅ REQUIRED - Scale + Fade for grids
ScaleAnimation(
  scale: 0.5,
  child: FadeInAnimation(
    child: widget,
  ),
)

// ✅ ALTERNATIVE - Flip + Fade for special effects
FlipAnimation(
  axis: FlipAxis.x,
  child: FadeInAnimation(
    child: widget,
  ),
)
```

## Required Configuration Parameters

```dart
// ✅ REQUIRED - Always specify duration
AnimationConfiguration.staggeredList(
  position: index,
  duration: const Duration(milliseconds: 375), // Required
  child: animations,
)

// ✅ REQUIRED - Always specify column count for grids
AnimationConfiguration.staggeredGrid(
  position: index,
  duration: const Duration(milliseconds: 375),
  columnCount: 2, // Required for proper grid timing
  child: animations,
)
```

## Benefits Enforced

- Professional Material Design compliant entrance animations
- Consistent staggered timing across all list presentations
- Improved perceived performance and user engagement
- Reduced animation complexity with pre-built combinations
- Proper animation limiting to prevent performance issues
- Smooth transitions that enhance user experience
- Easy implementation with minimal boilerplate code
