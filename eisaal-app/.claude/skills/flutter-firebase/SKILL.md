---
name: flutter-firebase
description: Expert Firebase integration for Flutter. Use when working with Firebase Auth, Firestore, Cloud Storage, Cloud Messaging (FCM), Analytics, Crashlytics, or Remote Config. Covers FlutterFire packages and best practices.
---

# Flutter Firebase Skill

## Setup

### Install FlutterFire CLI
```bash
dart pub global activate flutterfire_cli
```

### Configure Firebase
```bash
flutterfire configure
```

### Core Dependencies
```yaml
dependencies:
  firebase_core: ^2.27.0
  firebase_auth: ^4.17.0
  cloud_firestore: ^4.15.0
  firebase_storage: ^11.6.0
  firebase_messaging: ^14.7.0
  firebase_analytics: ^10.8.0
  firebase_crashlytics: ^3.4.0
```

### Initialize (main.dart)
```dart
import 'package:firebase_core/firebase_core.dart';
import 'firebase_options.dart';

Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  
  await Firebase.initializeApp(
    options: DefaultFirebaseOptions.currentPlatform,
  );
  
  // Optional: Crashlytics
  FlutterError.onError = FirebaseCrashlytics.instance.recordFlutterError;
  
  runApp(MyApp());
}
```

## Firebase Auth

### Email/Password Sign Up
```dart
Future<UserCredential> signUp(String email, String password) async {
  return await FirebaseAuth.instance.createUserWithEmailAndPassword(
    email: email,
    password: password,
  );
}
```

### Email/Password Sign In
```dart
Future<UserCredential> signIn(String email, String password) async {
  return await FirebaseAuth.instance.signInWithEmailAndPassword(
    email: email,
    password: password,
  );
}
```

### Google Sign In
```dart
// Add: google_sign_in: ^6.2.1

Future<UserCredential> signInWithGoogle() async {
  final GoogleSignInAccount? googleUser = await GoogleSignIn().signIn();
  
  if (googleUser == null) throw Exception('Sign in cancelled');
  
  final GoogleSignInAuthentication googleAuth = 
      await googleUser.authentication;
  
  final credential = GoogleAuthProvider.credential(
    accessToken: googleAuth.accessToken,
    idToken: googleAuth.idToken,
  );
  
  return await FirebaseAuth.instance.signInWithCredential(credential);
}
```

### Apple Sign In
```dart
// Add: sign_in_with_apple: ^5.0.0

Future<UserCredential> signInWithApple() async {
  final appleCredential = await SignInWithApple.getAppleIDCredential(
    scopes: [
      AppleIDAuthorizationScopes.email,
      AppleIDAuthorizationScopes.fullName,
    ],
  );
  
  final oauthCredential = OAuthProvider('apple.com').credential(
    idToken: appleCredential.identityToken,
    accessToken: appleCredential.authorizationCode,
  );
  
  return await FirebaseAuth.instance.signInWithCredential(oauthCredential);
}
```

### Sign Out
```dart
Future<void> signOut() async {
  await FirebaseAuth.instance.signOut();
  await GoogleSignIn().signOut();  // If using Google
}
```

### Current User
```dart
User? get currentUser => FirebaseAuth.instance.currentUser;
```

### Auth State Stream
```dart
@riverpod
Stream<User?> authState(Ref ref) {
  return FirebaseAuth.instance.authStateChanges();
}

// Or with more detail
Stream<User?> userChanges() {
  return FirebaseAuth.instance.userChanges();
}
```

### Password Reset
```dart
Future<void> resetPassword(String email) async {
  await FirebaseAuth.instance.sendPasswordResetEmail(email: email);
}
```

### Update Profile
```dart
Future<void> updateProfile({String? displayName, String? photoURL}) async {
  await FirebaseAuth.instance.currentUser?.updateDisplayName(displayName);
  await FirebaseAuth.instance.currentUser?.updatePhotoURL(photoURL);
}
```

## Cloud Firestore

### Get Collection
```dart
Future<List<Map<String, dynamic>>> getAll(String collection) async {
  final snapshot = await FirebaseFirestore.instance
      .collection(collection)
      .get();
  
  return snapshot.docs.map((doc) => {
    'id': doc.id,
    ...doc.data(),
  }).toList();
}
```

### Get Document
```dart
Future<Map<String, dynamic>?> getById(String collection, String id) async {
  final doc = await FirebaseFirestore.instance
      .collection(collection)
      .doc(id)
      .get();
  
  if (!doc.exists) return null;
  return {'id': doc.id, ...doc.data()!};
}
```

