# Documents — Agent Guide

This guide describes the Documents & Knowledge Base feature from an **agent's perspective**: what documents are, how to discover and read them, how to create and update content, and the exact tools available for each operation.

---

## Overview

Documents are a shared, versioned knowledge base built into Belfalas. They live inside **spaces** — containers that control visibility and access. There are three space types:

| Space Type | Description | Agent Access |
|---|---|---|
| **Personal** | Private to each user, auto-created | Always accessible for the current user's space |
| **Organizational** | Shared within a tenant, admin-created | Only if `agent_access_enabled` is true on the space |
| **External (Confluence)** | Connected to a Confluence instance | Only if `agent_access_enabled` is true on the space |
| **Global** | Platform-wide, read-only for tenants | Read-only, always accessible |

External spaces operate in one of two sync modes:

| Mode | Behavior for Agent Tools |
|---|---|
| **Proxy** | Content is fetched live from Confluence on every read. Search uses Confluence CQL. No local versioning. |
| **Cached** | Content is stored locally as Markdown. Search uses full-text. Auto-syncs if stale (>5 min). |

Documents follow a lifecycle: **Draft → Published → Archived**. Agents primarily work with **draft** documents in the current user's **personal space**. Publishing is a separate, confirmation-gated action.

Each document has an **AI Agent Access** toggle (enabled by default). If the document owner disables it, all agent tools will refuse to interact with that document.

---

## Access Rules

Before calling any tool, understand these constraints — they are enforced by every tool:

1. **Agent access toggle (document-level):** The document's `agentAccessEnabled` flag must be `true`. If disabled, the tool returns an error asking the owner to re-enable it.
2. **Agent access toggle (space-level):** For organizational and external spaces, the space's `agent_access_enabled` flag must be `true`. If disabled, an admin must enable it.
3. **Ownership for writes:** Agents can only create, update, move, or version documents **owned by the current user**. You cannot modify another user's document.
4. **Draft-only updates:** Agents can only update documents with `status: "draft"`. Published or archived documents must be reverted to draft by the user before the agent can edit them.
5. **Publishing requires confirmation:** The `publish_document` tool has `_requires_confirmation: true` — the user must approve before execution proceeds.
6. **External space writes:** When creating documents in an external (Confluence) space, the document is automatically synced to Confluence. The document is created as `published` (not `draft`) since it lives in Confluence.

---

## Available Tools

All document tools belong to the `knowledge_base` category. Each tool returns `{ success, result, summary }`.

### 1. `search_documents` — Find documents

Search the knowledge base using full-text search. Use this first to find relevant specs, runbooks, or architecture docs before working on a task.

For **proxy-mode Confluence spaces**, the search automatically uses Confluence's CQL query engine instead of local full-text search. Results include titles, URLs, and content snippets from Confluence.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `query` | string | yes | Search query (natural language or keywords) |
| `space_slug` | string | no | Limit search to a specific space by slug (e.g. `"engineering"`, `"legal"`) |
| `status` | string | no | Filter by status: `draft`, `published`, `archived`. Defaults to all. |
| `limit` | number | no | Max results (default: 10, max: 50) |

**Example call:**
```json
{
  "name": "search_documents",
  "arguments": {
    "query": "API authentication architecture",
    "space_slug": "engineering",
    "limit": 5
  }
}
```

**Returns:** A ranked list of matching documents with title, space, word count, status, and a text snippet with highlighted matches.

---

### 2. `read_document` — Read a document's content

Read the full markdown content of a specific document. You can look up by UUID or by space + document slugs.

For **Confluence documents**: proxy-mode documents are fetched live from Confluence on every read. Cached-mode documents are auto-synced if the local copy is older than 5 minutes.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `document_id` | string | no | Document UUID (if known) |
| `space_slug` | string | no | Space slug (used with `doc_slug`) |
| `doc_slug` | string | no | Document slug (used with `space_slug`) |

You must provide either `document_id` **or** both `space_slug` + `doc_slug`.

