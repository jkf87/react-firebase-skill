---
name: react-firebase
description: "React + Firebase fullstack application creation, deployment, and configuration"
---

# React Firebase Deployment

**CRITICAL**: When a user asks you to:
- "Create a React app with Firebase backend"
- "Make a todo app with Firebase"
- "Build a [X] app and deploy it with Firebase"
- "Use Firebase skill to build..."
- Mentions "Firebase" + "backend" + "deploy"

**YOU MUST immediately follow this skill workflow** - do NOT create files manually.

## What This Skill Does

Automates complete Firebase deployment in one shot:
1. Creates React project with Vite
2. Creates Firebase project via CLI (`npx firebase projects:create`)
3. Extracts real Firebase config via CLI (`npx firebase apps:sdkconfig`)
4. Deploys Firestore database with security rules
5. Deploys to Firebase Hosting
6. Provides live production URL

**Time**: 10-15 minutes from empty directory to live URL

**DO NOT**:
- ❌ Create package.json manually
- ❌ Write placeholder values like "YOUR_API_KEY"
- ❌ Create firebase.json without running CLI commands
- ❌ Skip Firebase CLI commands and just make files

**DO**:
- ✅ Run `npx firebase projects:create` to create real project
- ✅ Run `npx firebase apps:sdkconfig` to get real API keys
- ✅ Run `npx firebase deploy` to deploy
- ✅ Provide live URL at the end

## Creating a New React + Firebase Application

When the user requests a Firebase app (todo, chat, any app with backend):

### Step 1: Decide on Firebase Project

**IMPORTANT**: Check if user wants to use existing Firebase project or create new one.