### Query with Filters
```dart
Future<List<Map<String, dynamic>>> query(
  String collection, {
  required String field,
  required dynamic value,
}) async {
  final snapshot = await FirebaseFirestore.instance
      .collection(collection)
      .where(field, isEqualTo: value)
      .orderBy('createdAt', descending: true)
      .limit(20)
      .get();
  
  return snapshot.docs.map((doc) => {
    'id': doc.id,
    ...doc.data(),
  }).toList();
}
```

### Add Document (Auto ID)
```dart
Future<String> add(String collection, Map<String, dynamic> data) async {
  final doc = await FirebaseFirestore.instance
      .collection(collection)
      .add({
        ...data,
        'createdAt': FieldValue.serverTimestamp(),
      });
  return doc.id;
}
```

### Set Document (Custom ID)
```dart
Future<void> set(String collection, String id, Map<String, dynamic> data) async {
  await FirebaseFirestore.instance
      .collection(collection)
      .doc(id)
      .set({
        ...data,
        'updatedAt': FieldValue.serverTimestamp(),
      });
}
```

### Update Document
```dart
Future<void> update(String collection, String id, Map<String, dynamic> data) async {
  await FirebaseFirestore.instance
      .collection(collection)
      .doc(id)
      .update({
        ...data,
        'updatedAt': FieldValue.serverTimestamp(),
      });
}
```

### Delete Document
```dart
Future<void> delete(String collection, String id) async {
  await FirebaseFirestore.instance
      .collection(collection)
      .doc(id)
      .delete();
}
```

### Realtime Stream (Single Doc)
```dart
Stream<Map<String, dynamic>?> watchDocument(String collection, String id) {
  return FirebaseFirestore.instance
      .collection(collection)
      .doc(id)
      .snapshots()
      .map((doc) => doc.exists ? {'id': doc.id, ...doc.data()!} : null);
}
```

### Realtime Stream (Collection)
```dart
Stream<List<Map<String, dynamic>>> watchCollection(String collection) {
  return FirebaseFirestore.instance
      .collection(collection)
      .orderBy('createdAt', descending: true)
      .snapshots()
      .map((snapshot) => snapshot.docs.map((doc) => {
        'id': doc.id,
        ...doc.data(),
      }).toList());
}
```

### Batch Write
```dart
Future<void> batchWrite(List<Map<String, dynamic>> items) async {
  final batch = FirebaseFirestore.instance.batch();
  final collection = FirebaseFirestore.instance.collection('prayers');
  
  for (final item in items) {
    batch.set(collection.doc(), item);
  }
  
  await batch.commit();
}
```

### Transaction
```dart
Future<void> transfer(String fromId, String toId, int amount) async {
  await FirebaseFirestore.instance.runTransaction((transaction) async {
    final fromDoc = FirebaseFirestore.instance.collection('accounts').doc(fromId);
    final toDoc = FirebaseFirestore.instance.collection('accounts').doc(toId);
    
    final fromSnapshot = await transaction.get(fromDoc);
    final toSnapshot = await transaction.get(toDoc);
    
    final fromBalance = fromSnapshot.data()!['balance'] as int;
    final toBalance = toSnapshot.data()!['balance'] as int;
    
    transaction.update(fromDoc, {'balance': fromBalance - amount});
    transaction.update(toDoc, {'balance': toBalance + amount});
  });
}
```

### Query Operators
```dart
// Equal
.where('field', isEqualTo: value)

// Not equal
.where('field', isNotEqualTo: value)

// Greater than
.where('field', isGreaterThan: value)

// Less than
.where('field', isLessThan: value)

// Array contains
.where('tags', arrayContains: 'ramadan')

// Array contains any
.where('tags', arrayContainsAny: ['ramadan', 'fasting'])

// In
.where('status', whereIn: ['active', 'pending'])

// Not in
.where('status', whereNotIn: ['deleted', 'archived'])
```

## Firebase Storage

### Upload File
```dart
Future<String> uploadFile(String path, File file) async {
  final ref = FirebaseStorage.instance.ref().child(path);
  await ref.putFile(file);
  return await ref.getDownloadURL();
}
```

### Upload Bytes
```dart
Future<String> uploadBytes(String path, Uint8List bytes) async {
  final ref = FirebaseStorage.instance.ref().child(path);
  await ref.putData(bytes);
  return await ref.getDownloadURL();
}
```

### Download URL
```dart
Future<String> getDownloadUrl(String path) async {
  return await FirebaseStorage.instance.ref().child(path).getDownloadURL();
}
```

### Delete File
```dart
Future<void> deleteFile(String path) async {
  await FirebaseStorage.instance.ref().child(path).delete();
}
```

### Upload with Progress
```dart
Stream<double> uploadWithProgress(String path, File file) {
  final ref = FirebaseStorage.instance.ref().child(path);
  final task = ref.putFile(file);
  
  return task.snapshotEvents.map((snapshot) {
    return snapshot.bytesTransferred / snapshot.totalBytes;
  });
}
```

