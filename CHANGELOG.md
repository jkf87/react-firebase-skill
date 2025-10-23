# Changelog

All notable changes to the React Firebase Deployment Skill.

## [3.1.0] - 2025-10-23

### 🔥 Critical Fix - Skill Not Triggering on First Prompt

**Problem**: User requested "todo list앱을 만들어주세요 . firebase skill을 사용하고, 백엔드 ,db, 배포를 전부처리해주세요" but Claude didn't use the skill. Claude created files manually with placeholder values instead of running Firebase CLI commands.

**Root Cause**: SKILL.md lacked explicit trigger conditions. Claude didn't recognize when to activate the skill.

#### Added

- **Explicit trigger conditions** at top of SKILL.md:
  ```markdown
  **CRITICAL**: When a user asks you to:
  - "Create a React app with Firebase backend"
  - "Make a todo app with Firebase"
  - "Build a [X] app and deploy it with Firebase"
  - "Use Firebase skill to build..."
  - Mentions "Firebase" + "backend" + "deploy"

  **YOU MUST immediately follow this skill workflow**
  ```

- **DO/DO NOT checklist**:
  - ❌ DO NOT: Create package.json manually
  - ❌ DO NOT: Write placeholder values like "YOUR_API_KEY"
  - ❌ DO NOT: Skip Firebase CLI commands
  - ✅ DO: Run `npx firebase projects:create` to create real project
  - ✅ DO: Run `npx firebase apps:sdkconfig` to get real API keys
  - ✅ DO: Run `npx firebase deploy` to deploy

- **New/Existing project logic** (Step 1):
  - Default behavior: Create NEW Firebase project
  - Only use existing if `.firebaserc` exists or user explicitly mentions it
  - Clear instructions for both paths

#### Changed

- **Step 5** now explicitly says "ONLY IF creating NEW project"
- Added example commands with actual values
- Added fallback instructions if `firebase login` needed

#### Impact

- ✅ Claude now recognizes trigger keywords in user prompts
- ✅ Claude sees clear DO/DO NOT instructions upfront
- ✅ Claude knows to create NEW project by default (not search for existing)
- ✅ Skill activates on first prompt (not second)

#### Test Case

**Before (v3.0.0)**:
```
User: "todo list앱을 만들어주세요 . firebase skill을 사용하고"
Claude: *Creates package.json, firebase.json manually*
Claude: *Writes "YOUR_API_KEY" placeholders*
Result: No Firebase backend created
```

**After (v3.1.0)**:
```
User: "todo list앱을 만들어주세요 . firebase skill을 사용하고"
Claude: *Sees trigger: "firebase skill"*
Claude: *Runs npm create vite*
Claude: *Runs npx firebase projects:create todo-app-<timestamp>*
Claude: *Runs npx firebase apps:sdkconfig to get real keys*
Result: Live URL with working backend
```

---

## [3.0.0] - 2025-10-23

### 🚨 BREAKING CHANGE - Complete SKILL.md Rewrite

**Why**: Previous v2.1.0 format used "Execute:" prefix which Claude interpreted as documentation, not commands to execute. Claude ignored the skill and created files manually.

**Root Cause**: SKILL.md was written as **instructional guide** instead of **imperative workflow**.

#### Changed

- **Complete SKILL.md rewrite** following pptx skill pattern
- Changed from "Execute: `command`" to direct bash code blocks with workflow instructions
- Removed "Mandatory first step" and similar weak directives
- Changed from numbered phases to clear step-by-step workflow
- Simplified structure: Overview → Steps → Common Issues → Key Patterns

#### Format Transformation

**Before (v2.1.0 - Documentation style)**:
```markdown
**Mandatory first step**: Read `firebase-cli.md` completely.

Execute these commands in sequence. Stop if any command fails.

1. **Create React project with Vite**

   Execute: `npm create vite@latest <project-name> -- --template react`

   Then execute: `cd <project-name>`
```

**After (v3.0.0 - Imperative workflow)**:
```markdown
When a user asks you to create a React app with Firebase backend, follow this workflow...

### Step 2: Create React Project

\```bash
npm create vite@latest <project-name> -- --template react
cd <project-name>
npm install
\```

Verify `package.json` exists and contains "react" in dependencies.
```

#### Key Differences from v2.1.0

