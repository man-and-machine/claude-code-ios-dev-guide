# iOS Team Workflow Implementation Guide

> A practical guide for implementing PRD-driven iOS development with Claude Code CLI for a 2-developer team.

---

## Table of Contents

1. [Overview: What This Workflow Provides](#overview-what-this-workflow-provides)
2. [Initial Setup (Both Developers)](#initial-setup-both-developers)
3. [Project Configuration (One-Time Setup)](#project-configuration-one-time-setup)
4. [Daily Workflow for Your Team](#daily-workflow-for-your-team)
5. [Collaboration Best Practices](#collaboration-best-practices)
6. [Critical Things to Watch Out For](#critical-things-to-watch-out-for)
7. [Complete Feature Implementation Example](#complete-feature-implementation-example)

---

## Overview: What This Workflow Provides

This workflow enables you and your fellow developer to:

| Benefit | How It Works |
|---------|--------------|
| **Consistent AI assistance** | Shared `CLAUDE.md` and settings ensure Claude understands your project the same way for both developers |
| **Structured development** | PRD → Specs → Tasks → Code flow keeps everyone aligned |
| **Safe collaboration** | Permission controls and hooks prevent accidents |
| **Efficient builds** | XcodeBuildMCP handles all Xcode operations without leaving Claude |
| **Clear task ownership** | Task files track who's working on what |

### The Core Methodology (Separate Features)

Since you're each working on **different features**, the workflow looks like this:

```
┌─────────────────────────────────────────────────────────────────┐
│                 PARALLEL FEATURE DEVELOPMENT                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   DEVELOPER A                         DEVELOPER B               │
│   ───────────                         ───────────               │
│   Feature: User Profile               Feature: Settings         │
│                                                                 │
│   1. Create spec ◄────────────────────► 1. Create spec          │
│      docs/specs/profile.md               docs/specs/settings.md │
│                                                                 │
│   2. Create tasks ◄───────────────────► 2. Create tasks         │
│      docs/tasks/profile-tasks.md         docs/tasks/settings.md │
│                                                                 │
│   3. Implement ◄──────────────────────► 3. Implement            │
│      Features/Profile/                   Features/Settings/     │
│                                                                 │
│   4. Build & Test ◄───────────────────► 4. Build & Test         │
│                                                                 │
│                    ┌─────────────┐                              │
│                    │ SHARED CODE │ ◄── Coordination needed!     │
│                    │  Core/      │                              │
│                    │  Services/  │                              │
│                    │  Utils/     │                              │
│                    └─────────────┘                              │
│                           │                                     │
│                           ▼                                     │
│   5. PR + Review ◄────────────────────► 5. PR + Review          │
│                                                                 │
│                    ┌─────────────┐                              │
│                    │   MERGE     │                              │
│                    │ Integration │                              │
│                    │   Tests     │                              │
│                    └─────────────┘                              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**Key insight**: When working on separate features, you have full ownership of your feature directory. The main coordination points are:
1. **Shared code** (Core/, Services/, Extensions/)
2. **Architectural decisions** that affect both features
3. **Integration** when features need to interact

---

## Initial Setup (Both Developers)

### Step 1: Install Claude Code CLI

Each developer runs on their own machine:

```bash
# Install Claude Code CLI (native installation - recommended)
curl -fsSL https://claude.ai/install.sh | bash

# Verify installation
claude --version
```

### Step 2: Authenticate

```bash
# Start Claude and authenticate via OAuth
claude

# Or set your API key
export ANTHROPIC_API_KEY="your-key-here"
```

### Step 3: Install Node.js (Required for XcodeBuildMCP)

```bash
# If not already installed (via Homebrew)
brew install node

# Verify
node --version
npm --version
```

### Step 4: Configure Personal Global Settings

Each developer creates their personal settings at `~/.claude/settings.json`:

```json
{
  "model": "claude-sonnet-4-5-20250929",
  "permissions": {
    "allow": [
      "Bash(git *)",
      "Read",
      "Bash(swift *)"
    ]
  }
}
```

> **Note**: These are your personal defaults. Project settings will override them.

---

## Project Configuration (One-Time Setup)

**One developer** sets this up, commits, and pushes. The other pulls and is ready to go.

### Step 1: Create Directory Structure

In your iOS project root, create:

```bash
# Create Claude configuration directories
mkdir -p .claude/commands
mkdir -p .claude/agents
mkdir -p .claude/skills
mkdir -p .claude/hooks

# Create documentation structure
mkdir -p docs/specs
mkdir -p docs/tasks
```

### Step 2: Create `.mcp.json` (XcodeBuildMCP Configuration)

This is the **most important** configuration for iOS development. Create `.mcp.json` in project root:

```json
{
  "mcpServers": {
    "XcodeBuildMCP": {
      "command": "npx",
      "args": ["-y", "xcodebuildmcp@latest"],
      "env": {
        "INCREMENTAL_BUILDS_ENABLED": "true",
        "XCODEBUILDMCP_SENTRY_DISABLED": "true",
        "XCODEBUILDMCP_DYNAMIC_TOOLS": "true"
      }
    }
  }
}
```

### Step 3: Create `CLAUDE.md` (Project Context)

This file tells Claude everything about your project. Create `CLAUDE.md` in project root:

```markdown
# Project: [Your App Name]

## Quick Reference
- **Platform**: iOS 17+
- **Language**: Swift 6.0
- **UI Framework**: SwiftUI
- **Architecture**: MVVM with @Observable
- **Package Manager**: Swift Package Manager

## Project Structure
[Describe your actual project structure]

## Build Commands
- **Build**: Use `mcp__xcodebuildmcp__build_sim_name_proj`
- **Test**: Use `mcp__xcodebuildmcp__test_sim_name_proj`
- **Clean**: Use `mcp__xcodebuildmcp__clean`

## Coding Standards
- Use Swift 6 strict concurrency
- Prefer `@Observable` over `ObservableObject`
- Use `async/await` for all async operations
- Prefer value types (structs) over reference types

## Team Members
- Developer A: [Name] - Focus areas
- Developer B: [Name] - Focus areas

## Current Sprint Focus
[What the team is working on]

## DO NOT
- Force unwrap without justification
- Ignore Swift 6 concurrency warnings
- Commit to main directly
- Modify files in [protected paths]

## Memory Imports
@import docs/PRD.md
@import docs/ARCHITECTURE.md
```

### Step 4: Create Team Settings (`.claude/settings.json`)

This is **shared with the team** via git:

```json
{
  "model": "claude-sonnet-4-5-20250929",
  "permissions": {
    "allow": [
      "mcp__xcodebuildmcp__*",
      "Read",
      "Write",
      "Edit",
      "Glob",
      "Grep",
      "Bash(git status)",
      "Bash(git diff *)",
      "Bash(git log *)",
      "Bash(git add *)",
      "Bash(git commit *)",
      "Bash(git push *)",
      "Bash(git pull *)",
      "Bash(git checkout *)",
      "Bash(git branch *)",
      "Bash(swift build *)",
      "Bash(swift test *)",
      "Bash(swiftlint *)"
    ],
    "deny": [
      "Read(.env*)",
      "Read(**/Secrets.swift)",
      "Write(.env*)",
      "Write(**/Secrets.swift)",
      "Bash(rm -rf *)",
      "Bash(git push --force *)",
      "Bash(git reset --hard *)"
    ]
  },
  "env": {
    "PROJECT_NAME": "YourAppName",
    "DEFAULT_SIMULATOR": "iPhone 15"
  }
}
```

### Step 5: Create Personal Local Settings Template

Create `.claude/settings.local.json.template` (committed) as a template:

```json
{
  "_comment": "Copy this to settings.local.json and customize for your machine",
  "model": "claude-sonnet-4-5-20250929",
  "env": {
    "DEVELOPMENT_TEAM": "YOUR_TEAM_ID_HERE"
  }
}
```

Each developer copies this to `.claude/settings.local.json` (gitignored) with their personal settings.

### Step 6: Create Essential Slash Commands

**`.claude/commands/build.md`**:
```markdown
---
description: Build the iOS project
allowed-tools: mcp__xcodebuildmcp__*
---

# Build Project

1. Discover project: `mcp__xcodebuildmcp__discover_projects`
2. Build for simulator: `mcp__xcodebuildmcp__build_sim_name_proj`
3. Report any errors with suggested fixes
```

**`.claude/commands/test.md`**:
```markdown
---
description: Run all tests
allowed-tools: mcp__xcodebuildmcp__*, Read
---

# Run Tests

1. Run tests: `mcp__xcodebuildmcp__test_sim_name_proj`
2. Report failures with context
3. Suggest fixes for failing tests
```

**`.claude/commands/implement.md`**:
```markdown
---
description: Implement next task from task file
argument-hint: <feature-name>
allowed-tools: Read, Write, Edit, mcp__xcodebuildmcp__*
---

# Implement Feature: $ARGUMENTS

1. Read `docs/tasks/$ARGUMENTS-tasks.md`
2. Find the first uncompleted task
3. Implement it with tests
4. Build and run tests
5. Mark task complete in the task file
6. Stop and wait for approval before next task
```

### Step 7: Update `.gitignore`

Add these lines to your `.gitignore`:

```gitignore
# Claude Code local settings
.claude/settings.local.json
.claude/*.log

# Keep these committed
!.claude/commands/
!.claude/agents/
!.claude/skills/
!.claude/hooks/
!.claude/settings.json
```

### Step 8: Commit and Push

```bash
git add .claude/ .mcp.json CLAUDE.md docs/ .gitignore
git commit -m "Add Claude Code team workflow configuration"
git push
```

---

## Daily Workflow for Your Team

### Starting Your Day

```bash
# Pull latest changes
git pull

# Start Claude Code
cd /path/to/your/ios-project
claude

# Claude automatically loads:
# - CLAUDE.md (project context)
# - .mcp.json (XcodeBuildMCP)
# - .claude/settings.json (team settings)
# - docs/PRD.md (via @import)
```

### Typical Development Session

```
┌────────────────────────────────────────────────────────────────┐
│ DEVELOPER A's SESSION                                          │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│ You: "What's my next task for the authentication feature?"     │
│                                                                │
│ Claude: [Reads docs/tasks/authentication-tasks.md]             │
│         "Your next task is Step 3: Implement login API call"   │
│                                                                │
│ You: "Implement it using the /implement command"               │
│      /implement authentication                                 │
│                                                                │
│ Claude: [Implements, writes tests, builds, reports]            │
│                                                                │
│ You: "Looks good. Build and test."                             │
│      /test                                                     │
│                                                                │
│ Claude: [Runs tests via XcodeBuildMCP]                         │
│         "All 15 tests passed"                                  │
│                                                                │
│ You: "Commit this"                                             │
│                                                                │
│ Claude: [Commits with appropriate message]                     │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

### Permission Mode Cycling

Press `Shift+Tab` to cycle through modes:

| Mode | Use Case | Visual Indicator |
|------|----------|------------------|
| **Normal** | Default development | None |
| **Auto-Accept** | Trusted, repetitive work | `⏵⏵ accept edits on` |
| **Plan Mode** | Read-only analysis | `⏸ plan mode on` |

**Recommendation**: Use Plan Mode for exploring unfamiliar code, Normal Mode for development.

---

## Collaboration Best Practices (Separate Features)

### 1. Each Developer Owns Their Feature Entirely

Since you're working on separate features, each developer creates their **own** spec and task file:

**Developer A creates:**
- `docs/specs/user-profile.md`
- `docs/tasks/user-profile-tasks.md`
- Works in `Features/Profile/`

**Developer B creates:**
- `docs/specs/settings.md`
- `docs/tasks/settings-tasks.md`
- Works in `Features/Settings/`

### 2. Track Active Features in CLAUDE.md

Add a section to your shared `CLAUDE.md` so Claude knows who's working where:

```markdown
## Active Development

### Developer A: User Profile Feature
- **Branch**: `feature/user-profile`
- **Directory**: `Features/Profile/`
- **Spec**: `docs/specs/user-profile.md`
- **Status**: In Progress

### Developer B: Settings Feature
- **Branch**: `feature/settings`
- **Directory**: `Features/Settings/`
- **Spec**: `docs/specs/settings.md`
- **Status**: In Progress

## Off-Limits Directories
- Developer A: DO NOT modify `Features/Settings/`
- Developer B: DO NOT modify `Features/Profile/`
```

### 3. Branch Strategy for Separate Features

```bash
# Each developer has their own feature branch
git checkout -b feature/user-profile    # Developer A
git checkout -b feature/settings        # Developer B

# Both branch from and merge to develop
git checkout develop
git pull
git checkout -b feature/your-feature
```

### 4. Coordinate on Shared Code

The **critical coordination point** is shared code. Before modifying:

| Directory | Action Required |
|-----------|-----------------|
| `Core/` | Discuss with teammate first |
| `Services/` | Check if teammate is using it |
| `Extensions/` | Notify teammate of additions |
| `Models/` | Coordinate on shared models |
| `Networking/` | Agree on API changes |

**Example communication:**

```
Slack/Message to teammate:
"I need to add a `UserService` to Core/Services/ for my Profile feature.
Are you using or planning to create anything similar for Settings?"
```

### 5. Sync Points for Separate Features

```
Weekly Planning:
├── Discuss upcoming features
├── Identify potential shared code needs
└── Update CLAUDE.md with active features

Daily (Optional):
├── Quick check on shared code changes
├── Pull from develop to stay current
└── Flag any blocking issues

Before PR:
├── Pull latest develop
├── Run full test suite
├── Check for conflicts in shared code
└── Review impact on teammate's feature
```

### 6. Shared Context Updates

When you add shared code that your teammate might use:

```
You: "I added a new ImagePickerService in Core/Services/. Update
     CLAUDE.md to document this so my teammate's Claude knows about it."

Claude: [Updates CLAUDE.md with new service documentation]
```

**Commit the CLAUDE.md update** so your teammate's Claude session picks it up.

---

## Critical Things to Watch Out For (Separate Features)

### 1. Shared Code Conflicts (THE #1 ISSUE)

**Problem**: Both developers modify files in `Core/`, `Services/`, or `Extensions/` and create merge conflicts.

**Example**:
- Developer A adds `validateEmail()` to `Core/Extensions/String+Validation.swift`
- Developer B adds `validatePhone()` to the same file
- Merge conflict when both merge to develop

**Solution**:
- **Communicate before touching shared code** - a quick message saves hours
- Pull from develop frequently (daily minimum)
- Consider creating feature-specific services first, then extract shared code later

```
You: "Before adding to Core/Extensions, check if my teammate has
     pending changes there. Run: git fetch && git diff origin/develop -- Core/"

Claude: [Checks for teammate's pending changes]
```

### 2. Duplicate Code (Both Create Similar Utilities)

**Problem**: Without coordination, both developers create similar utilities independently.

**Example**:
- Developer A creates `ProfileImageLoader` in their feature
- Developer B creates `SettingsImageLoader` with nearly identical code

**Solution**:
- Review shared code needs during weekly planning
- Before creating a utility, search the codebase first:

```
You: "Before creating an image loading utility, search the codebase
     for existing image loading code"

Claude: [Searches and finds existing or similar utilities]
```

- Add new shared utilities to `CLAUDE.md` immediately

### 3. Architectural Drift

**Problem**: Each developer's Claude makes different architectural decisions, leading to inconsistent patterns.

**Example**:
- Developer A's Claude uses `@Observable` with dependency injection
- Developer B's Claude uses `@ObservableObject` with singletons

**Solution**: Be very specific in `CLAUDE.md` about architecture:

```markdown
## Architecture Requirements (MUST FOLLOW)
- ViewModels: ALWAYS use @Observable, NEVER @ObservableObject
- Dependencies: ALWAYS inject via init, NEVER use singletons
- Navigation: ALWAYS use NavigationStack with typed routes
- Networking: ALWAYS use the shared NetworkClient in Core/
- Error Handling: ALWAYS use the AppError enum from Core/
```

### 4. Integration Surprises

**Problem**: Features work in isolation but break when merged together.

**Example**:
- Profile feature uses UserModel with `name: String`
- Settings feature expects UserModel with `displayName: String`
- App crashes when both features access the same user

**Solution**:
- Share data models early - put them in `Core/Models/`
- Write integration tests that use multiple features
- Test on develop branch before final merge

```
You: "Check if my UserModel is compatible with what exists in Core/Models/"

Claude: [Compares and flags differences]
```

### 5. Stale CLAUDE.md

**Problem**: Your teammate adds shared code but doesn't update `CLAUDE.md`. Your Claude doesn't know about it and creates duplicates.

**Solution**:
- Make updating `CLAUDE.md` part of the shared code workflow
- Pull before starting each session
- Ask Claude to check for recent changes:

```
You: "What files in Core/ have been modified recently?
     Are there new utilities I should know about?"

Claude: [Checks git log for Core/ changes]
```

### 6. Accidentally Modifying Partner's Feature Directory

**Problem**: Claude edits files in your teammate's feature directory.

**Solution**: Use permissions in `.claude/settings.local.json`:

```json
{
  "permissions": {
    "deny": [
      "Write(Features/Settings/*)",
      "Edit(Features/Settings/*)"
    ]
  }
}
```

**Note**: Each developer sets this in their **local** settings to protect their teammate's directory.

### 7. Different Code Styles

**Problem**: Each developer's Claude generates code with slightly different styles.

**Solution**: Be extremely specific in `CLAUDE.md`:

```markdown
## Code Style Requirements (MANDATORY)
- Indent: 4 spaces (NOT tabs)
- Braces: Same line for functions and closures
- Max line length: 120 characters
- Trailing commas: Always in multi-line collections
- Self: Explicit in closures, implicit elsewhere
- Naming:
  - Booleans: `is`/`has`/`should` prefix
  - Arrays: Plural nouns (items, users)
  - Optionals: No special prefix
```

### 8. Version Mismatches

**Problem**: Different XcodeBuildMCP or Claude Code versions cause different behavior.

**Solution**: Pin versions and check them:

```json
// .mcp.json - pin XcodeBuildMCP version
{
  "mcpServers": {
    "XcodeBuildMCP": {
      "command": "npx",
      "args": ["-y", "xcodebuildmcp@1.2.3"]
    }
  }
}
```

```bash
# Both developers should run same Claude Code version
claude --version
```

### 9. Secrets Exposure

**Problem**: Claude might accidentally read or commit secrets.

**Solution**: Always deny in team settings:

```json
{
  "permissions": {
    "deny": [
      "Read(.env*)",
      "Read(**/Secrets.swift)",
      "Read(**/GoogleService-Info.plist)",
      "Write(.env*)",
      "Write(**/Secrets.swift)"
    ]
  }
}
```

---

## Complete Example: Two Developers, Two Features

Let's walk through a realistic scenario where:
- **Developer A** works on **User Profile** feature
- **Developer B** works on **App Settings** feature

Both features need shared code, which is where coordination happens.

---

### Week Start: Planning Session (Both Developers)

Before coding, have a quick sync to identify shared code needs:

```
Discussion:
├── Developer A: "Profile needs to fetch/update user data"
├── Developer B: "Settings needs to update user preferences"
├── Shared need identified: Both need UserService in Core/
└── Decision: Developer A creates UserService, Developer B uses it
```

**Update CLAUDE.md together:**

```markdown
## Active Development

### Developer A: User Profile Feature
- **Branch**: `feature/user-profile`
- **Directory**: `Features/Profile/`
- **Shared Code Responsibility**: Creating UserService in Core/Services/

### Developer B: App Settings Feature
- **Branch**: `feature/settings`
- **Directory**: `Features/Settings/`
- **Depends On**: UserService from Developer A

## Off-Limits
- Developer A: DO NOT modify Features/Settings/
- Developer B: DO NOT modify Features/Profile/
```

---

### Developer A's Workflow (User Profile)

**Day 1: Create spec and start feature**

```bash
git checkout -b feature/user-profile
claude
```

**Session:**

```
You: "I'm Developer A working on the User Profile feature. My teammate
     (Developer B) is working on Settings in Features/Settings/ - don't
     touch that directory.

     First, ultrathink and create a spec for User Profile at
     docs/specs/user-profile.md"

Claude: [Extended thinking...]
        [Creates comprehensive spec]

You: "Now create a task breakdown at docs/tasks/user-profile-tasks.md"

Claude: [Creates task file with 6 steps]
```

**Developer A's Task File (`docs/tasks/user-profile-tasks.md`):**

```markdown
# Tasks: User Profile

**Owner**: Developer A
**Branch**: feature/user-profile
**Spec**: docs/specs/user-profile.md

## Progress: 0/6 Complete

### Step 1: Create UserService in Core/ (SHARED)
- [ ] Create Core/Services/UserService.swift protocol
- [ ] Create Core/Services/UserAPIService.swift implementation
- [ ] Add to CLAUDE.md shared code documentation
- [ ] Notify Developer B that UserService is available
**Status**: Not Started
**Note**: Developer B's Settings feature will use this too

### Step 2: Create ProfileViewModel
- [ ] Create Features/Profile/ViewModels/ProfileViewModel.swift
- [ ] Use UserService for data fetching
- [ ] Add validation logic
- [ ] Write unit tests
**Status**: Not Started

### Step 3: Create ProfileView
- [ ] Create Features/Profile/Views/ProfileView.swift
- [ ] Display user info
- [ ] Add edit functionality
**Status**: Not Started

### Step 4: Add avatar editing
- [ ] Create ImagePicker integration
- [ ] Handle image upload
**Status**: Not Started

### Step 5: Write tests
- [ ] Unit tests for ViewModel
- [ ] Integration tests
**Status**: Not Started

### Step 6: Final review and PR
- [ ] Review against spec
- [ ] Create PR
**Status**: Not Started
```

**Implementing Step 1 (Shared Code):**

```
You: "Implement Step 1. This UserService will be used by both my
     Profile feature and Developer B's Settings feature. Make it
     general enough for both use cases."

Claude: [Creates Core/Services/UserService.swift]
        [Creates Core/Services/UserAPIService.swift]
        [Creates Core/Services/MockUserService.swift]
        [Creates UserServiceTests.swift]

You: "Update CLAUDE.md to document this new shared service"

Claude: [Updates CLAUDE.md with UserService documentation]

You: "Commit and push so Developer B can use it"

Claude: [Commits: "Add UserService to Core/ for user data management"]
        [Pushes to feature/user-profile]
```

**Message to Developer B:**
> "Hey, I pushed UserService to my branch. It has fetchUser(), updateUser(),
> and updatePreferences(). You can cherry-pick it or I can merge to develop."

---

### Developer B's Workflow (Settings) - Running in Parallel

**Day 1: Create spec and start feature**

```bash
git checkout -b feature/settings
claude
```

**Session:**

```
You: "I'm Developer B working on the App Settings feature. My teammate
     (Developer A) is working on Profile in Features/Profile/ - don't
     touch that directory.

     Developer A is creating a UserService in Core/Services/ that I'll
     use for user preferences.

     Create a spec for Settings at docs/specs/settings.md"

Claude: [Creates spec, noting dependency on UserService]

You: "Create tasks at docs/tasks/settings-tasks.md. Note that Step 1
     depends on Developer A's UserService."

Claude: [Creates task file]
```

**Developer B's Task File (`docs/tasks/settings-tasks.md`):**

```markdown
# Tasks: App Settings

**Owner**: Developer B
**Branch**: feature/settings
**Spec**: docs/specs/settings.md

## Progress: 0/5 Complete

### Step 1: Create SettingsViewModel
- [ ] Create Features/Settings/ViewModels/SettingsViewModel.swift
- [ ] Use UserService from Core/ (created by Developer A)
- [ ] Add preference management logic
**Status**: Blocked - waiting for UserService
**Depends On**: Developer A's UserService in Core/

### Step 2: Create SettingsView
- [ ] Create main settings screen
- [ ] Add preference toggles
- [ ] Add account section
**Status**: Not Started

### Step 3: Add notification settings
- [ ] Create notification preferences UI
- [ ] Integrate with system notifications
**Status**: Not Started

### Step 4: Add appearance settings
- [ ] Theme selection (light/dark/system)
- [ ] Font size options
**Status**: Not Started

### Step 5: Final review and PR
- [ ] Review against spec
- [ ] Create PR
**Status**: Not Started
```

**Working on non-blocked tasks first:**

```
You: "Step 1 is blocked. Start Step 2 with a placeholder for UserService"

Claude: [Creates SettingsView with mock data]
        [Creates Features/Settings/Views/SettingsView.swift]
        [Adds to navigation]

You: "/build"

Claude: [Builds successfully]
```

**Day 2: Developer A's UserService is ready**

```
You: "Developer A pushed UserService. Pull the latest and check if
     I can start Step 1 now."

Claude: [Runs git fetch && git merge origin/feature/user-profile --no-edit]
        "Merged Developer A's changes. UserService is now available in
         Core/Services/. You can proceed with Step 1."

You: "Implement Step 1 now using the real UserService"

Claude: [Creates SettingsViewModel using UserService]
        [Writes unit tests with MockUserService]

You: "Update the task file to mark Step 1 complete"

Claude: [Updates task file, notes the unblocking]
```

---

### Shared Code Coordination Scenario

**Developer B discovers they need a new utility:**

```
You: "I need a DateFormatter extension for displaying 'Last synced' time.
     Before creating it, check if Developer A has something similar or
     if there's an existing utility in Core/."

Claude: [Searches Core/Extensions/ and Features/Profile/]
        "I found that Developer A created Date+Formatting.swift in
         Core/Extensions/ with a relativeTimeString() method. You can
         use that instead of creating a new one."

You: "Perfect, use that existing extension"

Claude: [Uses existing Date+Formatting extension]
```

**If no existing utility:**

```
You: "I need a validation helper for email. Check Core/ first."

Claude: [Searches - nothing found]
        "No existing email validation found. Should I:
         1. Create it in Features/Settings/ (feature-specific)
         2. Create it in Core/Extensions/ (shared)

         Since Developer A might need email validation too, I recommend
         option 2."

You: "Create it in Core/Extensions/ and update CLAUDE.md"

Claude: [Creates Core/Extensions/String+Validation.swift]
        [Updates CLAUDE.md to document new utility]

You: "Commit with a clear message so Developer A knows"

Claude: [Commits: "Add email validation extension to Core/

         Added String+Validation.swift with isValidEmail property.
         Developer A: Feel free to use this in Profile feature."]
```

---

### Integration: Both Features Ready

**Developer A finishes and merges first:**

```
You: "All my tasks are done. Pull latest develop and run all tests"

Claude: [Pulls, runs tests]
        "All 34 tests pass. No conflicts with develop."

You: "Create a PR for the Profile feature"

Claude: [Creates PR #42: "Add User Profile feature"]
```

**Developer B integrates after:**

```
You: "Developer A's Profile feature is merged to develop. Pull it
     and make sure Settings still works."

Claude: [Pulls from develop]
        [Runs all tests]
        "Settings tests pass. However, I notice Developer A modified
         UserService to add an avatarURL field. Your SettingsViewModel
         should handle this gracefully."

You: "Update SettingsViewModel to work with the new UserService"

Claude: [Updates SettingsViewModel]
        [All tests pass]

You: "Create PR for Settings feature"

Claude: [Creates PR #43: "Add App Settings feature"]
```

---

### Final State

**docs/ directory after both features:**
```
docs/
├── PRD.md
├── ARCHITECTURE.md
├── specs/
│   ├── user-profile.md      (Developer A)
│   └── settings.md          (Developer B)
└── tasks/
    ├── user-profile-tasks.md (Complete)
    └── settings-tasks.md     (Complete)
```

**Features/ directory:**
```
Features/
├── Profile/                  (Developer A)
│   ├── Views/
│   │   └── ProfileView.swift
│   └── ViewModels/
│       └── ProfileViewModel.swift
└── Settings/                 (Developer B)
    ├── Views/
    │   ├── SettingsView.swift
    │   └── NotificationSettingsView.swift
    └── ViewModels/
        └── SettingsViewModel.swift
```

**Core/ (shared, coordinated):**
```
Core/
├── Services/
│   ├── UserService.swift         (Created by Developer A)
│   ├── UserAPIService.swift      (Created by Developer A)
│   └── MockUserService.swift     (Created by Developer A)
└── Extensions/
    ├── Date+Formatting.swift     (Created by Developer A)
    └── String+Validation.swift   (Created by Developer B)
```

---

## Quick Reference Card

### Essential Commands

| Command | Description |
|---------|-------------|
| `/build` | Build the project |
| `/test` | Run all tests |
| `/implement <feature>` | Implement next task |
| `/compact` | Compress conversation context |
| `/clear` | Clear context, start fresh |
| `/cost` | Show token usage |

### Keyboard Shortcuts

| Shortcut | Action |
|----------|--------|
| `Tab` | Toggle extended thinking |
| `Shift+Tab` | Cycle permission modes |
| `Ctrl+O` | Toggle verbose mode |
| `Ctrl+C` | Cancel operation |

### Thinking Keywords

| Keyword | Tokens | Use For |
|---------|--------|---------|
| `think` | ~4K | Simple planning |
| `think hard` | ~10K | Medium complexity |
| `ultrathink` | ~32K | Architecture decisions |

### File Locations

| File | Purpose | Git Status |
|------|---------|------------|
| `CLAUDE.md` | Project context | Committed |
| `.mcp.json` | XcodeBuildMCP config | Committed |
| `.claude/settings.json` | Team settings | Committed |
| `.claude/settings.local.json` | Personal settings | Gitignored |
| `docs/PRD.md` | Requirements | Committed |
| `docs/specs/*.md` | Feature specs | Committed |
| `docs/tasks/*.md` | Task tracking | Committed |

---

## Getting Started Checklist

### For the Developer Setting Up (One-Time)

- [ ] Install Claude Code CLI
- [ ] Create `.claude/` directory structure
- [ ] Create `CLAUDE.md`
- [ ] Create `.mcp.json`
- [ ] Create `.claude/settings.json`
- [ ] Create slash commands
- [ ] Create `docs/` structure
- [ ] Update `.gitignore`
- [ ] Commit and push

### For Both Developers (Each Machine)

- [ ] Install Claude Code CLI
- [ ] Install Node.js
- [ ] Pull the configured project
- [ ] Copy settings.local.json.template to settings.local.json
- [ ] Add your DEVELOPMENT_TEAM ID
- [ ] Test with `/build`
- [ ] Verify XcodeBuildMCP works

### Starting Your Own Feature

- [ ] Create your feature branch from develop
- [ ] Tell Claude about your feature and your teammate's off-limits directories
- [ ] Create your spec in `docs/specs/`
- [ ] Create your task file in `docs/tasks/`
- [ ] Update CLAUDE.md with your active feature info
- [ ] Identify shared code needs and coordinate with teammate
- [ ] Work independently in your feature directory
- [ ] Pull from develop regularly
- [ ] Create PR when complete

### Coordination Points (Both Developers)

- [ ] Weekly sync: Discuss upcoming shared code needs
- [ ] Before touching Core/: Message your teammate
- [ ] After adding shared code: Update CLAUDE.md and commit
- [ ] Before PR: Pull develop, run all tests, check for conflicts

---

*Happy coding together! This workflow scales well and keeps both developers productive with Claude Code.*