## Cloud Messaging (FCM)

### Setup
```dart
Future<void> setupFCM() async {
  final messaging = FirebaseMessaging.instance;
  
  // Request permission (iOS)
  await messaging.requestPermission(
    alert: true,
    badge: true,
    sound: true,
  );
  
  // Get token
  final token = await messaging.getToken();
  print('FCM Token: $token');
  
  // Listen for token refresh
  messaging.onTokenRefresh.listen((token) {
    // Save to server
  });
}
```

### Handle Messages
```dart
void setupMessageHandlers() {
  // Foreground messages
  FirebaseMessaging.onMessage.listen((message) {
    print('Foreground: ${message.notification?.title}');
    // Show local notification
  });
  
  // Background messages (app open)
  FirebaseMessaging.onMessageOpenedApp.listen((message) {
    print('Opened from background: ${message.data}');
    // Navigate to screen
  });
}

// Background handler (must be top-level function)
@pragma('vm:entry-point')
Future<void> _firebaseMessagingBackgroundHandler(RemoteMessage message) async {
  await Firebase.initializeApp();
  print('Background message: ${message.messageId}');
}

// Register in main()
FirebaseMessaging.onBackgroundMessage(_firebaseMessagingBackgroundHandler);
```

### Subscribe to Topic
```dart
Future<void> subscribeToTopic(String topic) async {
  await FirebaseMessaging.instance.subscribeToTopic(topic);
}

Future<void> unsubscribeFromTopic(String topic) async {
  await FirebaseMessaging.instance.unsubscribeFromTopic(topic);
}
```

## Analytics

```dart
final analytics = FirebaseAnalytics.instance;

// Log event
await analytics.logEvent(
  name: 'prayer_completed',
  parameters: {
    'prayer_name': 'fajr',
    'on_time': true,
  },
);

// Log screen
await analytics.logScreenView(
  screenName: 'HomeScreen',
  screenClass: 'HomePage',
);

// Set user property
await analytics.setUserProperty(
  name: 'preferred_language',
  value: 'urdu',
);

// Set user ID
await analytics.setUserId(id: userId);
```

## Crashlytics

```dart
final crashlytics = FirebaseCrashlytics.instance;

// Log error
await crashlytics.recordError(
  error,
  stackTrace,
  reason: 'Payment failed',
  fatal: false,
);

// Set user ID
await crashlytics.setUserIdentifier(userId);

// Custom key
await crashlytics.setCustomKey('screen', 'checkout');

// Log message
await crashlytics.log('User clicked checkout button');
```

## Error Handling Pattern

```dart
Future<Either<Failure, User>> getUser(String id) async {
  try {
    final doc = await FirebaseFirestore.instance
        .collection('users')
        .doc(id)
        .get();
    
    if (!doc.exists) {
      return Left(NotFoundFailure(message: 'User not found'));
    }
    
    return Right(UserModel.fromJson({
      'id': doc.id,
      ...doc.data()!,
    }).toEntity());
  } on FirebaseException catch (e) {
    return Left(FirebaseFailure(message: e.message ?? 'Firebase error'));
  } catch (e) {
    return Left(ServerFailure(message: e.toString()));
  }
}
```

## Repository Pattern

```dart
class PrayerRemoteDataSource {
  final FirebaseFirestore _firestore;

  PrayerRemoteDataSource(this._firestore);

  Future<List<PrayerModel>> getAll() async {
    final snapshot = await _firestore
        .collection('prayers')
        .orderBy('time')
        .get();
    
    return snapshot.docs
        .map((doc) => PrayerModel.fromJson({'id': doc.id, ...doc.data()}))
        .toList();
  }

  Stream<List<PrayerModel>> watchAll() {
    return _firestore
        .collection('prayers')
        .orderBy('time')
        .snapshots()
        .map((snapshot) => snapshot.docs
            .map((doc) => PrayerModel.fromJson({'id': doc.id, ...doc.data()}))
            .toList());
  }
}
```

## Common Mistakes

```dart
// ❌ Not awaiting Firebase.initializeApp()
void main() {
  Firebase.initializeApp();  // Missing await!
  runApp(MyApp());
}

// ✅ Correct
Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp(options: DefaultFirebaseOptions.currentPlatform);
  runApp(MyApp());
}

// ❌ Using .get() when you need realtime
final data = await FirebaseFirestore.instance.collection('x').get();

// ✅ Use .snapshots() for realtime
final stream = FirebaseFirestore.instance.collection('x').snapshots();

// ❌ Not handling null document
final doc = await FirebaseFirestore.instance.collection('x').doc(id).get();
return doc.data()!;  // Might crash!

// ✅ Handle null
if (!doc.exists) throw Exception('Not found');
return doc.data()!;
```