**Example calls:**
```json
{
  "name": "read_document",
  "arguments": {
    "document_id": "d8f3c2a1-..."
  }
}
```
```json
{
  "name": "read_document",
  "arguments": {
    "space_slug": "engineering",
    "doc_slug": "api-auth-design"
  }
}
```

**Returns:** Document title, metadata (space, status, version, author, word count, tags), and the full markdown content (truncated at 10,000 characters).

---

### 3. `create_document` — Create a new document

Create a new document. By default, documents are created as drafts in the user's **personal space**. You can optionally specify a `space_slug` to create in an organizational or external (Confluence) space.

When creating in an **external space**, the document is automatically synced to Confluence and created with `published` status.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `title` | string | yes | Document title |
| `content` | string | yes | Markdown content |
| `space_slug` | string | no | Target space slug (e.g. `"engineering"`, `"confluence-dev"`). Defaults to personal space. |
| `visibility` | string | no | `private` (default) or `workspace` |
| `tags` | string[] | no | Tags for categorization (e.g. `["api", "architecture"]`) |

**Example calls:**
```json
{
  "name": "create_document",
  "arguments": {
    "title": "API Rate Limiting Spec",
    "content": "# API Rate Limiting\n\n## Overview\n\nThis document describes...",
    "tags": ["api", "rate-limiting", "spec"]
  }
}
```
```json
{
  "name": "create_document",
  "arguments": {
    "title": "Deploy Runbook",
    "content": "# Deployment Steps\n\n...",
    "space_slug": "confluence-ops"
  }
}
```

**Returns:** Confirmation with the document ID, title, word count, space name, and sync status (for external spaces).

---

### 4. `update_document` — Update a draft document

Update the title, content, or tags of a draft document owned by the current user. Every update is versioned automatically.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `document_id` | string | yes | Document UUID |
| `title` | string | no | Updated title |
| `content` | string | no | Updated markdown content |
| `change_summary` | string | no | Brief description of what changed (e.g. `"Updated API endpoints"`) |
| `tags` | string[] | no | Updated tags array |

At least one of `title`, `content`, or `tags` should be provided alongside `document_id`.

**Example call:**
```json
{
  "name": "update_document",
  "arguments": {
    "document_id": "d8f3c2a1-...",
    "content": "# API Rate Limiting\n\n## Overview\n\nUpdated content...",
    "change_summary": "Added retry-after header section"
  }
}
```

**Constraints:**
- Document must be in `draft` status
- Document must be owned by the current user
- Agent access must be enabled on the document

**Returns:** Updated title, new version number, and list of changed fields.

---

### 5. `publish_document` — Publish to an organizational space

Publish a document to an organizational space. This **requires user confirmation** before execution.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `document_id` | string | yes | UUID of the document to publish |
| `space_id` | string | yes | UUID of the target organizational space |
| `version_number` | number | no | Specific version to publish (defaults to current) |
| `folder_id` | string | no | UUID of a folder within the target space |

**Example call:**
```json
{
  "name": "publish_document",
  "arguments": {
    "document_id": "d8f3c2a1-...",
    "space_id": "a1b2c3d4-..."
  }
}
```

**Constraints:**
- Document must be owned by the current user
- Both the document and the target space must have agent access enabled
- **Requires user confirmation** (`_requires_confirmation: true`)

**Returns:** Confirmation with the published version and target space name.

---

### 6. `link_document_to_ticket` — Link a document to a ticket

Create a bidirectional link between a document and a ticket. Use this to associate specs with implementation tickets, runbooks with deployments, or post-mortems with incidents.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `document_id` | string | yes | Document UUID |
| `ticket_id` | string | yes | Ticket UUID |
| `link_type` | string | no | One of: `reference` (default), `spec`, `runbook`, `post_mortem`, `requirement` |

**Example call:**
```json
{
  "name": "link_document_to_ticket",
  "arguments": {
    "document_id": "d8f3c2a1-...",
    "ticket_id": "t9e8f7a6-...",
    "link_type": "spec"
  }
}
```

**Returns:** Confirmation with the link type and link ID.

---

### 7. `manage_document_spaces` — Create, list, or update spaces

