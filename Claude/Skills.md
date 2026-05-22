---
description: Custom workflows and reusable prompts for this research vault
---

# Skills & Workflows

## Import a Zotero Paper
1. In Zotero, select the item → right-click → "Export to Obsidian" (or use the Obsidian Zotero Integration plugin)
2. Note lands in `Literature_Notes/` using the `import paper` export format
3. Ask Claude: "I just imported `[[citekey]]` — synthesize it and link it to relevant topics"

## Synthesize a Literature Note
Prompt: "Read [[citekey]] and extract: (1) core claim, (2) method, (3) key findings, (4) limitations, (5) how it connects to my current focus in [[North Star]]"

## Build a Topic Note
Prompt: "Create a topic note for `[concept]` — define it, list key papers from my vault that discuss it, and identify open questions"

## Weekly Review
Prompt: "Run a weekly review: list unread papers, summarize what I added this week, suggest 3 connections I might have missed, update [[Memories]]"

## Search My Vault (via qmd)
```bash
qmd query "your question in natural language"
qmd search "keyword" -c Literature_Notes
```
