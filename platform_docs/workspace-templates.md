# Workspace Templates

Create new workspaces pre-loaded with starter files, folder structures, and configuration using the built-in template catalog. Templates cover common project types — frontend apps, backend services, full-stack starters, data science notebooks, and more.

---

## Overview

Templates are curated starter projects that give you a working file structure in one click. Instead of creating an empty workspace and adding files manually, choose a template and the platform populates your workspace with ready-to-use code, configuration files, and folder hierarchies.

Each template includes:

- **Files & folders** — Source code, config files, README, and directory structure
- **Template variables** — Customizable placeholders (project name, author, options) that get replaced when you scaffold
- **Metadata** — Description, difficulty level, tech stack, and tags to help you find the right template

---

## Accessing the Template Catalog

### From the Workspace Panel

1. Click the **+** button in the Workspace Panel toolbar (or use the workspace menu)
2. Select **New from Template**
3. The Template Catalog modal opens

### Template Catalog Layout

The catalog modal has the following areas:

- **Category tabs (top)** — Filter templates by category (e.g., Frontend, Backend, Full-Stack, Data Science)
- **Search bar** — Search templates by name, tags, description, or tech stack
- **Template grid** — Browse available templates as cards showing the template name, icon, description, tech stack badges, and difficulty level
- **Detail panel (right)** — When you select a template, a detail panel shows the full description, file tree preview, variable inputs, and the scaffold button

---

## Browsing & Searching Templates

### Category Filtering

Click any category tab to filter templates to that category. The count next to each category name shows how many templates are available.

### Search

Type keywords in the search bar to search across template names, tags, and descriptions. Results are ranked by relevance — exact matches on name rank highest, followed by tag matches, then description matches.

**Examples:**

- `react` — Finds templates with "react" in the name, tags, or description
- `express api` — Finds templates matching both "express" and "api"
- `python` — Finds Python-related templates across all categories

### Sorting

Sort templates by:

| Sort Option | Behavior |
|---|---|
| **Name** | Alphabetical (A–Z) — default when not searching |
| **Relevance** | Best match first — default when searching |
| **Popularity** | Most-scaffolded templates first |
| **Difficulty** | Easiest first (Beginner → Intermediate → Advanced) |

### Featured Templates

Some templates are marked as **Featured** (⭐). These are recommended starting points curated by the platform. Toggle the "Featured" filter to see only featured templates.

---

## Previewing a Template

Click on any template card to open the **Detail Panel** on the right side of the catalog. The detail panel shows:

- **Template name** and icon
- **Full description** — What the template sets up and how to use it
- **Tech stack** — Badges for each technology (e.g., React, TypeScript, Vite)
- **Tags** — Searchable keywords
- **Difficulty** — Beginner, Intermediate, or Advanced
- **File count** — How many files will be created
- **File tree** — An expandable preview of every file and folder the template will create
- **Popularity** — How many times this template has been used across the platform

This lets you see exactly what will be added to your workspace before committing.

---

## Template Variables

Some templates include **variables** — customizable values that get inserted into the template files when you scaffold. For example, a React template might have a `PROJECT_NAME` variable that appears in the package.json `name` field, README title, and other files.

### Reserved Variables (Auto-Populated)

These variables are filled in automatically — you don't need to provide them:

| Variable | Value |
|---|---|
| `PROJECT_NAME` | Your project name (from the form) or workspace name |
| `AUTHOR` | Your display name |
| `DATE` | Today's date (YYYY-MM-DD) |
| `YEAR` | Current year (e.g., 2026) |
| `TENANT_NAME` | Your organization name |

### Custom Variables

Templates can define additional custom variables that appear as form fields in the detail panel. Each variable has a type:

| Type | Input | Example |
|---|---|---|
| **String** | Text input | Package scope (`@mycompany`) |
| **Number** | Number input | Port number (`3000`) |
| **Boolean** | Checkbox (true/false) | Include linting? |
| **Select** | Dropdown menu | CSS framework (Tailwind, CSS Modules, Styled Components) |

Variables marked as **required** must be filled in before you can scaffold. Optional variables show their default value and can be left as-is.

### How Variables Work

When you scaffold, every occurrence of `{{VARIABLE_NAME}}` in the template files is replaced with the value you provided. For example:

- `{{PROJECT_NAME}}` → `my-cool-app`
- `{{AUTHOR}}` → `Jane Smith`
- `{{CSS_FRAMEWORK}}` → `tailwind`

Variable substitution only applies to **text file contents** — filenames and folder names are not changed.

---

## Scaffolding a Template

Scaffolding creates the template's files and folders inside your workspace.

### Steps

1. **Select a template** from the catalog
2. **Review the file tree** in the detail panel to see what will be created
3. **Fill in any variables** (required variables must be completed)
4. **Choose a target location:**
   - **Workspace root** — Files are created at the top level of your workspace (default)
   - **Specific folder** — Select an existing folder to scaffold into
5. Click **Scaffold Project**
6. The files and folders are created in your workspace

### What Happens During Scaffolding

- All folders are created first (in order from shallowest to deepest)
- Files are then created inside the appropriate folders
- Template variables are replaced with your values in all text files
- Binary files (images, fonts) are copied as-is without variable substitution
- A scaffold event is recorded for analytics

### Conflict Prevention

If any of the template's top-level file or folder names already exist at the target location, you'll see a conflict warning listing the conflicting names. You can:

- Choose a different target folder
- Rename or move the conflicting items first
- Cancel and pick a different template

### After Scaffolding

Once scaffolding completes, you'll see a success message showing the number of folders and files created. Your workspace file tree updates immediately — you can start editing the new files right away.

---

## Plan-Based Access

Some templates may require a specific subscription plan. If a template requires a higher plan than your current subscription:

- The template appears in the catalog but cannot be scaffolded
- A message indicates the required plan level
- Contact your organization admin to upgrade if needed

Plan tiers (from most to least restrictive):

| Template Access Level | Available To |
|---|---|
| Free | All plans |
| Individual | Individual plan and above |
| Pro | Pro plan and above |
| Enterprise | Enterprise plan only |

---

## Tips

- **Start with Featured templates** if you're new to a technology — they're curated to follow best practices
- **Use search** with tech stack keywords (e.g., "typescript express") to find specific stacks
- **Check the file tree** before scaffolding — templates vary in size from minimal starters to full project structures
- **Scaffold into a subfolder** if you want the template alongside existing files
- **Variables save time** — templates with variables produce customized, ready-to-run projects without manual find-and-replace