**Default behavior**: Create a NEW Firebase project (don't ask user, just do it).

**Ask user ONLY if**:
- You detect they already have a Firebase project in current directory (`.firebaserc` exists)
- They explicitly mention "use my existing project"

**If creating NEW project**:

Ask the user for:
- Project name (lowercase, no spaces, e.g., "todo-app")
- Display name for Firebase (e.g., "My Todo App")

Generate these values:
- `<project-name>`: Use user's project name
- `<display-name>`: Use user's display name
- `<app-name>`: Use `<display-name> + " Web"`
- `<project-id>`: Construct as `<project-name>-<timestamp>` (run `date +%s` to get timestamp)

**If using EXISTING project**:

List available projects:
```bash
npx firebase projects:list
```

Show list to user and ask which project to use. Save the project ID they choose.

### Step 2: Create React Project

```bash
npm create vite@latest <project-name> -- --template react
cd <project-name>
npm install
```

Verify `package.json` exists and contains "react" in dependencies.

### Step 3: Install Dependencies

```bash
npm install firebase
npm install -D tailwindcss@^3 postcss autoprefixer
npx tailwindcss init -p
```

**Critical**: Use Tailwind v3 (not v4) - v4 has CLI compatibility issues.

Update `tailwind.config.js` content array:
```javascript
content: ['./index.html', './src/**/*.{js,jsx,ts,tsx}']
```

Update `src/index.css` - prepend these lines:
```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

### Step 4: Check Firebase CLI

```bash
npx firebase --version
```

If command fails or version < 14.0.0:
```bash
npm install -g firebase-tools@latest
```

Check Firebase login status:
```bash
npx firebase login:list
```

If no user listed, login now:
```bash
npx firebase login
```

This opens a browser for authentication.

### Step 5: Create Firebase Project

**ONLY IF creating NEW project** (Step 1 determined this).

**Skip this step if using existing project.**

Generate timestamp and construct project ID:
```bash
date +%s
# Use output to create: <project-name>-<timestamp>
# Example: todo-app-1729680000
```

Create the Firebase project:
```bash
npx firebase projects:create <project-id> --display-name "<display-name>"
```

**Example**:
```bash
npx firebase projects:create todo-app-1729680000 --display-name "My Todo App"
```

**Critical**: Parse the output for "Firebase project <project-id> is ready!" to confirm success. Save this project ID for all subsequent commands.

**If this command fails** with "Permission denied" or "Login required":
```bash
npx firebase login
# Then retry the create command
```

### Step 6: Register Web App

```bash
npx firebase apps:create WEB "<app-name>" --project <project-id>
```

Parse output for the App ID (format: `1:123456789:web:abcdef123456`). Save this app ID.

### Step 7: Get Firebase Configuration

```bash
npx firebase apps:sdkconfig WEB <app-id> --project <project-id>
```

This outputs JSON with these fields:
- `apiKey`
- `authDomain`
- `projectId`
- `storageBucket`
- `messagingSenderId`
- `appId`

**Critical**: Parse all these values and use them in the next step.

### Step 8: Create Environment File

Create `.env` with the actual values from step 7:
```env
VITE_FIREBASE_API_KEY=<apiKey-from-step-7>
VITE_FIREBASE_AUTH_DOMAIN=<authDomain-from-step-7>
VITE_FIREBASE_PROJECT_ID=<projectId-from-step-7>
VITE_FIREBASE_STORAGE_BUCKET=<storageBucket-from-step-7>
VITE_FIREBASE_MESSAGING_SENDER_ID=<messagingSenderId-from-step-7>
VITE_FIREBASE_APP_ID=<appId-from-step-7>
```

Verify the file was created correctly:
```bash
cat .env
```

Create `.env.example` with placeholders:
```env
VITE_FIREBASE_API_KEY=your_api_key_here
VITE_FIREBASE_AUTH_DOMAIN=your_project_id.firebaseapp.com
VITE_FIREBASE_PROJECT_ID=your_project_id
VITE_FIREBASE_STORAGE_BUCKET=your_project_id.firebasestorage.app
VITE_FIREBASE_MESSAGING_SENDER_ID=your_messaging_sender_id
VITE_FIREBASE_APP_ID=your_app_id
```

### Step 9: Create Firebase Config Files

Create `.firebaserc` with project ID from step 5:
```json
{
  "projects": {
    "default": "<project-id>"
  }
}
```

Verify:
```bash
cat .firebaserc | grep <project-id>
```

Create `firebase.json`:
```json
{
  "hosting": {
    "public": "dist",
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

### Step 10: Create Firestore Security Rules

Create `firestore.rules`:
```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /{collection}/{document} {
      allow read: if request.auth != null &&
                     resource.data.userId == request.auth.uid;
      allow create: if request.auth != null &&
                       request.resource.data.userId == request.auth.uid;
      allow update, delete: if request.auth != null &&
                               resource.data.userId == request.auth.uid;
    }
  }
}
```

Create `firestore.indexes.json`:
```json
{
  "indexes": [],
  "fieldOverrides": []
}
```

Verify:
```bash
cat firestore.indexes.json
```

### Step 11: Deploy Firestore

**Critical step**: This command creates the Firestore database and applies security rules.

```bash
npx firebase deploy --only firestore:rules,firestore:indexes --project <project-id>
```

This automatically:
- Enables Firestore API (takes 1-2 minutes first time)
- Creates Firestore database in us-central region
- Applies security rules
- Deploys indexes

Parse output for:
- "✔ firestore: deployed indexes"
- "✔ firestore: released rules"

If errors occur, stop and report to user. Otherwise, Firestore is ready.

### Step 12: Create Directory Structure

```bash
mkdir -p src/components/Auth
mkdir -p src/components/Layout
mkdir -p src/firebase
mkdir -p src/hooks
```

### Step 13: Implement Firebase Config

Create `src/firebase/config.js`:
```javascript
import { initializeApp } from 'firebase/app';
import { getAuth } from 'firebase/auth';
import { getFirestore } from 'firebase/firestore';

const firebaseConfig = {
  apiKey: import.meta.env.VITE_FIREBASE_API_KEY,
  authDomain: import.meta.env.VITE_FIREBASE_AUTH_DOMAIN,
  projectId: import.meta.env.VITE_FIREBASE_PROJECT_ID,
  storageBucket: import.meta.env.VITE_FIREBASE_STORAGE_BUCKET,
  messagingSenderId: import.meta.env.VITE_FIREBASE_MESSAGING_SENDER_ID,
  appId: import.meta.env.VITE_FIREBASE_APP_ID
};

const app = initializeApp(firebaseConfig);
export const auth = getAuth(app);
export const db = getFirestore(app);
```

**Critical**: Use `import.meta.env.VITE_*` for Vite environment variables (NOT `process.env`).

### Step 14: Implement Authentication Components

Create `src/components/Auth/Login.jsx`:
```javascript
import { useState } from 'react';
import { signInWithEmailAndPassword, createUserWithEmailAndPassword } from 'firebase/auth';
import { auth } from '../../firebase/config';

export default function Login() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [isSignUp, setIsSignUp] = useState(false);
  const [error, setError] = useState('');

  const handleSubmit = async (e) => {
    e.preventDefault();
    setError('');
    try {
      if (isSignUp) {
        await createUserWithEmailAndPassword(auth, email, password);
      } else {
        await signInWithEmailAndPassword(auth, email, password);
      }
    } catch (err) {
      setError(err.message);
    }
  };

  return (
    <div className="min-h-screen flex items-center justify-center bg-gray-100">
      <div className="bg-white p-8 rounded-lg shadow-md w-96">
        <h2 className="text-2xl font-bold mb-6 text-center">
          {isSignUp ? 'Sign Up' : 'Login'}
        </h2>
        <form onSubmit={handleSubmit} className="space-y-4">
          <input
            type="email"
            placeholder="Email"
            value={email}
            onChange={(e) => setEmail(e.target.value)}
            className="w-full p-2 border rounded"
            required
          />
          <input
            type="password"
            placeholder="Password"
            value={password}
            onChange={(e) => setPassword(e.target.value)}
            className="w-full p-2 border rounded"
            required
          />
          {error && <p className="text-red-500 text-sm">{error}</p>}
          <button
            type="submit"
            className="w-full bg-blue-500 text-white p-2 rounded hover:bg-blue-600"
          >
            {isSignUp ? 'Sign Up' : 'Login'}
          </button>
        </form>
        <button
          onClick={() => setIsSignUp(!isSignUp)}
          className="w-full mt-4 text-blue-500 hover:underline"
        >
          {isSignUp ? 'Already have an account? Login' : 'Need an account? Sign Up'}
        </button>
      </div>
    </div>
  );
}
```

Create `src/components/Layout/Header.jsx`:
```javascript
import { signOut } from 'firebase/auth';
import { auth } from '../../firebase/config';

export default function Header({ user }) {
  const handleLogout = () => {
    signOut(auth);
  };

  return (
    <header className="bg-blue-600 text-white p-4">
      <div className="container mx-auto flex justify-between items-center">
        <h1 className="text-2xl font-bold">My App</h1>
        <div className="flex items-center gap-4">
          <span>{user?.email}</span>
          <button
            onClick={handleLogout}
            className="bg-blue-700 px-4 py-2 rounded hover:bg-blue-800"
          >
            Logout
          </button>
        </div>
      </div>
    </header>
  );
}
```

### Step 15: Implement Feature Component (Example: Todos)

Create `src/hooks/useTodos.js`:
```javascript
import { useState, useEffect } from 'react';
import { collection, addDoc, deleteDoc, updateDoc, doc, query, where, onSnapshot, serverTimestamp } from 'firebase/firestore';
import { db } from '../firebase/config';

export function useTodos(userId) {
  const [todos, setTodos] = useState([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    if (!userId) return;

    const q = query(
      collection(db, 'todos'),
      where('userId', '==', userId)
    );

    const unsubscribe = onSnapshot(q, (snapshot) => {
      const todosData = snapshot.docs.map(doc => ({
        id: doc.id,
        ...doc.data()
      }));
      setTodos(todosData);
      setLoading(false);
    });

    return () => unsubscribe();
  }, [userId]);

  const addTodo = async (text) => {
    await addDoc(collection(db, 'todos'), {
      text,
      completed: false,
      userId,
      createdAt: serverTimestamp()
    });
  };

  const toggleTodo = async (id, completed) => {
    await updateDoc(doc(db, 'todos', id), { completed: !completed });
  };

  const deleteTodo = async (id) => {
    await deleteDoc(doc(db, 'todos', id));
  };

  return { todos, loading, addTodo, toggleTodo, deleteTodo };
}
```

Create `src/components/TodoList.jsx`:
```javascript
import { useState } from 'react';
import { useTodos } from '../hooks/useTodos';

export default function TodoList({ user }) {
  const [newTodo, setNewTodo] = useState('');
  const { todos, loading, addTodo, toggleTodo, deleteTodo } = useTodos(user.uid);

  const handleSubmit = (e) => {
    e.preventDefault();
    if (!newTodo.trim()) return;
    addTodo(newTodo);
    setNewTodo('');
  };

  if (loading) {
    return <div className="text-center p-8">Loading...</div>;
  }

  return (
    <div className="max-w-2xl mx-auto p-6">
      <form onSubmit={handleSubmit} className="mb-6 flex gap-2">
        <input
          type="text"
          value={newTodo}
          onChange={(e) => setNewTodo(e.target.value)}
          placeholder="Add a new todo..."
          className="flex-1 p-2 border rounded"
        />
        <button
          type="submit"
          className="bg-blue-500 text-white px-6 py-2 rounded hover:bg-blue-600"
        >
          Add
        </button>
      </form>
      <div className="space-y-2">
        {todos.map((todo) => (
          <div key={todo.id} className="flex items-center gap-2 p-3 bg-white rounded shadow">
            <input
              type="checkbox"
              checked={todo.completed}
              onChange={() => toggleTodo(todo.id, todo.completed)}
              className="w-5 h-5"
            />
            <span className={`flex-1 ${todo.completed ? 'line-through text-gray-400' : ''}`}>
              {todo.text}
            </span>
            <button
              onClick={() => deleteTodo(todo.id)}
              className="text-red-500 hover:text-red-700"
            >
              Delete
            </button>
          </div>
        ))}
      </div>
    </div>
  );
}
```

**Critical**: Always use `serverTimestamp()` for created dates (not `new Date()`).

### Step 16: Update App.jsx

Create `src/App.jsx`:
```javascript
import { useState, useEffect } from 'react';
import { onAuthStateChanged } from 'firebase/auth';
import { auth } from './firebase/config';
import Login from './components/Auth/Login';
import Header from './components/Layout/Header';
import TodoList from './components/TodoList';

export default function App() {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const unsubscribe = onAuthStateChanged(auth, (user) => {
      setUser(user);
      setLoading(false);
    });

    return () => unsubscribe();
  }, []);

  if (loading) {
    return <div className="min-h-screen flex items-center justify-center">Loading...</div>;
  }

  if (!user) {
    return <Login />;
  }

  return (
    <div className="min-h-screen bg-gray-100">
      <Header user={user} />
      <TodoList user={user} />
    </div>
  );
}
```

**Critical**: Always return unsubscribe function in useEffect to prevent memory leaks.

### Step 17: Build for Production

```bash
npm run build
```

Verify build succeeded:
```bash
ls dist/
ls dist/index.html
```

If build fails, check console errors and fix before proceeding.

### Step 18: Deploy to Firebase Hosting

```bash
npx firebase deploy --only hosting --project <project-id>
```

This command:
- Uploads all files from `dist/` directory
- Deploys to Firebase Hosting CDN
- Makes app live at `https://<project-id>.web.app`

Parse output for:
- "✔ hosting[<project-id>]: release complete"
- "Hosting URL: https://<project-id>.web.app"

Report to user: "✅ Deployed successfully to https://<project-id>.web.app"

### Step 19: Enable Authentication

Attempt to enable Identity Toolkit API automatically:

```bash
curl -X POST "https://serviceusage.googleapis.com/v1/projects/<project-id>/services/identitytoolkit.googleapis.com:enable" -H "Authorization: Bearer $(npx firebase login:ci --print-token 2>/dev/null || npx firebase login --print-token 2>/dev/null)"
```

If this fails (no OAuth token available), provide manual steps to user:

```
⚠️ Please enable Authentication manually (30 seconds):

1. Click here: https://console.firebase.google.com/project/<project-id>/authentication/providers

2. Click "Email/Password" provider
3. Enable toggle
4. Click "Save"

Let me know when done.
```

### Step 20: Rebuild After Authentication Setup

**Critical**: After enabling Authentication, rebuild to include API changes.

```bash
npm run build
npx firebase deploy --only hosting --project <project-id>
```

Parse output for deployment success and report new URL to user.

### Step 21: Verify Deployment

Test authentication with curl:
```bash
curl -X POST 'https://identitytoolkit.googleapis.com/v1/accounts:signUp?key=<api-key-from-env>' \
  -H 'Content-Type: application/json' \
  -d '{"email":"test@example.com","password":"test123","returnSecureToken":true}'
```

If response contains "idToken", authentication is working.

Report to user:
```
✅ Deployment complete!

Live URL: https://<project-id>.web.app

Test the app:
1. Open the URL above
2. Click "Sign Up"
3. Create an account
4. Add some todos

Your data is secure - each user sees only their own todos.
```

## Adding Firebase to Existing React Project

When user has existing React app and wants to add Firebase:

1. Run step 3 (install dependencies)
2. Run steps 4-11 (Firebase project setup and Firestore deployment)
3. Run steps 12-16 (implement Firebase in app)
4. Update build config if needed:
   - Vite: `"public": "dist"` in firebase.json
   - Create React App: `"public": "build"`
   - Next.js static: `"public": "out"`
5. Run steps 17-21 (deploy and verify)

## Common Issues

### Firebase CLI Not Found
If `npx firebase` fails:
```bash
npm install -g firebase-tools@latest
```

### Authentication Error: configuration-not-found
User forgot to enable Email/Password in Console. Provide direct link:
```
https://console.firebase.google.com/project/<project-id>/authentication/providers
```

### Environment Variables Undefined
Check `.env` file uses `VITE_` prefix (not just `FIREBASE_`).

Verify in code: `import.meta.env.VITE_FIREBASE_API_KEY` (NOT `process.env`).

### Firestore Permission Denied
User not logged in, or security rules not deployed. Re-run:
```bash
npx firebase deploy --only firestore:rules --project <project-id>
```

### Build Fails
Check Tailwind config has correct content paths:
```javascript
content: ['./index.html', './src/**/*.{js,jsx,ts,tsx}']
```

## Key Patterns

### Vite Environment Variables
```javascript
// Correct
const apiKey = import.meta.env.VITE_FIREBASE_API_KEY;

// Wrong (Node.js pattern, doesn't work in Vite)
const apiKey = process.env.VITE_FIREBASE_API_KEY;
```

### Firestore Timestamps
```javascript
import { serverTimestamp } from 'firebase/firestore';

// Correct - server time, consistent across timezones
createdAt: serverTimestamp()

// Wrong - client time, timezone issues
createdAt: new Date()
```

### Listener Cleanup
```javascript
useEffect(() => {
  const unsubscribe = onAuthStateChanged(auth, callback);
  return () => unsubscribe(); // Critical - prevents memory leaks
}, []);
```

### User-Specific Data
```javascript
// Always include userId when creating documents
await addDoc(collection(db, 'todos'), {
  text: 'Buy milk',
  userId: user.uid,  // Critical - enables security rules
  createdAt: serverTimestamp()
});
```

## Time Estimate

- New project: 10-15 minutes (automated)
- Add to existing: 5-10 minutes
- Manual setup: 2-3 hours

## Dependencies

- React 19
- Vite 7
- Firebase SDK 12
- Firebase CLI 14+
- Tailwind CSS 3
- Node.js 18+
