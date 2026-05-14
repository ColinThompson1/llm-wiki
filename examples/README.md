# Examples from Real Wiki

This directory contains real files from a knowledge base maintained with `llm-wiki` since April 2026.

The included example files still reflect the earlier single-project layout, including historical `raw/` references inside the sample content. The current skill generalizes that workflow to `sources/`, `wiki/projects/`, and `wiki/organization/`, but the article and log examples remain useful as format references.

## Files

| File | What it shows |
|------|---------------|
| `claude-code-statusline-landscape.md` | Compiled wiki article with structured data (tables, citations, cross-references) |
| `2026-03-19-claude-code-statusline-landscape.md` | Source material before compilation |
| `ai-coding-tools-index.md` | Topic index with one-line summaries |
| `log-sample.md` | Sample entries from operation log |

## Source vs Compiled Comparison

**Source file** (`2026-03-19-claude-code-statusline-landscape.md`):
- Original research notes
- Unstructured content
- Metadata header (Source, Collected, Published dates)

**Compiled article** (`claude-code-statusline-landscape.md`):
- Structured sections (Overview, Competitive Landscape, User Pain Points)
- Tables synthesized from multiple sources
- Cross-references to other wiki articles
- Updated across multiple ingest operations

## Operation Log

The log records every action:
- `Compile` — new article from source
- `Update` — cascade updates across related articles
- `Lint` — quality checks
- `Query` — archived query results

Recent activity shows daily maintenance: 87 entries in the last 7 days alone.
