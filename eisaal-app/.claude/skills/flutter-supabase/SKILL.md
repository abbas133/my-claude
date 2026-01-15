---
name: flutter-supabase
description: Expert Supabase integration for Flutter. Use when working with Supabase auth, database queries, realtime subscriptions, storage, edge functions, or RLS policies. Covers supabase_flutter package patterns.
---

# Flutter Supabase Skill

## Setup

### Dependencies
```yaml
dependencies:
  supabase_flutter: ^2.5.0
```

### Initialize (main.dart)
```dart
import 'package:supabase_flutter/supabase_flutter.dart';

Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  
  await Supabase.initialize(
    url: 'YOUR_SUPABASE_URL',
    anonKey: 'YOUR_ANON_KEY',
  );
  
  runApp(MyApp());
}

// Global client access
final supabase = Supabase.instance.client;
```

### With Riverpod
```dart
@Riverpod(keepAlive: true)
SupabaseClient supabaseClient(Ref ref) {
  return Supabase.instance.client;
}
```

## Authentication

### Email/Password Sign Up
```dart
Future<AuthResponse> signUp(String email, String password) async {
  return await supabase.auth.signUp(
    email: email,
    password: password,
  );
}
```

### Email/Password Sign In
```dart
Future<AuthResponse> signIn(String email, String password) async {
  return await supabase.auth.signInWithPassword(
    email: email,
    password: password,
  );
}
```

### Google OAuth
```dart
Future<void> signInWithGoogle() async {
  await supabase.auth.signInWithOAuth(
    OAuthProvider.google,
    redirectTo: 'io.supabase.yourapp://login-callback/',
  );
}
```

### Sign Out
```dart
Future<void> signOut() async {
  await supabase.auth.signOut();
}
```

### Get Current User
```dart
User? get currentUser => supabase.auth.currentUser;

// Or with null check
User getCurrentUser() {
  final user = supabase.auth.currentUser;
  if (user == null) throw Exception('Not authenticated');
  return user;
}
```

### Auth State Stream
```dart
@riverpod
Stream<AuthState> authState(Ref ref) {
  return supabase.auth.onAuthStateChange;
}

// Usage in widget
ref.listen(authStateProvider, (prev, next) {
  next.whenData((state) {
    if (state.event == AuthChangeEvent.signedOut) {
      context.go('/login');
    }
  });
});
```

## Database Queries

### Select All
```dart
Future<List<Map<String, dynamic>>> getAll() async {
  final response = await supabase
      .from('prayers')
      .select();
  return response;
}
```

### Select with Columns
```dart
Future<List<Map<String, dynamic>>> getNames() async {
  final response = await supabase
      .from('prayers')
      .select('id, name, time');
  return response;
}
```

### Select with Filter
```dart
Future<List<Map<String, dynamic>>> getByCategory(String category) async {
  final response = await supabase
      .from('duas')
      .select()
      .eq('category', category)
      .order('name');
  return response;
}
```

### Select Single Row
```dart
Future<Map<String, dynamic>> getById(String id) async {
  final response = await supabase
      .from('users')
      .select()
      .eq('id', id)
      .single();
  return response;
}
```

### Select with Relations (Joins)
```dart
Future<List<Map<String, dynamic>>> getPrayersWithMosque() async {
  final response = await supabase
      .from('prayers')
      .select('''
        id,
        name,
        time,
        mosque:mosques(id, name, address)
      ''');
  return response;
}
```

### Insert
```dart
Future<Map<String, dynamic>> create(Map<String, dynamic> data) async {
  final response = await supabase
      .from('prayers')
      .insert(data)
      .select()
      .single();
  return response;
}
```

### Insert Multiple
```dart
Future<List<Map<String, dynamic>>> createMany(List<Map<String, dynamic>> data) async {
  final response = await supabase
      .from('prayers')
      .insert(data)
      .select();
  return response;
}
```

### Update
```dart
Future<Map<String, dynamic>> update(String id, Map<String, dynamic> data) async {
  final response = await supabase
      .from('prayers')
      .update(data)
      .eq('id', id)
      .select()
      .single();
  return response;
}
```

### Upsert
```dart
Future<Map<String, dynamic>> upsert(Map<String, dynamic> data) async {
  final response = await supabase
      .from('prayers')
      .upsert(data)
      .select()
      .single();
  return response;
}
```

### Delete
```dart
Future<void> delete(String id) async {
  await supabase
      .from('prayers')
      .delete()
      .eq('id', id);
}
```

### Filter Operators
```dart
// Equal
.eq('column', value)

// Not equal
.neq('column', value)

// Greater than
.gt('column', value)

// Greater than or equal
.gte('column', value)

// Less than
.lt('column', value)

// Less than or equal
.lte('column', value)

// Pattern match (LIKE)
.like('name', '%prayer%')

// Case insensitive pattern
.ilike('name', '%Prayer%')

// In array
.inFilter('id', ['1', '2', '3'])

// Contains (for arrays)
.contains('tags', ['ramadan'])

// Is null
.isFilter('deleted_at', null)

// Range
.range(0, 9)  // First 10 rows

// Full text search
.textSearch('name', 'fajr')
```

