# Workspace Templates — Administration Guide

This guide covers template administration for **tenant admins**. For end-user documentation on browsing and scaffolding templates, see the Workspace Templates user guide.

---

## Overview

Tenant admins have access to a Template Administration dashboard that provides visibility into template usage across the organization:

- **Analytics** — Track which templates are most popular, view usage trends
- **Health & Deprecation** — Monitor template health status and manage deprecation
- **Scaffold Logs** — View detailed logs of every scaffold event

**Navigate to:** Admin dropdown → **Template Management**

The Template Management area has three tabs: **Analytics**, **Health**, and **Scaffold Logs**.

---

## Analytics

The Analytics tab shows aggregated template usage data across your organization.

### Key Metrics

| Metric | Description |
|---|---|
| **Total Scaffolds** | Total number of scaffold operations across all templates |
| **Success Rate** | Percentage of scaffolds that completed without errors |
| **Active Templates** | Number of unique templates that have been used at least once |

### Usage Table

The main table lists each template with its usage statistics:

| Column | Description |
|---|---|
| **Template** | Template name and ID |
| **Category** | Template category (e.g., Frontend, Backend) |
| **Total Uses** | Number of times the template has been scaffolded |
| **Success / Failed** | Count of successful vs. failed scaffold attempts |
| **Success Rate** | Percentage of successful scaffolds |
| **Last Used** | When the template was most recently scaffolded |

Click a column header to sort the table by that metric. The table supports pagination for organizations with many templates.

### Understanding Usage Patterns

- **High usage + high success rate** — Template is working well and popular
- **High usage + low success rate** — Template may have issues (conflicting files, broken structure)
- **Low usage** — Template may need better discoverability (tags, description) or may not match team needs

---

## Health & Deprecation

The Health tab helps you monitor template status and manage template lifecycle.

### Template Health Status

Templates can have the following status indicators:

| Status | Meaning |
|---|---|
| **Active** | Template is available and working normally |
| **Deprecated** | Template is marked for removal — hidden from the catalog by default but still functional if accessed directly |

### Deprecating a Template

When a template is outdated or being replaced:

1. Go to the **Health** tab
2. Find the template in the list
3. Click **Deprecate**
4. Enter a deprecation reason (e.g., "Replaced by react-vite-v2" or "Framework no longer supported")
5. Click **Confirm**

**What happens when a template is deprecated:**

- The template is hidden from the template catalog for regular users
- Existing workspaces that were scaffolded from it are **not affected**
- The deprecation reason is recorded for audit purposes
- The template can be un-deprecated if needed

### Un-Deprecating a Template

1. Go to the **Health** tab
2. Find the deprecated template (use the status filter to show deprecated templates)
3. Click **Restore**
4. The template reappears in the catalog

---

## Scaffold Logs

The Scaffold Logs tab provides an audit trail of all scaffold events in your organization.

### Log Entry Details

Each scaffold log entry includes:

| Field | Description |
|---|---|
| **Template** | Template name and ID |
| **Category** | Template category |
| **Workspace** | Target workspace name |
| **User** | Who performed the scaffold |
| **Status** | Success or Failed |
| **Files Created** | Number of files and folders created |
| **Timestamp** | When the scaffold occurred |
| **Error Details** | For failed scaffolds, the error message and any rollback status |

### Filtering Scaffold Logs

You can filter the log view by:

- **Template** — Show logs for a specific template
- **Status** — Filter to only successful or only failed scaffolds
- **User** — Filter to a specific user's scaffold events

### Investigating Failures

When a scaffold fails, the log entry includes:

- **Error message** — What went wrong (e.g., "Name conflict: package.json already exists")
- **Rollback status** — Whether the partial files were successfully cleaned up
  - `complete` — All partial files were removed
  - `partial` — Some files could not be removed (may need manual cleanup)
  - `failed` — Rollback could not be performed

**Common failure reasons:**

| Error | Cause | Resolution |
|---|---|---|
| Name conflict | Files already exist at the target location | User should scaffold into a subfolder or remove conflicting files |
| Scaffold in progress | Another scaffold is running on the same workspace | Wait for the first scaffold to complete |
| Template read error | Template files could not be read from disk | Check that the template repository is accessible and synchronized |
| Variable validation error | Required template variables were not provided or have invalid values | User needs to fill in all required variables correctly |

---

## Template Availability by Plan

Templates use an access tier system that maps to subscription plans. As a tenant admin, your organization's plan determines which templates are available to your users:

| Template Access Level | Required Plan |
|---|---|
| Free | All plans |
| Individual | Individual plan or higher |
| Pro | Pro plan or higher |
| Enterprise | Enterprise plan only |

If your organization needs access to templates at a higher tier, contact the platform administrator about upgrading your plan.

### Premium Templates Override

If your organization has the **Premium Templates** feature enabled (visible in your subscription settings), all templates are accessible regardless of their stated access tier.

---

## Analytics Data Refresh

Template analytics data is aggregated periodically by a background process. The data shown in the Analytics tab may have a slight delay (typically under 5 minutes) from when a scaffold event occurs to when it appears in the aggregated metrics. Individual scaffold events in the **Scaffold Logs** tab appear immediately.
