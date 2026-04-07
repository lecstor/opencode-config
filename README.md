# OpenCode Agent Configuration

A multi-agent configuration for [OpenCode](https://opencode.ai) that defines a structured system of specialized AI agents, reusable skills, and custom commands for software development workflows. The system is designed around a React/TypeScript stack and emphasizes test-driven development, accessibility-first design, and clear separation of concerns between agents.

## Overview

This repository lives at `~/.config/opencode` and serves as the global configuration for OpenCode. It defines:

- **10 agents** organised into primary (user-facing) and subagent (task-focused) roles
- **4 skills** providing domain-specific knowledge that agents can load on demand
- **1 custom command** for structured git commits
- **1 plugin** for dynamic context pruning to reduce token usage in long sessions
- **2 MCP integrations** for browser DevTools and library documentation lookup

The default agent is `build`, which writes, tests, and verifies code with full autonomy.

> **Looking for the Envoy pattern?** The [`envoy`](../../tree/envoy) branch has the previous setup where lightweight envoy agents act as intermediaries that refine user requests before dispatching them to subagents. If you prefer that workflow, check out that branch instead.

## Setup

### Installation

There are two ways to use this configuration:

**Use as-is** — Clone or copy the entire repository into your OpenCode config folder:

```sh
git clone <repo-url> ~/.config/opencode
```

This gives you the full agent system, all skills, commands, and MCP integrations out of the box.

**Pick and choose** — If you already have an OpenCode config folder, copy only the pieces you want into it. The main building blocks are:

- `agents/` — Agent definitions (copy individual `.md` files)
- `skills/` — Loadable knowledge bundles (copy entire skill directories)
- `commands/` — Custom slash commands (copy individual `.md` files)
- `opencode.jsonc` — MCP server configuration (merge the relevant sections into your existing config)

### Context7 API Key

The Context7 MCP integration (used by the `docs-lookup` agent to query library documentation) requires an API key. The key is referenced in `opencode.jsonc` via the `CONTEXT7_API_KEY` environment variable:

```jsonc
"headers": {
  "CONTEXT7_API_KEY": "{env:CONTEXT7_API_KEY}"
}
```

To set it up:

1. Get a free API key from [context7.com](https://context7.com).
2. Add the following line to `~/.zshenv` (or `~/.zshrc`):

   ```sh
   export CONTEXT7_API_KEY="your-key-here"
   ```

3. Restart your shell, or run `source ~/.zshenv` to pick up the change immediately.

Once the variable is set, OpenCode will pass the key to Context7 automatically when the `docs-lookup` agent resolves library IDs or queries documentation.

## How It Works

### Model Strategy

The configuration uses a deliberate three-tier model strategy:

| Tier | Model | Used By | Purpose |
|------|-------|---------|---------|
| **Fast / Cheap** | `claude-sonnet-4.6` | coder, review, browser | Scoped code edits, code review, multi-step browser interactions |
| **Powerful** | `claude-opus-4.6` | build, plan, general | Code generation, architectural planning, complex research |
| **Minimal** | `claude-haiku-4.5` | test-runner, search, explore, docs-lookup | Test execution, file search, codebase exploration, documentation lookup |

## Agent Architecture

### Primary Agents (User-Facing)

These agents are directly accessible to the user. They appear as selectable agents in the OpenCode UI.

| Agent | Model | Description |
|-------|-------|-------------|
| **build** (default) | Opus 4.6 | Writes, tests, and verifies code. Must run all code it writes. Delegates to subagents for test execution and code review. |
| **plan** | Opus 4.6 | Read-only planning agent. Explores the codebase, produces structured implementation plans with file-level changes, ordering, risks, and testing strategy. |
| **general** | Opus 4.6 | General-purpose research and execution agent with full tool access. |

### Subagents (Task-Focused)

These agents are invoked by primary agents or other subagents. Each has a tightly scoped permission set.

| Agent | Model | Permissions | Description |
|-------|-------|-------------|-------------|
| **coder** | Sonnet 4.6 | File editing (no bash, no task except search/explore/docs-lookup) | Code writing and editing specialist |
| **review** | Sonnet 4.6 | Read-only (no edit, bash, lsp) | Reviews code for quality; provides feedback without making changes |
| **test-runner** | Haiku 4.5 | Bash only (no edit, task, lsp) | Executes test suites and reports structured results without modifying code |
| **search** | Haiku 4.5 | Read-only (glob, grep, read) | Fast file and code search using patterns |
| **explore** | Haiku 4.5 | Read-only (glob, grep, read) | Fast, lightweight codebase exploration |
| **browser** | Sonnet 4.6 | chrome-devtools_* | Interacts with Chrome via DevTools MCP for screenshots, DOM inspection, navigation, performance traces |
| **docs-lookup** | Haiku 4.5 | context7_* | Looks up library/framework documentation using Context7 MCP |

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
- **Review is read-only** — can read code but cannot modify it, enforcing advisory-only feedback.
- **Plan is edit-restricted** — can read and analyse but cannot make changes.
- **Coder is edit-only** — can read and write files but cannot run commands or spawn most subagents (search, explore, and docs-lookup are allowed for context gathering).
- **Test-runner is execution-only** — can run shell commands but cannot edit files or spawn subagents.
- **Build must test** — the build agent's system prompt mandates running all code it writes before returning results. Non-code changes (config, docs) have a lighter path that skips tests and review when appropriate.

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

## Plugins

### Dynamic Context Pruning (DCP)

The [`@tarquinen/opencode-dcp`](https://github.com/Opencode-DCP/opencode-dynamic-context-pruning) plugin automatically reduces token usage by intelligently managing conversation context during long sessions. It is pinned to version **3.0.4** in `opencode.jsonc`:

```jsonc
"plugin": ["@tarquinen/opencode-dcp@3.0.4"]
```

#### What It Does

DCP intercepts messages before they are sent to the LLM and prunes redundant or stale content — without modifying your actual session history. It operates through four strategies:

- **Compress** — Exposes a `compress` tool to the model. Instead of OpenCode's built-in compaction (which triggers on the entire session at the context limit), Compress lets the model selectively summarise completed task ranges. Nested compressions preserve earlier summaries, and protected tool outputs (subagents, skills) are always retained.
- **Deduplication** — Detects repeated tool calls (same tool, same arguments) and keeps only the most recent output.
- **Purge Errors** — Prunes the potentially large input content from errored tool calls after a configurable number of turns (default: 4), while preserving the error messages themselves.
- **Supersede Writes** — Prunes write tool inputs when the file has been subsequently read, since the read output already contains the current content.

#### Why It's Here

Long coding sessions with the `build` agent — which reads, writes, tests, and iterates on code — can accumulate large amounts of context. DCP keeps sessions productive by trimming stale content before it pushes the model towards hallucination or triggers a costly full-context compaction.

For request-based billing providers like GitHub Copilot (used in this configuration), prompt-cache misses from pruning have no cost impact since billing is per-request, not per-token.

#### Configuration

DCP uses its own config file at `~/.config/opencode/dcp.jsonc`, separate from the main `opencode.jsonc`. The project includes a minimal config that references the plugin's JSON schema:

```jsonc
{
  "$schema": "https://raw.githubusercontent.com/Opencode-DCP/opencode-dynamic-context-pruning/master/dcp.schema.json"
}
```

All settings use defaults (compression enabled, deduplication enabled, error purging after 4 turns). Override any setting by adding it to `dcp.jsonc` — project-level configs in `.opencode/dcp.jsonc` take priority over the global file.

#### Commands

The plugin registers a `/dcp` slash command with subcommands:

| Command | Description |
|---------|-------------|
| `/dcp` | Shows available DCP commands |
| `/dcp context` | Token usage breakdown for the current session |
| `/dcp stats` | Cumulative pruning statistics across sessions |
| `/dcp sweep [n]` | Prune tool outputs since last user message (optionally last *n* tools) |
| `/dcp compress [focus]` | Trigger a manual compression, optionally focused on a topic |
| `/dcp decompress <id>` | Restore a specific compression by ID |
| `/dcp recompress <id>` | Re-apply a previously decompressed compression by ID |
| `/dcp manual [on\|off]` | Toggle manual mode (disables autonomous compression) |

## Configuration

### `opencode.jsonc`

The main configuration file:

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "default_agent": "build",
  "mcp": {
    "chrome-devtools": { ... },
    "context7": { ... }
  },
  "experimental": {
    "disable_paste_summary": true
  },
  "plugin": ["@tarquinen/opencode-dcp@3.0.4"]
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
├── opencode.jsonc           # Main configuration (MCP, plugins, default agent, experiments)
├── dcp.jsonc                # Dynamic Context Pruning plugin configuration
├── agents/                  # Agent definitions (10 agents)
│   ├── build.md             # Default agent — writes, tests, and verifies code
│   ├── plan.md              # Read-only planning/analysis agent
│   ├── general.md           # Unrestricted general-purpose agent
│   ├── coder.md             # File editing specialist subagent
│   ├── review.md            # Read-only code review subagent
│   ├── test-runner.md       # Test execution and reporting subagent
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

1. **Principle of Least Privilege** — Each agent has only the permissions it needs. Reviewers can't edit. Planners can't modify. Test runners can't write files.

2. **Test Everything** — The build agent is explicitly instructed to run all code it writes. No assumptions, no hallucinated test results. Evidence is required.

3. **Skills as Composable Knowledge** — Domain expertise is modular and loaded on demand rather than baked into system prompts, keeping context windows efficient.

4. **Accessibility Equals Testability** — The skills consistently reinforce that semantic HTML and ARIA attributes make both accessible software and reliable tests. One practice serves two goals.

5. **Model Cost Optimisation** — Only the orchestrator and planner use Opus. Code review and editing use Sonnet. Pure tool-calling tasks (test execution, file search, exploration, documentation lookup) use Haiku. Each agent sits on the cheapest tier that can reliably do its job.

## Usage

### Typical Workflow

1. Start OpenCode — the `build` agent is active by default.
2. Describe what you want in natural language.
3. The `build` agent writes code, runs tests, requests review, and reports results.
4. Review the output and continue the conversation.

### Switching Agents

- Use `@plan` for planning and analysis tasks (read-only — won't make changes).
- Use `@general` for unrestricted access to all tools.
- Subagents are invoked automatically — you don't call them directly.

### Using Commands

- `/commit` — Triggers the commit command to stage and commit changes with descriptive messages.
