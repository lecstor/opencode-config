# OpenCode Agent Configuration

A multi-agent configuration for [OpenCode](https://opencode.ai) that defines a structured system of specialized AI agents, reusable skills, and custom commands for software development workflows. The system is designed around a React/TypeScript stack and emphasizes test-driven development, accessibility-first design, and clear separation of concerns between agents.

## Overview

This repository lives at `~/.config/opencode` and serves as the global configuration for OpenCode. It defines:

- **11 agents** organised into primary (user-facing) and subagent (task-focused) roles
- **4 skills** providing domain-specific knowledge that agents can load on demand
- **1 custom command** for structured git commits
- **2 MCP integrations** for browser DevTools and library documentation lookup

The default agent is `build-envoy`, an intermediary that refines user requests before dispatching them to the `build` subagent for execution.

## How It Works

### The Envoy Pattern

The core workflow uses an "Envoy" pattern — a lightweight primary agent that acts as a linguistic bridge between the user and a powerful subagent:

```
User (casual request)
  └─> Envoy (refines prompt, asks for confirmation)
        └─> Subagent (executes the refined task)
              └─> Envoy (returns full response verbatim)
                    └─> User
```

1. **User** types a casual, possibly rough request.
2. **Envoy** rewrites it into a clear, structured prompt and presents it back for approval.
3. **User** confirms (or iterates on the draft).
4. **Envoy** dispatches the approved prompt to the target subagent.
5. **Subagent** performs the work (code, planning, review, etc.) and returns results.
6. **Envoy** relays the subagent's full response verbatim — no summarisation, no truncation.

This ensures prompts are high-quality before expensive models (Claude Opus) process them, while keeping the user in control.

<img width="1118" height="424" alt="envoy-at-work" src="https://github.com/user-attachments/assets/31d433ad-ae41-46c8-bda1-33540b5c6e6e" />

### Model Strategy

The configuration uses a deliberate two-tier model strategy:

| Tier | Model | Used By | Purpose |
|------|-------|---------|---------|
| **Fast / Cheap** | `claude-sonnet-4.6` | Envoys, browser, docs-lookup, explore, search | Prompt refinement, read-only tasks, lightweight operations |
| **Powerful** | `claude-opus-4.6` | build, plan, coder, review, general | Code generation, architectural planning, deep analysis |

## Agent Architecture

### Primary Agents (User-Facing)

These agents are directly accessible to the user. They appear as selectable agents in the OpenCode UI.

| Agent | Model | Description |
|-------|-------|-------------|
| **build-envoy** (default) | Sonnet 4.6 | Refines casual requests into precise prompts, dispatches to `@build` after user confirmation |
| **plan-envoy** | Sonnet 4.6 | Same envoy pattern, but dispatches to `@plan` for analysis and planning tasks |

### Subagents (Task-Focused)

These agents are invoked by primary agents or other subagents. Each has a tightly scoped permission set.

| Agent | Model | Permissions | Description |
|-------|-------|-------------|-------------|
| **build** | Opus 4.6 | Full tools + task(build) | Writes, tests, and verifies code. Must run all code it writes. |
| **plan** | Opus 4.6 | Read-only (no edit, no todo) | Restricted agent for planning and analysis only |
| **coder** | Opus 4.6 | Full file system + git | Code writing and editing specialist |
| **review** | Opus 4.6 | Read-only (no edit, bash, lsp) | Reviews code for quality; provides feedback without making changes |
| **search** | Sonnet 4.6 | Read-only (glob, grep, read) | Fast file and code search using patterns |
| **explore** | Sonnet 4.6 | Read-only | Fast, lightweight codebase exploration |
| **browser** | Sonnet 4.6 | chrome-devtools_* | Interacts with Chrome via DevTools MCP for screenshots, DOM inspection, navigation, performance traces |
| **docs-lookup** | Sonnet 4.6 | context7_* | Looks up library/framework documentation using Context7 MCP |
| **general** | Opus 4.6 | Unrestricted general-purpose agent with full tool access |

### Permission Model

Agents use a deny-by-default permission system. Each agent's frontmatter explicitly grants or denies access to tools:

```yaml
permission:
  task:
    "*": deny        # Deny spawning all subagents by default
    build: allow     # Except @build
  read: deny         # No file reading
  edit: deny         # No file editing
  bash: deny         # No shell commands
  # ...etc
```

Key design decisions:
- **Envoys are tool-less** — they can only spawn their designated subagent. No file access, no shell, no editing.
- **Review is read-only** — can read code but cannot modify it, enforcing advisory-only feedback.
- **Plan is edit-restricted** — can read and analyse but cannot make changes.
- **Build must test** — the build agent's system prompt mandates running all code it writes before returning results.

## Skills

Skills are domain-specific knowledge bundles that agents load on demand via the `skill` tool. Each skill is a directory of markdown files containing guides, examples, anti-patterns, and quickstart templates.

| Skill | Files | Description |
|-------|-------|-------------|
| **e2e-testing** | 8 files (~2,500 lines) | Playwright E2E testing with React Testing Library best practices. Covers locator strategy (getByRole hierarchy), web-first assertions, auto-waiting, test isolation, and debugging. |
| **react-best-practices** | 10 files (~5,600 lines) | Comprehensive React development guide. Covers Thinking in React (5-step process), hooks, state management, composition patterns, performance, Context API, and common pitfalls. |
| **react-testable-code** | 9 files (~6,100 lines) | Writing React code that's trivial to E2E test. Covers semantic HTML, accessibility-first design, component architecture, and the connection between accessibility and testability. |
| **testing-library** | 1 file (~640 lines) | React Testing Library for unit/component tests. Covers query priority order, userEvent, jest-dom matchers, async patterns, and anti-patterns. |

### How Skills Are Used

The `build` agent is instructed to load skills before implementing work:

- Writing Playwright tests → load `e2e-testing`
- Writing React components → load `react-best-practices`
- Creating testable components → load `react-testable-code`
- Writing component unit tests → load `testing-library`

Skills are loaded at runtime and injected into the agent's context, providing detailed instructions without bloating the base system prompt.

## Custom Commands

| Command | File | Description |
|---------|------|-------------|
| **commit** | `commands/commit.md` | Commits changes to Git in organised, scoped commits with clear descriptions of what and why changes were made. |

## MCP Integrations

Two Model Context Protocol (MCP) servers extend agent capabilities:

### Chrome DevTools MCP
- **Type:** Local (runs `npx chrome-devtools-mcp@latest`)
- **Used by:** `browser` agent
- **Capabilities:** Screenshots, DOM snapshots, page navigation, clicking/typing, network monitoring, performance traces, console messages, Lighthouse audits, JavaScript evaluation

### Context7 MCP
- **Type:** Remote (`https://mcp.context7.com/mcp`)
- **Used by:** `docs-lookup` agent
- **Capabilities:** Resolves library names to IDs, queries up-to-date documentation and code examples for any programming library or framework

## Configuration

### `opencode.jsonc`

The main configuration file:

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "default_agent": "build-envoy",
  "mcp": {
    "chrome-devtools": { ... },
    "context7": { ... }
  },
  "experimental": {
    "disable_paste_summary": true
  }
}
```

### Agent Definition Format

Each agent is a markdown file in `agents/` with YAML frontmatter:

```yaml
---
description: Human-readable description
mode: primary | subagent
model: github-copilot/claude-sonnet-4.6 | github-copilot/claude-opus-4.6
temperature: 0.1  # Optional
permission:
  tool_name: allow | deny
  task:
    "*": deny
    agent_name: allow