### Ordering & Pagination
```dart
final response = await supabase
    .from('duas')
    .select()
    .order('created_at', ascending: false)
    .range(0, 19);  // First 20 items
```

## Realtime Subscriptions

### Subscribe to Table Changes
```dart
@riverpod
Stream<List<Map<String, dynamic>>> prayerTimesStream(Ref ref) {
  return supabase
      .from('prayer_times')
      .stream(primaryKey: ['id'])
      .order('time');
}
```

### Subscribe with Filter
```dart
Stream<List<Map<String, dynamic>>> userPrayersStream(String userId) {
  return supabase
      .from('prayers')
      .stream(primaryKey: ['id'])
      .eq('user_id', userId);
}
```

### Broadcast Channels
```dart
final channel = supabase.channel('room1');

channel
    .onBroadcast(event: 'cursor', callback: (payload) {
      print('Received: $payload');
    })
    .subscribe();

// Send
channel.sendBroadcastMessage(
  event: 'cursor',
  payload: {'x': 100, 'y': 200},
);
```

## Storage

### Upload File
```dart
Future<String> uploadImage(String path, Uint8List bytes) async {
  await supabase.storage
      .from('avatars')
      .uploadBinary(path, bytes);
  
  return supabase.storage
      .from('avatars')
      .getPublicUrl(path);
}
```

### Upload from File
```dart
Future<String> uploadFile(String path, File file) async {
  await supabase.storage
      .from('documents')
      .upload(path, file);
  
  return supabase.storage
      .from('documents')
      .getPublicUrl(path);
}
```

### Download File
```dart
Future<Uint8List> downloadFile(String path) async {
  return await supabase.storage
      .from('documents')
      .download(path);
}
```

### Get Signed URL (Time-Limited)
```dart
Future<String> getSignedUrl(String path) async {
  return await supabase.storage
      .from('private')
      .createSignedUrl(path, 3600);  // 1 hour
}
```

### Delete File
```dart
Future<void> deleteFile(String path) async {
  await supabase.storage
      .from('avatars')
      .remove([path]);
}
```

### List Files
```dart
Future<List<FileObject>> listFiles(String folder) async {
  return await supabase.storage
      .from('documents')
      .list(path: folder);
}
```

## Edge Functions

```dart
Future<Map<String, dynamic>> callFunction(
  String name,
  Map<String, dynamic> body,
) async {
  final response = await supabase.functions.invoke(
    name,
    body: body,
  );
  return response.data;
}

// Example
final result = await callFunction('calculate-khums', {
  'income': 50000,
  'expenses': 30000,
  'marja': 'sistani',
});
```

## Error Handling Pattern

```dart
Future<Either<Failure, User>> getUser(String id) async {
  try {
    final response = await supabase
        .from('users')
        .select()
        .eq('id', id)
        .single();
    return Right(UserModel.fromJson(response).toEntity());
  } on PostgrestException catch (e) {
    return Left(DatabaseFailure(message: e.message));
  } on AuthException catch (e) {
    return Left(AuthFailure(message: e.message));
  } catch (e) {
    return Left(ServerFailure(message: e.toString()));
  }
}
```

## Repository Pattern

```dart
// lib/modules/prayers/data/datasources/prayer_remote_datasource.dart

class PrayerRemoteDataSource {
  final SupabaseClient _client;

  PrayerRemoteDataSource(this._client);

  Future<List<PrayerModel>> getAll() async {
    final response = await _client
        .from('prayers')
        .select()
        .order('time');
    return response.map((e) => PrayerModel.fromJson(e)).toList();
  }

  Future<PrayerModel> getById(String id) async {
    final response = await _client
        .from('prayers')
        .select()
        .eq('id', id)
        .single();
    return PrayerModel.fromJson(response);
  }

  Stream<List<PrayerModel>> watchAll() {
    return _client
        .from('prayers')
        .stream(primaryKey: ['id'])
        .order('time')
        .map((data) => data.map((e) => PrayerModel.fromJson(e)).toList());
  }
}
```

## RLS (Row Level Security) Notes

Always design queries assuming RLS is enabled:

```sql
-- Example RLS policy
CREATE POLICY "Users can view own data"
ON prayers FOR SELECT
USING (auth.uid() = user_id);

-- Your Dart code doesn't need .eq('user_id', userId)
-- RLS automatically filters!
```

## Common Mistakes

```dart
// ❌ Not awaiting
supabase.from('prayers').select();  // Missing await!

// ✅ Correct
await supabase.from('prayers').select();

// ❌ Using .single() when multiple rows possible
final response = await supabase.from('prayers').select().single();

// ✅ Use .maybeSingle() for optional single row
final response = await supabase.from('prayers').select().maybeSingle();

// ❌ Forgetting to handle null
final user = supabase.auth.currentUser;
user.id;  // Might be null!

// ✅ Null check
final user = supabase.auth.currentUser;
if (user == null) throw Exception('Not authenticated');
```
