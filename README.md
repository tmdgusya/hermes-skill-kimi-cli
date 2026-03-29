# hermes-skill-kimi-cli

A [Hermes Agent](https://github.com/tmdgusya/hermes-agent) skill for delegating coding tasks to [Kimi Code CLI](https://github.com/MoonshotAI/kimi-cli) — Moonshot AI's autonomous coding agent.

## What it does

This skill enables Hermes Agent to use Kimi CLI as a coding agent for:

- **Feature implementation** — describe features in natural language, Kimi writes the code
- **Bug fixing** — paste error logs or describe bugs, Kimi diagnoses and fixes
- **Code review** — review PRs, diffs, or entire codebases
- **Refactoring** — restructure code while preserving behavior
- **Batch processing** — parallel work on multiple independent tasks
- **Code exploration** — understand unfamiliar codebases

## Prerequisites

- [Kimi CLI](https://github.com/MoonshotAI/kimi-cli) installed and authenticated
  ```bash
  curl -LsSf https://code.kimi.com/install.sh | bash
  kimi  # first run, then /login to authenticate
  ```

## Install

```bash
hermes skills install tmdgusya/hermes-skill-kimi-cli
```

## Key features

- **Print mode** (`--print -p`) — non-interactive, auto-approves all actions, perfect for automation
- **Quiet mode** (`--quiet`) — final answer only, no intermediate output
- **No git repo required** — works in any directory (unlike some other agents)
- **262K context window** — handles large codebases
- **Built-in tools** — Shell, ReadFile, WriteFile, Grep, Glob, SearchWeb, FetchURL, and more
- **Subagents** — built-in coder, explore, and plan agent types
- **Multi-provider** — supports Kimi, OpenAI, Anthropic, Gemini, VertexAI backends
- **Session management** — continue previous sessions, resume by ID
- **JSON streaming** — structured JSONL output for programmatic integration
- **Exit codes** — 0 (success), 1 (permanent failure), 75 (transient/retry)

## Usage in Hermes

Once installed, the skill is available automatically when the agent needs to delegate coding tasks:

```
"Use kimi-cli to refactor the auth module"
```

Or load directly:
```
/kimi-cli
```

## Related skills

- [claude-code](https://github.com/tmdgusya/hermes-agent) — Delegate to Anthropic's Claude Code
- [codex](https://github.com/tmdgusya/hermes-agent) — Delegate to OpenAI's Codex CLI

## License

MIT
