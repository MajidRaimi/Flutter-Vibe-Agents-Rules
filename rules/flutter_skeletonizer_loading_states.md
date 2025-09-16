# Flutter Loading States with Skeletonizer

## When to Apply

This rule applies to ALL Flutter projects that need loading states for:

- Data fetching from APIs
- Image loading
- Content loading
- List/grid loading
- Any asynchronous operations that require user feedback

## Required Package

Always use `skeletonizer: ^2.1.0+1` as the ONLY solution for loading states instead of CircularProgressIndicator, LinearProgressIndicator, custom shimmer effects, or other loading indicators.

## Mandatory Implementation

### 1. Basic Skeletonizer Setup

Always wrap content that needs loading states with Skeletonizer:

```dart
import 'package:skeletonizer/skeletonizer.dart';

Skeletonizer(
  enabled: _isLoading,
  child: YourContentWidget(),
)
```

### 2. Global Configuration

Configure Skeletonizer globally in your app theme:

```dart
MaterialApp(
  theme: ThemeData(
    extensions: const [
      SkeletonizerConfigData(
        effect: ShimmerEffect(
          baseColor: Colors.grey[300]!,
          highlightColor: Colors.grey[100]!,
          duration: Duration(seconds: 1),
        ),
      ),
    ],
  ),
  darkTheme: ThemeData(
    brightness: Brightness.dark,
    extensions: const [
      SkeletonizerConfigData.dark(),
    ],
  ),
)
```

### 3. List Loading Pattern

Always use fake data for list loading states:

```dart
class UserListWidget extends StatefulWidget {
  @override
  State<UserListWidget> createState() => _UserListWidgetState();
}

class _UserListWidgetState extends State<UserListWidget> {
  List<User> users = [];
  bool _isLoading = true;

  @override
  void initState() {
    super.initState();
    _loadUsers();
  }

  @override
  Widget build(BuildContext context) {
    // Create fake data for loading state
    final displayUsers = _isLoading
        ? List.filled(5, User(
            name: BoneMock.name,
            email: BoneMock.email,
            jobTitle: BoneMock.words(2),
            avatar: '', // Empty for skeleton
          ))
        : users;

    return Skeletonizer(
      enabled: _isLoading,
      child: ListView.builder(
        itemCount: displayUsers.length,
        itemBuilder: (context, index) {
          return Card(
            child: ListTile(
              title: Text(displayUsers[index].name),
              subtitle: Text(displayUsers[index].email),
              leading: Skeleton.replace(
                width: 48,
                height: 48,
                child: CircleAvatar(
                  backgroundImage: displayUsers[index].avatar.isNotEmpty
                      ? NetworkImage(displayUsers[index].avatar)
                      : null,
                ),
              ),
              trailing: Skeleton.ignore(
                child: Icon(Icons.arrow_forward_ios),
              ),
            ),
          );
        },
      ),
    );
  }

  Future<void> _loadUsers() async {
    try {
      final loadedUsers = await UserService.fetchUsers();
      if (mounted) {
        setState(() {
          users = loadedUsers;
          _isLoading = false;
        });
      }
    } catch (error) {
      if (mounted) {
        setState(() {
          _isLoading = false;
        });
      }
      // Handle error
    }
  }
}
```

### 4. Manual Skeleton Creation

Use Bone widgets for custom skeleton layouts:

```dart
class ProductCardSkeleton extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Skeletonizer.zone(
      child: Card(
        child: Padding(
          padding: const EdgeInsets.all(16),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              Bone.square(size: 100), // Product image skeleton
              SizedBox(height: 8),
              Bone.text(words: 2), // Product title
              SizedBox(height: 4),
              Bone.text(words: 4), // Product description
              SizedBox(height: 8),
              Row(
                mainAxisAlignment: MainAxisAlignment.spaceBetween,
                children: [
                  Bone.text(words: 1), // Price
                  Bone.button(), // Add to cart button
                ],
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

### 5. Profile Page Loading Pattern

Create comprehensive skeleton for profile pages:

```dart
class ProfilePage extends StatefulWidget {
  @override
  State<ProfilePage> createState() => _ProfilePageState();
}

class _ProfilePageState extends State<ProfilePage> {
  UserProfile? profile;
  bool _isLoading = true;

