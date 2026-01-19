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

### The Core Methodology

```
┌─────────────────────────────────────────────────────────────────┐
│                      PRD-DRIVEN WORKFLOW                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   1. PRD.md ──────► 2. Feature Specs ──────► 3. Task Files     │
│   (What to build)    (How to build)          (Step-by-step)    │
│                                                                 │
│                            │                                    │
│                            ▼                                    │
│                                                                 │
│   Developer A: Works on Task 1-3    Developer B: Works on 4-6  │
│   ──────────────────────────────    ────────────────────────── │
│   Claude assists with:              Claude assists with:        │
│   • Code generation                 • Code generation           │
│   • Building/testing                • Building/testing          │
│   • Debugging                       • Debugging                 │
│                                                                 │
│                            │                                    │
│                            ▼                                    │
│                                                                 │
│   4. Merge ──────► 5. Integration Tests ──────► 6. Ship        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

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

## Collaboration Best Practices

### 1. Use Task Files to Divide Work

Create `docs/tasks/[feature]-tasks.md` with clear ownership:

```markdown
# Tasks: User Authentication

## Progress
- Completed: 0/6
- Developer A: Steps 1-3
- Developer B: Steps 4-6

## Steps

### Step 1: Create AuthService [Developer A]
- [x] Create AuthService protocol
- [x] Implement mock AuthService for testing
- [ ] Implement real AuthService with API calls
**Status**: In Progress

### Step 2: Create LoginViewModel [Developer A]
- [ ] Create @Observable LoginViewModel
- [ ] Add validation logic
- [ ] Write unit tests
**Status**: Not Started

### Step 3: Create LoginView [Developer A]
- [ ] Create SwiftUI LoginView
- [ ] Connect to LoginViewModel
- [ ] Add error handling UI
**Status**: Not Started

### Step 4: Create SignupViewModel [Developer B]
- [ ] ...
**Status**: Not Started

[Continue...]
```

### 2. Branch Strategy

```bash
# Feature branches for each developer
git checkout -b feature/auth-login-developer-a
git checkout -b feature/auth-signup-developer-b

# Merge to develop regularly
git checkout develop
git merge feature/auth-login-developer-a
```

### 3. Communication Points

Tell Claude about your teammate:

```
You: "I'm Developer A. Developer B is working on the signup flow in
     Features/Signup/. Don't modify any files there."
```

### 4. Sync Points

Establish sync points in your workflow:

```
Morning Sync:
├── Both developers pull latest
├── Review task file for conflicts
└── Assign tasks for the day

Mid-day Check:
├── Push completed work
├── Pull partner's changes
└── Resolve any integration issues

End of Day:
├── Commit all work in progress
├── Update task file with status
└── Push to feature branch
```

### 5. Shared Context Updates

When you make architectural decisions, update the shared docs:

```
You: "Update CLAUDE.md to note that we're using Keychain for
     credential storage, not UserDefaults"

Claude: [Updates CLAUDE.md]
```

This ensures both developers' Claude sessions know about the decision.

---

## Critical Things to Watch Out For

### 1. Merge Conflicts in Generated Code

**Problem**: Both developers' Claude sessions might generate similar code differently.

**Solution**:
- Divide features into non-overlapping modules
- Use task files to assign clear ownership
- Pull frequently and resolve conflicts early

### 2. Inconsistent Code Style

**Problem**: Without alignment, Claude might generate code in different styles.

**Solution**: Be specific in `CLAUDE.md`:

```markdown
## Code Style Requirements
- Indent: 4 spaces (NOT tabs)
- Braces: Same line for functions
- Max line length: 120 characters
- Always use explicit `self` in closures
- Name booleans with `is`/`has`/`should` prefix
```

### 3. Context Drift

**Problem**: Long Claude sessions can lose track of earlier decisions.

**Solution**:
- Use `/compact` regularly to summarize context
- Use `/clear` between unrelated tasks
- Keep `CLAUDE.md` updated with key decisions
- Start fresh sessions for new features

```
You: "/compact"
Claude: [Summarizes conversation, reduces context]

You: "/clear"
Claude: [Clears context, starts fresh]
```

### 4. Accidentally Modifying Partner's Code

**Problem**: Claude might edit files your partner is working on.

**Solution**: Add to your session:

```
You: "DO NOT modify any files in Features/Signup/ - my partner is
     working there. Only work in Features/Login/"
