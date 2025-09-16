# Flutter Project Structure and Architecture

## When to Apply

This rule applies to ALL Flutter projects to ensure:

- Consistent project organization across teams
- Maintainable and scalable codebase
- Clear separation of concerns
- Reusable components and utilities
- Proper data flow and state management architecture

## Mandatory Project Structure

### Root Structure

```
lib/
├── core/                    # REQUIRED - Shared components used across multiple features
├── features/               # REQUIRED - Feature-specific implementations
├── main.dart              # REQUIRED - App entry point
└── app.dart               # REQUIRED - App configuration
```

### Core Folder Structure

The `core/` folder contains ONLY components that are used in multiple features or across the entire application:

```
lib/core/
├── constants/             # App-wide constants
│   ├── api_constants.dart
│   ├── app_constants.dart
│   └── storage_keys.dart
├── fonts/                 # Custom font files
│   └── [font_files].ttf
├── helpers/               # Utility functions and extensions
│   ├── date_helper.dart
│   ├── string_extensions.dart
│   └── validation_helpers.dart
├── hooks/                 # Reusable flutter_hooks
│   ├── use_api_call.dart
│   └── use_debounce.dart
├── includes/              # Common imports and exports
│   ├── common_imports.dart
│   └── barrel_exports.dart
├── providers/             # Global Riverpod providers
│   ├── auth_provider.dart
│   ├── theme_provider.dart
│   └── app_state_provider.dart
├── styles/                # Global theming and styles
│   ├── app_theme.dart
│   ├── colors.dart
│   └── text_styles.dart
├── types/                 # Global type definitions
│   ├── api_response.dart
│   └── app_enums.dart
├── utils/                 # Utility classes and functions
│   ├── logger.dart
│   ├── device_info.dart
│   └── permissions.dart
├── validations/           # Global validation schemas
│   ├── common_validations.dart
│   └── form_validators.dart
└── models/               # Global/shared data models
    ├── user_model.dart
    └── api_error_model.dart
```

### Features Folder Structure

Each feature follows a consistent internal structure:

```
lib/features/[feature_name]/
├── models/               # Feature-specific models
│   ├── [feature]_model.dart
│   └── [feature]_request.dart
├── repository/           # API connections and data sources
│   └── [feature]_repository.dart
├── mutations/            # Data mutation operations
│   ├── create_[feature]_mutation.dart
│   ├── update_[feature]_mutation.dart
│   └── delete_[feature]_mutation.dart
├── queries/              # Data query operations
│   ├── get_[feature]_query.dart
│   └── list_[features]_query.dart
├── validations/          # Feature-specific validations
│   └── [feature]_validations.dart
├── widgets/              # Feature-specific widgets
│   ├── [feature]_card.dart
│   ├── [feature]_form.dart
│   └── [feature]_list_item.dart
└── pages/                # Feature pages/screens
    ├── [feature]_list_page.dart
    ├── [feature]_detail_page.dart
    └── [feature]_create_page.dart
```

## Implementation Rules

### 1. Core Folder Usage Rule

**Components belong in `core/` ONLY if they are used by 2 or more features:**

```dart
// ✅ CORRECT - Goes in core/helpers/ (used across features)
class DateHelper {
  static String formatDate(DateTime date) {
    return DateFormat('dd/MM/yyyy').format(date);
  }

  static bool isToday(DateTime date) {
    final now = DateTime.now();
    return date.year == now.year &&
           date.month == now.month &&
           date.day == now.day;
  }
}

// ✅ CORRECT - Goes in core/models/ (used by multiple features)
class User {
  final String id;
  final String name;
  final String email;

  const User({
    required this.id,
    required this.name,
    required this.email,
  });
}

// ❌ INCORRECT - Should be in features/products/models/
class ProductCategory {
  // Only used by products feature
}
```

### 2. Repository Structure

All external data connections go in the repository layer:

```dart
// features/products/repository/products_repository.dart
import 'package:supabase_flutter/supabase_flutter.dart';

class ProductsRepository {
  final SupabaseClient _supabase = Supabase.instance.client;

  // Get all products
  Future<List<Map<String, dynamic>>> getAllProducts() async {
    try {
      final response = await _supabase
          .from('products')
          .select()
          .order('created_at', ascending: false);

      return List<Map<String, dynamic>>.from(response);
    } catch (e) {
      throw Exception('Failed to fetch products: $e');
    }
  }

  // Get single product
  Future<Map<String, dynamic>?> getProductById(String id) async {
    try {
      final response = await _supabase
          .from('products')
          .select()
          .eq('id', id)
          .single();

      return response;
    } catch (e) {
      throw Exception('Failed to fetch product: $e');
    }
  }

  // Create product
  Future<Map<String, dynamic>> createProduct(Map<String, dynamic> data) async {
    try {
      final response = await _supabase
          .from('products')
          .insert(data)
          .select()
          .single();

      return response;
    } catch (e) {
      throw Exception('Failed to create product: $e');
    }
  }

  // Update product
  Future<Map<String, dynamic>> updateProduct(String id, Map<String, dynamic> data) async {
    try {
      final response = await _supabase
          .from('products')
          .update(data)
          .eq('id', id)
          .select()
          .single();

      return response;
    } catch (e) {
      throw Exception('Failed to update product: $e');
    }
  }

  // Delete product
  Future<void> deleteProduct(String id) async {
    try {
      await _supabase
          .from('products')
          .delete()
          .eq('id', id);
    } catch (e) {
      throw Exception('Failed to delete product: $e');
    }
  }
}
```

### 3. Queries Structure

Query providers ONLY use repository functions:

```dart
// features/products/queries/get_products_query.dart
import 'package:flutter_riverpod/flutter_riverpod.dart';
import '../repository/products_repository.dart';
import '../models/product_model.dart';

// Repository provider
final productsRepositoryProvider = Provider<ProductsRepository>((ref) {
  return ProductsRepository();
});

// Get all products query
final getProductsProvider = FutureProvider<List<Product>>((ref) async {
  final repository = ref.read(productsRepositoryProvider);
  final data = await repository.getAllProducts();

  return data.map((item) => Product.fromJson(item)).toList();
});

// Get single product query
final getProductProvider = FutureProvider.family<Product?, String>((ref, id) async {
  final repository = ref.read(productsRepositoryProvider);
  final data = await repository.getProductById(id);

  return data != null ? Product.fromJson(data) : null;
});

// Search products query
final searchProductsProvider = FutureProvider.family<List<Product>, String>((ref, query) async {
  final repository = ref.read(productsRepositoryProvider);
  // Implement search in repository first
  final data = await repository.searchProducts(query);

  return data.map((item) => Product.fromJson(item)).toList();
});
```

### 4. Mutations Structure

Mutation providers ONLY use repository functions:

```dart
// features/products/mutations/create_product_mutation.dart
import 'package:flutter_riverpod/flutter_riverpod.dart';
import '../repository/products_repository.dart';
import '../models/product_model.dart';

class CreateProductNotifier extends StateNotifier<AsyncValue<Product?>> {
  CreateProductNotifier(this.repository) : super(const AsyncValue.data(null));

  final ProductsRepository repository;

  Future<void> createProduct(Map<String, dynamic> productData) async {
    state = const AsyncValue.loading();

    try {
      final data = await repository.createProduct(productData);
      final product = Product.fromJson(data);
      state = AsyncValue.data(product);

      // Invalidate related queries
      ref.invalidate(getProductsProvider);
    } catch (error, stackTrace) {
      state = AsyncValue.error(error, stackTrace);
    }
  }
}

final createProductProvider = StateNotifierProvider<CreateProductNotifier, AsyncValue<Product?>>((ref) {
  final repository = ref.read(productsRepositoryProvider);
  return CreateProductNotifier(repository);
});

// features/products/mutations/update_product_mutation.dart
class UpdateProductNotifier extends StateNotifier<AsyncValue<Product?>> {
  UpdateProductNotifier(this.repository) : super(const AsyncValue.data(null));

  final ProductsRepository repository;

  Future<void> updateProduct(String id, Map<String, dynamic> updates) async {
    state = const AsyncValue.loading();

    try {
      final data = await repository.updateProduct(id, updates);
      final product = Product.fromJson(data);
      state = AsyncValue.data(product);

      // Invalidate related queries
      ref.invalidate(getProductsProvider);
      ref.invalidate(getProductProvider(id));
    } catch (error, stackTrace) {
      state = AsyncValue.error(error, stackTrace);
    }
  }
}

final updateProductProvider = StateNotifierProvider<UpdateProductNotifier, AsyncValue<Product?>>((ref) {
  final repository = ref.read(productsRepositoryProvider);
  return UpdateProductNotifier(repository);
});

// features/products/mutations/delete_product_mutation.dart
class DeleteProductNotifier extends StateNotifier<AsyncValue<void>> {
  DeleteProductNotifier(this.repository) : super(const AsyncValue.data(null));

  final ProductsRepository repository;

  Future<void> deleteProduct(String id) async {
    state = const AsyncValue.loading();

    try {
      await repository.deleteProduct(id);
      state = const AsyncValue.data(null);

      // Invalidate related queries
      ref.invalidate(getProductsProvider);
    } catch (error, stackTrace) {
      state = AsyncValue.error(error, stackTrace);
    }
  }
}

final deleteProductProvider = StateNotifierProvider<DeleteProductNotifier, AsyncValue<void>>((ref) {
  final repository = ref.read(productsRepositoryProvider);
  return DeleteProductNotifier(repository);
});
```

### 5. Models Structure

Feature-specific models in their respective folders:

```dart
// features/products/models/product_model.dart
import 'package:freezed_annotation/freezed_annotation.dart';

part 'product_model.freezed.dart';
part 'product_model.g.dart';

@freezed
class Product with _$Product {
  const factory Product({
    required String id,
    required String name,
    required String description,
    required double price,
    required String categoryId,
    String? imageUrl,
    required DateTime createdAt,
    required DateTime updatedAt,
  }) = _Product;

  factory Product.fromJson(Map<String, dynamic> json) => _$ProductFromJson(json);
}

@freezed
class CreateProductRequest with _$CreateProductRequest {
  const factory CreateProductRequest({
    required String name,
    required String description,
    required double price,
    required String categoryId,
    String? imageUrl,
  }) = _CreateProductRequest;

  factory CreateProductRequest.fromJson(Map<String, dynamic> json) =>
      _$CreateProductRequestFromJson(json);
}
```

### 6. Validations Structure

Feature-specific validations:

```dart
// features/products/validations/product_validations.dart
import 'package:acanthis/acanthis.dart' as v;

class ProductValidations {
  static final createProductSchema = v.object({
    'name': v.string().min(3).max(100),
    'description': v.string().min(10).max(500),
    'price': v.double().positive(),
    'categoryId': v.string().uuid(),
    'imageUrl': v.string().url().optional(),
  });

  static final updateProductSchema = v.object({
    'name': v.string().min(3).max(100).optional(),
    'description': v.string().min(10).max(500).optional(),
    'price': v.double().positive().optional(),
    'categoryId': v.string().uuid().optional(),
    'imageUrl': v.string().url().optional(),
  });

  static String? validateProductName(String? value) {
    if (value == null || value.isEmpty) {
      return 'Product name is required';
    }

    final result = v.string().min(3).max(100).tryParse(value);
    return result.success ? null : result.errors.first;
  }

  static String? validatePrice(String? value) {
    if (value == null || value.isEmpty) {
      return 'Price is required';
    }

    final numValue = double.tryParse(value);
    if (numValue == null) {
      return 'Please enter a valid price';
    }

    final result = v.double().positive().tryParse(numValue);
    return result.success ? null : result.errors.first;
  }
}
```

