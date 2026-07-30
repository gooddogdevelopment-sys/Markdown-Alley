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

## Page Template

New docs should follow this structure:

```markdown
# Title

One or two sentence summary of what this doc covers and when to use it.

## Prerequisites
(omit this section if none)
- ...

## Steps / Content
...

## See Also
(omit this section if none)
- [Related Doc](path/to/doc.md)
```

Not every page needs every section (e.g. a short reference page doesn't need "Prerequisites"), but longer setup/tutorial docs should include them.

## Checklist When Adding or Moving a Doc

1. Place the file per the naming/folder conventions above.
2. Update `README.md`'s table of contents.
3. Update `docs/index.md` if it's a new top-level topic.
4. If `mkdocs.yml` has an explicit `nav:` section, add the page there too.
5. Check for and fix any links elsewhere in the repo that reference the old path (if moving/renaming).

## Local Preview

```
docker-compose up
```

Serves the site at `http://localhost:8000` with live reload via `squidfunk/mkdocs-material`.
