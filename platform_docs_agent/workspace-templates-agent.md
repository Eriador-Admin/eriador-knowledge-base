# Workspace Templates — Agent Guide

This guide describes the Workspace Templates feature from an **agent's perspective**: how to discover templates, inspect their contents, and scaffold pre-configured project structures into workspaces using the available tools.

---

## Overview

Workspace Templates are curated starter projects stored in a template catalog. Each template contains a set of files and folders that can be scaffolded (copied) into a user's workspace. Templates also support **variables** — customizable placeholders like project name, author, and framework options that get substituted into file contents during scaffolding.

Use templates when:

- The user asks to **create a new project** or **set up a workspace** with a specific technology
- The user wants a **starter app**, **boilerplate**, or **scaffold**
- You need to **initialize a workspace** with standard file structures (React app, Express API, Python notebook, etc.)

### Template Structure

Each template has:

| Property | Description |
|---|---|
| `id` | Unique identifier (e.g., `react-vite`, `express-api`) |
| `name` | Human-readable name |
| `description` | What the template sets up |
| `category` | Category folder (e.g., `frontend`, `backend`, `fullstack`) |
| `tech` | Array of technologies (e.g., `["React", "TypeScript", "Vite"]`) |
| `tags` | Searchable keywords |
| `difficulty` | `beginner`, `intermediate`, or `advanced` |
| `featured` | Whether the template is a curated recommendation |
| `variables` | Array of customizable variable definitions (may be empty) |
| `file_count` | Number of files in the template |
| `tree` | Array of file paths showing the full file/folder structure |

### Template Variables

Templates can define variables that are substituted into file contents during scaffolding. Variables use mustache syntax: `{{VARIABLE_NAME}}`.

**Reserved variables** are auto-populated — you do NOT need to provide them:

| Variable | Auto-populated From |
|---|---|
| `PROJECT_NAME` | The `project_name` you pass, or the workspace name |
| `AUTHOR` | The current user's display name |
| `DATE` | Today's date (YYYY-MM-DD) |
| `YEAR` | Current year |
| `TENANT_NAME` | The organization name |

**Custom variables** are defined in the template's `variables` array. Each custom variable has:

- `name` — UPPER_SNAKE_CASE identifier
- `label` — Human-readable label
- `type` — `string`, `number`, `boolean`, or `select`
- `required` — Whether the variable must be provided
- `default` — Default value (used if not provided)
- `options` — For `select` type: array of valid choices

---

## Access Rules

1. **Plan-based access:** Templates may require a specific subscription tier. The tools enforce this — if a template requires a higher plan, the tool returns an error.
2. **Workspace ownership:** Scaffolding creates files in a workspace. The current user must own the target workspace.
3. **One scaffold at a time:** Only one scaffold can run on a workspace at a time. If another scaffold is in progress, the tool returns a concurrency error.

---

## Available Tools

All workspace template tools belong to the `workspace_templates` category. Each tool returns `{ success, result, summary }`.

### 1. `list_template_categories` — Browse categories

List all available template categories with template counts. Use this as a starting point to understand what's available.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| *(none)* | | | This tool takes no parameters |

**Example call:**
```json
{
  "name": "list_template_categories",
  "arguments": {}
}
```

**Returns:** A list of categories with names, descriptions, and template counts.

---

### 2. `search_templates` — Search and filter templates

Search the template catalog by keywords, filter by category, or list all available templates. Use this to find the right template for what the user needs.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `query` | string | no | Search keywords (e.g., `"react typescript"`, `"express api"`) |
| `category` | string | no | Filter to a specific category (e.g., `"frontend"`, `"backend"`) |
| `featured` | boolean | no | If `true`, only show featured/recommended templates |
| `limit` | number | no | Max results (default: 20, max: 50) |

**Example calls:**
```json
{
  "name": "search_templates",
  "arguments": {
    "query": "react typescript",
    "limit": 5
  }
}
```
```json
{
  "name": "search_templates",
  "arguments": {
    "category": "backend",
    "featured": true
  }
}
```

**Returns:** A ranked list of matching templates with id, name, description, category, tech stack, difficulty, and file count.

---

### 3. `get_template_details` — Inspect a specific template

Get the full details of a specific template including its file tree and variable definitions. Use this after finding a template via search to confirm it's the right one before scaffolding.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `template_id` | string | yes | Template ID (e.g., `"react-vite"`, `"express-api"`) |

**Example call:**
```json
{
  "name": "get_template_details",
  "arguments": {
    "template_id": "react-vite"
  }
}
```

