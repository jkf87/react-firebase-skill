# Firebase CLI Complete Reference

Detailed guide for Firebase project creation, configuration, and deployment using Firebase CLI.

## Overview

This guide covers Firebase CLI commands used in React + Firebase deployment workflow. All commands use `npx firebase` to avoid global installation requirements.

**Firebase CLI version**: 14.0.0+

---

## Project Creation

### Create New Firebase Project

```bash
npx firebase projects:create [project-id] --display-name "[Display Name]"
```

**Project ID requirements**:
- Globally unique across all Firebase projects
- Must start with lowercase letter
- Only lowercase letters, numbers, hyphens
- 6-30 characters

**Recommended pattern**: Use timestamp suffix for uniqueness
```bash
npx firebase projects:create my-app-$(date +%s) --display-name "My Application"
```

**Output example**:
```
✔ Creating Google Cloud Platform project
✔ Adding Firebase resources to Google Cloud Platform project

🎉 Firebase project my-app-1234567890 is ready!

Project ID: my-app-1234567890
Display Name: My Application
```

**Save the Project ID** - required for all subsequent commands.

### List Existing Projects

```bash
npx firebase projects:list
```

Shows all Firebase projects you have access to.

---

## Web App Registration

### Register Web Application

```bash
npx firebase apps:create WEB "[App Name]" --project [project-id]
```

**App Name**: User-facing name, can contain spaces and special characters.

**Output example**:
```
✔ Creating your Web app

🎉 Web app created!

App ID: 1:123456789012:web:abcdef1234567890
Display name: My Application
```

**Save the App ID** - needed to retrieve SDK configuration.

### Retrieve SDK Configuration

```bash
npx firebase apps:sdkconfig WEB [app-id] --project [project-id]
```

**Critical**: This command outputs your Firebase configuration. Copy all values immediately.

**Output example**:
```javascript
{
  apiKey: "AIzaSyAbc123Def456Ghi789Jkl012Mno345Pqr",
  authDomain: "my-app-1234567890.firebaseapp.com",
  projectId: "my-app-1234567890",
  storageBucket: "my-app-1234567890.firebasestorage.app",
  messagingSenderId: "123456789012",
  appId: "1:123456789012:web:abcdef1234567890"
}
```

### Create .env File from Output

Transform SDK output to Vite environment variables:

**Create `.env`**:
```env
VITE_FIREBASE_API_KEY=AIzaSyAbc123Def456Ghi789Jkl012Mno345Pqr
VITE_FIREBASE_AUTH_DOMAIN=my-app-1234567890.firebaseapp.com
VITE_FIREBASE_PROJECT_ID=my-app-1234567890
VITE_FIREBASE_STORAGE_BUCKET=my-app-1234567890.firebasestorage.app
VITE_FIREBASE_MESSAGING_SENDER_ID=123456789012
VITE_FIREBASE_APP_ID=1:123456789012:web:abcdef1234567890
```

**Create `.env.example`** (same structure, placeholder values):
```env
VITE_FIREBASE_API_KEY=your_api_key_here
VITE_FIREBASE_AUTH_DOMAIN=your_auth_domain_here
VITE_FIREBASE_PROJECT_ID=your_project_id_here
VITE_FIREBASE_STORAGE_BUCKET=your_storage_bucket_here
VITE_FIREBASE_MESSAGING_SENDER_ID=your_messaging_sender_id_here
VITE_FIREBASE_APP_ID=your_app_id_here
```

**Add to `.gitignore`**:
```
.env
```

---

## Firestore Deployment

### Deploy Rules and Indexes

```bash
npx firebase deploy --only firestore:rules,firestore:indexes --project [project-id]
```

**What this does**:
1. Enables Cloud Firestore API (if not already enabled)
2. Creates Firestore database (if not exists)
3. Deploys security rules from `firestore.rules`
4. Deploys indexes from `firestore.indexes.json`

**Output example**:
```
=== Deploying to 'my-app-1234567890'...

i  deploying firestore
i  firestore: reading indexes from firestore.indexes.json...
i  cloud.firestore: checking firestore.rules for compilation errors...
✔  cloud.firestore: rules file firestore.rules compiled successfully
i  firestore: uploading rules firestore.rules...
✔  firestore: deployed indexes in firestore.indexes.json successfully
✔  firestore: released rules firestore.rules to cloud.firestore

✔  Deploy complete!
```

**First-time deployment**: May take 2-3 minutes as API activation and database creation occur.

**Subsequent deployments**: ~10-30 seconds.

### Deploy Only Rules

```bash
npx firebase deploy --only firestore:rules --project [project-id]
```

Use when updating security rules without index changes.

### Deploy Only Indexes

```bash
npx firebase deploy --only firestore:indexes --project [project-id]
```

Use when adding composite indexes without rule changes.

---

## Firebase Hosting Deployment

### Initial Deploy

```bash
npm run build
npx firebase deploy --only hosting --project [project-id]
```

**Requirements**:
- `firebase.json` configured with correct public directory
- Build output exists (`dist/` for Vite, `build/` for CRA)

**Output example**:
```
=== Deploying to 'my-app-1234567890'...

i  deploying hosting
i  hosting[my-app-1234567890]: beginning deploy...
i  hosting[my-app-1234567890]: found 15 files in dist
✔  hosting[my-app-1234567890]: file upload complete
i  hosting[my-app-1234567890]: finalizing version...
✔  hosting[my-app-1234567890]: version finalized
i  hosting[my-app-1234567890]: releasing new version...
✔  hosting[my-app-1234567890]: release complete

✔  Deploy complete!

Project Console: https://console.firebase.google.com/project/my-app-1234567890/overview
Hosting URL: https://my-app-1234567890.web.app
```

**Save the Hosting URL** - this is your live application.