### 7. Widgets Structure

Feature-specific widgets:

```dart
// features/products/widgets/product_card.dart
import 'package:flutter/material.dart';
import 'package:cached_network_image/cached_network_image.dart';
import 'package:flutter_screenutil/flutter_screenutil.dart';
import '../models/product_model.dart';

class ProductCard extends StatelessWidget {
  final Product product;
  final VoidCallback? onTap;

  const ProductCard({
    Key? key,
    required this.product,
    this.onTap,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Card(
      margin: EdgeInsets.all(8.w),
      child: InkWell(
        onTap: onTap,
        borderRadius: BorderRadius.circular(8.r),
        child: Padding(
          padding: EdgeInsets.all(12.w),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              if (product.imageUrl != null)
                ClipRRect(
                  borderRadius: BorderRadius.circular(8.r),
                  child: CachedNetworkImage(
                    imageUrl: product.imageUrl!,
                    height: 120.h,
                    width: double.infinity,
                    fit: BoxFit.cover,
                    placeholder: (context, url) =>
                        Container(height: 120.h, color: Colors.grey[200]),
                    errorWidget: (context, url, error) =>
                        Container(height: 120.h, child: Icon(Icons.error)),
                  ),
                ),
              8.verticalSpace,
              Text(
                product.name,
                style: TextStyle(fontSize: 16.sp, fontWeight: FontWeight.bold),
                maxLines: 1,
                overflow: TextOverflow.ellipsis,
              ),
              4.verticalSpace,
              Text(
                product.description,
                style: TextStyle(fontSize: 12.sp, color: Colors.grey[600]),
                maxLines: 2,
                overflow: TextOverflow.ellipsis,
              ),
              8.verticalSpace,
              Text(
                '\$${product.price.toStringAsFixed(2)}',
                style: TextStyle(
                  fontSize: 14.sp,
                  fontWeight: FontWeight.bold,
                  color: Theme.of(context).primaryColor,
                ),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

## Forbidden Patterns

### ❌ Wrong Structure Examples:

```dart
// ❌ FORBIDDEN - Putting feature-specific code in core
core/models/product_model.dart  // Should be in features/products/models/

// ❌ FORBIDDEN - Direct API calls in queries/mutations
final getProductsProvider = FutureProvider<List<Product>>((ref) async {
  // Don't do direct Supabase calls here
  final response = await Supabase.instance.client.from('products').select();
  // ...
});

// ❌ FORBIDDEN - Business logic in repository
class ProductsRepository {
  Future<List<Product>> getDiscountedProducts() async {
    final data = await _supabase.from('products').select();

    // Don't do business logic here
    return data.where((p) => p['price'] < 100).map(...).toList();
  }
}
```

### ✅ Correct Structure Examples:

```dart
// ✅ CORRECT - Repository only handles data access
class ProductsRepository {
  Future<List<Map<String, dynamic>>> getProductsByPriceRange(double min, double max) async {
    return await _supabase
        .from('products')
        .select()
        .gte('price', min)
        .lte('price', max);
  }
}

// ✅ CORRECT - Business logic in queries
final discountedProductsProvider = FutureProvider<List<Product>>((ref) async {
  final repository = ref.read(productsRepositoryProvider);
  final data = await repository.getProductsByPriceRange(0, 100);
  return data.map((item) => Product.fromJson(item)).toList();
});
```

## Benefits Enforced

- Clear separation of data access, business logic, and UI
- Reusable components properly organized in core
- Consistent project structure across all features
- Easy testing with isolated repository layer
- Scalable architecture that supports growth
- Proper dependency injection with Riverpod
- Type-safe data models and validation
- Maintainable codebase with clear responsibilities
