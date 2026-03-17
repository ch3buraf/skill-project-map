name: project-map
description: >
  Generate an interactive code map of any software project — both as Markdown (for AI context) 
  and as a standalone HTML page (for humans). Use this skill whenever someone asks to visualize 
  project structure, create a code map, understand codebase architecture, map dependencies between 
  files, generate a project overview, or needs to know "where to modify" something in a codebase. 
  Also trigger when a user says things like "show me how files connect", "what depends on what", 
  "draw the architecture", "I need to understand this project", "create documentation of the structure",
  or "map out the codebase". Works with any language: Python, JS/TS, Go, Java, Rust, C/C++, HTML, 
  configs, Docker, shell scripts.
---

# Project Map Generator

Generate a complete interactive map of a software project that shows modules, files, and real 
dependencies between them. Output in two formats: **Markdown** (for AI/LLM context) and 
**HTML** (interactive visual graph for humans).

## When to use

- User asks to visualize, map, or document project structure
- User needs to understand where to make changes
- User wants to see dependencies between files
- Before starting a large refactoring task
- When onboarding onto an unfamiliar codebase

## Process

### Step 1: Analyze the project

Read every source file in the project. For each file, extract:

1. **What it exports** — functions, classes, components, routes, endpoints
2. **What it imports** — from other project files (not external packages)
3. **What it connects to** — databases, APIs, message queues, file I/O
4. **What it renders/includes** — templates, partials, static assets

See `references/analysis-patterns.md` for language-specific extraction patterns.

### Step 2: Identify modules

Group files into logical modules. A module is a cohesive unit of functionality. 
Use these signals to identify module boundaries:

- **Directory structure** — `src/auth/`, `components/`, `api/handlers/`
- **Naming conventions** — files with shared prefixes or domain terms
- **Dependency clusters** — files that import each other heavily form a module
- **Functional cohesion** — files that serve the same feature

Each module needs:
- `id` — slug identifier (`auth`, `api-gateway`, `web-ui`)
- `name` — human-readable name
- `description` — one sentence explaining what this module does
- `files[]` — list of files with `path` and `caption` (what the file does)

### Step 3: Map dependencies

For each pair of files that are both in the project, determine if a dependency exists.
A dependency is any of:

| Type | Example |
|------|---------|
| Import | `from app import db`, `import { Button } from './Button'` |
| Render | Flask `render_template('index.html')`, React `<Component />` |
| API call | `fetch('/api/users')` hitting a route defined in another file |
| Config reference | `docker-compose.yml` referencing a `Dockerfile` |
| Data flow | File A writes to DB, File B reads from same DB |
| Build chain | `Makefile` → source files, `entrypoint.sh` → app binary |

Label each dependency with a verb: `imports`, `renders`, `calls`, `builds`, `reads`, `writes`, `configures`, `orchestrates`.

### Step 4: Generate outputs

Generate **both** outputs. Always generate both unless the user explicitly asks for only one.

#### 4a: Markdown output → `PROJECT_MAP.md`

This file is optimized for AI consumption. An LLM reading this file should immediately 
understand the project architecture and know exactly which files to modify for any given task.

Use this exact structure:

```
# Project Map: {Project Name}

## Architecture Overview
{2-3 sentences describing what this project does and its high-level architecture}

## Modules

### {Module Name}
{One sentence description}

| File | Role | Key Exports |
|------|------|-------------|
| `path/to/file` | What it does | functions, classes, routes |

### {Next Module}
...

## Dependency Graph

### {filename}
- → `other-file` (verb: what the dependency is)
- ← `another-file` (verb: what depends on this)

## File Change Guide

| To change... | Modify these files | Why |
|-------------|-------------------|-----|
| Authentication logic | `auth/handler.py`, `auth/middleware.py` | Handler has login/logout, middleware checks tokens |
| Database schema | `models/user.py`, `migrations/` | Model defines schema, migration applies it |
| API endpoints | `api/routes.py`, `api/handlers/` | Routes maps URLs, handlers implement logic |

## Quick Reference

- **Entry point**: `{main file}`
- **Config**: `{config files}`
- **Database**: `{db-related files}`
- **Tests**: `{test directory}`
```

The "File Change Guide" section is the most important for AI use. Think about the 10 most 
common types of changes someone would make to this project and document which files they'd 
need to touch and why.

#### 4b: HTML output → `PROJECT_MAP.html`

Read the template from `assets/map-template.html`. This is a complete standalone HTML file 
with an interactive force-directed graph. You need to inject the project data into it.

Find the placeholder `__PROJECT_DATA__` in the template and replace it with a JSON object:

```json
{
  "projectName": "My Project",
  "modules": [
    {
      "id": "module-id",
      "name": "Module Name",
      "desc": "What this module does",
      "files": [
        {"path": "src/file.py", "caption": "What this file does"}
      ]
    }
  ],
  "deps": [
    {"from": "src/file.py", "to": "src/other.py", "label": "imports"}
  ]
}
```

The template handles all visualization — canvas rendering, force-directed layout, 
drag & drop, zoom, pan, hover tooltips, module filtering, file table. You only need to 
inject the data.

### Step 5: Verify

After generating both files, do a quick sanity check:

1. Every file in the project appears in at least one module
2. Every dependency has both endpoints present in the file list
3. The File Change Guide covers the main areas of the project
4. Module descriptions are specific (not "handles stuff"), say what the module actually does

## Important notes

- **Be thorough with dependencies.** Read the actual file contents, don't guess from filenames. 
  A file called `utils.py` could import from anywhere. The value of this map comes from 
  accurate dependency tracking.

- **Don't include external packages.** Only map dependencies between files within the project. 
  `import os` or `from flask import Flask` are external — skip them. 
  `from .models import User` is internal — include it.

- **Keep module count reasonable.** 3-8 modules for a small project, up to 15 for a large one. 
  If you have 20+ modules, you're splitting too fine. If you have 2, you're not splitting enough.

- **The HTML must be a single standalone file.** No external dependencies, no CDN links, 
  no separate CSS/JS files. Everything inline. The user should be able to open it by 
  double-clicking the file.