1. **First line sets context**: "When a user asks..." instead of "Automates creation..."
2. **Direct commands**: Bash blocks instead of "Execute: " prefix
3. **Inline verification**: "Verify X exists" instead of separate validation sections
4. **Removed external references**: No "See firebase-cli.md" - everything inline
5. **Removed phases**: Direct steps 1-21 instead of "Phase 1", "Phase 2", etc.

#### Impact

- ✅ Claude now **executes commands directly** instead of showing documentation
- ✅ Claude **follows workflow** instead of creating files manually
- ✅ Claude **uses Firebase CLI** to create real backend
- ✅ Claude **parses outputs** to extract IDs and config
- ✅ Complete automation from empty directory to live URL

#### Test Results

**v2.1.0 behavior** (broken):
- User: "Create todo app with Firebase"
- Claude: *Creates package.json, firebase.json manually*
- Claude: *Writes placeholder values in .env*
- Result: No Firebase backend, manual setup required

**v3.0.0 behavior** (fixed):
- User: "Create todo app with Firebase"
- Claude: *Runs npm create vite*
- Claude: *Runs firebase projects:create*
- Claude: *Parses SDK config and creates real .env*
- Result: Live URL with working backend

#### Files Modified

- `SKILL.md` - Complete rewrite (581 lines → 729 lines)
- Removed references to external files (`firebase-cli.md`, `component-patterns.md`)
- All Firebase CLI commands now inline with explanations
- All React components now inline with complete code

---

## [2.1.0] - 2025-10-23 [DEPRECATED]

### 🚀 Major Changes - Command Execution

**BREAKING**: Transformed from documentation skill to execution skill.

#### Changed
- **SKILL.md completely rewritten** to execute commands automatically
- Changed all instructions from "Run this command" to "Execute: <command>"
- Added automatic command execution following pptx skill pattern
- Added output parsing and validation after each step

#### Added
- **Firebase Tools auto-installation**
  - Checks for `npx firebase --version`
  - Auto-installs `firebase-tools@latest` if missing or < v14.0.0
  - Auto-triggers `firebase login` if not authenticated

- **Command execution with validation**
  - Execute: `npm create vite@latest <project-name>`
  - Execute: `npx firebase projects:create <project-id>`
  - Execute: `npx firebase apps:create WEB <app-name>`
  - Execute: `npx firebase apps:sdkconfig WEB <app-id>`
  - Execute: `npx firebase deploy --only firestore`
  - Execute: `npm run build`
  - Execute: `npx firebase deploy --only hosting`

- **Output parsing instructions**
  - Parse project ID from creation output
  - Extract app ID from registration output
  - Parse SDK config JSON for all Firebase values
  - Extract deployment URL from hosting output

- **Validation steps after each command**
  - Check `package.json` exists after npm install
  - Verify `.env` file populated with actual values
  - Confirm `dist/` directory created after build
  - Parse deployment success messages

- **Automatic .env file generation**
  - Uses parsed values from `firebase apps:sdkconfig`
  - Creates `.env` with actual Firebase config
  - Creates `.env.example` with placeholders

- **Authentication automation with fallback** (Phase 6)
  - Attempts API-based activation via curl + OAuth token
  - Falls back to direct Console link if token unavailable
  - **Critical rebuild step** after Auth activation to prevent API errors
  - Verification with curl test to confirm Auth is working

#### Fixed
- **Issue**: Skill was not executing commands, only showing documentation
- **Issue**: Firebase CLI was never installed automatically
- **Issue**: User had to manually run all Firebase commands
- **Issue**: No validation that commands succeeded
- **Issue**: Authentication not automated (only manual Console steps)
- **Issue**: Missing rebuild after Auth activation caused "Firebase API key invalid" errors
- **Issue**: No verification step to confirm Auth is working

#### Technical Details

**Before (Documentation format)**:
```markdown
1. **Create Firebase project**
   ```bash
   npx firebase projects:create [name]
   ```
```

**After (Execution format)**:
```markdown
1. **Create Firebase project**

   Execute: `npx firebase projects:create <project-id>`

   **Validation**: Parse output for "Firebase project <project-id> is ready!"

   **Save** the project ID from output for all subsequent commands.
```

#### Impact
- ✅ Claude now **executes** Firebase CLI commands instead of showing them
- ✅ Claude **parses** command output to extract IDs and config values
- ✅ Claude **validates** each step before proceeding
- ✅ Claude **creates files** with actual values (not placeholders)
- ✅ Complete automation from empty directory to deployed app

