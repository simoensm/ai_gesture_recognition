# AI Gesture Recognition — Knowledge Vault

Comparative study of 3D hand gesture recognition methods — **DTW**, **Edit-Distance**, **Three-Cents ($3)**, and **BiLSTM** — evaluated on the Huang et al. (2019) dataset with rigorous nested cross-validation, ablation study, and statistical hypothesis testing.

This repository doubles as an **Obsidian knowledge vault** tightly integrated with **Zotero** for academic reference management.

> **Note:** The actual project code (implementations, experiments, datasets, results) lives in the main project repository:
> 👉 [Matteo-glz/MLSMM2154_Artificial-Intelligence_gesture_recognition](https://github.com/Matteo-glz/MLSMM2154_Artificial-Intelligence_gesture_recognition)
>
> This repo is the **research workspace** for that project — literature notes, concept maps, paper annotations, and course material organised in Obsidian.

---

## Table of Contents

1. [Vault Structure](#vault-structure)
2. [Prerequisites](#prerequisites)
3. [Zotero Setup](#zotero-setup)
4. [Obsidian Setup](#obsidian-setup)
5. [Plugin Configuration](#plugin-configuration)
   - [Zotero Integration](#zotero-integration-obsidian-zotero-desktop-connector)
   - [Templater](#templater)
   - [Dataview](#dataview)
   - [Commander (cmdr)](#commander-cmdr)
6. [Workflow: Importing a Paper from Zotero](#workflow-importing-a-paper-from-zotero)
7. [Templates Reference](#templates-reference)
8. [Git & Collaboration Notes](#git--collaboration-notes)

---

## Vault Structure

```
ai-gesture-recognition/
├── 00_AI_Knowledge_Graph_MOC.md     # Master map of content
├── Concepts/                        # ML/AI concept notes (CNNs, Transformers, etc.)
├── Foundations/                     # Math foundations (probability, optimisation, …)
├── Papers/                          # Literature notes (one per paper, keyed by citekey)
│   └── images/                      # PDF annotation images exported by Zotero
├── Project - Gesture Recognition/   # Project-specific notes and workflow
├── Course - Sequence Modeling/      # Lecture notes
├── Workflows/                       # Reusable ML pipeline notes
├── Templates/                       # Obsidian note templates
│   ├── zotero-import-paper.md       # Primary Zotero import template
│   ├── zotero-template.md           # Lightweight Zotero template
│   ├── literature-note.md           # Manual literature note
│   ├── topic-note.md                # Concept / topic note
│   └── thinking-note.md             # Fleeting / thinking note
├── Zotero_Import/                   # BibTeX exports from Zotero (gitignored)
├── Daily Notes/                     # Daily log notes
└── .obsidian/                       # Obsidian config (plugins, settings)
```

**Note:** The `Claude/` folder (AI assistant memory) is gitignored and will not be pushed to GitHub.

---

## Prerequisites

| Tool | Version | Purpose |
|---|---|---|
| [Obsidian](https://obsidian.md) | ≥ 1.1.1 | Note-taking application (desktop) |
| [Zotero](https://www.zotero.org) | ≥ 6 | Reference manager |
| [Better BibTeX for Zotero](https://retorque.re/zotero-better-bibtex/) | latest | Citekey generation & BibTeX export |
| Git | any | Version control |

---

## Zotero Setup

### 1. Install Zotero

Download and install [Zotero](https://www.zotero.org/download/) for your platform. Create a free account for cloud sync if desired.

### 2. Install Better BibTeX (BBT)

Better BibTeX generates stable, human-readable citekeys (e.g., `Vaswani2017_Transformer`) and enables auto-export of `.bib` files.

1. Download the latest `.xpi` from the [BBT releases page](https://github.com/retorque-it/zotero-better-bibtex/releases).
2. In Zotero: **Tools → Add-ons → Install Add-on From File…** and select the `.xpi`.
3. Restart Zotero.

### 3. Configure the Citekey Format

The vault uses the pattern `[auth:capitalize][year]_[title:select=1:capitalize]` to produce keys like `Vaswani2017_Transformer`. To set this:

1. Open **Zotero → Edit → Preferences → Better BibTeX**.
2. Under **Citation keys**, set the format to:
   ```
   [auth:capitalize][year]_[shorttitle:capitalize]
   ```
3. Click **Refresh** to regenerate all keys.

### 4. Enable the Zotero Connector (for browser import)

Install the [Zotero Connector](https://www.zotero.org/download/connectors) browser extension. This lets you save papers directly from Google Scholar, arXiv, ACM DL, and most publisher sites into Zotero with one click.

### 5. (Optional) Export BibTeX to the vault

To keep a local `.bib` backup:

1. Right-click a Zotero collection → **Export Collection…**
2. Choose **Better BibTeX** format.
3. Enable **Keep Updated** for live auto-export.
4. Save to `Zotero_Import/` inside this vault.

> These `.bib` files are gitignored — they contain local file paths that differ between machines.

---

## Obsidian Setup

### 1. Open the vault

Clone or download this repository, then in Obsidian: **Open folder as vault** and select the repository root.

### 2. Trust plugins

On first open, Obsidian will warn about community plugins. Click **Trust author and enable plugins** to activate all four plugins listed below.

### 3. Allow network access (Zotero Integration)

The Zotero Integration plugin communicates with the running Zotero desktop app over a local HTTP connection. Make sure Zotero is **open** whenever you want to import notes.

---

## Plugin Configuration

Four community plugins are used. All are pre-installed in `.obsidian/plugins/`.

### Zotero Integration (`obsidian-zotero-desktop-connector`)

**Version:** 3.2.1 | **Author:** mgmeyers

This plugin connects to the running Zotero app and imports paper metadata, abstracts, highlights, and PDF annotations directly into Obsidian notes using a Nunjucks template.

**Saved configuration** (`.obsidian/plugins/obsidian-zotero-desktop-connector/data.json`):

| Setting | Value | Meaning |
|---|---|---|
| `database` | `Zotero` | Uses your local Zotero library |
| `noteImportFolder` | `Papers` | New notes land in `Papers/` |
| `outputPathTemplate` | `Papers/{{citekey}}.md` | One file per paper, named by citekey |
| `imageOutputPathTemplate` | `Papers/images/{{citekey}}/{{fileName}}` | PDF annotation images |
| `templatePath` | `Templates/zotero-import-paper.md` | Template applied to every import |
| `openNoteAfterImport` | `true` | Opens the note immediately after creation |

**How to import a paper:** see [Workflow](#workflow-importing-a-paper-from-zotero) below.

---

### Templater

**Version:** 2.20.5 | **Author:** SilentVoid / Zachatoo

Provides advanced Handlebars-style templating (`<% tp.* %>` syntax) for new notes. Used alongside the Zotero template's Nunjucks syntax.

**Recommended settings** (set manually in Obsidian → Settings → Templater):

| Setting | Recommended value |
|---|---|
| Template folder location | `Templates` |
| Trigger Templater on new file creation | Enabled |
| Automatic jump to cursor | Enabled |

---

### Dataview

**Version:** installed | **Author:** blacksmithgu

Treats your vault as a queryable database. Use it to generate dynamic lists of unread papers, papers by topic, etc.

**Example query** — all unread papers:

```dataview
TABLE authors, year, tags
FROM "Papers"
WHERE status = "unread"
SORT year DESC
```

**Example query** — papers tagged `transformer`:

```dataview
TABLE title, year
FROM "Papers"
WHERE contains(tags, "transformer")
SORT year ASC
```

---

### Commander (cmdr)

**Version:** installed | **Author:** phibr0

Adds custom commands to Obsidian's toolbar, ribbon, and context menus. Used here to create a one-click **"Import from Zotero"** ribbon button so you don't need to open the command palette every time.

---

## Workflow: Importing a Paper from Zotero

This is the standard workflow to go from a PDF in Zotero to a structured literature note in Obsidian.

### Step 1 — Add the paper to Zotero

- **From browser:** Click the Zotero Connector extension button on any article page (arXiv, ACM, Springer, etc.). Zotero auto-fills all metadata.
- **From PDF:** Drag the PDF into Zotero. Right-click → **Retrieve Metadata for PDF**.
- **Manual:** File → New Item → Journal Article, then fill in fields.

### Step 2 — Annotate the PDF (optional but recommended)

Open the PDF inside Zotero (double-click). Use the built-in PDF reader to:
- Highlight text (yellow = key claim, green = method, red = limitation — customise as you like).
- Add comments to highlights.

All highlights are stored in Zotero and will be pulled into Obsidian during import.

### Step 3 — Import into Obsidian

1. Make sure **Zotero is running**.
2. In Obsidian, open the Command Palette (`Cmd/Ctrl + P`).
3. Run **Zotero Integration: Import notes**.
4. Search for the paper by title, author, or citekey.
5. Press Enter — Obsidian creates `Papers/<citekey>.md` using `Templates/zotero-import-paper.md`.

The generated note contains:
- YAML frontmatter (title, authors, year, journal, citekey, DOI, tags, status)
- Abstract
- All highlights and annotations with page numbers
- Synthesis scaffold (Core Claim, Method, Key Findings, Limitations, Relevance)
- Backlink to open the item in Zotero

### Step 4 — Complete the synthesis

Open the new note and fill in the **Synthesis** section. Then link to concept notes:

```markdown
## Connections
- Core architecture → [[Concepts/Attention_Transformer]]
- Compared with → [[Papers/Hochreiter1997]]
```

---

## Templates Reference

### `Templates/zotero-import-paper.md` *(primary)*

Used automatically by the Zotero Integration plugin. Variables use **Nunjucks** syntax (`{{variable}}`). Key variables:

| Variable | Description |
|---|---|
| `{{title}}` | Paper title |
| `{{authors}}` | Author list |
| `{{date \| format("YYYY")}}` | Publication year |
| `{{publicationTitle}}` | Journal or conference name |
| `{{citekey}}` | Better BibTeX citekey |
| `{{DOI}}` | DOI string |
| `{{abstractNote}}` | Abstract text |
| `{{desktopURI}}` | Deep-link to open item in Zotero |
| `{{annotations}}` | Array of PDF annotations |
| `annotation.annotatedText` | Highlighted text |
| `annotation.page` | Page number |
| `annotation.comment` | Your annotation comment |

### `Templates/zotero-template.md` *(lightweight)*

A simpler alternative with the same Nunjucks variables, without the synthesis callout blocks. Useful for quick imports.

### `Templates/literature-note.md`

For manually created literature notes (no Zotero import). Uses Templater `<% tp.* %>` syntax.

### `Templates/topic-note.md`

For concept or topic notes in `Concepts/` and `Foundations/`.

### `Templates/thinking-note.md`

Fleeting / atomic thinking notes for the daily workflow.

---

## Git & Collaboration Notes

### What is tracked

- All `.md` note files (`Concepts/`, `Foundations/`, `Papers/`, `Course - Sequence Modeling/`, etc.)
- Obsidian plugin binaries and manifests (`.obsidian/plugins/`)
- Obsidian core settings (`.obsidian/app.json`, `community-plugins.json`, `core-plugins.json`)
- The Zotero Integration `data.json` (so template and output path config is shared)
- All templates in `Templates/`

### What is gitignored

| Path | Reason |
|---|---|
| `Claude/` | AI assistant memory — personal and session-specific |
| `.obsidian/workspace.json` | Local UI state (open tabs, panels) — differs per machine |
| `.obsidian/plugins/*/data.json` | Most plugin settings contain local paths; *exception:* Zotero Integration |
| `Zotero_Import/*.bib` | BibTeX exports contain absolute local file paths |
| `.DS_Store`, `._*` | macOS metadata |
| `__pycache__/`, `.venv/` | Python build artefacts |

### Cloning on a new machine

```bash
git clone https://github.com/<your-username>/ai-gesture-recognition.git
```

Then:
1. Open the folder as an Obsidian vault.
2. Install and start Zotero with Better BibTeX.
3. The Zotero Integration plugin will use the committed `data.json` — no manual reconfiguration needed.
