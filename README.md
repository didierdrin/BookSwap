# bookswap

A Flutter project.

While Signing up you'll need to verify your email which the email will be in the spam emails 

Connecting App to Firebase

Initial Setup Process:
The connection to Firebase was established using FlutterFire CLI for seamless integration. The process involved:

1. Firebase Project Creation: Created a new project in Firebase Console
2. Platform Configuration: Added Android and iOS apps to the project
3. Authentication Setup: Enabled Email/Password authentication with email verification
4. Firestore Database: Configured security rules and collection structure
5. Dependency Integration: Added necessary Firebase packages to pubspec.yaml

Challenges Encountered and Resolution

Challenge 1: Firebase Initialization Errors

[ERROR:flutter/runtime/dart_vm_initializer.cc(41)] Unhandled Exception: [core/not-initialized] Firebase has not been correctly initialized.

Resolution: Added proper async initialization in main.dart:
Dart code:
WidgetsFlutterBinding.ensureInitialized();
await Firebase.initializeApp(
  options: DefaultFirebaseOptions.currentPlatform,
);


Challenge 2: Firestore Security Rules

[cloud_firestore/permission-denied] Missing or insufficient permissions.

**Resolution**: Updated Firestore rules to allow authenticated access:
Javascript code
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /{document=**} {
      allow read, write: if request.auth != null;
    }
  }
}


Challenge 3: Image Upload Issues

[firebase_storage/unauthorized] User is not authorized to perform the desired action.

Resolution: Updated Storage rules and implemented base64 encoding for images to avoid storage bucket permissions.

 Database Design Summary 


Database Modeling & Collections Structure

1. books
   - Fields: title, author, condition, imageUrl, ownerId, ownerEmail, createdAt, status, swapFor
   - Indexed on: ownerId, createdAt

2. swaps
   - Fields: bookId, senderId, receiverId, status, createdAt, updatedAt
   - Indexed on: senderId, receiverId, status

3. threads (for chats)
   - Fields: members[], lastText, updatedAt
   - Subcollection: messages (from, to, text, createdAt)

ERD Representation

Users (Firebase Auth) → Books (1:N)
Books → Swaps (1:1 when active)
Users ↔ Swaps (N:M through senderId/receiverId)
Users ↔ Threads (N:M through members[])


Swap State Modeling

The swap states follow this lifecycle in Firestore:

Dart codes
enum SwapStatus {
  '',          // Available (no active swap)
  'Pending',   // Swap requested
  'Accepted',  // Swap agreed by both parties  
  'Rejected',  // Swap declined
  'Completed'  // Swap finalized
}


State Transitions:
1. Initial: Book created with empty status
2. Pending: When swap is requested, status updates in both books collection and swaps collection
3. Accepted/Rejected: Receiver action updates status across collections
4. Completed: Final state after swap execution

State Management Implementation

Provider Architecture

MultiProvider (
  AuthService → User authentication state
  BookProvider → Books and swaps management
  ChatProvider → Real-time messaging
  NotificationProvider → UI notifications
)


Key Features:
- Reactive Updates: Using StreamBuilder with Firestore real-time streams
- Local State: Combined with remote state for optimal UX
- Error Handling: Comprehensive error states and loading indicators

Design Trade-offs & Challenges

Trade-offs Made
1. Image Storage: Used base64 encoding in Firestore instead of Cloud Storage to simplify implementation, trading storage efficiency for development speed
2. Real-time vs Performance: Implemented real-time listeners on all collections, accepting higher read operations for better user experience
3. Security Rules: Started with open rules for development, implemented authenticated-only access for production

Challenges Overcome:
1. Email Verification Flow: Implemented automatic redirect upon verification detection
2. Offline Support: Limited offline capability due to real-time nature, focused on robust online experience
3. Image Performance: Base64 images impact performance but simplify storage management

Future Improvements:
- Implement Cloud Storage for images
- Add offline support with sync
- Enhanced security rules with document-level permissions
- Push notifications for swap updates
