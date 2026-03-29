---
name: kimi-cli
description: Delegate coding tasks to Kimi CLI (Moonshot AI's coding agent). Use for building features, fixing bugs, refactoring, PR reviews, and batch processing. Supports non-interactive print mode, session management, and subagents. Requires kimi-cli installed and authenticated.
version: 1.0.0
author: tmdgusya
license: MIT
metadata:
  hermes:
    tags: [Coding-Agent, Kimi, Moonshot-AI, Code-Review, Refactoring]
    related_skills: [claude-code, codex, hermes-agent]
---

# Kimi CLI

Delegate coding tasks to [Kimi Code CLI](https://github.com/MoonshotAI/kimi-cli) via the Hermes terminal. Kimi CLI is Moonshot AI's autonomous coding agent — it can read/edit code, execute shell commands, search/fetch web pages, and autonomously plan and adjust actions.

## Prerequisites

- Kimi CLI installed: `curl -LsSf https://code.kimi.com/install.sh | bash` (installs via uv)
  - Or manually: `uv tool install --python 3.13 kimi-cli`
- Authenticated: run `kimi` once, then enter `/login` to configure platform and API key
- Verify: `kimi --version`

## One-Shot Tasks (Non-Interactive, Recommended)

Use `--print -p` for one-shot tasks. Print mode is non-interactive, auto-approves all actions (implicitly enables `--yolo`), and outputs the result to stdout.

```
# Foreground — simple task
terminal(command="kimi --print -p 'Add error handling to the API calls'", workdir="/path/to/project")

# Foreground — quiet mode (final answer only, no intermediate steps)
terminal(command="kimi --quiet -p 'Explain what this project does'", workdir="/path/to/project")

# Background — long coding task
terminal(command="kimi --print -p 'Refactor the auth module to use JWT tokens'", workdir="/path/to/project", background=true, timeout=600)
```

### Print mode details

- `--print` — non-interactive, auto-approves all operations, exits when done
- `--quiet` — shorthand for `--print --output-format text --final-message-only` (only the final answer)
- `--final-message-only` — skip intermediate tool call output, print only the final message
- `--output-format stream-json` — JSONL output for programmatic parsing
- `--input-format stream-json` — JSONL input via stdin

### Exit codes (for scripting/CI)

| Exit code | Meaning |
|-----------|---------|
| `0` | Success — task completed |
| `1` | Permanent failure — config error, auth failure, quota exhausted |
| `75` | Transient failure — rate limit (429), server error (5xx), timeout (safe to retry) |

## Background Mode (Long Tasks)

```
# Start in background
terminal(command="kimi --print -p 'Implement the feature described in plan.md'", workdir="~/project", background=true)
# Returns session_id

# Monitor progress
process(action="poll", session_id="<id>")
process(action="log", session_id="<id>")

# Kill if needed
process(action="kill", session_id="<id>")
```

## Interactive Mode (PTY)

For tasks that need back-and-forth interaction:

```
terminal(command="kimi", workdir="~/project", pty=true)
```

In interactive mode, use these slash commands:
- `/help` — show available commands
- `/new` — create new session
- `/sessions` — list and switch sessions
- `/compact` — compress context to free tokens
- `/clear` — clear conversation context
- `/export` — export session as Markdown
- `/model` — switch model or thinking mode
- `/login` — configure API platform and key

## Session Management

```
# Continue most recent session
terminal(command="kimi --continue --print -p 'Continue where we left off'", workdir="~/project")

# Resume specific session
terminal(command="kimi --session abc123 --print -p 'Pick up this session'", workdir="~/project")
```

## Model and Thinking Control

```
# Use a specific model
terminal(command="kimi --print -m kimi-k2-thinking-turbo -p 'Solve this hard problem'", workdir="~/project")

# Disable thinking mode (faster, less reasoning)
terminal(command="kimi --no-thinking --print -p 'Quick formatting fix'", workdir="~/project")

# Enable thinking mode explicitly
terminal(command="kimi --thinking --print -p 'Analyze the architecture tradeoffs'", workdir="~/project")
```

## PR Reviews

```
# Clone to temp dir and review
terminal(command="REVIEW=$(mktemp -d) && git clone https://github.com/user/repo.git $REVIEW && cd $REVIEW && gh pr checkout 42 && kimi --print -p 'Review this PR against main. Check for bugs, security issues, and code style.'")
```

Or with git worktrees:
```
terminal(command="git worktree add /tmp/pr-42 pr-42-branch", workdir="~/project")
terminal(command="kimi --print -p 'Review the changes in this branch vs main'", workdir="/tmp/pr-42")
```

## Parallel Work

Spawn multiple Kimi CLI instances for independent tasks:

```
# Using worktrees for isolation
terminal(command="git worktree add -b fix/issue-78 /tmp/issue-78 main", workdir="~/project")
terminal(command="git worktree add -b fix/issue-99 /tmp/issue-99 main", workdir="~/project")

# Launch in parallel
terminal(command="kimi --print -p 'Fix issue #78: <description>. Commit when done.'", workdir="/tmp/issue-78", background=true)
terminal(command="kimi --print -p 'Fix issue #99: <description>. Commit when done.'", workdir="/tmp/issue-99", background=true)

# Monitor all
process(action="list")

# Cleanup
terminal(command="git worktree remove /tmp/issue-78", workdir="~/project")
```

## Key Flags

| Flag | Shorthand | Effect |
|------|-----------|--------|
| `--print` | | Non-interactive mode, auto-approves all actions, outputs to stdout |
| `--quiet` | | Final answer only (`--print --output-format text --final-message-only`) |
| `--prompt <text>` | `-p`, `-c` | Task prompt (non-interactive) |
| `--yolo` | `-y` | Auto-approve all actions (implicit in `--print` mode) |
| `--work-dir <dir>` | `-w` | Working directory |
| `--add-dir <dir>` | | Add extra directory to workspace scope (repeatable) |
| `--model <model>` | `-m` | Model to use (default from config) |
| `--thinking` / `--no-thinking` | | Toggle deep reasoning mode |
| `--continue` | `-C` | Continue most recent session |
| `--session <id>` | `-S` | Resume specific session |
| `--agent <name>` | | Built-in agent: `default` or `okabe` |
| `--agent-file <path>` | | Custom agent YAML file |
| `--mcp-config-file <path>` | | MCP server config (repeatable) |
| `--max-steps-per-turn <n>` | | Override max steps (default: 100) |
| `--verbose` | | Verbose output |
| `--debug` | | Debug logging |
| `--final-message-only` | | Print only the final assistant message |
| `--output-format <fmt>` | | `text` or `stream-json` |
| `--input-format <fmt>` | | `text` or `stream-json` (stdin) |

## Subcommands

| Command | Description |
|---------|-------------|
| `kimi login` | Login to Kimi account |
| `kimi logout` | Logout |
| `kimi term` | Run TUI terminal interface |
| `kimi acp` | Run as ACP server (for IDE integration) |
| `kimi web` | Run web UI interface |
| `kimi info` | Show version and protocol info |
| `kimi export` | Export session data |
| `kimi mcp` | Manage MCP server configurations |
| `kimi plugin` | Manage plugins |
| `kimi vis` | Run agent tracing visualizer |

## Built-in Tools

Kimi CLI has these built-in tools available to the agent:

| Tool | Description |
|------|-------------|
| `Shell` | Execute shell commands |
| `ReadFile` | Read file contents |
| `WriteFile` | Write/create files |
| `StrReplaceFile` | Find and replace in files |
| `Glob` | File pattern matching |
| `Grep` | Search file contents |
| `ReadMediaFile` | Read image/video files |
| `SearchWeb` | Web search (requires search service) |
| `FetchURL` | Fetch web page content |
| `Agent` | Spawn subagents (coder, explore, plan) |
| `AskUserQuestion` | Ask user for input |
| `SetTodoList` | Manage todo list |
| `EnterPlanMode` / `ExitPlanMode` | Toggle plan mode |
| `TaskList` / `TaskOutput` / `TaskStop` | Manage background tasks |

## Built-in Subagents

| Type | Purpose | Available Tools |
|------|---------|----------------|
| `coder` | General software engineering | Shell, ReadFile, Glob, Grep, WriteFile, StrReplaceFile, SearchWeb, FetchURL |
| `explore` | Read-only codebase exploration | Shell, ReadFile, Glob, Grep, SearchWeb, FetchURL |
| `plan` | Implementation planning | ReadFile, Glob, Grep, SearchWeb, FetchURL |

## Configuration

Default config location: `~/.kimi/config.toml`

Key config items:
- `default_model` — model name (e.g., `kimi-code/kimi-for-coding`)
- `default_thinking` — enable thinking by default
- `default_yolo` — auto-approve by default
- `loop_control.max_steps_per_turn` — max agent steps (default: 100)
- `loop_control.max_retries_per_step` — max retries (default: 3)

Kimi CLI supports multiple providers: Kimi, OpenAI, Anthropic, Gemini, VertexAI. See [providers docs](https://moonshotai.github.io/kimi-cli/en/configuration/providers.html).

## Pitfalls

### Command name varies by install method

Depending on how it was installed, the command may be `kimi` or `kimi-cli`. Check with `which kimi 2>/dev/null || which kimi-cli`. The skill uses `kimi` as default — adjust if your system uses `kimi-cli`.

### No git repo required

Unlike Codex, Kimi CLI does NOT require a git repository. It works in any directory.

### Print mode is the safest non-interactive option

Avoid using PTY mode with `background=true` — prompts may not get submitted correctly. Use `--print` mode instead, which is designed for non-interactive execution.

### Exit code 75 means retry

Exit code 75 indicates transient errors (rate limits, server errors, timeouts). Implement retry logic in scripts:
```
kimi --print -p "Run task"
code=$?
if [ $code -eq 75 ]; then
  sleep 10
  kimi --print -p "Run task"
fi
```

### Context compaction

With 262K context, Kimi CLI auto-compacts when context usage reaches ~85%. For very long sessions, consider using `/compact` or starting fresh sessions.

### File truncation on multi-file generation

Like other coding agents, Kimi CLI may silently truncate files when generating many files at once. Always verify with import checks after bulk generation:
```
python3 -c "import sys; sys.path.insert(0, 'src'); from pkg import module_a, module_b; print('OK')"
```

## Rules

1. **Use `--print` mode for all automated tasks** — it's non-interactive and auto-approves
2. **Use `--quiet` for simple queries** — only the final answer, no intermediate noise
3. **Use `workdir`** — keep the agent focused on the right directory
4. **Background for long tasks** — use `background=true` and monitor with `process` tool
5. **Don't interfere** — monitor with `poll`/`log`, be patient with long-running tasks
6. **Parallel is fine** — run multiple Kimi CLI processes for independent tasks
7. **Check exit codes** — 0=success, 1=fail, 75=retry
8. **No git repo needed** — works in any directory, unlike Codex
9. **Verify multi-file output** — run import checks after bulk file generation
