# Flutter Async State Management with fquery

## When to Apply

This rule applies to ALL Flutter projects that involve:

- API calls (REST, GraphQL, etc.)
- Database operations (SQLite, Firebase, etc.)
- Any asynchronous data fetching
- Server state management
- Data caching requirements

## Required Package

Always use `fquery: ^1.5.4-beta.2` instead of manual FutureBuilder, Riverpod for async data, or custom caching solutions.

## Mandatory Implementation

### 1. App Setup

```dart
void main() {
  final queryClient = QueryClient(
    defaultQueryOptions: DefaultQueryOptions(
      cacheDuration: Duration(minutes: 5),
      staleDuration: Duration(seconds: 30),
      retryCount: 3,
    ),
  );

  runApp(
    QueryClientProvider(
      queryClient: queryClient,
      child: MyApp(),
    ),
  );
}
```

### 2. Query Implementation

For data fetching, ALWAYS use either:

**Option A: With HookWidget**

```dart
class DataWidget extends HookWidget {
  @override
  Widget build(BuildContext context) {
    final data = useQuery(['key'], fetchFunction);

    if (data.isLoading) return CircularProgressIndicator();
    if (data.isError) return Text('Error: ${data.error}');

    return YourWidget(data: data.data!);
  }
}
```

**Option B: With QueryBuilder**

```dart
QueryBuilder<DataType, Error>(
  ['key'],
  fetchFunction,
  builder: (context, query) {
    if (query.isLoading) return CircularProgressIndicator();
    if (query.isError) return Text('Error: ${query.error}');

    return YourWidget(data: query.data!);
  },
)
```

### 3. Mutations for Data Updates

```dart
final mutation = useMutation<ReturnType, Error, InputType>(
  mutationFunction,
  onSuccess: (data, variables, context) {
    // Invalidate related queries
    queryClient.invalidateQueries(['related-key']);
  },
);

// Usage
mutation.mutate(inputData);
```

### 4. Infinite Scrolling

```dart
final infiniteQuery = useInfiniteQuery<PageResult, Error, int>(
  ['infinite-key'],
  (pageParam) => fetchPage(pageParam),
  initialPageParam: 1,
  getNextPageParam: (lastPage, allPages, lastPageParam, allPageParams) {
    return lastPage.hasMore ? lastPageParam + 1 : null;
  },
);
```

## Forbidden Patterns

- ❌ Manual FutureBuilder for server data
- ❌ setState() for async data management
- ❌ Custom caching logic
- ❌ Riverpod/Bloc for simple async data fetching
- ❌ Direct HTTP calls without caching

## Required Dependencies

```yaml
dependencies:
  flutter_hooks: ^0.20.5
  fquery: ^1.5.4-beta.2
```

## Exception Cases

Only bypass this rule for:

- One-time operations that don't need caching
- Real-time streams (use StreamBuilder)
- Local state that doesn't come from async sources

## Benefits Enforced

- Automatic caching and garbage collection
- Automatic re-fetching of stale data
- Built-in loading and error states
- Query invalidation and manual updates
- Optimistic updates support
- Parallel and dependent queries
- No boilerplate code