Manage organizational document spaces. Spaces are containers that group related documents (e.g. "Engineering", "Legal", "Product").

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `action` | string | yes | `create`, `list`, or `update` |
| `name` | string | create/update | Space name |
| `space_id` | string | update | Space UUID |
| `description` | string | no | Space description |
| `icon` | string | no | Emoji icon (e.g. `"📚"`) |
| `visibility` | string | no | `private`, `workspace` (default), or `published` |
| `agent_access_enabled` | boolean | no | Enable/disable agent access (defaults to `false` for new spaces) |

**Example calls:**
```json
{
  "name": "manage_document_spaces",
  "arguments": {
    "action": "list"
  }
}
```
```json
{
  "name": "manage_document_spaces",
  "arguments": {
    "action": "create",
    "name": "Engineering",
    "description": "Technical architecture and design docs",
    "icon": "⚙️",
    "visibility": "workspace",
    "agent_access_enabled": true
  }
}
```

**Returns:**
- `list`: All accessible spaces with name, slug, document count, visibility, and agent access status
- `create`: Created space with name, slug, visibility, and ID
- `update`: Confirmation of update

---

### 8. `manage_document_folders` — Create, list, or delete folders

Manage folders within a document space. Folders can be nested under other folders.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `action` | string | yes | `create`, `list`, or `delete` |
| `space_id` | string | create/list | Space UUID |
| `name` | string | create | Folder name |
| `parent_id` | string | no | Parent folder UUID for nesting |
| `folder_id` | string | delete | Folder UUID to delete |

**Example calls:**
```json
{
  "name": "manage_document_folders",
  "arguments": {
    "action": "list",
    "space_id": "a1b2c3d4-..."
  }
}
```
```json
{
  "name": "manage_document_folders",
  "arguments": {
    "action": "create",
    "space_id": "a1b2c3d4-...",
    "name": "Architecture Decisions",
    "parent_id": null
  }
}
```

**Note:** When a folder is deleted, documents inside it are moved to the space root — they are not deleted.

**Returns:**
- `list`: Folder tree with names, document counts, nesting, and IDs
- `create`: Created folder with name, slug, and ID
- `delete`: Confirmation that the folder was removed

---

### 9. `manage_document_versions` — Create or list version snapshots

Create a point-in-time version snapshot of a document, or list the version history.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `action` | string | yes | `create` or `list` |
| `document_id` | string | yes | Document UUID |
| `change_summary` | string | no | Description of what changed (for `create`) |
| `limit` | number | no | Max versions to return (for `list`, default: 20, max: 50) |

**Example calls:**
```json
{
  "name": "manage_document_versions",
  "arguments": {
    "action": "create",
    "document_id": "d8f3c2a1-...",
    "change_summary": "Finalized rate limiting thresholds"
  }
}
```
```json
{
  "name": "manage_document_versions",
  "arguments": {
    "action": "list",
    "document_id": "d8f3c2a1-...",
    "limit": 10
  }
}
```

**Constraints:**
- Document must be owned by the current user
- Agent access must be enabled
- If all version slots are occupied by published/kept versions, creation fails with `ALL_VERSIONS_PUBLISHED`

**Returns:**
- `create`: New version number, word count, and change summary
- `list`: Chronological version history with version numbers, word counts, dates, summaries, and authors

---

### 10. `move_document_to_folder` — Move a document within its space

Move a document into a folder within its current space, or back to the space root.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `document_id` | string | yes | Document UUID |
| `folder_id` | string | no | Target folder UUID, or `null`/empty to move to space root |

**Example call:**
```json
{
  "name": "move_document_to_folder",
  "arguments": {
    "document_id": "d8f3c2a1-...",
    "folder_id": "f5e6d7c8-..."
  }
}
```

**Constraints:**
- Document must be owned by the current user
- The target folder must belong to the same space as the document

**Returns:** Confirmation with the document title and destination folder name (or "space root").

---

## Common Workflows

### Research before implementing a ticket

1. **Search** for related documents using `search_documents` with keywords from the ticket
2. **Read** the most relevant results using `read_document`
3. Use the content to inform your implementation

