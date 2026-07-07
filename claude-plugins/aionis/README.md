# Aionis Claude Code Plugin

This plugin gives Claude Code Aionis execution memory through two paths:

- Lifecycle hooks that inject governed context and record tool, subagent, and
  Agent Team outcomes.
- MCP tools for explicit context, handoff, Memory Firewall, snapshots, and Flight Recorder.

The injected prompt context is the SDK AgentContext
`execution.guideAgentContextForRole().agent_prompt`; the plugin is a Claude Code
transport for that contract, not a separate context surface.

## Install From This Marketplace

From Claude Code:

```text
/plugin marketplace add https://github.com/ostinatocc/aionis-claude-code
/plugin install aionis@aionis-claude-code
/aionis:onboard
```

Runtime should be reachable before you run the doctor command. For the
recommended isolated Claude Code Runtime, install with:

```bash
npx @aionis/create@latest .aionis-runtime --with-claude-code
cd .aionis-runtime
npm run -s lite:start
```

For local development of this repo:

```text
/plugin marketplace add /Volumes/ziel/aionis-claude-code
/plugin install aionis@aionis-claude-code
/aionis:doctor
```

## Runtime URL

The plugin defaults to the isolated Claude Code Runtime URL:

```text
http://127.0.0.1:3101
```

Set `AIONIS_BASE_URL` before starting Claude Code to point at a different
Runtime. A plain Aionis Runtime still defaults to `http://127.0.0.1:3001`;
the `@aionis/create --with-claude-code` path writes `PORT=3101` so plugin and
Runtime match out of the box.

## Scope

The plugin defaults to `AIONIS_SCOPE_FROM=workspace` and `AIONIS_WORKSPACE_ID_STORE=user`.
That gives each project a stable Aionis scope without writing identity files into every repo.

## Claude Code Multi-Agent Events

The plugin records ordinary Claude Code sessions and Claude Code multi-agent
surfaces into the same Aionis workspace memory:

- `UserPromptSubmit` receives governed execution context before the main Claude
  Code prompt.
- `SubagentStart` receives role-aware context for the subagent.
- `SubagentStop` writes the subagent result as shared advisory handoff evidence.
- `TaskCreated` / `TaskCompleted` record Agent Team task boundaries and
  teammate completion handoffs.
- `PostToolUse` for the `Agent` tool records the delegated result and refreshes
  parent context.

Aionis does not orchestrate Claude Code agents. Claude Code still manages
subagents and teams; Aionis supplies shared execution memory, admission
decisions, handoff evidence, and Flight Recorder traces.
