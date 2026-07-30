# CLAUDE.md

Guidance for working in this repo. This is a personal MkDocs Material documentation site deployed to GitHub Pages (see `.github/workflows/deploy.yml`).

## Folder Structure

Docs live under `docs/`, organized by topic:

```
docs/
  AI/
    Ollama/
  Dotnet/
    CI-CD/
    Classes/
    Coding-Standards/
    Database/
    Middleware/
    Templates/
    Tutorials/
  Typescript/
    NestJS/
      Controllers/
    NextJs/
      ClerkAuth/
  General/
```

When adding a new topic, prefer nesting under an existing top-level folder (`AI`, `Dotnet`, `Typescript`, `General`) before creating a new one.

## Naming Conventions

- **Folders and files:** PascalCase, no spaces (use hyphens if a word break is needed, e.g. `Coding-Standards`, not `Coding Standards`). Spaces force URL-encoding in links and are easy to typo.
- **Top-level topic folders:** use the short form consistently — `Dotnet` and `Typescript`, not `.NET`/`TypeScript`. Prose inside docs and in README/index.md can use the proper names (".NET", "TypeScript"); only the folder names use the short form.
- **File names:** describe the content, not the doc type (`DotNetDbContext.md`, not `dbcontext-setup.md` or `Guide.md`).

## Document Types

Every doc belongs to exactly one of the five types below. Pick the type first, then follow that type's rules. All types share these baseline rules:

- The `#` H1 title is the first line, describes the *content* (not "Setup" or "Guide"), and is the only H1.
- A one-line summary sentence directly under the title.
- Heading levels don't skip (H1 → H2 → H3, never H1 → H3).
- No leading byte-order mark (BOM); files are UTF-8 without BOM.
- Cross-link related docs in a `## See Also` section when they exist.

### 1. Tutorial / Guide

Goal-oriented: walks the reader from nothing to a working result via ordered steps. Lives in `Tutorials/`, `CI-CD/`, or a topic setup folder.

Required: title + one-line "what you'll build/do" summary; `## Prerequisites` (omit only if truly none); ordered `##`/numbered steps; a closing step that shows the result or how to verify it. `## Project Structure` when the doc builds a multi-file project. `## See Also` when related.

### 2. Reference / Explanation

Explains a concept or catalogs techniques with examples; not a start-to-finish build. Lives in `Classes/`, `Database/`, `Controllers/`, etc.

Required: title + one-line summary; one `##` per concept/technique; a runnable example per section where applicable; a link to the canonical upstream docs (e.g. Microsoft Learn, NestJS docs). No Prerequisites section needed.

### 3. Snippet / Cheatsheet

A block of copy-paste config or code with thin prose. Lives in `Coding-Standards/`, or alongside its topic.

Required: title + one sentence stating *what it is and when to use it*; the fenced code/config block with a language tag; a note on where the snippet goes (file path / placement) if not obvious. Keep it short — if it grows ordered steps, it's a Tutorial/Guide.

### 4. Template Doc

Describes a scaffolded starter repo and what it ships with. Lives in `Templates/`.

Required: title + `## Overview` (framework, database, repo name); `## Project Structure` tree; a "what's included out of the box" list; how to clone/use it.

### 5. Link Collection

Curated list of external resources. Lives in `General/`.

Required: title + one-line summary; entries grouped under `##` category headings; a consistent entry format within the file — either a table (`Name | Description | …`) or `[Label](url)` bullets, not a mix. Prefer the table format when a "free tier" or similar attribute is worth tracking.

Not every page needs every section, but a doc that omits its type's required sections should be treated as incomplete.

## Checklist When Adding or Moving a Doc

1. Pick the doc's type (see "Document Types") and include that type's required sections.
2. Place the file per the naming/folder conventions above.
2. Update `README.md`'s table of contents.
3. Update `docs/index.md` if it's a new top-level topic.
4. If `mkdocs.yml` has an explicit `nav:` section, add the page there too.
5. Check for and fix any links elsewhere in the repo that reference the old path (if moving/renaming).

## Local Preview

```
docker-compose up
```

Serves the site at `http://localhost:8000` with live reload via `squidfunk/mkdocs-material`.