```

Or use permissions in `.claude/settings.json`:

```json
{
  "permissions": {
    "deny": [
      "Write(Features/Signup/*)",
      "Edit(Features/Signup/*)"
    ]
  }
}
```

### 5. Different XcodeBuildMCP Versions

**Problem**: Different MCP versions might behave differently.

**Solution**: Pin the version in `.mcp.json`:

```json
{
  "mcpServers": {
    "XcodeBuildMCP": {
      "command": "npx",
      "args": ["-y", "xcodebuildmcp@1.2.3"]
    }
  }
}
```

### 6. Secrets in Code

**Problem**: Claude might accidentally expose or commit secrets.

**Solution**: Always deny access to sensitive files:

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

### 7. Breaking Changes Without Communication

**Problem**: One developer's changes break another's code.

**Solution**:
- Update `docs/ARCHITECTURE.md` when making breaking changes
- Add notes to task files
- Use feature flags for in-progress work

---

## Complete Feature Implementation Example

Let's walk through implementing a **"User Profile"** feature as a 2-developer team.

### Phase 1: Planning (Developer A Leads)

**Developer A starts Claude in Plan Mode:**

```bash
claude --permission-mode plan
```

**Developer A's Session:**

```
You: "Ultrathink about implementing a User Profile feature for our app.
     Users need to:
     - View their profile information
     - Edit their name and avatar
     - Change notification preferences

     Read our codebase structure first, then create a comprehensive plan."

Claude: [Extended thinking... analyzes codebase]
        [Creates detailed implementation plan]
        [Identifies dependencies]

You: "Create a feature spec at docs/specs/user-profile.md"

Claude: [Creates comprehensive spec]
```

**The spec Claude creates (`docs/specs/user-profile.md`):**

```markdown
# Feature Specification: User Profile

**Status**: Approved
**Priority**: P1
**Author**: Developer A
**Last Updated**: 2026-01-19

## Overview
Allow users to view and edit their profile information.

## User Stories
1. As a user, I want to view my profile so I can see my account info
2. As a user, I want to edit my name so I can correct mistakes
3. As a user, I want to change my avatar so I can personalize my account
4. As a user, I want to manage notifications so I control what alerts I get

## Acceptance Criteria
- [ ] Profile screen displays name, email, avatar
- [ ] Name can be edited with validation (2-50 chars)
- [ ] Avatar can be selected from library or camera
- [ ] Notification toggles persist and affect push behavior
- [ ] All changes sync to backend
- [ ] Offline support with sync on reconnect

## Technical Design

### Data Models
```swift
struct UserProfile: Codable, Identifiable {
    let id: UUID
    var name: String
    var email: String
    var avatarURL: URL?
    var notificationPreferences: NotificationPreferences
}

struct NotificationPreferences: Codable {
    var pushEnabled: Bool
    var emailEnabled: Bool
    var weeklyDigest: Bool
}
```

### Architecture
- ProfileViewModel (@Observable) manages state
- ProfileService protocol for API calls
- Uses existing NetworkClient from Core/Networking

## Dependencies
- Core/Networking module
- Core/ImagePicker utility
- SwiftData for offline cache
```

**Developer A then creates tasks:**

```
You: "Create a task breakdown at docs/tasks/user-profile-tasks.md.
     Split tasks between two developers - assign UI work to Developer A
     and backend/data work to Developer B."

Claude: [Creates task file]
```

**The task file Claude creates (`docs/tasks/user-profile-tasks.md`):**

```markdown
# Tasks: User Profile

**Feature Spec**: docs/specs/user-profile.md
**Status**: In Progress

## Progress Summary
- Total Steps: 8
- Completed: 0
- Developer A (UI): Steps 1, 2, 5, 7
- Developer B (Data): Steps 3, 4, 6, 8

---

## Developer A Tasks (UI)

### Step 1: Create ProfileView skeleton [Developer A]
- [ ] Create Features/Profile/ directory structure
- [ ] Create ProfileView.swift with basic layout
- [ ] Add placeholder content
- [ ] Add to navigation
**Status**: Not Started
**Estimated**: Small

### Step 2: Create ProfileViewModel [Developer A]
- [ ] Create ProfileViewModel.swift as @Observable
- [ ] Add published properties for profile data
- [ ] Add validation methods
- [ ] Write unit tests for validation
**Status**: Not Started
**Estimated**: Medium
**Depends on**: Step 3 (ProfileService protocol)

### Step 5: Implement ProfileView UI [Developer A]
- [ ] Build complete profile display UI
- [ ] Add edit mode toggle
- [ ] Implement form validation feedback
- [ ] Add avatar picker integration
**Status**: Not Started
**Estimated**: Large
**Depends on**: Steps 2, 4

### Step 7: Add notification preferences UI [Developer A]
- [ ] Create NotificationPreferencesView
- [ ] Add toggle controls
- [ ] Connect to ViewModel
**Status**: Not Started
**Estimated**: Small

---

## Developer B Tasks (Data/Backend)

### Step 3: Create ProfileService [Developer B]
- [ ] Create ProfileService protocol
- [ ] Create MockProfileService for testing
- [ ] Create ProfileAPIService implementation
- [ ] Write unit tests
**Status**: Not Started
**Estimated**: Medium

