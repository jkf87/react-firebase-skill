# Next.js + Firebase Setup Guide

Alternative to Vite when SSR, SSG, or App Router features are needed.

## When to Use Next.js Instead of Vite

- Need server-side rendering (SEO-critical pages)
- Want static site generation for blog/docs
- Require API routes without separate backend
- Planning to use middleware for auth protection

## Initial Setup

```bash
npx create-next-app@latest [name] --typescript --tailwind --app
cd [name]
npm install firebase
```

Configuration options:
- ✅ TypeScript (recommended)
- ✅ App Router (use App Router, not Pages Router)
- ✅ Tailwind CSS
- ❌ src/ directory (optional, but cleaner)

## Firebase Client Initialization

**app/firebase/config.ts**

```typescript
import { initializeApp, getApps } from 'firebase/app';
import { getAuth } from 'firebase/auth';
import { getFirestore } from 'firebase/firestore';

const firebaseConfig = {
  apiKey: process.env.NEXT_PUBLIC_FIREBASE_API_KEY,
  authDomain: process.env.NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN,
  projectId: process.env.NEXT_PUBLIC_FIREBASE_PROJECT_ID,
  storageBucket: process.env.NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET,
  messagingSenderId: process.env.NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID,
  appId: process.env.NEXT_PUBLIC_FIREBASE_APP_ID
};

// Prevent re-initialization in development (hot reload)
const app = getApps().length === 0 ? initializeApp(firebaseConfig) : getApps()[0];

export const auth = getAuth(app);
export const db = getFirestore(app);
export default app;
```

## Environment Variables

**.env.local** (not `.env`):
```env
NEXT_PUBLIC_FIREBASE_API_KEY=...
NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN=...
NEXT_PUBLIC_FIREBASE_PROJECT_ID=...
NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET=...
NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID=...
NEXT_PUBLIC_FIREBASE_APP_ID=...
```

**Critical**: Use `NEXT_PUBLIC_` prefix for client-side variables.

## Server vs Client Components

### Authentication Context (Client Component)

**app/providers/AuthProvider.tsx**

```typescript
'use client';

import { createContext, useContext, useEffect, useState } from 'react';
import { User, onAuthStateChanged } from 'firebase/auth';
import { auth } from '@/app/firebase/config';

const AuthContext = createContext<{ user: User | null }>({ user: null });

export function AuthProvider({ children }: { children: React.ReactNode }) {
  const [user, setUser] = useState<User | null>(null);

  useEffect(() => {
    const unsubscribe = onAuthStateChanged(auth, setUser);
    return () => unsubscribe();
  }, []);

  return <AuthContext.Provider value={{ user }}>{children}</AuthContext.Provider>;
}

export const useAuth = () => useContext(AuthContext);
```

### Root Layout (Server Component)

**app/layout.tsx**

```typescript
import { AuthProvider } from './providers/AuthProvider';
import './globals.css';

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="ko">
      <body>
        <AuthProvider>
          {children}
        </AuthProvider>
      </body>
    </html>
  );
}
```

## Protected Routes with Middleware

**middleware.ts** (project root)

```typescript
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  const authCookie = request.cookies.get('auth-token');

  // Redirect to login if no auth cookie
  if (!authCookie && request.nextUrl.pathname !== '/login') {
    return NextResponse.redirect(new URL('/login', request.url));
  }

  return NextResponse.next();
}

export const config = {
  matcher: ['/dashboard/:path*', '/profile/:path*'],
};
```

**Note**: For production, use Firebase Admin SDK in middleware for server-side token verification.

## Firebase Deployment Configuration

**firebase.json**

```json
{
  "hosting": {
    "public": "out",
    "ignore": ["firebase.json", "**/.*", "**/node_modules/**"],
    "rewrites": [
      {
        "source": "**",
        "destination": "/index.html"
      }
    ]
  },
  "firestore": {
    "rules": "firestore.rules",
    "indexes": "firestore.indexes.json"
  }
}
```

## Build and Deploy

### Static Export (SSG)

**next.config.mjs**

```javascript
/** @type {import('next').NextConfig} */
const nextConfig = {
  output: 'export',
  images: {
    unoptimized: true
  }
};

export default nextConfig;
```

Build and deploy:
```bash
npm run build
npx firebase deploy --only hosting
```

### Server-Side Rendering (Requires Node.js Server)

For SSR, use Firebase Cloud Functions or other Node.js hosting (Vercel, Netlify).

**Not compatible** with `firebase deploy --only hosting` (static only).

## Key Differences from Vite

| Feature | Vite | Next.js |
|---------|------|---------|
| Env vars | `import.meta.env.VITE_*` | `process.env.NEXT_PUBLIC_*` |
| Init check | Not needed | `getApps().length === 0` check |
| Client components | All by default | Need `'use client'` directive |
| Routing | React Router | File-based (app/ directory) |
| Static export | `dist/` | `out/` |

## Component Example

**app/components/TodoList.tsx**

```typescript
'use client';

import { useEffect, useState } from 'react';
import { collection, query, where, onSnapshot } from 'firebase/firestore';
import { db } from '@/app/firebase/config';
import { useAuth } from '@/app/providers/AuthProvider';

export default function TodoList() {
  const { user } = useAuth();
  const [todos, setTodos] = useState([]);

  useEffect(() => {
    if (!user) return;

    const q = query(
      collection(db, 'todos'),
      where('userId', '==', user.uid)
    );

    const unsubscribe = onSnapshot(q, (snapshot) => {
      setTodos(snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() })));
    });

    return () => unsubscribe();
  }, [user]);

  // ... render
}
```

## Common Issues

**"You're importing a component that needs useState..."**
- Add `'use client'` to top of file

**"process is not defined"**
- Use `NEXT_PUBLIC_` prefix for client-side env vars

**Firebase re-initializes on hot reload**
- Add `getApps().length === 0` check in config

**Middleware doesn't work with static export**
- Use client-side auth checks instead, or deploy to Node.js server

## When to Choose Next.js

✅ **Use Next.js if**:
- SEO is critical (blog, marketing site)
- Need API routes
- Want file-based routing
- Planning to scale to SSR later

❌ **Use Vite if**:
- Building admin dashboard (no SEO needed)
- Want fastest dev server
- Prefer simpler mental model
- Don't need SSR/SSG features

## Resources

- Next.js App Router: https://nextjs.org/docs/app
- Firebase with Next.js: https://firebase.google.com/docs/web/setup#nextjs
