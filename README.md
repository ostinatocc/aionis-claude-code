# Aionis Claude Code

Claude Code plugin, MCP bridge wiring, and lifecycle hooks for Aionis execution
memory.

Use this repo when you want Claude Code to receive governed execution context
before prompts, record Bash/Edit/Write outcomes after tool use, and expose
Aionis MCP tools such as context, handoff, snapshot, measure, and Flight
Recorder.

Runtime core lives in [ostinatocc/Aionis](https://github.com/ostinatocc/Aionis).
This repo is the Claude Code adapter layer only.

## Install

Start an isolated local Aionis Runtime:

```bash
npx @aionis/create@latest .aionis-runtime --with-claude-code
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
`@aionis/create --with-claude-code`.

## What It Adds

- User-level Claude Code plugin install.
- Aionis MCP server named `aionis`.
- Lifecycle hooks for `SessionStart`, `UserPromptSubmit`, `PostToolUse`,
  `PostToolUseFailure`, compaction, and session end.
- Slash commands: `/aionis:onboard`, `/aionis:doctor`, `/aionis:status`.
- Stable workspace identity storage outside individual repos.

## CLI Fallback

If you do not want to use Claude Code plugins:

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

- `@aionis/claude-code`: CLI fallback and hook implementation used by the
  plugin wrapper scripts.

The package depends on the public `@aionis/sdk` package and talks to Aionis
Runtime through the product HTTP API.
