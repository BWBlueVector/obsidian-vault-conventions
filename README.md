# obsidian-vault-conventions

A Claude Code [Agent Skill](https://agentskills.io/specification) that keeps an AI assistant's persistent memory vault correctly formatted for **both** of its readers at once: the AI itself, and a human browsing the same files in Obsidian as a knowledge graph.

## Why this exists

I run a personal AI assistant on Claude Code with a persistent memory bank — markdown files with frontmatter, linked with `[[wikilinks]]`, that I also browse directly in Obsidian to see the whole knowledge graph visually. That dual-use setup sounds simple until you actually build it: an AI memory system and a note-taking app have different, unstated assumptions about what a "valid" file looks like, and neither side tells you when the other one breaks.

Two production bugs prove it, and this skill exists specifically to stop them recurring:

1. **Wikilinks pointed at the wrong thing.** I was linking files by their frontmatter `name:` field. Obsidian resolves links by *filename*, not frontmatter. The two looked similar enough (same words, different casing) that it took real debugging to notice every cross-reference was silently broken.
2. **Tags kept vanishing.** I added `tags:` and `aliases:` as frontmatter fields — standard Obsidian practice. Some background process in my Claude Code memory pipeline kept re-nesting them under a `metadata:` key, where Obsidian's tag pane can't see them. I fixed it, watched it happen again on a file I hadn't even touched, and only stopped chasing it once I moved tagging out of frontmatter entirely into an inline `#tag` line in the document body — which the normalization never touches.

Neither of those is an exotic edge case. They're the two most basic things a linked-notes system needs to get right (linking, tagging), and both failed silently in a live system. This skill is the fix, written down so it doesn't need re-deriving.

## What it does

Encodes four rules as a Claude Code skill that loads automatically when relevant:

- **Filenames are the source of truth for links** — write frontmatter `name:` and the real filename in parallel conventions, and always link by filename.
- **Tags live in the document body, not frontmatter** — a durable workaround for a real, confirmed-recurring normalization bug, not a style preference.
- **A project-folder convention** (readme / progress / status / decisions) for anything with ongoing history, vs. a single file for one-off facts — with a naming rule that keeps every file uniquely addressable even once you have dozens of similarly-structured project folders.
- **A verification step** built into the process itself: re-read what you just wrote before trusting it, because this system has a documented history of silently reformatting itself.

## Who this is for

- **Builders:** if you're wiring an AI agent to a persistent, human-readable memory store (Obsidian, a wiki, any linked-notes system), this is a concrete pattern for the failure modes you'll hit, not just the happy path.
- **Anyone evaluating AI-agent reliability:** this is a small, real example of treating an agent's own infrastructure with the same rigor as production code — documenting a bug, ruling out causes systematically, and building the fix into the process so it can't quietly regress.

## Install

Drop `SKILL.md` into `.claude/skills/obsidian-vault-conventions/` in your project. Claude Code picks up project-level skills automatically at session start.

## License

MIT — see [LICENSE](LICENSE).