  @override
  Widget build(BuildContext context) {
    final displayProfile = _isLoading
        ? UserProfile(
            name: BoneMock.name,
            bio: BoneMock.sentences(3),
            email: BoneMock.email,
            phone: BoneMock.phone,
            avatar: '',
            stats: ProfileStats(
              followers: 0,
              following: 0,
              posts: 0,
            ),
          )
        : profile!;

    return Scaffold(
      body: Skeletonizer(
        enabled: _isLoading,
        child: SingleChildScrollView(
          padding: EdgeInsets.all(16),
          child: Column(
            children: [
              // Profile Header
              Skeleton.replace(
                width: 100,
                height: 100,
                child: CircleAvatar(
                  radius: 50,
                  backgroundImage: displayProfile.avatar.isNotEmpty
                      ? NetworkImage(displayProfile.avatar)
                      : null,
                ),
              ),
              SizedBox(height: 16),
              Text(
                displayProfile.name,
                style: Theme.of(context).textTheme.headlineSmall,
              ),
              SizedBox(height: 8),
              Text(displayProfile.bio),
              SizedBox(height: 16),

              // Stats Row
              Row(
                mainAxisAlignment: MainAxisAlignment.spaceEvenly,
                children: [
                  _buildStatColumn('Posts', displayProfile.stats.posts.toString()),
                  _buildStatColumn('Followers', displayProfile.stats.followers.toString()),
                  _buildStatColumn('Following', displayProfile.stats.following.toString()),
                ],
              ),

              SizedBox(height: 24),

              // Contact Info
              Card(
                child: Padding(
                  padding: EdgeInsets.all(16),
                  child: Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    children: [
                      Text('Contact Information',
                          style: Theme.of(context).textTheme.titleMedium),
                      SizedBox(height: 8),
                      ListTile(
                        leading: Skeleton.keep(child: Icon(Icons.email)),
                        title: Text(displayProfile.email),
                      ),
                      ListTile(
                        leading: Skeleton.keep(child: Icon(Icons.phone)),
                        title: Text(displayProfile.phone),
                      ),
                    ],
                  ),
                ),
              ),
            ],
          ),
        ),
      ),
    );
  }

  Widget _buildStatColumn(String label, String value) {
    return Column(
      children: [
        Text(value, style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold)),
        Text(label, style: TextStyle(color: Colors.grey)),
      ],
    );
  }
}
```

### 6. Grid Loading Pattern

Implement skeleton loading for image grids:

```dart
class PhotoGrid extends StatefulWidget {
  @override
  State<PhotoGrid> createState() => _PhotoGridState();
}

class _PhotoGridState extends State<PhotoGrid> {
  List<Photo> photos = [];
  bool _isLoading = true;

  @override
  Widget build(BuildContext context) {
    final displayPhotos = _isLoading
        ? List.filled(12, Photo(
            id: BoneMock.uuid,
            title: BoneMock.words(2),
            url: '',
          ))
        : photos;

    return Skeletonizer(
      enabled: _isLoading,
      child: GridView.builder(
        gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
          crossAxisCount: 2,
          crossAxisSpacing: 8,
          mainAxisSpacing: 8,
        ),
        itemCount: displayPhotos.length,
        itemBuilder: (context, index) {
          return Card(
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.stretch,
              children: [
                Expanded(
                  child: Skeleton.replace(
                    child: displayPhotos[index].url.isNotEmpty
                        ? Image.network(displayPhotos[index].url, fit: BoxFit.cover)
                        : Container(),
                  ),
                ),
                Padding(
                  padding: EdgeInsets.all(8),
                  child: Text(displayPhotos[index].title),
                ),
              ],
            ),
          );
        },
      ),
    );
  }
}
```

### 7. Sliver Loading Pattern

Use SliverSkeletonizer for custom scrollables:

```dart
class CustomScrollViewWithSkeleton extends StatefulWidget {
  @override
  State<CustomScrollViewWithSkeleton> createState() => _CustomScrollViewWithSkeletonState();
}

class _CustomScrollViewWithSkeletonState extends State<CustomScrollViewWithSkeleton> {
  bool _isLoading = true;
  List<Article> articles = [];

  @override
  Widget build(BuildContext context) {
    final displayArticles = _isLoading
        ? List.filled(8, Article(
            title: BoneMock.sentences(1),
            content: BoneMock.sentences(3),
            author: BoneMock.name,
            publishedAt: BoneMock.date,
            imageUrl: '',
          ))
        : articles;

    return CustomScrollView(
      slivers: [
        SliverAppBar(
          title: Text('Articles'),
          floating: true,
        ),
        SliverSkeletonizer(
          enabled: _isLoading,
          child: SliverList(
            delegate: SliverChildBuilderDelegate(
              (context, index) {
                final article = displayArticles[index];
                return Card(
                  margin: EdgeInsets.all(8),
                  child: Padding(
                    padding: EdgeInsets.all(16),
                    child: Column(
                      crossAxisAlignment: CrossAxisAlignment.start,
                      children: [
                        Text(
                          article.title,
                          style: Theme.of(context).textTheme.titleLarge,
                        ),
                        SizedBox(height: 8),
                        Text(article.content),
                        SizedBox(height: 8),
                        Row(
                          children: [
                            Text('By ${article.author}'),
                            Spacer(),
                            Skeleton.ignore(
                              child: Icon(Icons.more_vert),
                            ),
                          ],
                        ),
                      ],
                    ),
                  ),
                );
              },
              childCount: displayArticles.length,
            ),
          ),
        ),
      ],
    );
  }
}
```

### 8. Skeleton Annotations Usage

Use appropriate annotations for different widget behaviors:

```dart
// Ignore widgets that shouldn't be skeletonized
Skeleton.ignore(
  child: FloatingActionButton(
    onPressed: () {},
    child: Icon(Icons.add),
  ),
)

