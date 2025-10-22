# React Firebase Deployment Skill

Automates creation and deployment of production-ready React applications with Firebase backend (Firestore, Authentication, Hosting).

## Overview

This skill provides complete automation for:
- Creating new React + Vite projects
- Setting up Firebase backend infrastructure
- Configuring Firestore database with security rules
- Deploying to Firebase Hosting
- Implementing authentication (Email/Password, Google Sign-in)

**Time savings**: Reduces 2-3 hours of manual setup to 10-15 minutes.

## Features

### Three Primary Workflows

1. **New Project Creation** - Complete automation from empty directory to live deployment
2. **Adding Firebase to Existing Project** - Integrate Firebase into existing React apps
3. **TypeScript Conversion** - Migrate JavaScript implementation to TypeScript

### What You Get

- ✅ React 19 + Vite 7 + Tailwind CSS 3
- ✅ Firebase project with Firestore database
- ✅ User-specific security rules configured
- ✅ Live deployment on Firebase Hosting
- ✅ Authentication setup guide (Email/Password + Google)
- ✅ Real-time data synchronization
- ✅ Complete component implementations
- ✅ Custom hooks for Firestore operations

## Installation

### For Plugin Marketplace

This skill is designed to be distributed via Claude Code plugin marketplace.

**Directory structure**:
```
react-firebase-skill/
├── SKILL.md                     # Main skill workflow
├── firebase-cli.md              # Firebase CLI reference
├── component-patterns.md        # Implementation patterns
├── README.md                    # This file
├── LICENSE.txt                  # License
└── guides/
    ├── typescript-setup.md
    ├── advanced-security.md
    └── nextjs-variant.md
```

### Manual Installation

1. Copy entire directory to `.claude/skills/react-firebase/`
2. Skill will be automatically detected by Claude Code
3. Invoke by requesting React + Firebase project creation

## Quick Start

### Using the Skill

Trigger this skill by requesting:
- "Create a React app with Firebase backend"
- "Build a todo app using React and Firestore"
- "Set up a React + Firebase project with authentication"

The skill will automatically:
1. Create Vite React project
2. Configure Tailwind CSS
3. Create Firebase project via CLI
4. Deploy Firestore database with security rules
5. Generate authentication components
6. Deploy to Firebase Hosting
7. Provide manual authentication setup instructions

## Documentation

### Core Files

- **SKILL.md** - Main entry point with 3 complete workflows
  - Workflow 1: New project creation (22 steps)
  - Workflow 2: Adding Firebase to existing project
  - Workflow 3: TypeScript conversion

- **firebase-cli.md** - Firebase CLI complete reference
  - Project creation commands
  - Web app registration
  - Firestore deployment
  - Hosting deployment
  - Troubleshooting guide

- **component-patterns.md** - React + Firebase implementation patterns
  - Firebase configuration
  - Authentication components (Login, SignUp)
  - Layout components (Header)
  - Custom hooks (useFirestore, useTodos)
  - App structure with auth state
  - Firestore integration patterns

### Advanced Guides

- **guides/typescript-setup.md**
  - TypeScript configuration for Vite + React
  - Firebase type definitions
  - Environment variable types
  - Component prop types

- **guides/advanced-security.md**
  - Email verification implementation
  - Custom claims (role-based access)
  - Field-level security rules
  - Rate limiting patterns

- **guides/nextjs-variant.md**
  - Next.js App Router setup
  - Server vs Client components
  - Environment variable differences
  - Static export configuration
  - When to choose Next.js vs Vite

## Key Patterns

### Vite Environment Variables
```javascript
// Correct for Vite
const apiKey = import.meta.env.VITE_FIREBASE_API_KEY;

// Wrong (this is Node.js, not Vite)
const apiKey = process.env.VITE_FIREBASE_API_KEY;
```

### Firestore Timestamps
```javascript
import { serverTimestamp } from 'firebase/firestore';

// Correct - server time, timezone-safe
createdAt: serverTimestamp()

// Wrong - client time, timezone issues
createdAt: new Date()
```

### Real-time Listeners
```javascript
// Always cleanup to prevent memory leaks
useEffect(() => {
  const unsubscribe = onSnapshot(query, callback);
  return () => unsubscribe(); // Critical cleanup
}, []);
```

## Limitations

### Firebase CLI Cannot Automate Authentication

Authentication providers (Email/Password, Google Sign-in) cannot be enabled via Firebase CLI due to OAuth2 requirements.

**Workaround**: Skill provides detailed Console setup instructions with step-by-step guide.

**Time required**: 2-3 minutes manual setup.

## Dependencies

Tested versions:
- React 19
- Vite 7
- Firebase SDK 12
- Firebase CLI 14
- Tailwind CSS 3
- Node.js 18+

## Time Estimates

| Workflow | Duration |
|----------|----------|
| New project (Workflow 1) | 10-15 min |
| Add to existing (Workflow 2) | 5-10 min |
| TypeScript conversion (Workflow 3) | 15-30 min |

**Compare to manual setup**: 2-3 hours

## Common Issues

### Tailwind CSS v4 Incompatibility
**Fix**: Skill automatically uses Tailwind v3 (v4 has CLI compatibility issues)

### Firestore API Not Enabled
**Fix**: Auto-resolved by `firebase deploy --only firestore`

### Authentication Not Configured
**Fix**: Follow Phase 6 manual setup instructions provided by skill

### Google Sign-in Popup Blocked
**Fix**: Skill instructs user to allow popups for localhost and Firebase domain

## Version History

- **2.0.0** (2025-10-22) - Complete restructure following official skill format
  - Separated workflows into SKILL.md
  - Created firebase-cli.md reference
  - Created component-patterns.md implementation guide
  - Added advanced guides (TypeScript, security, Next.js)
  - Removed redundant content (progressive disclosure)

- **1.0.0** (2025-10-21) - Initial release

## License

MIT License

Copyright (c) 2025

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

## Contributing

This skill follows [Anthropic's Skill Best Practices](https://docs.claude.com/en/docs/agents-and-tools/agent-skills/best-practices):
- Conciseness (removes Claude's existing knowledge)
- Progressive Disclosure (main file references detailed guides)
- Clear activation triggers in description
- Validation checklists for each workflow

## Support

For issues or questions:
1. Check `SKILL.md` for complete workflows
2. Check `firebase-cli.md` for CLI command reference
3. Check `component-patterns.md` for implementation details
4. Check relevant guides in `guides/` directory

---

**Made with** ❤️ **for rapid React + Firebase development**
