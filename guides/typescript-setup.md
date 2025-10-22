# TypeScript Setup Guide

Converting the React + Firebase project to TypeScript.

## Initial Setup

```bash
# Install TypeScript dependencies
npm install -D typescript @types/react @types/react-dom

# Generate tsconfig.json
npx tsc --init
```

## TypeScript Configuration

`tsconfig.json`:
```json
{
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "skipLibCheck": true,

    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",

    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true
  },
  "include": ["src"],
  "references": [{ "path": "./tsconfig.node.json" }]
}
```

## Type Definitions

### Firebase Types

```typescript
// src/types/firebase.ts
import { Timestamp } from 'firebase/firestore';

export interface FirebaseConfig {
  apiKey: string;
  authDomain: string;
  projectId: string;
  storageBucket: string;
  messagingSenderId: string;
  appId: string;
}

export interface FirestoreDocument {
  id: string;
  userId: string;
  createdAt: Timestamp;
  updatedAt: Timestamp;
}

export interface TodoItem extends FirestoreDocument {
  text: string;
  completed: boolean;
}
```

### Component Props

```typescript
// src/components/Auth/Login.tsx
interface LoginProps {
  onToggleForm: () => void;
}

export default function Login({ onToggleForm }: LoginProps) {
  // ...
}
```

## Firebase Config with Types

```typescript
// src/firebase/config.ts
import { initializeApp, FirebaseApp } from 'firebase/app';
import { getAuth, Auth } from 'firebase/auth';
import { getFirestore, Firestore } from 'firebase/firestore';
import { FirebaseConfig } from '../types/firebase';

const firebaseConfig: FirebaseConfig = {
  apiKey: import.meta.env.VITE_FIREBASE_API_KEY,
  authDomain: import.meta.env.VITE_FIREBASE_AUTH_DOMAIN,
  projectId: import.meta.env.VITE_FIREBASE_PROJECT_ID,
  storageBucket: import.meta.env.VITE_FIREBASE_STORAGE_BUCKET,
  messagingSenderId: import.meta.env.VITE_FIREBASE_MESSAGING_SENDER_ID,
  appId: import.meta.env.VITE_FIREBASE_APP_ID
};

const app: FirebaseApp = initializeApp(firebaseConfig);
export const auth: Auth = getAuth(app);
export const db: Firestore = getFirestore(app);
export default app;
```

## Custom Hook with Types

```typescript
// src/hooks/useFirestore.ts
import { useState, useEffect } from 'react';
import {
  collection,
  query,
  where,
  orderBy,
  onSnapshot,
  addDoc,
  updateDoc,
  deleteDoc,
  doc,
  serverTimestamp,
  QueryDocumentSnapshot
} from 'firebase/firestore';
import { db } from '../firebase/config';
import { FirestoreDocument } from '../types/firebase';

interface UseFirestoreReturn<T extends FirestoreDocument> {
  items: T[];
  loading: boolean;
  addItem: (data: Omit<T, keyof FirestoreDocument>) => Promise<void>;
  updateItem: (id: string, updates: Partial<Omit<T, keyof FirestoreDocument>>) => Promise<void>;
  deleteItem: (id: string) => Promise<void>;
}

export default function useFirestore<T extends FirestoreDocument>(
  collectionName: string,
  userId: string | null
): UseFirestoreReturn<T> {
  const [items, setItems] = useState<T[]>([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    if (!userId) {
      setItems([]);
      setLoading(false);
      return;
    }

    const q = query(
      collection(db, collectionName),
      where('userId', '==', userId),
      orderBy('createdAt', 'desc')
    );

    const unsubscribe = onSnapshot(q, (snapshot) => {
      const data = snapshot.docs.map((doc: QueryDocumentSnapshot) => ({
        id: doc.id,
        ...doc.data()
      })) as T[];
      setItems(data);
      setLoading(false);
    });

    return () => unsubscribe();
  }, [collectionName, userId]);

  const addItem = async (data: Omit<T, keyof FirestoreDocument>) => {
    await addDoc(collection(db, collectionName), {
      ...data,
      userId,
      createdAt: serverTimestamp(),
      updatedAt: serverTimestamp()
    });
  };

  const updateItem = async (id: string, updates: Partial<Omit<T, keyof FirestoreDocument>>) => {
    await updateDoc(doc(db, collectionName, id), {
      ...updates,
      updatedAt: serverTimestamp()
    });
  };

  const deleteItem = async (id: string) => {
    await deleteDoc(doc(db, collectionName, id));
  };

  return { items, loading, addItem, updateItem, deleteItem };
}
```

## Environment Variables Types

```typescript
// src/vite-env.d.ts
/// <reference types="vite/client" />

interface ImportMetaEnv {
  readonly VITE_FIREBASE_API_KEY: string
  readonly VITE_FIREBASE_AUTH_DOMAIN: string
  readonly VITE_FIREBASE_PROJECT_ID: string
  readonly VITE_FIREBASE_STORAGE_BUCKET: string
  readonly VITE_FIREBASE_MESSAGING_SENDER_ID: string
  readonly VITE_FIREBASE_APP_ID: string
}

interface ImportMeta {
  readonly env: ImportMetaEnv
}
```

## File Renaming

Rename all `.jsx` files to `.tsx`:
```bash
find src -name "*.jsx" -exec sh -c 'mv "$1" "${1%.jsx}.tsx"' _ {} \;
```