// Keep widgets as-is during skeleton state
Skeleton.keep(
  child: AppBar(title: Text('My App')),
)

// Replace problematic widgets (like network images)
Skeleton.replace(
  width: 100,
  height: 100,
  child: Image.network(imageUrl),
)

// Unite multiple small elements into one bone
Skeleton.unite(
  child: Row(
    children: [
      Icon(Icons.star),
      SizedBox(width: 4),
      Text('4.5'),
      SizedBox(width: 4),
      Text('(123 reviews)'),
    ],
  ),
)

// Shade custom painted widgets
Skeleton.shade(
  child: CustomPaint(painter: MyPainter()),
)
```

### 9. Animation Configuration

Configure skeleton animations for better UX:

```dart
Skeletonizer(
  enabled: _isLoading,
  enableSwitchAnimation: true,
  effect: const ShimmerEffect(
    baseColor: Colors.grey,
    highlightColor: Colors.white,
    duration: Duration(seconds: 1),
  ),
  switchAnimationConfig: SwitchAnimationConfig(
    duration: Duration(milliseconds: 300),
    switchInCurve: Curves.easeIn,
    switchOutCurve: Curves.easeOut,
  ),
  child: YourContent(),
)
```

### 10. Service Integration Pattern

Create a service helper for consistent skeleton usage:

```dart
class SkeletonService {
  static Widget wrapWithSkeleton<T>({
    required bool isLoading,
    required Widget child,
    required List<T> fakeData,
    SkeletonizerConfigData? config,
  }) {
    return Skeletonizer(
      enabled: isLoading,
      effect: config?.effect ?? const ShimmerEffect(),
      child: child,
    );
  }

  static List<T> getFakeData<T>(T Function() generator, int count) {
    return List.filled(count, generator());
  }
}

// Usage
class DataWidget extends StatelessWidget {
  final bool isLoading;
  final List<User> users;

  @override
  Widget build(BuildContext context) {
    final displayUsers = isLoading
        ? SkeletonService.getFakeData(() => User.fake(), 5)
        : users;

    return SkeletonService.wrapWithSkeleton(
      isLoading: isLoading,
      fakeData: displayUsers,
      child: ListView.builder(
        itemCount: displayUsers.length,
        itemBuilder: (context, index) => UserTile(user: displayUsers[index]),
      ),
    );
  }
}
```

## Required Dependencies

```yaml
dependencies:
  skeletonizer: ^2.1.0+1
```

## Forbidden Patterns

- ❌ Using CircularProgressIndicator for content loading
- ❌ Using LinearProgressIndicator for content loading
- ❌ Custom shimmer effect implementations
- ❌ Empty states during loading (always provide fake data)
- ❌ Loading states without skeleton animation
- ❌ Not using appropriate skeleton annotations
- ❌ Not configuring skeleton globally in theme
- ❌ Using other loading indicator packages

## Required Implementation Rules

### Always Provide Fake Data

```dart
// ✅ REQUIRED - Always provide fake data
final displayData = isLoading
    ? List.filled(5, MyModel.fake())
    : realData;

// ❌ FORBIDDEN - Empty loading state
final displayData = isLoading ? [] : realData;
```

### Always Handle Network Images

```dart
// ✅ REQUIRED - Replace network images
Skeleton.replace(
  width: 100,
  height: 100,
  child: Image.network(imageUrl),
)

// ❌ FORBIDDEN - Network images without replacement
Image.network(imageUrl) // Will cause errors with empty URLs
```

### Always Configure Effects

```dart
// ✅ REQUIRED - Configure skeleton effects
Skeletonizer(
  enabled: isLoading,
  effect: const ShimmerEffect(
    duration: Duration(seconds: 1),
  ),
  child: content,
)
```

## Benefits Enforced

- Consistent and professional loading experience
- Automatic skeleton generation from existing layouts
- No need to maintain separate skeleton layouts
- Better perceived performance with shimmer effects
- Type-safe fake data generation with BoneMock
- Flexible annotation system for edge cases
- Smooth animations between loading and content states
- Reduced development time for loading states
