# Confluence Integration — Agent Guide

This guide describes how agents interact with Confluence-connected document spaces. No new tools are required — the existing `knowledge_base` tools work transparently across native and Confluence spaces. This guide covers the behavioral differences agents should understand.

---

## Overview

Confluence spaces connected to Belfalas appear as **external** spaces with `spaceType: "external"`. They operate in one of two modes:

| Mode | Agent Behavior |
|---|---|
| **Proxy** | `read_document` fetches live from Confluence. `search_documents` uses Confluence CQL. No versioning tools. |
| **Cached** | `read_document` auto-syncs if stale (>5 min). `search_documents` uses local full-text search. Full versioning. |

All 10 existing document tools work with Confluence spaces. The tools automatically detect the sync mode and adjust behavior.

---

## Access Rules for Confluence Spaces

The same access rules from the main Documents agent guide apply, plus:

1. **Space-level agent access** must be enabled by an admin on each Confluence space
2. **Global setting** — The admin setting `Allow Agent Sharing on Confluence Spaces` must be enabled for personal Confluence spaces
3. **Document-level toggle** — Still applies per-document (enabled by default)

If any layer is disabled, tools return an error explaining which toggle needs to be enabled.

---

## Tool Behavior by Sync Mode

### `search_documents`

**Proxy spaces:** When `space_slug` points to a proxy-mode Confluence space, the search is routed to Confluence's CQL engine. Results include titles, URLs, and content excerpts directly from Confluence.

**Cached spaces:** Standard full-text search against local Markdown copies. Same behavior as organizational spaces.

**No space filter:** When searching without `space_slug`, only native and cached spaces are searched. To search a proxy space, you must specify its `space_slug`.

```json
{
  "name": "search_documents",
  "arguments": {
    "query": "deployment checklist",
    "space_slug": "confluence-ops",
    "limit": 5
  }
}
```

### `read_document`

**Proxy documents:** Content is fetched live from Confluence on every call. You always get the latest version. If Confluence is unreachable, the tool returns an error.

**Cached documents:** The tool checks if the local copy is stale (>5 minutes since last sync). If stale, it automatically syncs before returning the content. If sync fails, the existing local copy is returned.

```json
{
  "name": "read_document",
  "arguments": {
    "space_slug": "confluence-dev",
    "doc_slug": "api-auth-design"
  }
}
```

### `create_document`

When `space_slug` points to an external space, the document is:
- Created in Confluence immediately (the page appears in the Confluence space)
- Status is `published` (not `draft`) since it lives in Confluence
- The tool returns a sync status confirmation

```json
{
  "name": "create_document",
  "arguments": {
    "title": "Incident Response Runbook",
    "content": "# Incident Response\n\n## Step 1: Assess severity...",
    "space_slug": "confluence-ops"
  }
}
```

### `update_document`

Works only on **cached** Confluence documents that the user owns. Proxy documents are edited by the user directly in the editor (auto-saves to Confluence).

### `manage_document_versions`

- **Cached spaces:** Full versioning support — create snapshots, list history
- **Proxy spaces:** Not supported — versioning is managed by Confluence. The tool will return an error.

### `manage_document_folders`

Works for both modes. Creating or deleting folders in an external space creates or deletes the corresponding Confluence page hierarchy.

### `move_document_to_folder`

Works for both modes. Moving a document updates its position in the Confluence page tree.

### `publish_document`

- **Cached documents:** Can be published to organizational spaces like native documents
- **Proxy documents:** Cannot be published to other spaces (the tool will reject the request)

### `manage_document_spaces`

The `list` action includes Confluence spaces in the results. External spaces show additional metadata: `spaceType: "external"`, sync mode, and the Confluence space key.

### `link_document_to_ticket`

Works identically for both native and Confluence documents.

---

## Common Workflows

### Search Confluence for context before implementing a ticket

```
search_documents { query: "database migration strategy", space_slug: "confluence-eng" }
  → Found 4 results from Confluence CQL
read_document { space_slug: "confluence-eng", doc_slug: "db-migration-playbook" }
  → Full content fetched live from Confluence
```

### Create a document directly in Confluence

```
create_document { 
  title: "Sprint 42 Retrospective",
  content: "# Sprint 42 Retro\n\n## What went well...",
  space_slug: "confluence-team"
}
  → Created in Confluence, synced immediately
```

### Find and read across mixed spaces

```
search_documents { query: "authentication flow" }
  → Returns results from native spaces + cached Confluence spaces

search_documents { query: "authentication flow", space_slug: "confluence-security" }
  → Returns results from Confluence CQL (proxy space)
```

---

## Error Handling

In addition to the standard document tool errors, Confluence-specific errors include:

| Error | Cause | Resolution |
|---|---|---|
| `"Confluence search failed: ..."` | Confluence is unreachable or CQL query failed | Retry; if persistent, the integration may need reconfiguration |
| `"Failed to fetch live content from Confluence: ..."` | Network issue or Confluence page was deleted | Verify the page still exists in Confluence |
| `"warning: Confluence sync failed: ..."` | Document was created locally but sync to Confluence failed | The document exists in Belfalas; sync will be retried |
| `"Agent access is disabled for space ..."` | Space-level or global agent access is off | Ask an admin to enable agent access on the space |
