# Aionis Claude Code

Claude Code plugin, MCP bridge wiring, and lifecycle hooks for Aionis execution
memory.

Use this repo when you want Claude Code sessions, subagents, and Agent Team
tasks to receive governed execution context, record tool outcomes, and expose
Aionis MCP tools such as context, handoff, snapshot, measure, and Flight
Recorder.

Claude Code prompt injection uses the public SDK AgentContext path
`execution.guideAgentContextForRole().agent_prompt`. This repository is an
integration transport, not a separate Aionis context compiler.

Runtime core lives in [ostinatocc/Aionis](https://github.com/ostinatocc/Aionis).
This repo publishes the Claude Code integration package.

## Install

Start an isolated local Aionis Runtime:

```bash
npx aionis setup .aionis-runtime --with-claude-code
cd .aionis-runtime
npm run -s lite:start
```

Then in Claude Code:

```text
/plugin marketplace add https://github.com/ostinatocc/aionis-claude-code
/plugin install aionis@aionis-claude-code
/aionis:doctor
```

The plugin defaults to `http://127.0.0.1:3101`, matching
`npx aionis setup --with-claude-code`.

Version `0.3.0` maps Claude Code subagents and Agent Team tasks into Aionis
shared execution memory. Subagent start events receive role-aware guide context,
subagent/team completion events write advisory handoffs, and Agent tool returns
refresh parent context so the next Claude Code agent can see governed state
without reading raw history.

## Verified Flow

The v0.3.0 adapter keeps the isolated real-project flow: one session can use a
verifier subagent, record validated execution evidence, and let a later session
recover the governed route from Aionis without reading raw history.

See [docs/claude-code-real-project-flow-v0.2.14.md](docs/claude-code-real-project-flow-v0.2.14.md).

## What It Adds

- User-level Claude Code plugin install.
- Aionis MCP server named `aionis`.
- Lifecycle hooks for `SessionStart`, `UserPromptSubmit`, `PostToolUse`,
  `PostToolUseFailure`, `SubagentStart`, `SubagentStop`, `TaskCreated`,
  `TaskCompleted`, compaction, and session end.
- Verified session handoff records for successful file-changing sessions.
- Shared execution memory across Claude Code subagents and Agent Teams using
  Aionis `team_id`, derived `agent_id`, and role-aware guide calls.
- Slash commands: `/aionis:onboard`, `/aionis:doctor`, `/aionis:status`.
- Stable workspace identity storage outside individual repos.

## CLI Onboarding

Use the CLI onboarding command for scripted setup:

```bash
npx @aionis/claude-code@latest onboard --base-url http://127.0.0.1:3101
```

## Development

```bash
npm install
npm run -s build
npm test
npm run -s plugin:validate
```

Local plugin testing:

```text
/plugin marketplace add /Volumes/ziel/aionis-claude-code
/plugin install aionis@aionis-claude-code
/aionis:doctor
```

## Package

This repo publishes:

- `@aionis/claude-code`: CLI onboarding and hook implementation used by the
  plugin wrapper scripts.

The package depends on the public `@aionis/sdk` package and talks to Aionis
Runtime through the product HTTP API.
