# Changelog

All notable changes to the React Firebase Deployment Skill.

## [2.1.0] - 2025-10-23

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

#### Fixed
- **Issue**: Skill was not executing commands, only showing documentation
- **Issue**: Firebase CLI was never installed automatically
- **Issue**: User had to manually run all Firebase commands
- **Issue**: No validation that commands succeeded

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
| 2.1.0 | 2025-10-23 | Major | Command execution automation |
| 2.0.0 | 2025-10-22 | Major | Restructure following official patterns |
| 1.0.0 | 2025-10-21 | Initial | First public release |

---

## Upgrade Guide

### From 2.0.0 to 2.1.0

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

**After (v2.1.0)**:
- User: "Create React Firebase app"
- Claude: *Executes all commands automatically*
- Result: *Deployed app with live URL*

---

## Breaking Changes

### v2.1.0
- SKILL.md format completely changed from documentation to execution
- Claude now expects to execute commands, not just display them
- May require Claude Code restart after installation

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
