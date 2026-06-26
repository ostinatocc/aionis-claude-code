# @aionis/claude-code

Claude Code lifecycle integration for Aionis execution memory.

Use this package when MCP-only is not enough and you want Claude Code sessions,
subagents, and Agent Team tasks to pass through Aionis automatically.

Recommended Claude Code plugin setup:

```text
/plugin marketplace add https://github.com/ostinatocc/aionis-claude-code
/plugin install aionis@aionis-claude-code
/aionis:onboard
```

The plugin loads user-level lifecycle hooks, an Aionis MCP server, and slash
commands. After that, run `claude` from any project. Aionis derives a stable
workspace scope per project without requiring manual project setup.

CLI fallback:

```bash
npx @aionis/claude-code@latest onboard --base-url http://127.0.0.1:3101
```

`onboard` installs the same hook + MCP integration through user-level Claude
Code settings when you do not want to use plugins.

Hooks call Aionis through the SDK:

- `SessionStart`: injects a compact Aionis activation context.
- `UserPromptSubmit`: runs Aionis guide before every user prompt.
- `SubagentStart`: injects role-aware Aionis context into Claude Code
  subagents.
- `SubagentStop`: records subagent results as shared advisory handoff evidence.
- `TaskCreated` / `TaskCompleted`: records Claude Code Agent Team task
  boundaries and completed teammate handoffs.
- `PostToolUse` / `PostToolUseFailure`: records Agent/Bash/Edit/Write
  execution evidence. Agent tool returns also refresh parent context.
- `PostCompact`: records the compacted session summary as handoff evidence.
- `SessionEnd`: records a verified handoff only when files changed and
  validation passed; otherwise it skips writing execution memory to avoid
  generic session-end noise.

Version `0.3.0` maps Claude Code subagents and Agent Team tasks into Aionis
shared execution memory with stable workspace `team_id`, derived `agent_id`,
and role-aware guide calls. Aionis Runtime can compile those handoffs into
active execution context for future Claude Code agents in the same workspace.
Tool observation summaries remain bounded before Runtime write so large patch
payloads do not break the execution-memory contract.

MCP remains available for explicit tools such as `aionis_context`,
`aionis_record_step`, `aionis_flight_recorder`, and `aionis_snapshot`.

Check everything:

```bash
npx @aionis/claude-code@latest doctor --base-url http://127.0.0.1:3101
```

Project-only isolated install is still available:

```bash
npx @aionis/claude-code@latest install \
  --settings local \
  --claude-scope local \
  --base-url http://127.0.0.1:3101
```