**Returns:** Full template metadata including name, description, tech stack, tags, difficulty, file tree (all file paths), and variable definitions (with names, types, defaults, and whether they're required).

---

### 4. `scaffold_template` — Create a workspace from a template

Scaffold a template into a workspace, creating all files and folders with variable substitution. This is the primary action tool — it creates the actual project structure.

**This tool requires user confirmation** (`_requires_confirmation: true`) because it creates many files in the workspace.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `template_id` | string | yes | Template ID to scaffold |
| `workspace_id` | string | yes | Target workspace UUID |
| `target_folder_id` | string | no | Folder UUID within the workspace to scaffold into (null = workspace root) |
| `project_name` | string | no | Project name for the `PROJECT_NAME` variable (defaults to template ID) |
| `variables` | object | no | Custom variable values as `{ "VAR_NAME": "value" }` |

**Example calls:**
```json
{
  "name": "scaffold_template",
  "arguments": {
    "template_id": "react-vite",
    "workspace_id": "a1b2c3d4-...",
    "project_name": "my-dashboard"
  }
}
```
```json
{
  "name": "scaffold_template",
  "arguments": {
    "template_id": "express-api",
    "workspace_id": "a1b2c3d4-...",
    "target_folder_id": "f5e6d7c8-...",
    "project_name": "user-service",
    "variables": {
      "PORT": "4000",
      "DATABASE": "postgresql",
      "INCLUDE_AUTH": "true"
    }
  }
}
```

**Constraints:**
- The current user must own the target workspace
- No file/folder name conflicts at the target location
- Only one scaffold per workspace at a time
- All required template variables must be provided
- Template must be accessible under the tenant's plan
- **Requires user confirmation** (`_requires_confirmation: true`)

**Returns:** Confirmation with the template ID, workspace ID, and counts of folders and files created.

---

## Common Workflows

### Set up a new project from scratch

1. **Ask the user** what kind of project they want (or infer from context)
2. **Search templates** to find matching options
3. **Show the user** the top results and let them pick (or recommend one)
4. **Get details** on the chosen template to check variables
5. **Scaffold** the template into their workspace

```
search_templates { query: "react typescript" }
  → Found 3 templates, present top options to user
get_template_details { template_id: "react-ts-vite" }
  → Has 2 custom variables: CSS_FRAMEWORK (select), INCLUDE_TESTS (boolean)
scaffold_template { template_id: "react-ts-vite", workspace_id: "...", project_name: "dashboard", variables: { "CSS_FRAMEWORK": "tailwind", "INCLUDE_TESTS": "true" } }
  → [user confirms] → Created 12 folders and 28 files
```

### Help the user choose a template

1. **List categories** to see what's available
2. **Browse a category** or search with keywords
3. **Compare templates** by getting details on 2-3 options
4. Present a summary comparison to the user

```
list_template_categories {}
  → 5 categories: frontend (8), backend (6), fullstack (4), data-science (3), devops (2)
search_templates { category: "frontend", limit: 10 }
  → 8 templates listed with names, descriptions, and tech stacks
get_template_details { template_id: "react-vite" }
  → File tree: 15 files, React + Vite + TypeScript
get_template_details { template_id: "next-app" }
  → File tree: 22 files, Next.js + TypeScript + Tailwind
```

### Scaffold into a subfolder

When the workspace already has files and the user wants the template alongside them:

1. **Create a folder** in the workspace using `create_file` (the file tool with folder creation)
2. **Scaffold** into that specific folder

```
scaffold_template { template_id: "express-api", workspace_id: "...", target_folder_id: "folder-uuid", project_name: "api-server" }
  → [user confirms] → Created 5 folders and 14 files in the "api-server" folder
```

### Initialize a workspace for a specific ticket

When implementing a ticket that requires a new project structure:

1. **Read the ticket** to understand requirements
2. **Search templates** matching the required tech stack
3. **Scaffold** the best match
4. **Customize** files as needed for the ticket's specific requirements

```
search_templates { query: "express postgresql" }
  → Found "express-pg-api" template
scaffold_template { template_id: "express-pg-api", workspace_id: "...", project_name: "billing-service", variables: { "PORT": "5002", "DB_NAME": "billing" } }
  → [user confirms] → Project scaffolded
edit_file { ... }  → Customize routes for billing domain
```

---

## Error Handling

All tools return `{ success: false, result: "<message>" }` on failure. Common errors:

| Error | Cause | Resolution |
|---|---|---|
| `"Template not found"` | Invalid template ID | Use `search_templates` to find valid template IDs |
| `"Template requires a higher plan tier"` | Tenant plan doesn't include this template | Inform the user they need to upgrade or choose a different template |
| `"Name conflict: ... already exist"` | Files at the target location have the same names | Scaffold into a subfolder or ask the user to move conflicting files |
| `"A scaffold is already running"` | Another scaffold is in progress on this workspace | Wait and retry, or inform the user |
| `"Workspace not found"` | Invalid workspace ID | Verify the workspace ID |
| `"You do not own this workspace"` | User doesn't own the target workspace | Can only scaffold into workspaces the user owns |
| `"Variable validation failed"` | Required variables missing or invalid values | Check `get_template_details` for required variables and provide valid values |
| `"Target folder not found"` | Invalid target folder ID | Use `list_files` to find valid folder IDs in the workspace |

---

## Key Concepts

- **Template IDs** are lowercase, alphanumeric with hyphens (e.g., `react-vite`, `express-api`, `python-flask`). Use them for lookups and scaffolding.
- **Categories** group templates by project type. A template belongs to exactly one category.
- **Variables** are substituted at scaffold time — they only affect file contents, not file names or folder names.
- **Scaffolding is atomic** — if it fails partway through, any partially created files are rolled back (soft-deleted).
- **Featured templates** are curated starting points. Recommend them when the user has no strong preference.
- **Deprecated templates** are hidden from search unless specifically requested. They still function if scaffolded by ID, but newer alternatives exist.
