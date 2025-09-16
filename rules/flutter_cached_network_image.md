# Flutter Network Images with Cached Network Image

## When to Apply

This rule applies to ALL Flutter projects that display:

- Images from URLs/network sources
- User profile pictures
- Product images
- Gallery images
- Any remote image content

## Required Package

Always use `cached_network_image: ^3.4.1` as the ONLY solution for displaying network images instead of Image.network, NetworkImage, or custom image loading solutions.

## Mandatory Implementation

### 1. Basic Network Image Display

Always use CachedNetworkImage with required error handling:

```dart
import 'package:cached_network_image/cached_network_image.dart';

CachedNetworkImage(
  imageUrl: "https://example.com/image.jpg",
  placeholder: (context, url) => const CircularProgressIndicator(),
  errorWidget: (context, url, error) => const Icon(
    Icons.error,
    color: Colors.red,
  ),
)
```

### 2. Progress Indicator Implementation

Use progress indicators for better user experience:

```dart
CachedNetworkImage(
  imageUrl: imageUrl,
  progressIndicatorBuilder: (context, url, downloadProgress) =>
    CircularProgressIndicator(
      value: downloadProgress.progress,
      backgroundColor: Colors.grey[300],
      valueColor: AlwaysStoppedAnimation<Color>(Colors.blue),
    ),
  errorWidget: (context, url, error) => Container(
    color: Colors.grey[300],
    child: const Icon(
      Icons.broken_image,
      color: Colors.grey,
      size: 50,
    ),
  ),
)
```

### 3. Custom Image Builder Pattern

Use imageBuilder for complex image layouts:

```dart
CachedNetworkImage(
  imageUrl: imageUrl,
  imageBuilder: (context, imageProvider) => Container(
    decoration: BoxDecoration(
      borderRadius: BorderRadius.circular(12),
      image: DecorationImage(
        image: imageProvider,
        fit: BoxFit.cover,
      ),
    ),
  ),
  placeholder: (context, url) => Container(
    decoration: BoxDecoration(
      color: Colors.grey[300],
      borderRadius: BorderRadius.circular(12),
    ),
    child: const Center(
      child: CircularProgressIndicator(),
    ),
  ),
  errorWidget: (context, url, error) => Container(
    decoration: BoxDecoration(
      color: Colors.grey[300],
      borderRadius: BorderRadius.circular(12),
    ),
    child: const Icon(Icons.error),
  ),
)
```

### 4. Profile Picture Component

Create reusable profile picture widgets:

```dart
class ProfilePicture extends StatelessWidget {
  final String? imageUrl;
  final double size;
  final String? fallbackText;

  const ProfilePicture({
    Key? key,
    this.imageUrl,
    required this.size,
    this.fallbackText,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return ClipRRect(
      borderRadius: BorderRadius.circular(size / 2),
      child: SizedBox(
        width: size,
        height: size,
        child: imageUrl != null && imageUrl!.isNotEmpty
            ? CachedNetworkImage(
                imageUrl: imageUrl!,
                fit: BoxFit.cover,
                placeholder: (context, url) => Container(
                  color: Colors.grey[300],
                  child: const Center(
                    child: CircularProgressIndicator(strokeWidth: 2),
                  ),
                ),
                errorWidget: (context, url, error) => Container(
                  color: Colors.grey[400],
                  child: fallbackText != null
                      ? Center(
                          child: Text(
                            fallbackText![0].toUpperCase(),
                            style: TextStyle(
                              fontSize: size * 0.4,
                              fontWeight: FontWeight.bold,
                              color: Colors.white,
                            ),
                          ),
                        )
                      : const Icon(Icons.person, color: Colors.white),
                ),
              )
            : Container(
                color: Colors.grey[400],
                child: const Icon(Icons.person, color: Colors.white),
              ),
      ),
    );
  }
}
```

### 5. Product/Gallery Image Component

Create components for product images with zoom functionality:

