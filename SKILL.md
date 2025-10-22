---
name: react-firebase
description: "React + Firebase fullstack application creation, deployment, and configuration"
---

# React Firebase Deployment

Automates creation and deployment of production-ready React applications with Firebase backend (Firestore, Authentication, Hosting).

## Three Primary Workflows

### 1. Creating New Project from Scratch
Complete automation from empty directory to live deployment with authentication and database.

### 2. Adding Firebase to Existing React Project
Integrate Firebase backend into an existing React/Vite application.

### 3. Converting to TypeScript
Migrate JavaScript implementation to TypeScript with proper Firebase type definitions.

---

## Workflow 1: New Project Creation

**Mandatory first step**: Read `firebase-cli.md` completely to understand Firebase project creation and configuration patterns.

### Phase 1: Project Initialization

1. **Create React project with Vite**
   ```bash
   npm create vite@latest [project-name] -- --template react
   cd [project-name]
   npm install
   ```

2. **Setup Tailwind CSS v3**
   ```bash
   npm install -D tailwindcss@^3 postcss autoprefixer
   npx tailwindcss init -p
   ```

   **Critical**: Must use Tailwind v3. Version 4 has CLI compatibility issues.

   Update `tailwind.config.js`:
   ```javascript
   content: ['./index.html', './src/**/*.{js,jsx,ts,tsx}']
   ```

   Update `src/index.css`:
   ```css
   @tailwind base;
   @tailwind components;
   @tailwind utilities;
   ```

3. **Install Firebase dependencies**
   ```bash
   npm install firebase firebase-tools
   ```

### Phase 2: Firebase Project Setup

See `firebase-cli.md` for complete Firebase CLI workflow.

4. **Create Firebase project**
   ```bash
   npx firebase projects:create [name]-$(date +%s) --display-name "[Display Name]"
   ```

   Save the project ID from output.

5. **Register web application**
   ```bash
   npx firebase apps:create WEB "[App Name]" --project [project-id]
   ```

   Save the app ID from output.

6. **Retrieve SDK configuration**
   ```bash
   npx firebase apps:sdkconfig WEB [app-id] --project [project-id]
   ```

   Copy all configuration values from output.

7. **Create environment file**

   Create `.env`:
   ```env
   VITE_FIREBASE_API_KEY=...
   VITE_FIREBASE_AUTH_DOMAIN=...
   VITE_FIREBASE_PROJECT_ID=...
   VITE_FIREBASE_STORAGE_BUCKET=...
   VITE_FIREBASE_MESSAGING_SENDER_ID=...
   VITE_FIREBASE_APP_ID=...
   ```

   Create `.env.example` (same structure, placeholder values).

8. **Create Firebase configuration files**

   Create `.firebaserc`:
   ```json
   {
     "projects": {
       "default": "[project-id]"
     }
   }
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

### Phase 3: Firestore Database Configuration

9. **Create Firestore security rules**

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

   This enforces user-specific data isolation across all collections.

10. **Create indexes file**

    Create `firestore.indexes.json`:
    ```json
    {
      "indexes": [],
      "fieldOverrides": []
    }
    ```

11. **Deploy Firestore configuration**
    ```bash
    npx firebase deploy --only firestore:rules,firestore:indexes --project [project-id]
    ```

    **Important**: This command automatically:
    - Enables Firestore API
    - Creates Firestore database
    - Applies security rules

    No manual Console steps required.

### Phase 4: Application Implementation

**Mandatory first step**: Read `component-patterns.md` completely for implementation details.

12. **Create directory structure**
    ```bash
    mkdir -p src/components/Auth
    mkdir -p src/components/Layout
    mkdir -p src/firebase
    mkdir -p src/hooks
    ```

13. **Implement Firebase configuration**

    Create `src/firebase/config.js` - see `component-patterns.md` for complete implementation.

14. **Implement authentication components**

    Required components:
    - `src/components/Auth/Login.jsx` - Email/Password + Google sign-in
    - `src/components/Auth/SignUp.jsx` - User registration
    - `src/components/Layout/Header.jsx` - Navigation + logout

    See `component-patterns.md` for complete implementations.

15. **Implement feature components with Firestore**

    Create custom hooks (e.g., `src/hooks/useTodos.js`) for Firestore operations.
    See `component-patterns.md` for patterns.

16. **Update App.jsx with authentication state**

    Implement `onAuthStateChanged` listener - see `component-patterns.md`.

### Phase 5: Deployment

17. **Build for production**
    ```bash
    npm run build
    ```

    Verify `dist/` directory created successfully.

18. **Deploy to Firebase Hosting**
    ```bash
    npx firebase deploy --only hosting --project [project-id]
    ```

    Save the deployment URL from output: `https://[project-id].web.app`

### Phase 6: Authentication Setup

⚠️ **CLI Limitation**: Authentication providers cannot be enabled via CLI (OAuth2 requirement).

19. **Manual Console steps required**

    Provide these instructions to user:

    **Email/Password Authentication**:
    1. Visit https://console.firebase.google.com/project/[project-id]/authentication
    2. Click "Get started" button
    3. Click "Sign-in method" tab
    4. Click "Email/Password" provider
    5. Enable "Email/Password" toggle
    6. Click "Save"

    **Google Sign-in** (optional):
    1. Same page, click "Google" provider
    2. Enable toggle
    3. Select project support email from dropdown
    4. Click "Save"

20. **Verify authentication**

    Test locally:
    ```bash
    npm run dev
    ```

    Test signup flow, verify user appears in Authentication console.