### Redeploy After Changes

```bash
npm run build
npx firebase deploy --only hosting --project [project-id]
```

Overwrites previous version. Previous versions accessible via Console.

---

## Combined Deployment

### Deploy Everything

```bash
npx firebase deploy --project [project-id]
```

Deploys all configured services:
- Firestore rules
- Firestore indexes
- Hosting
- Cloud Functions (if configured)
- Storage rules (if configured)

**Use cases**:
- Initial project deployment
- Major updates affecting multiple services
- Production releases

---

## CLI Limitations

### Authentication Cannot Be Automated

**What CANNOT be done via CLI**:
- Enable Email/Password authentication
- Enable Google Sign-in
- Configure OAuth redirect URIs
- Set up authentication templates

**Why**: OAuth2 provider configuration requires interactive consent flow.

**Workaround**: Provide Console instructions to user (see SKILL.md Phase 6).

### Database Location Cannot Be Changed

**Default**: Firestore database created in `us-central` region on first deployment.

**To change**: Must specify before first deployment via Console:
1. Visit https://console.firebase.google.com/project/[project-id]/firestore
2. Click "Create database"
3. Select region
4. Choose production mode

Then run CLI deployment.

### API Keys Cannot Be Restricted via CLI

**Security consideration**: API keys returned by `apps:sdkconfig` are unrestricted.

**Best practice**: After deployment, restrict in Console:
1. Visit https://console.firebase.google.com/project/[project-id]/settings/general
2. Scroll to "Your apps" → Web app
3. Click "API keys" link
4. Add application restrictions (HTTP referrers for web)

---

## Configuration Files

### .firebaserc

Stores project aliases for deployment.

**Structure**:
```json
{
  "projects": {
    "default": "my-app-1234567890",
    "production": "my-app-prod-1234567890",
    "staging": "my-app-staging-1234567890"
  }
}
```

**Deploy to specific alias**:
```bash
npx firebase deploy --only hosting -P staging
```

### firebase.json

Master configuration for all Firebase services.

**Complete example**:
```json
{
  "hosting": {
    "public": "dist",
    "ignore": [
      "firebase.json",
      "**/.*",
      "**/node_modules/**"
    ],
    "rewrites": [
      {
        "source": "**",
        "destination": "/index.html"
      }
    ],
    "headers": [
      {
        "source": "**/*.@(jpg|jpeg|gif|png|svg|webp)",
        "headers": [
          {
            "key": "Cache-Control",
            "value": "max-age=31536000"
          }
        ]
      }
    ]
  },
  "firestore": {
    "rules": "firestore.rules",
    "indexes": "firestore.indexes.json"
  }
}
```

**Key settings**:

- `hosting.public`: Build output directory
  - Vite: `"dist"`
  - Create React App: `"build"`
  - Next.js (static): `"out"`

- `hosting.rewrites`: SPA routing (redirect all to index.html)

- `hosting.headers`: Cache control for static assets

---

## Troubleshooting

### "Project ID does not exist"

**Error**: `HTTP Error: 404, Project [project-id] does not exist`

**Cause**: Typo in project ID or project not created

**Fix**:
```bash
npx firebase projects:list
```
Verify exact project ID.

### "Permission denied"

**Error**: `HTTP Error: 403, Permission denied`

**Cause**: Not authenticated or insufficient permissions

**Fix**:
```bash
npx firebase login
```

Then verify account has Owner or Editor role in Console.

### "Deployment quota exceeded"

**Error**: `Quota exceeded for quota metric 'Deployments'`

**Cause**: Too many deployments in short time (Spark plan: 10/day)

**Fix**: Wait 24 hours or upgrade to Blaze plan.

### "Build directory not found"

**Error**: `Error: Cannot find module 'dist'`

**Cause**: Haven't run build or wrong directory in `firebase.json`

**Fix**:
```bash
npm run build
```

Verify `dist/index.html` exists.

---

## Best Practices

### 1. Always Specify Project ID

```bash
# Good - explicit project
npx firebase deploy --only hosting --project my-app-1234567890

# Risky - depends on .firebaserc
npx firebase deploy --only hosting
```

### 2. Deploy Specific Targets

```bash
# Good - only affected service
npx firebase deploy --only hosting

# Wasteful - redeploys everything
npx firebase deploy
```

### 3. Version Control Configuration

**Commit**:
- `.firebaserc`
- `firebase.json`
- `firestore.rules`
- `firestore.indexes.json`
- `.env.example`

**Never commit**:
- `.env` (contains secrets)

### 4. Use Non-Interactive Mode in CI/CD

```bash
npx firebase deploy --only hosting --non-interactive --token $FIREBASE_TOKEN
```

Generate token:
```bash
npx firebase login:ci
```

---

## Command Reference

### Project Management
```bash
npx firebase projects:list                                    # List all projects
npx firebase projects:create [id] --display-name "[name]"     # Create project
npx firebase use [project-id]                                 # Set active project
```

### App Management
```bash
npx firebase apps:list --project [project-id]                 # List apps
npx firebase apps:create WEB "[name]" --project [project-id]  # Create web app
npx firebase apps:sdkconfig WEB [app-id] --project [project-id] # Get config
```

### Deployment
```bash
npx firebase deploy --project [project-id]                    # Deploy all
npx firebase deploy --only hosting --project [project-id]     # Deploy hosting
npx firebase deploy --only firestore --project [project-id]   # Deploy Firestore
npx firebase deploy --only firestore:rules --project [project-id] # Rules only
```

### Local Development
```bash
npx firebase serve --only hosting                             # Local hosting preview
npx firebase emulators:start                                  # Start all emulators
```

---

**Version**: 2.0.0
**Last Updated**: 2025-10-22
**Firebase CLI**: 14.0.0+
