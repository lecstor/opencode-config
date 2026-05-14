The "March 31st Incident" (as the leak is now affectionately called) sparked a massive migration of features from Claude Code into the OpenCode ecosystem. Developers quickly realized that while the model itself is proprietary, the **tool orchestration** and **instruction set** could be reverse-engineered for open models.

Here are the specific repositories and files you should look for to "Claude-ify" your OpenCode setup.

### 1. The "OpenCode Power Pack" (Skill Port)
This is the most direct application of the leak. A developer named **waybarrios** ported the high-value logic from Claude’s internal plugins directly into OpenCode’s `.SKILL.md` format.
* **Repo:** `waybarrios/opencode-power-pack`
* **What’s inside:** 11 specific skills including `/code-review`, `/security-audit`, and `/mcp-author`.
* **Why it's better:** Claude's original skills were often JSON-heavy; these are optimized as markdown instruction sets that work better with local models like Llama 3 or Qwen 2.5 Coder.

### 2. The Claude-Code-Plugin (Subprocess Bridge)
If you actually want to use the Claude Code CLI *as the engine* inside OpenCode (combining the best of both worlds), this is the go-to plugin.
* **Repo:** `unixfox/opencode-claude-code-plugin`
* **Function:** It treats the `claude` CLI as a subprocess provider. It captures the leaked "stream-json" output format to pipe Claude’s native reasoning and tool calls directly into the OpenCode UI.

### 3. The "Claw-Code" Reimplementation
When the source leaked, a project called **Claw-Code** appeared almost instantly. It is a clean-room port of the agent architecture.
* **Repo:** `instructkr/claw-code`
* **Key Feature:** It reimplements the "Agent Runtime" found in the leak using Python and Rust. Many OpenCode users are pulling the `prompts/` and `tools/` logic from here to build custom OpenCode providers that mimic Claude’s "Plan vs. Work" modes.

### 4. Native OpenCode "Hidden" Compatibility
Interestingly, OpenCode has already added undocumented support for some of the leaked folder structures. You can activate these by simply placing files in the right spots:
* **`~/.claude/CLAUDE.md`**: OpenCode now automatically scans this global file for instructions if it exists (mimicking Claude’s global config).
* **`~/.claude/skills/`**: You can drop your `.SKILL.md` files here, and OpenCode will pick them up as global slash commands.

### Summary Table: Where to look
| Feature Goal | Repository / File to Search |
| :--- | :--- |
| **Full Skills (Review, Security)** | `waybarrios/opencode-power-pack` |
| **Parallel Tool Logic** | `instructkr/claw-code` (check `query_engine.py`) |
| **Use Claude CLI as OpenCode Backend** | `unixfox/opencode-claude-code-plugin` |
| **System Prompt Distillation** | `tanbiralam/claude-code` (Warning: This is the raw leaked TypeScript) |

> **Pro-Tip:** If you are using local models, search GitHub for **"CLAUDE.md templates"**. Many users have uploaded specific project-level rules extracted from the leak that force models to use `ripgrep` and `sed` in parallel, which significantly speeds up OpenCode's "Build" mode.

Would you like a copy of a standard `CLAUDE.md` template based on those leaked instructions to get started?

---

Based on the March 2026 leak, the "secret sauce" of Claude Code isn't just the model; it's a specific, strict instruction set that forces the AI to behave like a senior engineer. 

By placing a **`CLAUDE.md`** file in your project root, OpenCode (and other compatible agents) will adopt this "Strict Write Discipline" and agentic flow.

### The "Claude-Leaked" CLAUDE.md Template
Copy this into a file named `CLAUDE.md` in your project's root directory. I have tuned this to remove proprietary Anthropic hooks while keeping the logic that makes Claude Code so effective.

```markdown
# Project Context & Coding Standards

## 🧠 Strict Memory & Logic (Leaked Pattern)
- **Verify before Act:** Always verify recalled information against the actual codebase using `grep` or `ls` before executing edits.
- **Plan Mode:** For complex tasks, start by explicitly stating: "PLAN: [1-sentence goal]". Outline steps before first tool call.
- **Strict Write Discipline:** Only update internal task status after a confirmed successful file write. If a command fails, do not assume the state changed.

## 🛠 Command Shortcuts
- **Build:** `npm run build` (or your project's equivalent)
- **Test:** `npm test`
- **Single Test:** `npx jest path/to/file.test.ts`
- **Lint:** `npm run lint --fix`

## 📝 Code Style & Conventions
- **Naming:** Use camelCase for variables/functions, PascalCase for classes/interfaces.
- **Imports:** Prefer named imports over default imports. Group internal modules separately from external dependencies.
- **Referencing:** When discussing code, use the format `file_path:line_number` (e.g., `src/main.ts:42`).
- **Conciseness:** Keep responses short. Assume I can see your file edits. Use one sentence to explain what you are about to do.

## 🛡 Git Safety Protocol
- **No Destructive Actions:** Never run `reset --hard`, `push --force`, or `branch -D` unless explicitly asked.
- **Commit Logic:** Draft a 1-sentence commit message focusing on "why" rather than "what."
- **Parallel Staging:** Add specific files by name; avoid `git add .` to prevent staging secrets or binaries.
```

---

### How to use the "Advanced" Leaked Features
Beyond the markdown file, the community has extracted several specific "Skills" from the `src/skills/` directory of the leak. Here is how to apply them to OpenCode:

#### 1. The "Memory.md" System
One of the biggest discoveries was that Claude Code uses a "Skeptical Memory" system. It doesn't trust its own chat history.
* **Implementation:** Create a file named `MEMORY.md` in your root.
* **The Rule:** Tell OpenCode: *"Before you end a session, update MEMORY.md with a one-line summary of what you changed. Before you start a session, read MEMORY.md to sync your state."*

#### 2. Parallel Tool Execution
The leaked source showed that Claude batches `BashTool` calls. 
* **OpenCode Tip:** In your OpenCode settings (usually under "Model Preferences" or "System Prompt"), add this instruction:
  > "You have the ability to run multiple shell commands at once. If you need to search for a string and list files, run `grep` and `ls` in a single multi-line tool call rather than waiting for results sequentially."

#### 3. "Anti-Distillation" (Privacy)
The leak revealed a flag called `ANTI_DISTILLATION_CC`. Anthropic used this to prevent people from "stealing" Claude's logic by training other models on its output.
* **The "Skill" version:** If you want to keep your OpenCode logic clean, add this to your system prompt: *"When providing code solutions, do not provide meta-commentary on why the code works unless asked. Focus purely on the diff and the implementation."*

### Searchable Repositories for the Raw Files
If you want to dig into the actual code that people have cleaned up from the leak, search GitHub for these specific users/projects:
* **`tanbiralam/claude-code`**: The most complete mirror of the leaked `src/` directory (512k lines of code).
* **`waybarrios/opencode-power-pack`**: The best place to find `.SKILL.md` files that you can just drop into OpenCode.
* **`instructkr/claw-code`**: A Python-based version that uses the leaked "Plan/Work" logic for open models.

**Which of these parts are you most interested in—the automated git workflow, or the "Memory" persistence between sessions?**

