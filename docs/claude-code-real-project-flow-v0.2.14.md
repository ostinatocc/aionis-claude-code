# Claude Code Real Project Flow - v0.2.14

This note records a real, isolated Claude Code run against the published
`aionis@aionis-claude-code` plugin v0.2.14.

It is evidence for the Claude Code adapter layer only. It does not change
Aionis Runtime behavior and should not be treated as a core Runtime rule.

## Environment

- Plugin release: https://github.com/ostinatocc/aionis-claude-code/releases/tag/v0.2.14
- Claude Code plugin: `aionis@aionis-claude-code` v0.2.14, user scope, enabled
- MCP status:
  - `plugin:aionis:aionis` connected
  - `aionis-local` connected to `http://127.0.0.1:3101`
- Runtime URL: `http://127.0.0.1:3101`
- Isolated repo: `/tmp/aionis-claude-code-real-flow-v214`
- Scope mode: `workspace`

## Fixture

The isolated project starts with one failing test:

```js
// src/route.js
export function routeStatus() {
  return "legacy";
}
```

```js
// test/route.test.js
assert.equal(routeStatus(), "ready");
```

Baseline result:

```text
1 fail: 'legacy' !== 'ready'
```

## Session 1 - Subagent Inspection and Fix

Prompt:

```text
Use the verifier subagent via the Agent tool to inspect this repository and
identify why npm test fails. Then, as the main agent, make the minimal code
change needed to pass the tests. Run npm test after the change. Do not modify
unrelated files.
```

Observed behavior:

- `SessionStart` reported Aionis active and Runtime reachable.
- `UserPromptSubmit` injected an empty-history Aionis execution context.
- Claude Code used the `Agent` tool with `subagent_type: verifier`.
- `SubagentStart:verifier` injected role-aware Aionis context for the verifier.
- The verifier ran `npm test`, found the failing assertion, and reported:
  - failing test: `test/route.test.js`
  - target implementation: `src/route.js`
  - expected value: `"ready"`
  - actual value: `"legacy"`
  - minimal fix: change `"legacy"` to `"ready"`
- `SubagentStop` recorded the verifier result as shared handoff evidence.
- `PostToolUse:Agent` refreshed parent context after the Agent result.
- The main agent edited only `src/route.js`.
- The main agent ran `npm test`.
- Final result: `1 pass, 0 fail`.

The isolated repo commit after the session:

```text
6db874f Fix route ready state
```

## Session 2 - Cross-Session Recovery

The second Claude Code invocation was started in the same isolated project
after committing the session 1 fix.

Prompt:

```text
This is a new session in the same project. Use the Aionis context if available.
Summarize what was already fixed and validated in the previous session, then
run npm test once to confirm the current state. Do not edit files.
```

The `UserPromptSubmit` hook injected Aionis execution memory before Claude Code
processed the prompt. The injected contract included:

- `ACTIVE_TARGETS`: `/private/tmp/aionis-claude-code-real-flow-v214/src/route.js`
- `SHOULD_CONTINUE`: verified implementation route, `npm test 2>&1 passed`
- `INSPECT_BEFORE_USE`: prior verifier finding and candidate workflow evidence
- `DO_NOT_USE`: prior failed `npm test` branch
- `BASE_AIONIS_CONTEXT`: compact current/procedure/inspect/avoid context

Claude Code used that context to summarize the previous session:

- the prior failure was `routeStatus returns the active ready state`
- the failure was caused by `legacy` vs `ready`
- `src/route.js` had already been fixed
- the previous `npm test` passed

It then ran `npm test` once and did not edit files.

Final result:

```text
1 pass, 0 fail
```

## What This Proves

This run verifies that the Claude Code adapter can:

- install as a user-level Claude Code plugin;
- expose an Aionis MCP server;
- inject Aionis execution context on session start and user prompt submit;
- map Claude Code subagents into Aionis role-aware execution memory;
- record subagent handoff evidence;
- refresh parent context after `Agent` tool results;
- recover a previous verified route in a new Claude Code session;
- keep failed test history separate from the active continuation route.

## Remaining Issue

During the real Claude Code run, two `PostToolUse:Bash` events returned:

```text
Aionis Claude Code hook skipped: Aionis request failed: 400 /v1/observe
```

The hook returned exit code 0, so Claude Code was not blocked. Manual replay of
the same class of `PostToolUse:Bash` payload succeeded against the same Runtime.

Current interpretation:

- this did not affect context injection or task completion;
- it is not a deterministic Bash payload schema failure;
- the adapter should keep treating Runtime write failures as non-blocking;
- the SDK/Runtime error path should later expose the response body so 400s can
  be diagnosed without reproducing under a debugger.

## Conclusion

v0.2.14 is a working Claude Code adapter baseline for real project flows:
subagent inspection, parent-agent implementation, verified session handoff, and
cross-session execution-memory recovery all worked in an isolated repository.
