# Advanced Security Guide

Enhanced security patterns for Firebase applications.

## Email Verification

### Setup

```typescript
import { sendEmailVerification, User } from 'firebase/auth';

async function sendVerification(user: User) {
  try {
    await sendEmailVerification(user);
    console.log('Verification email sent');
  } catch (error) {
    console.error('Error sending verification:', error);
  }
}
```

### After Sign Up

```typescript
const handleSignUp = async (email: string, password: string) => {
  try {
    const credential = await createUserWithEmailAndPassword(auth, email, password);
    await sendEmailVerification(credential.user);
    alert('Please check your email to verify your account');
  } catch (error) {
    // Handle error
  }
};
```

### Firestore Rules with Email Verification

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /{collection}/{document} {
      allow read, write: if request.auth != null &&
                            request.auth.token.email_verified == true &&
                            resource.data.userId == request.auth.uid;
    }
  }
}
```

## Custom Claims (Role-Based Access)

### Setup (Cloud Functions)

```typescript
// functions/src/index.ts
import * as functions from 'firebase-functions';
import * as admin from 'firebase-admin';

admin.initializeApp();

export const setUserRole = functions.https.onCall(async (data, context) => {
  // Only admins can set roles
  if (!context.auth || !context.auth.token.admin) {
    throw new functions.https.HttpsError('permission-denied', 'Must be admin');
  }

  const { userId, role } = data;

  await admin.auth().setCustomUserClaims(userId, { role });

  return { message: `Role ${role} set for user ${userId}` };
});
```

### Client-Side Usage

```typescript
import { getIdTokenResult } from 'firebase/auth';

async function checkUserRole(user: User) {
  const idTokenResult = await getIdTokenResult(user);
  const role = idTokenResult.claims.role;

  return role; // 'admin', 'user', etc.
}
```

### Firestore Rules with Roles

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /admin/{document} {
      allow read, write: if request.auth != null &&
                            request.auth.token.role == 'admin';
    }

    match /users/{userId} {
      allow read: if request.auth != null;
      allow write: if request.auth != null &&
                      (request.auth.uid == userId || request.auth.token.role == 'admin');
    }
  }
}
```

## Field-Level Security

### Immutable Fields

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /posts/{postId} {
      allow create: if request.auth != null &&
                       request.resource.data.authorId == request.auth.uid;

      allow update: if request.auth != null &&
                       resource.data.authorId == request.auth.uid &&
                       // Prevent changing authorId
                       request.resource.data.authorId == resource.data.authorId;
    }
  }
}
```

### Validate Data Types

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /todos/{todoId} {
      function validTodo() {
        return request.resource.data.keys().hasAll(['text', 'completed', 'userId']) &&
               request.resource.data.text is string &&
               request.resource.data.text.size() > 0 &&
               request.resource.data.text.size() <= 500 &&
               request.resource.data.completed is bool &&
               request.resource.data.userId is string;
      }

      allow create: if request.auth != null &&
                       validTodo() &&
                       request.resource.data.userId == request.auth.uid;

      allow update: if request.auth != null &&
                       resource.data.userId == request.auth.uid &&
                       validTodo();
    }
  }
}
```

## Rate Limiting

### Firestore Rules with Time-Based Limits

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /posts/{postId} {
      allow create: if request.auth != null &&
                       // Only allow 1 post per minute
                       (!exists(/databases/$(database)/documents/users/$(request.auth.uid)) ||
                        !exists(/databases/$(database)/documents/users/$(request.auth.uid)/lastPost) ||
                        request.time > resource.data.lastPost + duration.value(1, 'm'));
    }

    match /users/{userId}/lastPost {
      allow write: if request.auth != null && request.auth.uid == userId;
    }
  }
}
```

### Client-Side Implementation

```typescript
async function createPost(content: string) {
  const batch = writeBatch(db);

  // Create post
  const postRef = doc(collection(db, 'posts'));
  batch.set(postRef, {
    content,
    authorId: auth.currentUser!.uid,
    createdAt: serverTimestamp()
  });

  // Update lastPost timestamp
  const lastPostRef = doc(db, `users/${auth.currentUser!.uid}/lastPost`, 'timestamp');
  batch.set(lastPostRef, { lastPost: serverTimestamp() });

  await batch.commit();
}
```

## Security Audit Checklist

- ✅ All collections have security rules
- ✅ No `allow read, write: if true;` in production
- ✅ User data isolated by userId
- ✅ Email verification enforced for sensitive operations
- ✅ Admin operations use custom claims
- ✅ Data validation in rules
- ✅ Rate limiting for write operations
- ✅ API keys restricted to app domains
- ✅ `.env` file in `.gitignore`
- ✅ Regular security audits via Firebase Console
