---
name: context-os
description: Query and manage a user's Context OS knowledge base (ContextLM). Use when the user wants to ask questions against their own documents, upload files/URLs for cited Q&A, organize sources into a project, or open a project for guest Q&A via a public link. Requires a Context OS key (cloud or local) and a workspace_id.
---

# Context OS

Context OS (by ContextLM) is a personal AI knowledge base. A **project** (`workspace_id`) holds sources; agents ingest, retrieve, and query it through the `workspace.*` tool catalog over the MCP-compatible JSON-RPC endpoint. Two access tracks share the same catalog:

| Track | Content | Endpoint | Credential |
| --- | --- | --- | --- |
| Cloud API | Hosted on `app.contextlm.top` | `POST {api_base}/api/v1/mcp` (JSON-RPC; `Authorization: Bearer <cloud_key>`) | Cloud key, scoped to one project |
| Local MCP | Stays on the user's machine | `context-os-mcp` (stdio) → `http://127.0.0.1:18080` | Local key from the local instance |

## Prerequisites — check before acting

A project key (`api_key` + `workspace_id`) is required for every `workspace.*` call — put `workspace_id` in `arguments` each time.

- **If the user has not provided an Agent Pack or key**: ask them to open their project → share center → **Cloud API** tab → **Create cloud key** → **Copy agent pack**, then paste the JSON here. One human step; do not improvise credentials. Point them to the quickstart if needed: `https://app.contextlm.top/help/api-access/agent-quickstart`.
- **If there is no project yet**: project keys cannot provision one (`workspace_key_cannot_call_org_tools`). The human creates the project in the product UI first, then issues a key for it. `account.create_workspace` exists only for user-session credentials — not for project keys.
- **If content must stay on the user's machine**: use the Local MCP track instead of uploading.

## Decision flow

1. **Ask a question against the project** → `workspace.rag_query` (`query` = the user's question). Answers cite source documents.
2. **Add a file** → orchestrate: `workspace.create_upload` → HTTP PUT the bytes to `upload_url` → `workspace.complete_upload` → poll `workspace.document_status` until `completed` → then `rag_query`.
3. **Add a web page / URL** → `workspace.add_url_source` → poll `document_status` → query.
4. **See what is indexed** → `workspace.list_sources`.
5. **Web search instead of the knowledge base** → `workspace.search_query`.
6. **Share the project for guest Q&A** → `workspace.share_*` tools need a **user session**, not a project key (`api_key_forbidden`). Tell the user to open the share center in the UI instead — the link lets guests ask without an account, owner pays.
7. **Check remaining share slots** → `account.share_quota` (user session only, same restriction).

## Failure handling

- `tools/call` failures return HTTP 200 with a JSON-RPC `error` object. Read `error.data.agent_operation_guide` first and adjust the call accordingly before retrying.
- Common codes: `workspace_scope_mismatch` (wrong `workspace_id` for this key), `missing_permission` (key lacks `index`/`query`), `docscope_required` (no completed sources yet — ingest first), `api_key_forbidden` (called a user-session tool with a project key).
- Full reference, tool catalog, and error table: `https://app.contextlm.top/help/api-access/agents` (also as `.md`).

## Boundaries

- Never invent credentials, `workspace_id`s, or document IDs. Read them from the Agent Pack / local config the user supplies.
- Guest-facing share links are a cloud capability; local content must be uploaded to a cloud project before it can be shared.