```dart
class ProductImage extends StatelessWidget {
  final String imageUrl;
  final double? width;
  final double? height;
  final BoxFit fit;

  const ProductImage({
    Key? key,
    required this.imageUrl,
    this.width,
    this.height,
    this.fit = BoxFit.cover,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: () => _showFullScreenImage(context),
      child: Hero(
        tag: imageUrl,
        child: CachedNetworkImage(
          imageUrl: imageUrl,
          width: width,
          height: height,
          fit: fit,
          placeholder: (context, url) => Container(
            width: width,
            height: height,
            color: Colors.grey[200],
            child: const Center(
              child: CircularProgressIndicator(),
            ),
          ),
          errorWidget: (context, url, error) => Container(
            width: width,
            height: height,
            color: Colors.grey[300],
            child: const Column(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                Icon(Icons.broken_image, size: 50, color: Colors.grey),
                Text('Image not available', style: TextStyle(color: Colors.grey)),
              ],
            ),
          ),
        ),
      ),
    );
  }

  void _showFullScreenImage(BuildContext context) {
    Navigator.push(
      context,
      MaterialPageRoute(
        builder: (context) => Scaffold(
          backgroundColor: Colors.black,
          appBar: AppBar(
            backgroundColor: Colors.transparent,
            iconTheme: const IconThemeData(color: Colors.white),
          ),
          body: Center(
            child: Hero(
              tag: imageUrl,
              child: InteractiveViewer(
                child: CachedNetworkImage(
                  imageUrl: imageUrl,
                  fit: BoxFit.contain,
                  placeholder: (context, url) => const CircularProgressIndicator(
                    color: Colors.white,
                  ),
                  errorWidget: (context, url, error) => const Icon(
                    Icons.error,
                    color: Colors.white,
                    size: 100,
                  ),
                ),
              ),
            ),
          ),
        ),
      ),
    );
  }
}
```

### 6. Image Grid/List Implementation

For image lists and grids:

```dart
class ImageGrid extends StatelessWidget {
  final List<String> imageUrls;
  final double aspectRatio;

  const ImageGrid({
    Key? key,
    required this.imageUrls,
    this.aspectRatio = 1.0,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return GridView.builder(
      gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
        crossAxisCount: 2,
        crossAxisSpacing: 8,
        mainAxisSpacing: 8,
        childAspectRatio: aspectRatio,
      ),
      itemCount: imageUrls.length,
      itemBuilder: (context, index) {
        return ClipRRect(
          borderRadius: BorderRadius.circular(8),
          child: CachedNetworkImage(
            imageUrl: imageUrls[index],
            fit: BoxFit.cover,
            placeholder: (context, url) => Container(
              color: Colors.grey[200],
              child: const Center(
                child: CircularProgressIndicator(),
              ),
            ),
            errorWidget: (context, url, error) => Container(
              color: Colors.grey[300],
              child: const Icon(Icons.broken_image),
            ),
          ),
        );
      },
    );
  }
}
```

### 7. Image Provider Usage

When using with other widgets that need ImageProvider:

```dart
// For Container decoration
Container(
  decoration: BoxDecoration(
    image: DecorationImage(
      image: CachedNetworkImageProvider(imageUrl),
      fit: BoxFit.cover,
    ),
  ),
)

// For CircleAvatar
CircleAvatar(
  backgroundImage: CachedNetworkImageProvider(imageUrl),
  onBackgroundImageError: (error, stackTrace) {
    // Handle error
  },
)

// For custom image handling
class CustomImageWidget extends StatelessWidget {
  final String imageUrl;

  @override
  Widget build(BuildContext context) {
    return CachedNetworkImage(
      imageUrl: imageUrl,
      imageBuilder: (context, imageProvider) => MyCustomWidget(
        imageProvider: imageProvider,
      ),
      placeholder: (context, url) => const CircularProgressIndicator(),
      errorWidget: (context, url, error) => const Icon(Icons.error),
    );
  }
}
```

### 8. Cache Configuration

Configure caching behavior globally:

```dart
import 'package:flutter_cache_manager/flutter_cache_manager.dart';

class CustomCacheManager extends CacheManager with ImageCacheManager {
  static const key = 'customCacheKey';

  static final CustomCacheManager _instance = CustomCacheManager._();
  factory CustomCacheManager() => _instance;

  CustomCacheManager._()
      : super(
          Config(
            key,
            stalePeriod: const Duration(days: 7),
            maxNrOfCacheObjects: 20,
            repo: JsonCacheInfoRepository(databaseName: key),
            fileService: HttpFileService(),
          ),
        );
}

// Usage with custom cache manager
CachedNetworkImage(
  imageUrl: imageUrl,
  cacheManager: CustomCacheManager(),
  placeholder: (context, url) => const CircularProgressIndicator(),
  errorWidget: (context, url, error) => const Icon(Icons.error),
)
```

