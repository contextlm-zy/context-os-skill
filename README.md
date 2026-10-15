# context-os-skill

Official agent skill for [Context OS](https://app.contextlm.top) (ContextLM) — the personal AI knowledge base agents can query.

Teaches an agent how to drive the `workspace.*` tool catalog: ingest files and URLs, run cited RAG queries, and find the guest-facing share surfaces. Two access tracks share the same catalog — **Cloud API** (hosted content, remote MCP endpoint + cloud key) and **Local MCP** (`context-os-mcp` stdio, content stays on-device).

## Status

Staged in the monorepo at `packages/context-os-skill/` pending extraction to a standalone public repository (`contextlm/context-os-skill`, decision pending). Everything needed for a plugin install lives under this directory.

## Install

Once published as a plugin marketplace repo, in Claude Code:

```text
/plugin marketplace add contextlm/context-os-skill
/plugin install context-os@context-os-skill
```

For other harnesses, copy `skills/context-os/SKILL.md` into the harness's skill directory, or point it at this repo.

## Before first use

The skill needs a project key and `workspace_id` — one human step:

1. Open a project → share center → **Cloud API** tab → **Create cloud key** (grant `index`/`query`) → **Copy agent pack**.
2. Paste the Agent Pack JSON into the agent session.

Full walkthrough: https://app.contextlm.top/help/api-access/agent-quickstart
Agent API reference: https://app.contextlm.top/help/api-access/agents
