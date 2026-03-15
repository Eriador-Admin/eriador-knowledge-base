# Confluence Integration — Administration Guide

This guide covers how tenant admins connect, manage, and control Confluence spaces within Belfalas.

---

## Prerequisites

Before connecting a Confluence space, ensure:

1. **A Confluence integration is configured** — Go to Settings → Integrations and add a Confluence (Atlassian) integration with your domain and credentials
2. **You have admin access** — Only tenant admins can connect and manage external spaces

---

## Connecting a Confluence Space

1. Navigate to **Admin → Document Management**
2. Click **+ Add** in the toolbar, then select **🔗 Connect Confluence**
3. The connect modal opens with these steps:

### Step 1: Select Integration

Choose the Confluence integration from the dropdown. It displays as `{domain} ({email})`. Once selected, available spaces are loaded automatically.

### Step 2: Select Space

Pick from the dropdown of available Confluence spaces (`{name} ({key})`). If the space doesn't appear in the list, switch to **manual entry** and type the space key directly. The system validates the key and shows "✓ Found" or an error.

### Step 3: Choose Sync Mode

| Mode | Description | When to Use |
|---|---|---|
| **Cached** | Syncs all pages locally as Markdown. Full-text search, versioning, cross-space publishing. | Teams that want fast local access and Belfalas features on top of Confluence content |
| **Proxy** | Zero local storage. Every read fetches live from Confluence. Search uses Confluence CQL. | Teams that need real-time accuracy and don't want data duplication |

### Step 4: Agent Access

Toggle **Enable Agent Access** to allow AI agents to search and reference documents in this space. If the global agent access setting is not yet enabled, it will be auto-enabled when you toggle this on.

### Step 5: Connect

Click **Connect**. The space appears in the sidebar under **External Spaces** with a 🔗 icon and a sync mode badge (yellow for proxy, green for cached).

---

## Managing Connected Spaces

Select any external space in the Document Management sidebar to view its detail panel.

### Space Detail Panel

The detail panel shows:

- **Header** — Space name with **Confluence** badge and sync mode badge
- **Space Settings** — Agent Access toggle
- **Connection Details** — Space Key, Space ID, Integration ID, sync mode
- **Access Permissions** — User and group permission grants

### Disconnecting a Space

1. Select the space in the sidebar
2. Click **Disconnect** in the header
3. Confirm the action

Disconnecting removes the space from Belfalas. For cached spaces, all local copies are deleted. The original content in Confluence is never affected.

---

## Access Permissions

External spaces use the same permission model as organizational spaces:

| Permission Level | What It Allows |
|---|---|
| Read | View documents in the space |
| Write | Create, edit, and update documents |
| Admin | Full control — manage permissions, folders, delete documents |

**Granting access:**
1. Select the connected space
2. Scroll to **Access Permissions**
3. Click **+ Add** and select a user or group
4. Choose the permission level
5. Click **Grant**

**Revoking access:**
- Click the revoke button next to any permission entry

> **Tip:** Use groups for team-wide access instead of individual user grants.

---

## Agent Access Controls

Agent access for Confluence spaces is governed by a three-layer hierarchy:

```
Admin Settings (global)  →  Space Setting  →  Document Setting
```

1. **Global setting** — `Allow Agent Sharing on Confluence Spaces` in Document Settings. Controls whether agents can access any personal Confluence space.
2. **Space-level toggle** — Each connected space has its own Agent Access toggle in the detail panel. Must be enabled for agents to access documents in that space.
3. **Document-level toggle** — Individual documents have an AI Agent Access toggle (enabled by default). Users can disable it for sensitive content.

All three layers must allow access for an agent to read a document.

> When you enable agent access on a space and the global setting is off, Belfalas auto-enables the global setting for you.

---

## Cached vs. Proxy — Admin Considerations

| Aspect | Cached | Proxy |
|---|---|---|
| **Local storage** | Full Markdown copies of all pages | None |
| **Sync operations** | Manual sync per-space or per-document | N/A (always live) |
| **Search** | Belfalas full-text search | Confluence CQL |
| **Versioning** | Belfalas version snapshots available | Managed by Confluence |
| **Cross-space publishing** | Supported | Not supported |
| **Data freshness** | May lag behind Confluence (pull to refresh) | Always current |
| **Offline access** | Yes (local copies) | No |

Choose proxy mode when real-time accuracy matters and your team always has connectivity. Choose cached mode when you want full Belfalas feature coverage including versioning, search, and cross-space publishing.

---

## Document Settings for Confluence

The **Document Settings** tab includes these Confluence-specific controls:

| Setting | Description |
|---|---|
| **Allow Confluence Document Sharing** | Whether users can share Confluence-sourced documents via the Share modal |
| **Allow Agent Sharing on Confluence Spaces** | Whether AI agents can access documents in personal Confluence spaces |
| **Allow User Confluence Connections** | Whether regular users can connect their own Confluence accounts |

---

## Troubleshooting

| Problem | Cause | Fix |
|---|---|---|
| Space not appearing in dropdown | Integration credentials may be expired or space is restricted | Verify credentials in Settings → Integrations |
| "Validation failed" on manual entry | Space key doesn't exist or is inaccessible | Check the space key in Confluence and ensure the integration has access |
| Documents not loading in proxy mode | Confluence is unreachable or rate-limited | Check connectivity; try again after a moment |
| Agent can't access Confluence docs | One of the three access layers is disabled | Check global setting, space toggle, and document toggle |