---

System prompt content in markdown below the frontmatter.
```

### Skill Definition Format

Each skill directory contains a `SKILL.md` with optional YAML frontmatter and comprehensive instructional content. Supporting files (EXAMPLES.md, ANTI-PATTERNS.md, QUICKSTART.md, etc.) provide depth.

## Directory Structure

```
~/.config/opencode/
├── opencode.jsonc           # Main configuration (MCP, default agent, experiments)
├── agents/                  # Agent definitions (11 agents)
│   ├── build-envoy.md       # Default agent — refines prompts for @build
│   ├── plan-envoy.md        # Refines prompts for @plan
│   ├── general.md           # Unrestricted general-purpose agent
│   ├── build.md             # Code writing + testing subagent
│   ├── plan.md              # Read-only planning/analysis subagent
│   ├── coder.md             # File editing specialist subagent
│   ├── review.md            # Read-only code review subagent
│   ├── search.md            # File/code search subagent
│   ├── explore.md           # Codebase exploration subagent
│   ├── browser.md           # Chrome DevTools interaction subagent
│   └── docs-lookup.md       # Documentation lookup subagent
├── commands/                # Custom slash commands
│   └── commit.md            # Git commit command
├── skills/                  # Loadable knowledge bundles
│   ├── e2e-testing/         # Playwright E2E testing guide (8 files)
│   ├── react-best-practices/# React development patterns (10 files)
│   ├── react-testable-code/ # Testable React components (9 files)
│   └── testing-library/     # React Testing Library guide (1 file)
└── package.json             # Dependencies (@opencode-ai/plugin)
```

## Design Principles

1. **Prompt Quality Over Speed** — The envoy pattern ensures prompts are refined before reaching expensive models, reducing wasted computation and improving output quality.

2. **Principle of Least Privilege** — Each agent has only the permissions it needs. Reviewers can't edit. Planners can't modify. Envoys can't access files.

3. **Test Everything** — The build agent is explicitly instructed to run all code it writes. No assumptions, no hallucinated test results. Evidence is required.

4. **Skills as Composable Knowledge** — Domain expertise is modular and loaded on demand rather than baked into system prompts, keeping context windows efficient.

5. **Accessibility Equals Testability** — The skills consistently reinforce that semantic HTML and ARIA attributes make both accessible software and reliable tests. One practice serves two goals.

6. **Model Cost Optimisation** — Lightweight tasks (search, exploration, prompt refinement) use the cheaper Sonnet model; heavy tasks (code generation, planning, review) use Opus.

## Usage

### Typical Workflow

1. Start OpenCode — the `build-envoy` agent is active by default.
2. Describe what you want in natural language (casual is fine).
3. The envoy refines your request and presents a polished prompt.
4. Confirm with "Yes", "Go", or "Send it" (or iterate).
5. The `build` subagent executes: writes code, runs tests, reports results.
6. Review the output and continue the conversation.

### Switching Agents

- Use `@plan-envoy` for planning and analysis tasks.
- Use `@general` for unrestricted access to all tools.
- Subagents are invoked automatically — you don't call them directly.

### Using Commands

- `/commit` — Triggers the commit command to stage and commit changes with descriptive messages.
