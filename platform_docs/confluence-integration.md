# Confluence Integration

Belfalas connects to your organization's Confluence instance, letting you browse, search, and edit Confluence documents directly from the Document Hub — no tab-switching required.

---

## How It Works

Your tenant admin connects one or more Confluence spaces to Belfalas. Once connected, those spaces appear under **External Spaces** in the Document Hub sidebar with a 🔗 link icon. You interact with them like any other space — search, browse, open, and edit — but the content lives in Confluence.

Each connected space uses one of two sync modes:

| Mode | Badge | What You See |
|---|---|---|
| **Proxy** | 🟡 yellow | Documents are fetched live from Confluence every time you open them. Nothing is stored locally. |
| **Cached** | 🟢 green | Documents are synced and stored locally as Markdown copies. You can pull latest or push changes back. |

The badge appears next to the space name in the sidebar so you always know which mode you're in.

---

## Browsing Confluence Spaces

1. Open the **Document Hub** (top nav → Documents)
2. In the sidebar, scroll to **External Spaces**
3. Click a Confluence space to browse its documents
4. Folders from Confluence appear at the top of the list — click to drill in
5. Click any document card to open it

In **proxy mode**, each click fetches the latest version directly from Confluence. In **cached mode**, you see the locally stored copy (which may be a few minutes behind).

### Search

Use the Document Hub search bar to search across all spaces including Confluence:

- **Cached spaces** — Searched using Belfalas's full-text search engine (fast, works on local copies)
- **Proxy spaces** — Searched using Confluence's native CQL query engine (always up to date)

---

## Editing Confluence Documents

### Proxy Mode

When you open a proxy document in the editor:

- The content is fetched live from Confluence
- Edits auto-save directly to Confluence (approximately every 5 seconds)
- A save indicator shows the current status: **saved**, **saving**, **unsaved**, or **error**
- Changes appear in Confluence immediately — there is no separate "push" step

**Limitations in proxy mode:**
- No version snapshots (versioning is managed by Confluence)
- Cannot publish to other Belfalas spaces (the document only lives in Confluence)
- No local offline access

### Cached Mode

When you open a cached document in the editor:

- You see the locally stored Markdown copy
- Edits are saved locally with full Belfalas versioning support
- Use **Pull Latest** to sync the latest content from Confluence
- Use **Push Local Copy** to send your changes back to Confluence
- You can publish cached documents to organizational spaces like any native document

---

## Creating Documents in Confluence

You can create new documents directly in a Confluence space from Belfalas:

1. Navigate to the Confluence space (or a folder within it)
2. Click **+ New Document**
3. Enter a title and content
4. The document is created in Confluence immediately

In proxy mode, the new page appears in Confluence right away. In cached mode, the page is created in Confluence and a local copy is stored automatically.

---

## Folder Navigation

Confluence spaces support the same folder hierarchy you see in Confluence:

- Confluence page trees are represented as folders in Belfalas
- Navigate into folders to see child pages
- Breadcrumbs show your current path
- In proxy mode, folder contents are fetched live
- You can create new folders and move documents between folders within the same space

---

## What You Cannot Do

These actions are managed by your tenant admin and are not available to regular users:

- Connect or disconnect Confluence spaces
- Change the sync mode (proxy vs. cached)
- Enable or disable agent access on a Confluence space
- Trigger a full sync of a cached space

If you need any of these, ask your tenant admin.

---

## Quick Reference

| Action | Proxy Mode | Cached Mode |
|---|---|---|
| Browse documents | ✅ Live from Confluence | ✅ Local copies |
| Search | ✅ Confluence CQL | ✅ Full-text local |
| Edit documents | ✅ Auto-saves to Confluence | ✅ Local edits, manual push |
| Version snapshots | ❌ Managed by Confluence | ✅ Full Belfalas versioning |
| Publish to org spaces | ❌ Not supported | ✅ Supported |
| Create documents | ✅ Creates in Confluence | ✅ Creates in Confluence + local copy |
| AI agent access | ✅ Controlled by admin | ✅ Controlled by admin |
