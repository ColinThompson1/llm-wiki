# llm-wiki

**A reusable skill for building multi-repo LLM wikis with Claude Code, Cursor, Codex, and other Agent Skills tools.**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![GitHub stars](https://img.shields.io/github/stars/ColinThompson1/llm-wiki?style=social)](https://github.com/ColinThompson1/llm-wiki)
[![GitHub forks](https://img.shields.io/github/forks/ColinThompson1/llm-wiki?style=social)](https://github.com/ColinThompson1/llm-wiki)
[![Agent Skills](https://img.shields.io/badge/Agent_Skills-compatible-blue)](https://agentskills.io)
[![Install](https://img.shields.io/badge/Install-npx_add--skill-green)](https://github.com/ColinThompson1/llm-wiki#install)

<p align="center">
  <img src="assets/karpathy-tweet.png" alt="Inspiration for the LLM wiki workflow" width="560">
</p>

`llm-wiki` packages the [LLM wiki workflow](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) into one installable [Agent Skills](https://agentskills.io) skill. Your coding agent ingests sources into `sources/`, compiles durable knowledge pages into `wiki/`, answers questions with citations, lints the wiki for consistency, and commits wiki updates with reasons.

## What Is an LLM Wiki?

An **LLM wiki** is a knowledge system where the LLM maintains structured wiki pages instead of re-searching raw documents on every question. New sources are compiled into durable markdown pages, cross-references are updated over time, and answers cite the wiki pages that already contain the synthesized knowledge.

This skill gives you three operations:

| Operation | What it does | Output |
|-----------|--------------|--------|
| **Ingest** | Collects a source into `sources/` and compiles it into the wiki | New or updated wiki pages plus a git commit |
| **Query** | Searches the wiki and answers with citations | Grounded answers linking to markdown pages |
| **Lint** | Checks index integrity, links, and wiki health | Auto-fixes plus reported issues, with commits when files change |

See [SKILL.md](SKILL.md) for the full skill specification.

## LLM Wiki vs RAG

| Approach | Knowledge lives in | When synthesis happens | Good for |
|----------|--------------------|------------------------|----------|
| **RAG** | Raw chunks and embeddings | At query time | Broad retrieval across large corpora |
| **LLM Wiki** | Curated markdown pages | During ingest and maintenance | Compounding knowledge, summaries, and durable cross-links |

This skill is optimized for the wiki model: knowledge that improves over time instead of re-deriving relationships on every query.

## Multi-Repo Model

The wiki uses two scopes:

- `projects/` for repo-specific knowledge
- `organization/` for reusable and cross-repo knowledge

`organization/` starts intentionally small:

- `patterns/`
- `architecture/`
- `topics/`

This keeps repo detail close to the source while still letting the wiki accumulate shared lessons across many codebases.

## Usage Stats

Based on a production knowledge base maintained daily since April 2026:

- **94** wiki articles across **13** topic directories
- **99** source materials ingested
- **87** operation log entries in the last 7 days

See [examples/](examples/) for sample wiki pages, source files, and operation logs.

## Install

```bash
npx add-skill ColinThompson1/llm-wiki
```

Works with any tool that supports the [Agent Skills](https://agentskills.io) standard.

## Quick Start

### 1. Ingest your first source

Give the skill a URL, a file, or pasted text:

> "Ingest this article: https://example.com/attention-is-all-you-need"

The skill stores the source in `sources/`, then compiles or updates the right knowledge pages in `wiki/`.

### 2. Ask your wiki a question

> "What do I know about attention mechanisms?"

The skill searches the wiki and answers with citations linking back to your markdown pages.

### 3. Keep the wiki healthy

> "Lint my wiki"

Checks for broken links, missing index entries, stale cross-references, and related issues.

## How the Workflow Works

The core idea behind this workflow: the LLM maintains the wiki while the human focuses on choosing sources and asking good questions.

```text
knowledge-base/
├── sources/
│   ├── external/
│   │   └── 2026-04-03-source-article.md
│   └── projects/
│       └── repo-a/
│           └── 2026-04-03-design-note.md
├── wiki/
│   ├── projects/
│   │   └── repo-a/
│   │       └── concept-name.md
│   ├── organization/
│   │   ├── patterns/
│   │   ├── architecture/
│   │   └── topics/
│   ├── index.md
│   └── log.md
├── projects.md
```

Each new source can update multiple pages, strengthen cross-references, record contradictions, and produce a git commit that explains why the knowledge changed. That is what makes the wiki compound over time.

## Git-Tracked Knowledge Changes

Every wiki-writing operation should create a git commit. By default, use one commit per ingest, archive, or auto-fixing lint run. If a single operation changes unrelated topics, split the commits.

When facts change, the commit message should explain why:

```text
wiki: ingest retry-strategies

Reason: new source showed jitter is mandatory for distributed retries, which changes the prior summary.
Files: wiki/projects/repo-a/retry-worker.md, wiki/organization/patterns/retry-strategies.md
```

## Tool Compatibility

This skill follows the [agentskills.io](https://agentskills.io) open standard:

| Tool | Install method |
|------|----------------|
| Claude Code | `npx add-skill ColinThompson1/llm-wiki` |
| Cursor | `npx add-skill ColinThompson1/llm-wiki` |
| Codex CLI | Copy to `.agents/skills/llm-wiki/` |
| OpenCode | `npx add-skill ColinThompson1/llm-wiki` |
| Other tools | Copy `SKILL.md` and `references/` into the tool's skill directory |

## FAQ

### What is the difference between an LLM wiki and a personal wiki?

An LLM wiki is maintained by the model. It updates summaries, cross-links, index entries, and contradictions as new material arrives. A normal personal wiki depends on manual editing.

### What sources can I ingest?

Web pages, papers, blog posts, PDFs, markdown files, text files, and pasted text. The skill converts everything into markdown under `sources/` and compiles it into `wiki/`.

### Is this production-ready?

The workflow is based on a real knowledge base with 94 articles and 99 sources maintained daily since April 2026. The repo includes examples, templates, and a design spec.

## Inspired By

Implementation of the workflow inspired by [Karpathy's LLM Wiki idea](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f). The value here is the reusable workflow, prompt structure, and battle-tested knowledge-compilation rules.

See also: [lucasastorian/llmwiki](https://github.com/lucasastorian/llmwiki), [atomicmemory/llm-wiki-compiler](https://github.com/atomicmemory/llm-wiki-compiler).

## License

[MIT](LICENSE)