```
search_documents { query: "authentication middleware" }
  → Found 3 docs, read the top result
read_document { document_id: "..." }
  → Full content of the auth middleware design doc
```

### Create a design document for a new feature

1. **Create** the document as a draft using `create_document`
2. **Create a version snapshot** with `manage_document_versions` once the content is stable
3. Optionally **link it to a ticket** using `link_document_to_ticket`
4. Optionally **publish** to an org space using `publish_document` (requires user confirmation)

```
create_document { title: "...", content: "...", tags: [...] }
  → Draft created in personal space, ID returned
manage_document_versions { action: "create", document_id: "...", change_summary: "Initial draft" }
  → v1 snapshot created
link_document_to_ticket { document_id: "...", ticket_id: "...", link_type: "spec" }
  → Linked as spec
publish_document { document_id: "...", space_id: "..." }
  → [user confirms] → Published to org space
```

### Update an existing document after changes

1. **Read** the current document using `read_document`
2. **Update** the content using `update_document` with a change summary
3. **Create a version snapshot** if this is a meaningful checkpoint

```
read_document { document_id: "..." }
  → Current content
update_document { document_id: "...", content: "...", change_summary: "Added error handling section" }
  → Updated to v3
manage_document_versions { action: "create", document_id: "...", change_summary: "Error handling complete" }
  → Snapshot v3 created
```

### Organize documents into folders

1. **List spaces** to find the target space using `manage_document_spaces`
2. **List folders** in that space using `manage_document_folders`
3. **Create a folder** if needed
4. **Move the document** using `move_document_to_folder`

```
manage_document_spaces { action: "list" }
  → Engineering space ID: a1b2...
manage_document_folders { action: "list", space_id: "a1b2..." }
  → 3 folders listed
manage_document_folders { action: "create", space_id: "a1b2...", name: "API Specs" }
  → Folder created, ID: f5e6...
move_document_to_folder { document_id: "d8f3...", folder_id: "f5e6..." }
  → Moved into "API Specs"
```

---

## Error Handling

All tools return `{ success: false, result: "<message>" }` on failure. Common errors:

| Error | Cause | Resolution |
|---|---|---|
| `"Agent access is disabled for this document"` | Document owner turned off the AI Agent Access toggle | Ask the user to re-enable it in document settings |
| `"Agent access is disabled for space ..."` | Org or external space has `agent_access_enabled = false` | An admin must enable agent access on the space |
| `"Agents can only update documents owned by the current user"` | Attempting to modify another user's document | Only the document owner's agent can modify it |
| `"Cannot update a published document"` | Document is not in draft status | User must unpublish/revert to draft first |
| `"Document not found"` | Invalid ID or the document was deleted | Verify the ID or search again |
| `"Space not found"` | Invalid space ID or slug | Use `manage_document_spaces { action: "list" }` to find valid spaces |
| `"ALL_VERSIONS_PUBLISHED"` | All version slots are occupied by published/kept versions | User must replace or unpublish a version first |

---

## Key Concepts

- **Slugs** are URL-friendly identifiers auto-generated from names (e.g. `"API Auth Design"` → `"api-auth-design"`). Use them for human-readable lookups via `read_document` with `space_slug` + `doc_slug`.
- **Visibility** controls default access: `private` (explicit access only), `workspace` (all org members can read), `published` (all org members can read, displayed prominently).
- **Version snapshots** are immutable point-in-time captures. The current auto-saved content is separate from version snapshots. Creating a version freezes the current state.
- **Publishing** creates a snapshot visible to space readers. It does not move the document — it remains in the author's personal space. A single document can be published to multiple spaces simultaneously.
- **Ticket links** are bidirectional — they appear on both the document and the ticket. Link types (`spec`, `runbook`, `post_mortem`, `requirement`, `reference`) categorize the relationship.
- **Confluence integration** — External spaces connect to Confluence. Proxy mode fetches live; cached mode stores locally. Creating documents in external spaces automatically syncs them to Confluence. Agent tools work transparently across both modes.