### 9. Performance Optimization

Always set appropriate image dimensions:

```dart
CachedNetworkImage(
  imageUrl: imageUrl,
  width: 200, // Always specify dimensions when known
  height: 200,
  fit: BoxFit.cover,
  memCacheWidth: 200, // Optimize memory cache
  memCacheHeight: 200,
  placeholder: (context, url) => const SizedBox(
    width: 200,
    height: 200,
    child: Center(child: CircularProgressIndicator()),
  ),
  errorWidget: (context, url, error) => const SizedBox(
    width: 200,
    height: 200,
    child: Icon(Icons.error),
  ),
)
```

### 10. Error Handling and Retry Logic

Implement comprehensive error handling:

```dart
class RobustNetworkImage extends StatefulWidget {
  final String imageUrl;
  final Widget? placeholder;
  final int maxRetries;

  const RobustNetworkImage({
    Key? key,
    required this.imageUrl,
    this.placeholder,
    this.maxRetries = 3,
  }) : super(key: key);

  @override
  State<RobustNetworkImage> createState() => _RobustNetworkImageState();
}

class _RobustNetworkImageState extends State<RobustNetworkImage> {
  int retryCount = 0;
  String? currentImageUrl;

  @override
  void initState() {
    super.initState();
    currentImageUrl = widget.imageUrl;
  }

  @override
  Widget build(BuildContext context) {
    return CachedNetworkImage(
      key: ValueKey('${widget.imageUrl}_$retryCount'),
      imageUrl: currentImageUrl!,
      placeholder: (context, url) => widget.placeholder ??
        const CircularProgressIndicator(),
      errorWidget: (context, url, error) {
        if (retryCount < widget.maxRetries) {
          return GestureDetector(
            onTap: () {
              setState(() {
                retryCount++;
                currentImageUrl = '${widget.imageUrl}?retry=$retryCount';
              });
            },
            child: Container(
              color: Colors.grey[300],
              child: const Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  Icon(Icons.refresh, color: Colors.grey),
                  Text('Tap to retry', style: TextStyle(color: Colors.grey)),
                ],
              ),
            ),
          );
        }

        return Container(
          color: Colors.grey[300],
          child: const Icon(Icons.broken_image, color: Colors.grey),
        );
      },
    );
  }
}
```

## Required Dependencies

```yaml
dependencies:
  cached_network_image: ^3.4.1
```

## Forbidden Patterns

- ❌ Using Image.network() for remote images
- ❌ Using NetworkImage() directly
- ❌ Not providing error widgets
- ❌ Not providing placeholder widgets
- ❌ Creating custom image caching solutions
- ❌ Not handling image loading failures
- ❌ Not specifying image dimensions when known
- ❌ Ignoring memory optimization parameters

## Required Error Handling

Always provide meaningful error widgets:

```dart
// ✅ REQUIRED - Proper error widget
errorWidget: (context, url, error) => Container(
  color: Colors.grey[300],
  child: const Column(
    mainAxisAlignment: MainAxisAlignment.center,
    children: [
      Icon(Icons.broken_image, color: Colors.grey),
      Text('Image not available', style: TextStyle(color: Colors.grey, fontSize: 12)),
    ],
  ),
)

// ❌ FORBIDDEN - No error handling
CachedNetworkImage(imageUrl: url) // Missing error widget
```

## Required Placeholder Patterns

Always provide loading indicators:

```dart
// ✅ REQUIRED - Proper placeholder
placeholder: (context, url) => Container(
  color: Colors.grey[200],
  child: const Center(
    child: CircularProgressIndicator(),
  ),
)

// ✅ ALTERNATIVE - Progress indicator
progressIndicatorBuilder: (context, url, downloadProgress) =>
  CircularProgressIndicator(value: downloadProgress.progress)
```

## Benefits Enforced

- Automatic image caching and memory management
- Improved app performance and reduced network usage
- Better user experience with loading states
- Consistent error handling across the app
- Optimized memory usage for image display
- Support for various image formats and sources
- Built-in retry mechanisms and cache invalidation