### Phase 7: Validation

21. **Run validation checklist**

    - [ ] Local dev server runs: `npm run dev` (no errors)
    - [ ] Live URL accessible: `https://[project-id].web.app`
    - [ ] Firestore database exists in Console
    - [ ] Security rules applied (test unauthorized access)
    - [ ] Authentication works (signup, login, logout)
    - [ ] User data isolated (create items as different users)
    - [ ] No console errors in browser DevTools

22. **Generate user documentation**

    Create these files:
    - `QUICK_START.md` - Running locally, adding features
    - `DEPLOYMENT_GUIDE.md` - Firebase configuration, deployment process
    - `AUTH_SETUP.md` - Authentication activation steps with screenshots

---

## Workflow 2: Adding Firebase to Existing Project

Use when integrating Firebase into an existing React application.

1. **Install dependencies**
   ```bash
   npm install firebase firebase-tools
   ```

2. **Follow Phase 2-3** from Workflow 1 (Firebase project setup, Firestore configuration)

3. **Follow Phase 4** from Workflow 1 (application implementation)

4. **Update build configuration if needed**

   Verify `firebase.json` points to correct build output directory:
   - Vite: `"public": "dist"`
   - Create React App: `"public": "build"`
   - Next.js (static): `"public": "out"`

5. **Deploy**: Follow Phase 5-7 from Workflow 1

---

## Workflow 3: TypeScript Conversion

**Mandatory first step**: Read `typescript-setup.md` completely.

Use when converting existing JavaScript implementation to TypeScript.

1. **Install TypeScript dependencies**
   ```bash
   npm install -D typescript @types/react @types/react-dom
   ```

2. **Generate tsconfig.json**
   ```bash
   npx tsc --init
   ```

3. **Follow typescript-setup.md** for:
   - TypeScript configuration
   - Firebase type definitions
   - Environment variable types
   - Component prop types
   - Custom hook types

4. **Rename files**
   ```bash
   find src -name "*.jsx" -exec sh -c 'mv "$1" "${1%.jsx}.tsx"' _ {} \;
   find src -name "*.js" -exec sh -c 'mv "$1" "${1%.js}.ts"' _ {} \;
   ```

5. **Fix type errors incrementally**
   ```bash
   npx tsc --noEmit
   ```

---

## Common Issues

### Firestore API Not Enabled
**Error**: `HTTP Error: 403, Cloud Firestore API has not been used`

**Cause**: Firestore API not activated

**Fix**: Automatically resolved by `firebase deploy --only firestore`

### Authentication Not Configured
**Error**: `auth/configuration-not-found`

**Cause**: Authentication providers not enabled in Console

**Fix**: Follow Phase 6 manual setup steps

### Tailwind CSS Not Working
**Error**: Styles not applied

**Cause**: Tailwind v4 installed (CLI incompatibility)

**Fix**:
```bash
npm uninstall tailwindcss
npm install -D tailwindcss@^3
npx tailwindcss init -p
```

### Google Sign-in Popup Blocked
**Error**: Popup window doesn't open

**Cause**: Browser blocking popup

**Fix**: Instruct user to allow popups for localhost and Firebase domain

### Environment Variables Not Loading
**Error**: `undefined` values in config

**Cause**: Missing `VITE_` prefix or wrong variable name

**Fix**: All Vite environment variables must start with `VITE_`

---

## Key Non-Obvious Patterns

### Vite Environment Variables
```javascript
// Correct
const apiKey = import.meta.env.VITE_FIREBASE_API_KEY;

// Wrong (this is for Node.js, not Vite)
const apiKey = process.env.VITE_FIREBASE_API_KEY;
```

### Firestore Timestamps
```javascript
import { serverTimestamp } from 'firebase/firestore';

// Correct - server time, consistent across timezones
await addDoc(collection(db, 'items'), {
  createdAt: serverTimestamp()
});

// Wrong - client time, timezone issues
await addDoc(collection(db, 'items'), {
  createdAt: new Date()
});
```

### Listener Cleanup
```javascript
// Always return unsubscribe function to prevent memory leaks
useEffect(() => {
  const unsubscribe = onAuthStateChanged(auth, (user) => {
    setUser(user);
  });

  return () => unsubscribe(); // Critical: cleanup on unmount
}, []);
```

### Query Ordering Requirements
```javascript
// If using orderBy, must create composite index
const q = query(
  collection(db, 'todos'),
  where('userId', '==', userId),
  orderBy('createdAt', 'desc') // Requires index
);

// Firebase will provide index creation link in console error
```

---

## Advanced Features

See separate documentation for advanced patterns:

- **Advanced security**: `advanced-security.md`
  - Email verification
  - Custom claims (role-based access)
  - Field-level security rules
  - Rate limiting

- **Next.js variant**: Use when SSR/SSG needed
  - Different environment variable pattern (`NEXT_PUBLIC_`)
  - Client vs Server component considerations
  - Static export configuration

---

## Dependencies

Tested versions:
- React 19
- Vite 7
- Firebase SDK 12
- Firebase CLI 14
- Tailwind CSS 3
- Node.js 18+

## Time Estimate

- New project (Workflow 1): 10-15 minutes
- Adding to existing (Workflow 2): 5-10 minutes
- TypeScript conversion (Workflow 3): 15-30 minutes

Compare to manual setup: 2-3 hours

---

**Version**: 2.0.0
**Last Updated**: 2025-10-22
