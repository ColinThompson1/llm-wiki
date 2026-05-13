---
name: karpathy-llm-wiki
description: "Use when building or maintaining a personal or multi-repo LLM-powered knowledge base. Triggers: ingesting sources into a wiki, querying wiki knowledge, linting wiki quality, 'add to wiki', 'what do I know about', or any mention of 'LLM wiki' or 'Karpathy wiki'."
---

# Karpathy LLM Wiki

Build and maintain a knowledge base using LLMs. You manage immutable source material under `sources/` and compiled knowledge articles under `wiki/`. The wiki is organized into two scopes:

- `projects/` for repo-specific knowledge
- `organization/` for reusable or cross-repo knowledge

Core ideas from Karpathy:
- "The LLM writes and maintains the wiki; the human reads and asks questions."
- "The wiki is a persistent, compounding artifact."

## Architecture

Four layers, all under the user's knowledge-base root:

**sources/** — Immutable source material. You read, never modify after creation.
- `sources/projects/<repo>/` holds repo-specific source material.
- `sources/external/` holds source material not owned by a single repo.

**wiki/** — Compiled knowledge articles. You have full ownership.
- `wiki/projects/<repo>/` holds repo-specific articles.
- `wiki/organization/patterns/` holds reusable implementation and workflow patterns.
- `wiki/organization/architecture/` holds design principles, architecture notes, and system approaches.
- `wiki/organization/topics/` holds cross-repo material that does not yet deserve a more specific category.
- `wiki/index.md` is the global index.
- `wiki/log.md` is the append-only operation log.

**projects.md** — Project registry. Maps repo identifiers to paths or short descriptions. Use it to normalize repo naming before writing into `wiki/projects/` or `sources/projects/`.

**SKILL.md** (this file) — Schema layer. Defines structure and workflow rules.

Templates live in `references/` relative to this file. Read them when you need the exact format for source files, wiki articles, archive pages, the index, or the project registry.

### Initialization

Triggers only on the first Ingest. Check whether the structure exists. Create only what is missing; never overwrite existing files:

- `sources/` directory (with `.gitkeep` only if needed)
- `sources/projects/` directory
- `sources/external/` directory
- `wiki/` directory
- `wiki/projects/` directory
- `wiki/organization/patterns/` directory
- `wiki/organization/architecture/` directory
- `wiki/organization/topics/` directory
- `wiki/index.md` — heading `# Knowledge Base Index`, empty body
- `wiki/log.md` — heading `# Wiki Log`, empty body
- `projects.md` — initialize from `references/projects-template.md`

If Query or Lint cannot find the wiki structure, tell the user: "Run an ingest first to initialize the wiki." Do not auto-create.

---

## Ingest

Fetch a source into `sources/`, then compile it into `wiki/`. Always both steps, no exceptions.

### Fetch (`sources/`)

1. Get the source content using whatever web or file tools your environment provides. If nothing can reach the source, ask the user to paste it directly.

2. Determine source provenance:
   - If it is tied to one repo, store it under `sources/projects/<repo>/`.
   - Otherwise store it under `sources/external/`.
   - Normalize `<repo>` against `projects.md`. If the repo is new, add it before saving the source.

3. Save using the correct provenance path:
   - `sources/projects/<repo>/YYYY-MM-DD-descriptive-slug.md`
   - `sources/external/YYYY-MM-DD-descriptive-slug.md`
   - Slug from source title, kebab-case, max 60 characters.
   - Published date unknown → omit the date prefix from the file name (for example `descriptive-slug.md`). The metadata Published field still appears; set it to `Unknown`.
   - If a file with the same name already exists, append a numeric suffix.
   - Include metadata header: source URL or origin, collected date, published date, and project when applicable.
   - Preserve original text. Clean formatting noise. Do not rewrite opinions.

   See `references/raw-template.md` for the exact format.

### Compile (`wiki/`)

Classify the knowledge scope before writing:

- **Project scope** → The content is mainly about one repo's implementation, architecture, decisions, or behavior. Write or update `wiki/projects/<repo>/<article>.md`.
- **Organization scope** → The content applies across repos, captures a reusable pattern, compares approaches, or should outlive a single repo. Write or update one of:
  - `wiki/organization/patterns/<article>.md`
  - `wiki/organization/architecture/<article>.md`
  - `wiki/organization/topics/<article>.md`

Default routing rules:

- If the knowledge is mainly about one repo, always update the project article first.
- If the same source reveals a reusable lesson, also update the relevant organization article.
- If an organization article is grounded in project evidence, keep the project article links in the organization page's Source Files field or body citations.

Determine whether the content belongs in an existing article:

- **Same core thesis as an existing article** → Merge into that article. Add the new source file to Source Files. Update affected sections.
- **New concept** → Create a new article in the most relevant scope and directory. Name the file after the concept, not the source file.
- **Touches unrelated topics** → Split the work into separate articles, and later split commits if the resulting updates are unrelated.

These are not mutually exclusive. A single source may update one project article and one organization article. In all cases, check for factual conflicts: if the new source contradicts existing content, annotate the disagreement with source attribution. When merging, note the conflict within the merged article. When the conflicting content lives in separate articles, note it in both and cross-link them.

See `references/article-template.md` for article format. Key points:
- Sources field: author, organization, or publication name plus date, semicolon-separated.
- Source Files field: markdown links to `sources/` files, semicolon-separated.
- All links inside wiki files are relative to the current file.

### Cascade Updates

After the primary article, check for ripple effects:

1. Scan sibling articles in the same scope for materially affected content.
2. Scan `wiki/index.md` for related project or organization articles in other directories.
3. Update every article whose knowledge content is materially affected. Each updated file gets its Updated date refreshed.

Archive pages are never cascade-updated. They are point-in-time snapshots.

### Post-Ingest

Update `wiki/index.md`: add or update entries for every touched article. Group entries first by scope, then by repo or organization category. When adding a new section, include a one-line description. The Updated date reflects when the article's knowledge content last changed, not the file system timestamp. See `references/index-template.md` for format.

Append to `wiki/log.md`:

```text
## [YYYY-MM-DD] ingest | <primary article title>
- Updated: <cascade-updated article title>
- Updated: <another cascade-updated article title>
```

Omit `- Updated:` lines when no cascade updates occur.

### Git Commit Requirement

Any operation that writes wiki content must also create a git commit:

- ingest
- archived query
- lint when it auto-fixes files

Commit rules:

- Default to one commit per operation.
- If a single run changes clearly unrelated topics, split into separate commits.
- Never create empty commits.

Commit message format:

```text
wiki: <operation> <short subject>

Reason: <why the factual or structural change happened>
Files: <comma-separated relative paths>
```

Reason rules:

- If a factual claim changed, `Reason:` must explain what changed and why.
- Good triggers include: new source, contradiction resolution, superseded information, corrected attribution, or synthesis from multiple project pages.
- If the change is non-factual, `Reason:` can be concise: link repair, index sync, cross-reference cleanup, or archive creation.

If the knowledge-base root is not in a git repository, tell the user that automatic commit logging is unavailable and continue with the wiki update only if they still want the change.

---

## Query

Search the wiki and answer questions. Examples of triggers:
- "What do I know about X?"
- "Summarize everything related to Y"
- "Compare how repo A and repo B handle Z"

### Steps

1. Read `wiki/index.md` to locate relevant articles.
2. Search organization articles first, then project articles that provide supporting evidence.
3. Prefer wiki content over your own training knowledge.
4. Cite wiki sources with project-root-relative markdown links, for example `[Retry Strategies](wiki/organization/patterns/retry-strategies.md)` or `[Queue Worker](wiki/projects/repo-a/queue-worker.md)`.
5. Output the answer in the conversation. Do not write files unless asked.

### Archiving

When the user explicitly asks to archive or save the answer to the wiki:

1. Write the answer as a new wiki page. See `references/archive-template.md`.
   - Sources: markdown links to the wiki articles cited in the answer.
   - No Source Files field.
   - File name reflects the query topic.
   - Place in the most relevant scope and directory.
2. Always create a new page. Never merge archive content into existing articles.
3. Update `wiki/index.md`. Prefix the Summary with `[Archived]`.
4. Append to `wiki/log.md`:
   ```text
   ## [YYYY-MM-DD] query | Archived: <page title>
   ```
5. Create a git commit following the commit rules above. For archived pages, the reason should explain the synthesis or question captured.

---

## Lint

Quality checks on the wiki. Two categories with different authority levels.

### Deterministic Checks (auto-fix)

Fix these automatically:

**Index consistency** — compare `wiki/index.md` against actual wiki files, excluding `index.md` and `log.md`:
- File exists but missing from index → add entry with `(no summary)` placeholder. For Updated, use the article's metadata Updated date if present; otherwise fall back to the file's last modified date.
- Index entry points to nonexistent file → mark it as `[MISSING]` in the index. Do not delete the entry.

**Internal links** — for every markdown link in wiki article files, excluding `index.md` and `log.md`:
- Target does not exist → search `wiki/` for a file with the same name elsewhere.
  - Exactly one match → fix the path.
  - Zero or multiple matches → report to the user.

**Source Files references** — every link in a Source Files field must point to an existing `sources/` file:
- Target does not exist → search `sources/` for a file with the same name elsewhere.
  - Exactly one match → fix the path.
  - Zero or multiple matches → report to the user.

**See Also** — within each repo directory and organization category:
- Add obviously missing cross-references between related articles.
- Remove links to deleted files.

### Heuristic Checks (report only)

These rely on your judgment. Report findings without auto-fixing:

- Factual contradictions across articles
- Outdated claims superseded by newer sources
- Missing conflict annotations where sources disagree
- Orphan pages with no inbound links from other wiki articles
- Organization pages that make cross-repo claims without project evidence
- Missing cross-links between project articles and the organization pages derived from them
- Concepts frequently mentioned but lacking a dedicated page
- Archive pages whose cited source articles have been substantially updated since archival

### Post-Lint

Append to `wiki/log.md`:

```text
## [YYYY-MM-DD] lint | <N> issues found, <M> auto-fixed
```

If lint auto-fixes files, create a git commit. If lint only reports issues and writes nothing, do not commit.

---

## Conventions

- Standard markdown with relative links throughout wiki files.
- `wiki/projects/` is organized by repo. `wiki/organization/` is organized by category.
- Today's date is used for log entries, Collected dates, and Archived dates. Updated dates reflect when the article's knowledge content last changed. Published dates come from the source; use `Unknown` when unavailable.
- Inside wiki files, use relative links. In conversation output, use project-root-relative links such as `wiki/projects/repo-a/queue-worker.md`.
- Ingest updates both `wiki/index.md` and `wiki/log.md`. Archive updates both. Lint updates `wiki/log.md`, and `wiki/index.md` when needed for auto-fixes. Plain queries do not write files.
- Git commit history is the authoritative explanation of why wiki content changed. `wiki/log.md` is the operational timeline.