---

## [2.0.0] - 2025-10-22

### Added
- Complete skill restructure following official Anthropic best practices
- Separated workflows into main SKILL.md
- Created firebase-cli.md with complete CLI reference
- Created component-patterns.md with React implementation patterns
- Added progressive disclosure with guides/ directory
  - typescript-setup.md
  - advanced-security.md
  - nextjs-variant.md
- Added proper marketplace.json for plugin distribution
- Added comprehensive DEPLOYMENT_GUIDE.md

### Changed
- Restructured from single react-firebase.md to multiple focused files
- Removed redundant content (Claude's existing knowledge)
- Applied progressive disclosure pattern
- Updated to follow official pptx skill structure

### Fixed
- marketplace.json schema validation errors
  - Changed `source` from `"."` to `"./"`
  - Changed `author` from string to object `{name}`
  - Removed unrecognized fields: displayName, features, dependencies, files

---

## [1.0.0] - 2025-10-21

### Added
- Initial release
- React 19 + Vite 7 + Tailwind CSS 3 setup
- Firebase CLI automation workflow
- Firestore configuration and deployment
- Authentication setup guide (Email/Password + Google)
- Firebase Hosting deployment
- Complete documentation generation (4 files)
- User-specific security rules
- Real-time Firestore integration

### Features
- 3 complete workflows (new project, add to existing, TypeScript)
- Time savings: 2-3 hours → 10-15 minutes

---

## Version History Summary

| Version | Date | Type | Description |
|---------|------|------|-------------|
| 3.1.0 | 2025-10-23 | Patch | Fixed skill not triggering on first prompt |
| 3.0.0 | 2025-10-23 | Breaking | Imperative workflow format (actually executes) |
| 2.1.0 | 2025-10-23 | Deprecated | Command execution (didn't work) |
| 2.0.0 | 2025-10-22 | Major | Restructure following official patterns |
| 1.0.0 | 2025-10-21 | Initial | First public release |

---

## Upgrade Guide

### From 2.1.0 to 3.0.0

**Action required**: Reinstall the skill to get working command execution.

v2.1.0 had a critical bug where Claude ignored the skill and created files manually. v3.0.0 fixes this by rewriting SKILL.md in imperative workflow format.

### From 2.0.0 to 3.0.0

**Action required**: Reinstall the skill to get command execution features.

```bash
# Remove old version
/plugin marketplace remove react-firebase-marketplace
/plugin uninstall react-firebase

# Install new version
/plugin marketplace add jkf87/react-firebase-skill
/plugin install react-firebase
```

**What's different**:
- Skill now **executes commands automatically**
- No need to manually run Firebase CLI commands
- Claude handles output parsing and validation
- Automatic .env file generation with real values

**Before (v2.0.0)**:
- User: "Create React Firebase app"
- Claude: *Shows documentation and command examples*
- User: *Manually copies and runs each command*

**After (v3.0.0)**:
- User: "Create React Firebase app"
- Claude: *Executes all commands automatically*
- Result: *Deployed app with live URL*

---

## Breaking Changes

### v3.0.0
- SKILL.md completely rewritten in imperative workflow format
- Removed external file references (firebase-cli.md, component-patterns.md)
- All code and commands now inline in SKILL.md
- Changed from "Execute: " prefix to direct bash code blocks
- May require Claude Code restart after installation

### v2.1.0 [DEPRECATED]
- SKILL.md format completely changed from documentation to execution
- ❌ Bug: Claude ignored "Execute:" prefix and created files manually
- ❌ Fixed in v3.0.0

### v2.0.0
- File structure changed (single file → multiple files)
- marketplace.json schema updated
- plugin.json fields removed

---

## Roadmap

### v2.2.0 (Planned)
- [ ] Add TypeScript template option
- [ ] Add Next.js variant support
- [ ] Implement Cloud Functions deployment
- [ ] Add Firebase Storage configuration

### v3.0.0 (Future)
- [ ] Support for monorepo structures
- [ ] CI/CD pipeline generation
- [ ] Custom domain setup automation
- [ ] Advanced security rules templates

---

**Repository**: https://github.com/jkf87/react-firebase-skill
**Issues**: https://github.com/jkf87/react-firebase-skill/issues
**License**: MIT
