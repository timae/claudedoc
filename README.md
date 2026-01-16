# Claude Code Engineering Documentation

> **Version:** 1.0
> **Last Updated:** January 2026
> **Audience:** Engineering Teams

---

## Table of Contents

1. [What is Claude Code?](#what-is-claude-code)
2. [Installation Guide](#installation-guide)
3. [Agents vs Skills](#agents-vs-skills)
4. [Building Project-Specific CLAUDE.md Files](#building-project-specific-claudemd-files)
5. [Security and Confidence Guidelines](#security-and-confidence-guidelines)
6. [Real-World Workflow Examples](#real-world-workflow-examples)
7. [Dos and Don'ts](#dos-and-donts)
8. [Quick Reference](#quick-reference)

---

## What is Claude Code?

Claude Code is Anthropic's official command-line interface (CLI) tool that brings Claude's AI capabilities directly into your development environment. It operates as an **agentic coding assistant** that can read, write, and execute code while respecting security boundaries you define.

### Core Capabilities

| Capability | Description |
|------------|-------------|
| **Code Generation** | Write new code, functions, modules, and complete features |
| **Code Modification** | Edit existing files with context-aware changes |
| **Code Analysis** | Understand codebases, find bugs, explain logic |
| **Command Execution** | Run shell commands, tests, build processes |
| **Git Operations** | Commits, branches, PRs, rebases, history analysis |
| **Multi-file Refactoring** | Coordinate changes across multiple files |

### How It Works

```
┌─────────────────────────────────────────────────────────────┐
│                      Claude Code CLI                        │
├─────────────────────────────────────────────────────────────┤
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐       │
│  │   CLAUDE.md  │  │    Tools     │  │  Permissions │       │
│  │   (Context)  │  │  (Actions)   │  │   (Safety)   │       │
│  └──────────────┘  └──────────────┘  └──────────────┘       │
│           │                │                 │              │
│           └────────────────┼─────────────────┘              │
│                            ▼                                │
│                 ┌──────────────────┐                        │
│                 │   Claude Model   │                        │
│                 │   (Reasoning)    │                        │
│                 └──────────────────┘                        │
└─────────────────────────────────────────────────────────────┘
```

### Key Concepts

- **CLAUDE.md Files**: Configuration files that provide project context, constraints, and workflows
- **Tools**: Actions Claude can perform (Read, Write, Edit, Bash, Glob, Grep, etc.)
- **Permissions**: Safety controls determining what Claude can do without asking
- **Subagents**: Specialized AI instances for specific tasks (testing, reviewing, etc.)
- **Skills**: Reusable slash commands that encode common workflows

---

## Installation Guide

### Prerequisites

| Requirement | macOS | Linux |
|-------------|-------|-------|
| **OS Version** | macOS 10.15+ | Ubuntu 20.04+ / Debian 10+ |
| **Shell** | zsh or bash | bash or zsh |
| **Internet** | Required for installation | Required for installation |

> **Note:** The native installer does NOT require Node.js. Only the legacy npm method requires Node.js 18+.

### Installation on macOS

#### Method 1: Native Binary (Recommended)

```bash
# Download and install Claude Code
curl -fsSL https://claude.ai/install.sh | bash

# Reload shell configuration
source ~/.zshrc  # or source ~/.bashrc if using bash

# Verify installation
claude --version

# Run diagnostics
claude doctor
```

#### Method 2: Homebrew (Alternative)

```bash
# Install via Homebrew
brew install claude-code

# Verify installation
claude --version
```

### Installation on Linux

#### Native Binary Installation

```bash
# Download and install Claude Code
curl -fsSL https://claude.ai/install.sh | bash

# Add to PATH if not automatically added
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc

# Verify installation
claude --version

# Run diagnostics
claude doctor
```

#### Alpine Linux / musl-based Distributions

```bash
# Install required dependencies first
apk add libgcc libstdc++ ripgrep

# Then run the standard installer
curl -fsSL https://claude.ai/install.sh | bash
```

### Authentication Setup

```bash
# Start Claude Code (will prompt for authentication)
claude

# Or authenticate with API key directly
export ANTHROPIC_API_KEY="your-api-key-here"
claude
```

### Post-Installation Verification

```bash
# Check installation health
claude doctor

# Expected output should show:
# ✓ Claude Code version: x.x.x
# ✓ Installation type: native
# ✓ Authentication: configured
# ✓ Permissions: default
```

### Migrating from npm Installation

If you previously installed via npm:

```bash
# Uninstall npm version
npm uninstall -g @anthropic-ai/claude-code

# Install native version
claude install

# Verify migration
claude doctor
```

### Troubleshooting

| Issue | Solution |
|-------|----------|
| `command not found: claude` | Add PATH: `echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc && source ~/.bashrc` |
| Permission denied | Do NOT use sudo. Run as regular user. |
| SSL/Network errors | Check firewall settings; Claude requires HTTPS access |
| Old version persisting | Remove npm version first, then reinstall native |

---

## Agents vs Skills

Understanding the difference between **agents** (subagents) and **skills** is crucial for effective Claude Code usage.

### Conceptual Overview

```
┌─────────────────────────────────────────────────────────────┐
│                     AGENTS (Subagents)                      │
│  • Autonomous task executors                                │
│  • Independent context windows                              │
│  • Domain-specific expertise                                │
│  • Spawned for complex, multi-step tasks                    │
└─────────────────────────────────────────────────────────────┘
                            vs
┌─────────────────────────────────────────────────────────────┐
│                         SKILLS                              │
│  • Reusable prompt templates                                │
│  • Slash command shortcuts                                  │
│  • Parameterized workflows                                  │
│  • Invoked in current context                               │
└─────────────────────────────────────────────────────────────┘
```

### Comparison Table

| Aspect | Agents (Subagents) | Skills |
|--------|-------------------|--------|
| **Purpose** | Execute complex autonomous tasks | Quick reusable commands |
| **Context** | Isolated context window | Shared with main session |
| **Invocation** | Task tool with subagent_type | Slash command (`/skill-name`) |
| **Complexity** | Multi-step reasoning | Single prompt template |
| **State** | Independent, returns result | Inline execution |
| **Tool Access** | Configurable per agent | Inherits session permissions |
| **Use Case** | Research, code review, testing | Formatting, scaffolding, commits |

### Agent Types and Tool Assignments

| Agent Type | Tools Available | Use Case |
|------------|-----------------|----------|
| **Read-only** | Read, Grep, Glob | Code review, security audit |
| **Research** | Read, Grep, Glob, WebFetch, WebSearch | Documentation lookup, API research |
| **Code Writer** | Read, Write, Edit, Bash, Glob, Grep | Feature implementation, bug fixes |
| **Documentation** | Read, Write, Edit, Glob, Grep, WebFetch | Doc generation, README updates |

### Agent Configuration Example

Create agents in `.claude/agents/` directory:

```yaml
# .claude/agents/code-reviewer.md
---
name: code-reviewer
description: Reviews code for quality, security, and best practices
tools: Read, Grep, Glob
---

You are a senior code reviewer. Your responsibilities:

## Review Checklist
- [ ] Check for security vulnerabilities (injection, XSS, etc.)
- [ ] Verify error handling coverage
- [ ] Assess code readability and maintainability
- [ ] Validate naming conventions
- [ ] Check for code duplication

## Output Format
Provide findings as:
1. **Critical Issues** (must fix)
2. **Warnings** (should fix)
3. **Suggestions** (nice to have)

Always cite specific file:line references.
```

```yaml
# .claude/agents/test-writer.md
---
name: test-writer
description: Generates comprehensive test suites
tools: Read, Write, Edit, Bash, Glob, Grep
---

You are a test engineering specialist. When writing tests:

## Guidelines
- Use the project's existing test framework
- Cover happy path, edge cases, and error conditions
- Follow AAA pattern (Arrange, Act, Assert)
- Mock external dependencies appropriately
- Aim for meaningful coverage, not percentage targets

## Before Writing
1. Read existing tests to understand patterns
2. Identify the function/component under test
3. List test scenarios

## After Writing
1. Run the test suite
2. Verify tests pass
3. Check coverage if available
```

### Skill Configuration Example

Create skills in `.claude/commands/` directory:

```markdown
# .claude/commands/commit.md
Generate a conventional commit message for staged changes.

Analyze the git diff and create a message following:
- type(scope): description
- Types: feat, fix, docs, style, refactor, test, chore
- Keep description under 72 characters
- Add body if changes are complex

Run: git add -A && git commit
```

```markdown
# .claude/commands/scaffold.md
# Scaffold: $ARGUMENTS

Create a new $ARGUMENTS with:
1. Appropriate file structure
2. Basic boilerplate code
3. Test file stub
4. Type definitions if applicable

Follow existing project patterns and conventions.
```

```markdown
# .claude/commands/review-pr.md
# Review PR: $ARGUMENTS

Fetch and review pull request #$ARGUMENTS:

1. Get PR diff and description using gh CLI
2. Analyze changes for:
   - Correctness
   - Security issues
   - Performance implications
   - Test coverage
3. Provide structured feedback
4. Suggest improvements if needed
```

### Using Agents and Skills

```bash
# Invoke a skill (slash command)
/commit
/scaffold UserService
/review-pr 123

# Agents are invoked automatically by Claude when appropriate
# Or explicitly request: "Use the code-reviewer agent to review auth.ts"
```

### Directory Structure

```
your-project/
├── .claude/
│   ├── agents/
│   │   ├── code-reviewer.md
│   │   ├── test-writer.md
│   │   ├── security-auditor.md
│   │   └── doc-generator.md
│   ├── commands/
│   │   ├── commit.md
│   │   ├── scaffold.md
│   │   ├── review-pr.md
│   │   └── deploy-check.md
│   └── settings.json
├── CLAUDE.md
└── ... (project files)
```

---

## Building Project-Specific CLAUDE.md Files

The `CLAUDE.md` file is your primary mechanism for providing Claude with project context. It's loaded automatically at the start of every session.

### File Locations and Hierarchy

| Location | Scope | Use Case |
|----------|-------|----------|
| `~/.claude/CLAUDE.md` | Global (all projects) | Personal preferences, global conventions |
| `../CLAUDE.md` | Parent directory | Monorepo root configuration |
| `./CLAUDE.md` | Project root | Project-specific context |
| `./subdir/CLAUDE.md` | Subdirectory | Module-specific overrides |

Files are loaded in order; later files can override earlier ones.

### Content Framework: WHAT, WHY, HOW

Structure your CLAUDE.md around three dimensions:

```markdown
# Project: [Name]

## WHAT (Tech Stack & Structure)
- Technologies, frameworks, languages
- Project structure and key directories
- Important files and their purposes

## WHY (Purpose & Context)
- What this project does
- Business domain knowledge
- Key abstractions and patterns

## HOW (Workflows & Commands)
- Build and run commands
- Test execution
- Deployment process
- Common development tasks
```

### Guidelines for Effective CLAUDE.md Files

| Do | Don't |
|-----|-------|
| Keep under 300 lines | Create massive documentation dumps |
| Focus on universally applicable info | Include task-specific guidance |
| Use file:line pointers | Copy large code blocks |
| Reference external docs | Duplicate linter rules |
| Update manually and thoughtfully | Auto-generate content |

### Template: Standard Project

```markdown
# Project: MyApp

## Overview
MyApp is a [brief description]. Built with [stack].

## Tech Stack
- **Backend:** Node.js 20, Express, TypeScript
- **Database:** PostgreSQL 15, Prisma ORM
- **Frontend:** React 18, Vite, TailwindCSS
- **Testing:** Vitest, Playwright

## Project Structure
```src/
├── api/          # REST endpoints
├── services/     # Business logic
├── models/       # Prisma models
├── utils/        # Shared utilities
└── types/        # TypeScript types
```

## Commands
```bash
# Development
npm run dev          # Start dev server (port 3000)
npm run db:migrate   # Run Prisma migrations
npm run db:seed      # Seed development data

# Testing
npm test             # Run unit tests
npm run test:e2e     # Run Playwright tests
npm run test:cov     # Generate coverage report

# Build
npm run build        # Production build
npm run typecheck    # TypeScript validation
```

## Conventions
- Use named exports, not default exports
- Error handling: throw typed errors from `src/errors/`
- API responses follow `{ data, error, meta }` structure
- All endpoints require authentication except `/health` and `/auth/*`

## Key Files
- `src/api/routes.ts` - Route definitions
- `src/services/auth.ts` - Authentication logic
- `prisma/schema.prisma` - Database schema
- `.env.example` - Required environment variables

## External Docs
- API docs: `docs/api.md`
- Architecture: `docs/architecture.md`
- Deployment: `docs/deploy.md`
```

### Template: Monorepo

```markdown
# Monorepo: Platform

## Structure
```packages/
├── api/           # Backend service (NestJS)
├── web/           # Customer portal (Next.js)
├── admin/         # Admin dashboard (React)
├── shared/        # Shared types and utilities
└── config/        # Shared configuration
```

## Package-Specific Commands
Each package has its own README. Key commands:

| Package | Dev | Test | Build |
|---------|-----|------|-------|
| api | `npm run dev -w api` | `npm test -w api` | `npm run build -w api` |
| web | `npm run dev -w web` | `npm test -w web` | `npm run build -w web` |

## Shared Dependencies
- Changes to `packages/shared` require rebuilding dependents
- Run `npm run build:shared` before testing dependent packages

## Cross-Package Imports
```typescript
// From any package, import shared types:
import { User, ApiResponse } from '@platform/shared';
```

## CI/CD
- PRs trigger affected package tests only
- Main branch deploys to staging
- Tagged releases deploy to production
```

### Template: Library/SDK

```markdown
# Library: utils-sdk

## Purpose
Shared utility library for [company] projects.

## Publishing
- Version follows semver
- Changelog in CHANGELOG.md
- Publish: `npm run release`

## Development
```bash
npm install          # Install dependencies
npm run build        # Compile TypeScript
npm test             # Run tests (100% coverage required)
npm run docs         # Generate API docs
```

## API Design Principles
- All public functions must have JSDoc comments
- Prefer pure functions over stateful classes
- Every export must have corresponding test
- No external runtime dependencies

## File Organization
- `src/[category]/[function].ts` - Implementation
- `src/[category]/[function].test.ts` - Co-located tests
- `src/[category]/index.ts` - Category exports
- `src/index.ts` - Public API surface
```

### Progressive Disclosure Pattern

Instead of cramming everything into CLAUDE.md, use references:

```markdown
# Project: ComplexApp

## Quick Reference
[Core commands and conventions here - keep brief]

## Deep Dives
For detailed information, read these files:
- Architecture decisions: `docs/architecture.md`
- API conventions: `docs/api-guidelines.md`
- Testing strategy: `docs/testing.md`
- Security policies: `docs/security.md`

When working on specific areas, read the relevant doc first.
```

---

## Security and Confidence Guidelines

### Security Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    SECURITY LAYERS                          │
├─────────────────────────────────────────────────────────────┤
│  Layer 1: Permission System                                 │
│  ├── Default: Read-only                                     │
│  ├── Explicit approval for writes/executes                  │
│  └── Allowlist for trusted commands                         │
├─────────────────────────────────────────────────────────────┤
│  Layer 2: Sandboxing                                        │
│  ├── Filesystem isolation                                   │
│  ├── Network isolation                                      │
│  └── OS-level enforcement (bubblewrap/seatbelt)             │
├─────────────────────────────────────────────────────────────┤
│  Layer 3: Input Sanitization                                │
│  ├── Command injection prevention                           │
│  ├── Prompt injection detection                             │
│  └── Suspicious command flagging                            │
├─────────────────────────────────────────────────────────────┤
│  Layer 4: Operational Controls                              │
│  ├── Data retention limits                                  │
│  ├── Audit logging                                          │
│  └── Environment isolation                                  │
└─────────────────────────────────────────────────────────────┘
```

### Data Leak Prevention Policy

#### High-Risk Files - NEVER Allow Access

| File Pattern | Risk | Mitigation |
|--------------|------|------------|
| `.env`, `.env.*` | API keys, secrets | Use placeholder values in code |
| `*.pem`, `*.key` | Private keys | Store outside project directory |
| `credentials.json` | Service accounts | Use secret managers |
| `~/.ssh/*` | SSH keys | Filesystem isolation |
| `~/.aws/*` | AWS credentials | Use IAM roles instead |
| `*.secrets.*` | Various secrets | Exclude from project |

#### Permission Configuration

Create `.claude/settings.json`:

```json
{
  "permissions": {
    "deny": [
      "Bash(cat ~/.aws/*)",
      "Bash(cat ~/.ssh/*)",
      "Bash(*credentials*)",
      "Bash(curl*)",
      "Bash(wget*)",
      "Bash(nc *)",
      "Read(~/.aws/*)",
      "Read(~/.ssh/*)",
      "Read(**/.env)",
      "Read(**/.env.*)",
      "Read(**/secrets.*)"
    ],
    "ask": [
      "Bash(npm publish*)",
      "Bash(git push*)",
      "Bash(docker push*)",
      "Write(**/*.sql)",
      "Edit(**/*.sql)"
    ],
    "allow": [
      "Read",
      "Glob",
      "Grep",
      "Bash(npm test*)",
      "Bash(npm run build*)",
      "Bash(git status)",
      "Bash(git diff*)",
      "Bash(git log*)"
    ]
  }
}
```

### Sandboxing Configuration

Enable sandboxing for maximum isolation:

```bash
# Enable sandbox mode in session
/sandbox

# Or configure in settings.json
{
  "sandbox": {
    "enabled": true,
    "filesystem": {
      "allowedPaths": [
        "${workspaceFolder}",
        "/tmp/claude-*"
      ],
      "deniedPaths": [
        "${HOME}/.ssh",
        "${HOME}/.aws",
        "${HOME}/.config"
      ]
    },
    "network": {
      "enabled": false,
      "allowedDomains": []
    }
  }
}
```

### Environment-Specific Guidelines

#### Development Environment

```markdown
## Security: Development

### Allowed
- Read/write project files
- Run tests and builds
- Access local databases
- Git operations (except force push)

### Requires Approval
- Installing new dependencies
- Running arbitrary scripts
- Network requests to external APIs
- Database migrations

### Denied
- Access to credential files
- Production API endpoints
- Deployment commands
```

#### CI/CD Environment

```bash
# Use headless mode with strict permissions
claude -p "Run tests and report failures" \
  --allowedTools "Read,Grep,Glob,Bash(npm test*)" \
  --output-format json
```

#### Production-Adjacent Work

```markdown
## Security: Production Access

⚠️ CRITICAL: When working with production data:

1. NEVER use real credentials - use read replicas or sanitized data
2. NEVER commit connection strings or API keys
3. ALWAYS use placeholder values in examples
4. ALWAYS verify commands before approval
5. PREFER read-only operations
```

### Operational Security Checklist

```markdown
## Pre-Session Checklist

- [ ] Verify working directory is correct
- [ ] Check `.env` files are in `.gitignore`
- [ ] Confirm no secrets in CLAUDE.md
- [ ] Review permission settings

## During Session

- [ ] Review all commands before approval
- [ ] Don't pipe untrusted content directly
- [ ] Verify changes to critical files
- [ ] Monitor for unusual file access patterns

## Post-Session

- [ ] Review git diff before committing
- [ ] Check no secrets were accidentally added
- [ ] Clear sensitive context if needed (/clear)
```

### Confidence Levels for Operations

| Confidence | Operation Type | Approval |
|------------|----------------|----------|
| **High** | Read project files | Auto-allow |
| **High** | Run defined test commands | Auto-allow |
| **High** | Git status/diff/log | Auto-allow |
| **Medium** | Write to project files | Ask once |
| **Medium** | Install dependencies | Ask each time |
| **Low** | Run arbitrary scripts | Always ask |
| **Low** | Network operations | Always ask |
| **Critical** | Production operations | Manual review required |

### MCP Server Security

```json
{
  "mcpServers": {
    "trusted-memory": {
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-memory"],
      "trust": "verified"
    },
    "github": {
      "command": "gh",
      "args": ["mcp"],
      "trust": "verified"
    }
  }
}
```

**MCP Security Rules:**
- Only use MCP servers you've written or from trusted providers
- Anthropic does not audit third-party MCP servers
- Review MCP server code before enabling
- Configure permissions per-server

---

## Real-World Workflow Examples

### Workflow 1: Feature Development (Explore → Plan → Code → Commit)

```bash
# Step 1: Explore existing code
> claude
You: I need to add user profile picture upload. First, explore how file
     uploads are currently handled in this codebase.

# Claude reads relevant files, identifies patterns

# Step 2: Create a plan
You: Create a detailed implementation plan. Use extended thinking.

# Claude outlines:
# - Storage service changes
# - API endpoint additions
# - Frontend component updates
# - Migration requirements

# Step 3: Implement with approval
You: Implement step 1 of the plan - the storage service changes.

# Review Claude's changes before proceeding

# Step 4: Test and commit
You: /commit  # Use the commit skill
```

### Workflow 2: Test-Driven Development

```bash
# Step 1: Write failing tests first
You: Write tests for a new validateEmail function that should:
     - Accept valid email formats
     - Reject invalid formats
     - Handle edge cases (empty string, null, unicode)

     Don't implement the function yet.

# Step 2: Verify tests fail
You: Run the tests to confirm they fail.

# Step 3: Implement to pass
You: Now implement validateEmail to pass all tests.

# Step 4: Iterate until green
You: Run tests again. Fix any failures.
```

### Workflow 3: Bug Investigation and Fix

```bash
# Step 1: Describe the bug
You: Users report that login fails intermittently with a 500 error.
     The error appears in logs as "session store connection timeout".
     Investigate the root cause.

# Claude searches logs, traces code paths, identifies issue

# Step 2: Propose solutions
You: What are the options to fix this? Consider tradeoffs.

# Claude presents options with pros/cons

# Step 3: Implement chosen solution
You: Implement option 2 (connection pooling) with retry logic.

# Step 4: Add regression test
You: Add a test that would catch this issue if it regresses.
```

### Workflow 4: Code Review Automation

```bash
# Create a review workflow
> /review-pr 456

# Or manually:
You: Review PR #456 focusing on:
     - Security vulnerabilities
     - Performance implications
     - Test coverage
     - API compatibility

# Claude fetches PR, analyzes changes, provides structured feedback
```

### Workflow 5: Large Migration

```bash
# Step 1: Analyze scope
You: We need to migrate from Moment.js to date-fns.
     Find all usages and create a migration checklist.

# Claude creates markdown checklist:
# - [ ] src/utils/dates.ts (15 usages)
# - [ ] src/components/Calendar.tsx (8 usages)
# - [ ] ...

# Step 2: Migrate systematically
You: Migrate the first file on the checklist. Update the checklist
     when complete.

# Repeat for each file, with Claude tracking progress
```

### Workflow 6: Multi-Claude Parallel Development

```bash
# Terminal 1: Feature work
cd ~/project
git checkout -b feature-a
claude
You: Implement the new dashboard widget.

# Terminal 2: Bug fixes (separate worktree)
cd ~/project-worktree
git checkout -b bugfix-123
claude
You: Fix the pagination bug in the user list.

# Both Claude instances work independently
```

### Workflow 7: CI/CD Integration

```bash
#!/bin/bash
# .github/workflows/claude-review.yml

- name: AI Code Review
  run: |
    claude -p "Review the changes in this PR for security issues.
               Output findings as JSON." \
      --allowedTools "Read,Grep,Glob" \
      --output-format json > review-results.json

    # Parse results and post as PR comment
    cat review-results.json | jq '.findings' | gh pr comment $PR_NUMBER
```

### Workflow 8: Documentation Generation

```bash
You: Generate API documentation for all endpoints in src/api/.
     Follow the existing format in docs/api.md.
     Include request/response examples.

# Claude reads endpoints, generates consistent documentation

You: Now create a mermaid diagram showing the API flow for user
     registration.
```

---

## Dos and Don'ts

### General Usage

| Do | Don't |
|-----|-------|
| Be specific and detailed in prompts | Use vague requests like "make it better" |
| Provide context about your goals | Assume Claude knows your intentions |
| Review changes before accepting | Blindly accept all suggestions |
| Use `/clear` between unrelated tasks | Let context grow unbounded |
| Approve commands one-by-one initially | Use `--dangerously-skip-permissions` in production |

### CLAUDE.md Files

| Do | Don't |
|-----|-------|
| Keep under 300 lines | Create exhaustive documentation |
| Focus on universally applicable info | Include one-off task instructions |
| Use file:line references | Copy large code blocks |
| Update manually and thoughtfully | Auto-generate content |
| Reference external detailed docs | Duplicate information from other sources |
| Include common commands | List every possible command |

### Security

| Do | Don't |
|-----|-------|
| Use placeholder values for secrets | Put real credentials in code |
| Configure deny rules for sensitive paths | Trust default permissions blindly |
| Run in sandboxed environments | Disable sandbox in production |
| Review commands before approval | Pipe untrusted content to Claude |
| Use read-only access when possible | Grant write access unnecessarily |
| Audit permissions regularly | Set and forget permissions |

### Code Quality

| Do | Don't |
|-----|-------|
| Ask Claude to explain changes | Accept code you don't understand |
| Request tests with implementations | Skip testing new code |
| Follow existing project patterns | Let Claude introduce new patterns |
| Keep changes focused and minimal | Allow scope creep |
| Verify builds pass after changes | Commit without verification |

### Collaboration

| Do | Don't |
|-----|-------|
| Share `.claude/` configs in git | Keep configurations local only |
| Document custom skills for team | Create undocumented shortcuts |
| Establish team permission policies | Leave permissions to individual choice |
| Review generated code in PRs | Merge AI code without review |

### Performance

| Do | Don't |
|-----|-------|
| Use `/clear` to reset context | Keep growing context indefinitely |
| Be concise in prompts | Write essay-length requests |
| Use agents for complex searches | Run many sequential searches |
| Provide file paths directly | Make Claude search for known files |

---

## Quick Reference

### Essential Commands

```bash
# Start Claude Code
claude

# Headless mode (CI/CD)
claude -p "your prompt"

# Continue previous conversation
claude --continue

# Check installation
claude doctor

# View/edit permissions
/permissions

# Clear context
/clear

# Enable sandbox
/sandbox

# Get help
/help
```

### File Locations

```
~/.claude/
├── CLAUDE.md           # Global preferences
├── settings.json       # Global settings
└── credentials         # Auth tokens

./
├── CLAUDE.md           # Project context
└── .claude/
    ├── settings.json   # Project settings
    ├── agents/         # Subagent definitions
    │   └── *.md
    └── commands/       # Skill definitions
        └── *.md
```

### Permission Syntax

```json
{
  "permissions": {
    "allow": ["Tool(pattern)"],
    "ask": ["Tool(pattern)"],
    "deny": ["Tool(pattern)"]
  }
}
```

### Environment Variables

```bash
ANTHROPIC_API_KEY    # API authentication
CLAUDE_CODE_SANDBOX  # Enable sandbox (true/false)
CLAUDE_CODE_DEBUG    # Debug logging (true/false)
```

---

## Sources

This documentation was compiled from:

- [Anthropic Claude Code Best Practices](https://www.anthropic.com/engineering/claude-code-best-practices)
- [HumanLayer: Writing a Good CLAUDE.md](https://www.humanlayer.dev/blog/writing-a-good-claude-md)
- [VoltAgent Awesome Claude Code Subagents](https://github.com/VoltAgent/awesome-claude-code-subagents)
- [Claude Code Official Documentation](https://code.claude.com/docs/en/setup)
- [Claude Code Security Documentation](https://code.claude.com/docs/en/security)
- [Anthropic Engineering: Claude Code Sandboxing](https://www.anthropic.com/engineering/claude-code-sandboxing)

---

*Document maintained by Engineering Team. Last reviewed: January 2026*