### Step 4: Implement API integration [Developer B]
- [ ] Add profile endpoints to NetworkClient
- [ ] Implement fetch profile
- [ ] Implement update profile
- [ ] Add error handling
**Status**: Not Started
**Estimated**: Medium

### Step 6: Add offline support [Developer B]
- [ ] Create SwiftData model for profile cache
- [ ] Implement cache-first loading
- [ ] Add sync on reconnect
- [ ] Write integration tests
**Status**: Not Started
**Estimated**: Large

### Step 8: Integration testing [Both]
- [ ] Full feature integration test
- [ ] Edge case testing
- [ ] Performance testing
**Status**: Not Started
**Estimated**: Medium

---

## Changes Log
| Date | Step | Developer | Changes |
|------|------|-----------|---------|
| 2026-01-19 | - | A | Initial task breakdown created |
```

**Developer A commits and pushes:**

```
You: "Commit the spec and task files"

Claude: [Commits: "Add User Profile feature spec and task breakdown"]

You: "Push to the feature branch"

Claude: [Pushes to feature/user-profile]
```

### Phase 2: Parallel Development

**Developer B pulls and starts their work:**

```bash
git pull
git checkout feature/user-profile
claude
```

**Developer B's Session:**

```
You: "I'm Developer B. Read the user profile task file and start on
     Step 3: Create ProfileService."

Claude: [Reads docs/tasks/user-profile-tasks.md]
        "I'll create the ProfileService. Let me start with the protocol
         and mock implementation for testing."

        [Creates Features/Profile/Services/ProfileService.swift]
        [Creates Features/Profile/Services/MockProfileService.swift]
        [Creates Features/Profile/Services/ProfileAPIService.swift]
        [Creates ProfileServiceTests.swift]

You: "/build"

Claude: [Builds successfully]

You: "/test"

Claude: [Runs tests - all pass]

You: "Update the task file to mark Step 3 complete"

Claude: [Updates docs/tasks/user-profile-tasks.md]
        Changed Step 3 status to Complete
        Added to Changes Log
```

**Meanwhile, Developer A works on Step 1:**

```
You: "I'm Developer A. Read the task file and start Step 1."

Claude: [Reads task file]
        "I'll create the ProfileView skeleton. First, let me create
         the directory structure."

        [Creates Features/Profile/Views/ProfileView.swift]
        [Creates Features/Profile/ViewModels/]
        [Adds to app navigation]

You: "/build"

Claude: [Builds successfully]
```

### Phase 3: Dependent Tasks

**Developer A needs to wait for Developer B's work:**

```
You: "Can I start Step 2? It depends on Step 3."

Claude: [Reads task file]
        "Step 3 is now marked complete by Developer B. You can proceed
         with Step 2: Create ProfileViewModel."
```

**Developer A pulls Developer B's changes:**

```
You: "Pull latest changes first"

Claude: [Runs git pull]
        "Pulled Developer B's ProfileService implementation. No conflicts."

You: "Now implement Step 2"

Claude: [Creates ProfileViewModel using the ProfileService protocol]
        [Creates ProfileViewModelTests.swift]
```

### Phase 4: Integration

**Both developers sync up for Step 8:**

```markdown
# In docs/tasks/user-profile-tasks.md

### Step 8: Integration testing [Both]
- [x] Full feature integration test
- [x] Edge case testing
- [x] Performance testing
**Status**: Complete
**Notes**:
  - Developer A wrote UI integration tests
  - Developer B wrote API integration tests
  - All 47 tests passing
```

### Phase 5: Final Review

**Either developer can do the final review:**

```
You: "Use Plan Mode to review the entire User Profile feature
     implementation before we merge."

Claude: [Switches to Plan Mode]
        [Reviews all files in Features/Profile/]
        [Checks test coverage]
        [Verifies against acceptance criteria]

        "Review complete:
         ✅ All acceptance criteria met
         ✅ 47 tests passing (92% coverage)
         ✅ No SwiftLint warnings
         ✅ Follows project architecture

         Recommendations:
         - Add accessibility labels to avatar picker
         - Consider adding loading state to save button"

You: "Implement those recommendations"

Claude: [Switches to Normal Mode]
        [Adds accessibility labels]
        [Adds loading state]
```

### Phase 6: Merge to Main

```
You: "Create a pull request for this feature"

Claude: [Creates PR with summary of all changes]
        "PR #42 created: Add User Profile feature
         - 8 tasks completed
         - 47 tests added
         - 12 new files"
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

### Starting a New Feature (Together)

- [ ] Discuss feature requirements
- [ ] One developer creates PRD/spec in Plan Mode
- [ ] Create task breakdown with ownership
- [ ] Divide tasks by module/layer
- [ ] Work in parallel on separate branches
- [ ] Sync and integrate regularly
- [ ] Review together before merge

---

*Happy coding together! This workflow scales well and keeps both developers productive with Claude Code.*
