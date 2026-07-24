---
name: obsidian-vault-conventions
description: >
  Enforces Obsidian-compatible writing conventions for a Claude Code memory
  vault that a human also browses directly in Obsidian, not just an
  AI-facing memory store. Use this skill whenever: (1) starting a new
  ongoing project and deciding flat-file vs. project-folder, (2) creating
  or editing any file inside that memory directory (project_*.md,
  playbook_*.md, howto_*.md, lesson_*.md, feedback_*.md, an index file),
  (3) adding a wikilink between memory files, (4) adding tags or aliases,
  (5) immediately after writing to a memory file, to verify the frontmatter
  wasn't silently reformatted.
license: MIT
compatibility: Claude Code sessions using an Obsidian-vault-as-memory pattern; portable to any similar agent-memory setup.
metadata:
  author: braxton-williams
  version: "1.0.0"
allowed-tools: Read Write Edit Glob Grep
---

# Obsidian vault conventions

A memory vault like this is read by two different consumers with two different rules: the AI's own memory-loading (reads an index file and the files it points to) and Obsidian itself (resolves wikilinks and tags by real filename and top-level frontmatter keys). Every rule below exists because getting it wrong broke one or the other in production use — confirmed the hard way, not theoretical.

## Process

- [ ] **Decide flat file vs. project folder.** A one-off fact, lesson, or piece of feedback is a single file. An ongoing project (anything that will accumulate a history, a current state, and decisions over time) gets a folder. Don't force the folder split on something small/paused/simple just for consistency — use judgment.
- [ ] **If it's a project folder**, copy a blank template and rename both the folder (human-readable, e.g. `Some New Project/`) and all four files inside it (see naming rule below).
- [ ] **Name every file uniquely, vault-wide, in snake_case.** For a project folder: `project_<name>_readme.md`, `project_<name>_progress.md`, `project_<name>_status.md`, `project_<name>_decisions.md`. For a reusable checklist: `playbook_<topic>.md`. For a human-facing teaching doc: `howto_<topic>.md`. This must hold even though the folder groups files visually — the folder is for human browsing only.
- [ ] **Write frontmatter in the exact shape below** — see Quick Reference.
- [ ] **Add tags as an inline `#tag` line in the body, not frontmatter `tags:`.** Frontmatter `tags:`/`aliases:` are confirmed to get silently re-nested and stop working in at least one production Claude Code memory system (see Known Issues) — inline tags are the durable mechanism, not a fallback.
- [ ] **Wikilink by filename, never by the frontmatter `name:` field.** Filename is snake_case; `name:` is kebab-case. They look similar but only the filename resolves in Obsidian.
- [ ] **After writing or editing, re-read the file and verify frontmatter didn't grow unrequested nested keys.** See Known Issues.
- [ ] **Update your index file** with a one-line pointer if this is a new project or a significant new file — it's what auto-loads each session.

## Quick Reference

Standard project-folder file frontmatter — deliberately minimal, nothing beyond what a basic AI memory system needs, since anything extra is what gets silently restructured:

```yaml
---
name: project-some-new-project-readme
description: "One line: what this file covers."
metadata:
  type: project
---

Tags: #project #business-line-tag

## What it is
...
```

- **Tags go inline in the body**, right after the frontmatter closes, as a plain line: `Tags: #project #your-tag`. Obsidian indexes inline `#tags` the same as frontmatter tags for its tag pane, but the body isn't touched by frontmatter renormalization — this is the durable version. Put the same tag line on all files of a project so they cluster under each tag regardless of folder. Minimum: a `#project` type tag plus one shared business-line tag. Add a status tag (`#active`, `#paused`, `#dropped`) if useful.
- **Don't bother with frontmatter `aliases:`.** It provides nothing that a pipe-aliased wikilink doesn't already give you (`[[project_some_new_project_readme|Some New Project]]`), and it's just as vulnerable to the renormalization as `tags:` was. Always link with the real filename and a pipe display alias instead.
- Single-fact files (`lesson_*.md`, `feedback_*.md`, `reference_*.md`) use the same minimal frontmatter shape and can skip the inline tag line entirely unless cross-project browsing would genuinely help.

Wikilink examples:

```markdown
[[project_your_project_readme]]                       <- correct: real filename
[[project_your_project_readme|Your Project]]           <- correct: filename + display alias, use this form by default
[[project-your-project-readme]]                         <- WRONG: this is the frontmatter name:, not the filename — will not resolve
[[Your Project/readme]]                                 <- WRONG: folder-qualified path, breaks the moment the filename convention is followed correctly
```

## Known issues

### Frontmatter tags:/aliases: getting silently nested under metadata: — confirmed recurring, not a one-time glitch

**What happens:** top-level `tags:` (and `aliases:`) get reformatted into:

```yaml
metadata:
  tags:
    - project
    - your-tag
  type: project
```

This makes them invisible to Obsidian's tag pane and alias resolution. Confirmed in production use to recur repeatedly — it happened to freshly-written files, was fixed, and reverted again within the same session; it also happened to a file created by a different process entirely, ruling out "this only happens right after creation." Re-flattening is not a durable fix.

**Likely cause:** background memory-normalization in some Claude Code memory setups enforces a canonical `name` / `description` / `metadata.type` frontmatter shape and doesn't recognize `tags`/`aliases` as legitimate top-level keys, folding them into `metadata` as unrecognized extras — repeatedly, on an ongoing basis, not just at file creation. Before assuming this, rule out an Obsidian plugin or setting: check the vault's `.obsidian/plugins/` folder and `.obsidian/app.json` for any property-type configuration that might explain it.

**The actual fix: don't put tags in frontmatter at all.** Use an inline `Tags: #project #your-tag` line in the body instead (see Quick Reference). Whatever is renormalizing the frontmatter only touches the YAML block, not the body, so inline tags are immune.

### Filenames repeated across folders break link resolution

**What happens:** two project folders both contain `readme.md` (or `progress.md`, etc.), and a wikilink meant for one resolves to the wrong file, or Obsidian can't tell which one you mean.

**What to do:** this is why every filename must be unique vault-wide (see naming rule above). If you inherit or find generic filenames like this, rename them to the `project_<name>_<file>.md` pattern and update every wikilink that pointed at the old path-qualified form.

## Ground rules

- ALWAYS tag with an inline `Tags: #project #business-line` line in the body — NEVER rely on frontmatter `tags:`/`aliases:` if you've seen them get silently restructured in your setup.
- ALWAYS make filenames globally unique across the vault, snake_case, regardless of which folder they live in.
- ALWAYS wikilink by actual filename with a pipe display alias (`[[real_filename|Display Text]]`), never by the frontmatter `name:` field and never by a frontmatter `aliases:` entry.
- ALWAYS re-read a file after writing to confirm the frontmatter is still the minimal `name`/`description`/`metadata.type` shape — extra top-level keys are what get silently restructured.
- PREFER a project folder over a flat file once a project has real ongoing history, but don't force it on something small or paused.
