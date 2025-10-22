# React + Firebase Component Implementation Patterns

Complete implementation reference for React components with Firebase integration.

## Table of Contents

1. [Firebase Configuration](#firebase-configuration)
2. [Authentication Components](#authentication-components)
3. [Layout Components](#layout-components)
4. [Custom Hooks](#custom-hooks)
5. [App Structure](#app-structure)
6. [Firestore Integration Patterns](#firestore-integration-patterns)

---

## Firebase Configuration

### src/firebase/config.js

Firebase initialization module. Exports `auth` and `db` instances used throughout application.

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
export default app;
```

**Critical points**:
- Use `import.meta.env.VITE_*` for Vite projects (NOT `process.env`)
- All environment variables must start with `VITE_` prefix
- Export named exports (`auth`, `db`) for convenience

---

## Authentication Components

### Login Component

**File**: `src/components/Auth/Login.jsx`

Implements Email/Password and Google Sign-in authentication.

```javascript
import { useState } from 'react';
import {
  signInWithEmailAndPassword,
  signInWithPopup,
  GoogleAuthProvider
} from 'firebase/auth';
import { auth } from '../../firebase/config';

export default function Login({ onToggleForm }) {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [error, setError] = useState('');
  const [loading, setLoading] = useState(false);

  const handleEmailLogin = async (e) => {
    e.preventDefault();
    setError('');
    setLoading(true);

    try {
      await signInWithEmailAndPassword(auth, email, password);
      // Navigation handled by App.jsx auth state change
    } catch (err) {
      setError(getErrorMessage(err.code));
    } finally {
      setLoading(false);
    }
  };

  const handleGoogleLogin = async () => {
    setError('');
    setLoading(true);

    try {
      const provider = new GoogleAuthProvider();
      await signInWithPopup(auth, provider);
    } catch (err) {
      // User cancelled popup
      if (err.code === 'auth/popup-closed-by-user') {
        setError('');
      } else {
        setError(getErrorMessage(err.code));
      }
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="min-h-screen flex items-center justify-center bg-gradient-to-br from-blue-50 to-indigo-100 px-4">
      <div className="max-w-md w-full bg-white rounded-lg shadow-xl p-8">
        <h2 className="text-3xl font-bold text-center text-gray-800 mb-8">
          Login
        </h2>

        {error && (
          <div className="mb-4 p-3 bg-red-100 border border-red-400 text-red-700 rounded">
            {error}
          </div>
        )}

        <form onSubmit={handleEmailLogin} className="space-y-6">
          <div>
            <label className="block text-sm font-medium text-gray-700 mb-2">
              Email
            </label>
            <input
              type="email"
              value={email}
              onChange={(e) => setEmail(e.target.value)}
              required
              className="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent"
              disabled={loading}
            />
          </div>

          <div>
            <label className="block text-sm font-medium text-gray-700 mb-2">
              Password
            </label>
            <input
              type="password"
              value={password}
              onChange={(e) => setPassword(e.target.value)}
              required
              className="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent"
              disabled={loading}
            />
          </div>

          <button
            type="submit"
            disabled={loading}
            className="w-full bg-blue-600 text-white py-2 px-4 rounded-lg hover:bg-blue-700 disabled:opacity-50 disabled:cursor-not-allowed transition"
          >
            {loading ? 'Logging in...' : 'Login'}
          </button>
        </form>

        <div className="mt-6">
          <div className="relative">
            <div className="absolute inset-0 flex items-center">
              <div className="w-full border-t border-gray-300"></div>
            </div>
            <div className="relative flex justify-center text-sm">
              <span className="px-2 bg-white text-gray-500">Or</span>
            </div>
          </div>

          <button
            onClick={handleGoogleLogin}
            disabled={loading}
            className="mt-6 w-full bg-white border border-gray-300 text-gray-700 py-2 px-4 rounded-lg hover:bg-gray-50 disabled:opacity-50 disabled:cursor-not-allowed transition"
          >
            Sign in with Google
          </button>
        </div>

        <p className="mt-8 text-center text-sm text-gray-600">
          Don't have an account?{' '}
          <button
            onClick={onToggleForm}
            className="text-blue-600 hover:text-blue-700 font-medium"
          >
            Sign up
          </button>
        </p>
      </div>
    </div>
  );
}

function getErrorMessage(code) {
  switch (code) {
    case 'auth/user-not-found':
    case 'auth/wrong-password':
      return 'Invalid email or password';
    case 'auth/too-many-requests':
      return 'Too many failed attempts. Please try again later.';
    case 'auth/network-request-failed':
      return 'Network error. Please check your connection.';
    case 'auth/popup-blocked':
      return 'Popup blocked. Please allow popups for this site.';
    default:
      return 'An error occurred. Please try again.';
  }
}
```

**Key patterns**:
- Use `loading` state to disable form during submission
- Handle both email/password and Google authentication
- Provide user-friendly error messages
- `onToggleForm` prop for switching between login/signup
- Form submission handled via `onSubmit` (not button `onClick`)

### SignUp Component

**File**: `src/components/Auth/SignUp.jsx`

User registration with email/password.

```javascript
import { useState } from 'react';
import { createUserWithEmailAndPassword } from 'firebase/auth';
import { auth } from '../../firebase/config';

export default function SignUp({ onToggleForm }) {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [confirmPassword, setConfirmPassword] = useState('');
  const [error, setError] = useState('');
  const [loading, setLoading] = useState(false);

  const handleSignUp = async (e) => {
    e.preventDefault();
    setError('');

    // Client-side validation
    if (password !== confirmPassword) {
      setError('Passwords do not match');
      return;
    }

    if (password.length < 6) {
      setError('Password must be at least 6 characters');
      return;
    }

    setLoading(true);

    try {
      await createUserWithEmailAndPassword(auth, email, password);
      // Navigation handled by App.jsx auth state change
    } catch (err) {
      setError(getErrorMessage(err.code));
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="min-h-screen flex items-center justify-center bg-gradient-to-br from-blue-50 to-indigo-100 px-4">
      <div className="max-w-md w-full bg-white rounded-lg shadow-xl p-8">
        <h2 className="text-3xl font-bold text-center text-gray-800 mb-8">
          Sign Up
        </h2>

        {error && (
          <div className="mb-4 p-3 bg-red-100 border border-red-400 text-red-700 rounded">
            {error}
          </div>
        )}

        <form onSubmit={handleSignUp} className="space-y-6">
          <div>
            <label className="block text-sm font-medium text-gray-700 mb-2">
              Email
            </label>
            <input
              type="email"
              value={email}
              onChange={(e) => setEmail(e.target.value)}
              required
              className="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent"
              disabled={loading}
            />
          </div>

          <div>
            <label className="block text-sm font-medium text-gray-700 mb-2">
              Password
            </label>
            <input
              type="password"
              value={password}
              onChange={(e) => setPassword(e.target.value)}
              required
              minLength={6}
              className="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent"
              disabled={loading}
            />
          </div>

          <div>
            <label className="block text-sm font-medium text-gray-700 mb-2">
              Confirm Password
            </label>
            <input
              type="password"
              value={confirmPassword}
              onChange={(e) => setConfirmPassword(e.target.value)}
              required
              minLength={6}
              className="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent"
              disabled={loading}
            />
          </div>

          <button
            type="submit"
            disabled={loading}
            className="w-full bg-blue-600 text-white py-2 px-4 rounded-lg hover:bg-blue-700 disabled:opacity-50 disabled:cursor-not-allowed transition"
          >
            {loading ? 'Creating account...' : 'Sign Up'}
          </button>
        </form>

        <p className="mt-8 text-center text-sm text-gray-600">
          Already have an account?{' '}
          <button
            onClick={onToggleForm}
            className="text-blue-600 hover:text-blue-700 font-medium"
          >
            Login
          </button>
        </p>
      </div>
    </div>
  );
}

function getErrorMessage(code) {
  switch (code) {
    case 'auth/email-already-in-use':
      return 'Email already registered';
    case 'auth/invalid-email':
      return 'Invalid email address';
    case 'auth/weak-password':
      return 'Password is too weak';
    case 'auth/network-request-failed':
      return 'Network error. Please check your connection.';
    default:
      return 'An error occurred. Please try again.';
  }
}
```

**Key patterns**:
- Client-side password validation before submission
- Confirm password matching
- Minimum password length enforcement (6 characters - Firebase requirement)

---

## Layout Components

### Header Component

**File**: `src/components/Layout/Header.jsx`

Navigation bar with user info and logout.

```javascript
import { signOut } from 'firebase/auth';
import { auth } from '../../firebase/config';

export default function Header({ user }) {
  const handleLogout = async () => {
    try {
      await signOut(auth);
    } catch (err) {
      console.error('Logout error:', err);
    }
  };

  return (
    <header className="bg-white shadow-md">
      <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-4">
        <div className="flex justify-between items-center">
          <h1 className="text-2xl font-bold text-gray-800">
            My Application
          </h1>

          <div className="flex items-center space-x-4">
            <span className="text-sm text-gray-600">
              {user.email}
            </span>
            <button
              onClick={handleLogout}
              className="bg-red-600 text-white px-4 py-2 rounded-lg hover:bg-red-700 transition"
            >
              Logout
            </button>
          </div>
        </div>
      </div>
    </header>
  );
}
```

**Key patterns**:
- `signOut(auth)` for logout
- Display user email from auth state
- Error handling in logout (rare but possible)

---

## Custom Hooks

### useFirestore Hook

**File**: `src/hooks/useFirestore.js`

Reusable hook for Firestore CRUD operations with real-time synchronization.

```javascript
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
  serverTimestamp
} from 'firebase/firestore';
import { db } from '../firebase/config';

export default function useFirestore(collectionName, userId) {
  const [items, setItems] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

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

    // Real-time listener
    const unsubscribe = onSnapshot(
      q,
      (snapshot) => {
        const data = snapshot.docs.map((doc) => ({
          id: doc.id,
          ...doc.data()
        }));
        setItems(data);
        setLoading(false);
        setError(null);
      },
      (err) => {
        console.error('Firestore error:', err);
        setError(err.message);
        setLoading(false);
      }
    );

    // Cleanup function prevents memory leaks
    return () => unsubscribe();
  }, [collectionName, userId]);

  const addItem = async (data) => {
    try {
      await addDoc(collection(db, collectionName), {
        ...data,
        userId,
        createdAt: serverTimestamp(),
        updatedAt: serverTimestamp()
      });
    } catch (err) {
      console.error('Add error:', err);
      throw err;
    }
  };

  const updateItem = async (id, updates) => {
    try {
      await updateDoc(doc(db, collectionName, id), {
        ...updates,
        updatedAt: serverTimestamp()
      });
    } catch (err) {
      console.error('Update error:', err);
      throw err;
    }
  };

  const deleteItem = async (id) => {
    try {
      await deleteDoc(doc(db, collectionName, id));
    } catch (err) {
      console.error('Delete error:', err);
      throw err;
    }
  };

  return { items, loading, error, addItem, updateItem, deleteItem };
}
```

**Key patterns**:
- Real-time updates via `onSnapshot` (not manual queries)
- User-specific data filtering with `where('userId', '==', userId)`
- Always use `serverTimestamp()` (not `new Date()`)
- Return `unsubscribe()` in cleanup to prevent memory leaks
- Automatic `userId` injection on create
- Automatic `updatedAt` timestamp on update

**Usage example**:
```javascript
const { items, loading, addItem, updateItem, deleteItem } = useFirestore('todos', user.uid);

// Add new item
await addItem({ text: 'New todo', completed: false });

// Update item
await updateItem(itemId, { completed: true });

// Delete item
await deleteItem(itemId);
```

### Feature-Specific Hook Example

**File**: `src/hooks/useTodos.js`

Specialized hook for todo items.

```javascript
import { useState } from 'react';
import useFirestore from './useFirestore';

export default function useTodos(userId) {
  const { items: todos, loading, addItem, updateItem, deleteItem } = useFirestore('todos', userId);
  const [adding, setAdding] = useState(false);

  const addTodo = async (text) => {
    if (!text.trim()) return;

    setAdding(true);
    try {
      await addItem({
        text: text.trim(),
        completed: false
      });
    } finally {
      setAdding(false);
    }
  };

  const toggleTodo = async (id, currentCompleted) => {
    await updateItem(id, { completed: !currentCompleted });
  };

  const deleteTodo = async (id) => {
    await deleteItem(id);
  };

  return {
    todos,
    loading,
    adding,
    addTodo,
    toggleTodo,
    deleteTodo
  };
}
```

**Pattern**: Wrap generic `useFirestore` with domain-specific logic.

---

## App Structure

### App.jsx

**File**: `src/App.jsx`

Root component managing authentication state and routing.

```javascript
import { useState, useEffect } from 'react';
import { onAuthStateChanged } from 'firebase/auth';
import { auth } from './firebase/config';
import Login from './components/Auth/Login';
import SignUp from './components/Auth/SignUp';
import Header from './components/Layout/Header';
import MainContent from './components/MainContent';

function App() {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [showSignUp, setShowSignUp] = useState(false);

  useEffect(() => {
    // Listen for auth state changes
    const unsubscribe = onAuthStateChanged(auth, (currentUser) => {
      setUser(currentUser);
      setLoading(false);
    });

    // Cleanup listener on unmount
    return () => unsubscribe();
  }, []);

  if (loading) {
    return (
      <div className="min-h-screen flex items-center justify-center">
        <div className="text-xl text-gray-600">Loading...</div>
      </div>
    );
  }

  if (!user) {
    return showSignUp ? (
      <SignUp onToggleForm={() => setShowSignUp(false)} />
    ) : (
      <Login onToggleForm={() => setShowSignUp(true)} />
    );
  }

  return (
    <div className="min-h-screen bg-gray-50">
      <Header user={user} />
      <MainContent user={user} />
    </div>
  );
}

export default App;
```

**Key patterns**:
- Single `onAuthStateChanged` listener at app root
- Loading state prevents flash of login screen
- Conditional rendering based on auth state
- `unsubscribe()` cleanup prevents memory leaks

---

## Firestore Integration Patterns

### Basic CRUD Component

**File**: `src/components/TodoList.jsx`

Example feature component using Firestore.

```javascript
import { useState } from 'react';
import useTodos from '../hooks/useTodos';

export default function TodoList({ user }) {
  const [newTodoText, setNewTodoText] = useState('');
  const { todos, loading, adding, addTodo, toggleTodo, deleteTodo } = useTodos(user.uid);

  const handleSubmit = async (e) => {
    e.preventDefault();
    await addTodo(newTodoText);
    setNewTodoText('');
  };

  if (loading) {
    return <div className="text-center py-8">Loading todos...</div>;
  }

  return (
    <div className="max-w-2xl mx-auto p-6">
      <form onSubmit={handleSubmit} className="mb-6">
        <div className="flex space-x-2">
          <input
            type="text"
            value={newTodoText}
            onChange={(e) => setNewTodoText(e.target.value)}
            placeholder="Add new todo..."
            className="flex-1 px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500"
            disabled={adding}
          />
          <button
            type="submit"
            disabled={adding || !newTodoText.trim()}
            className="bg-blue-600 text-white px-6 py-2 rounded-lg hover:bg-blue-700 disabled:opacity-50 disabled:cursor-not-allowed"
          >
            {adding ? 'Adding...' : 'Add'}
          </button>
        </div>
      </form>

      <div className="space-y-2">
        {todos.map((todo) => (
          <div
            key={todo.id}
            className="flex items-center space-x-3 p-4 bg-white rounded-lg shadow"
          >
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
              className="text-red-600 hover:text-red-700"
            >
              Delete
            </button>
          </div>
        ))}

        {todos.length === 0 && (
          <div className="text-center text-gray-500 py-8">
            No todos yet. Add one above!
          </div>
        )}
      </div>
    </div>
  );
}
```

**Key patterns**:
- Separate loading state for initial load vs. adding
- Disable form during submission
- Optimistic UI (React state updates before server confirms)
- Empty state messaging

---

## Additional Patterns

### Error Boundary

**File**: `src/components/ErrorBoundary.jsx`

Catches React errors and displays fallback UI.

```javascript
import { Component } from 'react';

export default class ErrorBoundary extends Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  componentDidCatch(error, errorInfo) {
    console.error('Error caught:', error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return (
        <div className="min-h-screen flex items-center justify-center bg-gray-100">
          <div className="max-w-md p-6 bg-white rounded-lg shadow-xl">
            <h2 className="text-2xl font-bold text-red-600 mb-4">
              Something went wrong
            </h2>
            <p className="text-gray-600 mb-4">
              {this.state.error?.message || 'An unexpected error occurred'}
            </p>
            <button
              onClick={() => window.location.reload()}
              className="bg-blue-600 text-white px-4 py-2 rounded-lg hover:bg-blue-700"
            >
              Reload page
            </button>
          </div>
        </div>
      );
    }

    return this.props.children;
  }
}
```

**Usage in main.jsx**:
```javascript
import ErrorBoundary from './components/ErrorBoundary';

ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <ErrorBoundary>
      <App />
    </ErrorBoundary>
  </React.StrictMode>
);
```

---

**Version**: 2.0.0
**Last Updated**: 2025-10-22
