# FORGE — Logos Workshop
## Complete Build Manifest for AI Agents (Claude Code / Codex / Any)

> **One sentence:** A desktop app where the user writes in an Obsidian/Notion hybrid editor, every character lives in an addressable Excel-like grid underneath, and an AI layer below executes any instruction the user gives by highlighting text — eliminating the need for plugins forever.

**Version:** 1.1.0  
**Stack:** Tauri 2 · React 19 · TypeScript · TipTap (ProseMirror) · Rust · PostgreSQL  
**Identifier:** `com.theophysics.forge`  
**Author:** David Lowery / POF 2828  
**Last Updated:** 2026-03-13  

---

## TABLE OF CONTENTS

1. [The Three-Layer Editor](#the-three-layer-editor)
2. [The Four Pillars](#the-four-pillars)
3. [What's Already Built](#whats-already-built)
4. [What Needs to Be Built](#what-needs-to-be-built)
5. [The Core Loop (CANONICAL)](#the-core-loop)
6. [Data Ingestion Layer](#data-ingestion-layer)
7. [Plugin Platform](#plugin-platform)
8. [Repo Structure](#repo-structure)
9. [Setup & Development](#setup--development)
10. [Reference Docs](#reference-docs)

---

## THE THREE-LAYER EDITOR

This is the breakthrough. This is the product.

### Layer 1: Surface (What the user sees)
- Obsidian-style markdown writing + Notion-style structured blocks
- Rich text, headings, links, tables, callouts, YAML frontmatter
- **Built with TipTap (ProseMirror) in React + TypeScript** ✅

### Layer 2: Grid (The addressable substrate) ✅ BUILT
- Every block in the TipTap document maps to a GridRow
- Every word within a block maps to a GridCell with `[row, col]` coordinates
- Every cell carries ProseMirror `from/to` absolute positions for precise cursor bridging
- Parallel data structure that stays in sync with TipTap's document model via debounced rebuild (150ms)
- Extensible metadata per cell and per row: tags, flags, links, color, confidence, notes
- Immutable mutation API for React state compatibility
- Serialization/deserialization for metadata persistence across doc edits
- Query API: by tag, by flag, by text, by cell range
- Toggle grid view (`Ctrl+G`) shows side-by-side cell inspector
- Status bar shows live `{rows}r/{cells}c` count

### Layer 3: AI (The execution engine) ✅ BUILT — enhancement pending
- Three-provider streaming engine: Anthropic (Claude Sonnet 4), OpenAI (GPT-4.1), local Ollama
- Three-role system: Interface (direct interaction), Logic (structural validation), Copilot (predictive next actions)
- Per-role provider routing: shared engine or split engine (each role on its own provider/model)
- Inline AI chat bubble: select text → chat box appears at selection → type instruction → AI executes → result in Layer 1
- Inline chat is grid-aware: shows `[row,col]` coordinates, node type, tags, flags in context header
- Quick actions: Tag, Flag Row, Explain, Fix Grammar, Summarize, Link To
- Smart replace detection: instructions containing "fix", "rewrite", "translate", etc. auto-replace selection
- Promoted block commands: `/PROBE` (structural integrity), `/EAST` (steelman objection), `/CONNECT` (cross-domain bridge)
- AI runtime event logging with deduplication (8-minute cooldown, 120-event ring buffer)
- Background roles fire on 6-second debounce after content changes
- **Pending:** Cached instruction enforcement engine (instructions stored and auto-enforced on future edits)

---

## THE FOUR PILLARS

### Pillar 1: Version Control ❌ NOT BUILT
- Built-in git-style versioning (not requiring external git)
- Every save creates a timestamped snapshot
- Diff view between versions
- Rollback to any point
- Branch for experimental edits

### Pillar 2: Content Layer (Three-Layer Editor) ✅ CORE COMPLETE
- Obsidian/Notion hybrid surface ✅
- Excel grid underneath ✅
- AI underneath that ✅
- File tree sidebar ✅
- Notebook/vault concept ✅

### Pillar 3: Data Mirror ❌ NOT BUILT
- Every content folder has a shadow `_data/` mirror directory
- Content folder: your markdown files (clean, readable)
- Mirror folder: everything GENERATED (HTML reports, CSV, graphs, audio, images, Python outputs)
- Content stays clean. Data stays organized. Linked but separate.
- Third-party developers can drop Python scripts into mirror folders

### Pillar 4: Global Engine (YAML-Driven) ❌ NOT BUILT
- Every capability is: Python script + YAML config = run/don't run
- `_engines/` folder in each vault
- Settings page shows all available engines with on/off toggles
- Engine execution triggered by: manual, checkbox, save, schedule, or event
- Example engines: TTS, semantic tagging, link extraction, PDF/HTML/DOCX export, API calls, AI analysis

---

## WHAT'S ALREADY BUILT

### Tech Stack Status

| Component | Technology | Status |
|-----------|-----------|--------|
| Runtime | Tauri v2 (Rust backend + webview) | ✅ Built |
| Frontend | React 19 + TypeScript + Vite 7 | ✅ Built |
| Editor | TipTap (ProseMirror) | ✅ Built (Layer 1) |
| Styling | Tailwind CSS 4 | ✅ Built |
| AI Service | Anthropic + OpenAI + Ollama streaming | ✅ Built |
| AI Roles | Interface / Logic / Copilot (3-role system) | ✅ Built |
| AI Prompts | /PROBE, /EAST, /CONNECT (Theophysics-aligned) | ✅ Built |
| AI Runtime | Event logging, dedup, cooldown, ring buffer | ✅ Built |
| AI Sidecar | Python script (ai_sidecar.py) | ✅ Built |
| Grid Layer | Parallel addressable substrate (`grid.ts` + `useGrid.ts`) | ✅ Built |
| Inline AI Chat | Selection → Instruct bubble (grid-aware) | ✅ Built |
| Database | PostgreSQL (192.168.1.177:2665) | ✅ Connected |
| Icons | Lucide React | ✅ Installed |
| Animation | Framer Motion | ✅ Installed |
| Cached Instructions | Instruction enforcement engine | ❌ NOT BUILT |
| Version Control | — | ❌ NOT BUILT |
| Data Mirror | — | ❌ NOT BUILT |
| Global Engine | — | ❌ NOT BUILT |
| YAML Config System | — | ❌ NOT BUILT |
| Plugin Platform | — | ❌ NOT BUILT |
| Data Ingestion Layer | — | ❌ NOT BUILT |
| Command Palette | — | ❌ NOT BUILT |

---

### Grid Layer — Technical Architecture (`src/lib/grid.ts` — ~300 lines)

The grid is a pure, stateless parallel data structure. It takes a ProseMirror JSON document and produces a `GridSnapshot`.

**Data flow:**
```
TipTap ProseMirror Doc (JSON)
  → buildGrid(doc) walks all block nodes
    → Each block node → GridRow (paragraph, heading, listItem, etc.)
      → Text tokenized by word boundaries (regex \S+) → GridCell[]
        → Each cell: [row, col] + word text + from/to ProseMirror positions + CellMeta
```

**Types:**
```typescript
interface GridCell {
  row: number;
  col: number;
  word: string;           // text snapshot
  from: number;           // ProseMirror absolute position (start)
  to: number;             // ProseMirror absolute position (end)
  meta: CellMeta;         // tags, flags, links, color, confidence, notes, extensible
}

interface GridRow {
  index: number;
  nodeId: string;         // stable ID for TipTap node
  nodeType: string;       // 'paragraph', 'heading', 'listItem', etc.
  level?: number;         // heading level (1-6)
  from: number;           // node start position
  to: number;             // node end position
  cells: GridCell[];
  meta: CellMeta;         // row-level metadata
}

interface GridSnapshot {
  rows: GridRow[];
  version: number;
  timestamp: number;
  totalRows: number;
  totalCells: number;
}
```

**Query API:** `getCell(row, col)`, `getRow(row)`, `getCellRange(fromRow, fromCol, toRow, toCol)`, `queryByTag(tag)`, `queryByFlag(flag)`, `queryByText(search)`

**Mutation API (immutable):** `setCellMeta`, `setRowMeta`, `addTagToCell`, `addFlagToRow`, `removeFlagFromRow` — all return new GridSnapshot objects.

**Serialization:** `serializeGridMeta` saves only cells/rows with non-empty metadata. `deserializeGridMeta` rehydrates with drift-tolerance — if a cell's word doesn't match at the expected column after an edit, it searches the row for the word by text matching.

**React hook** (`src/hooks/useGrid.ts` — ~180 lines):
```typescript
const grid = useGrid(editor);
// Exposes: snapshot, getCell, getRow, getCellRange, queryByTag, queryByFlag,
//          queryByText, addTag, setFlag, removeFlag, updateCellMeta,
//          updateRowMeta, highlightCell, highlightRow, rebuild
```

Listens to `editor.on('update')`, debounces at 150ms, rebuilds from `editor.getJSON()`, re-applies stored metadata across rebuilds via serialize/deserialize cycle.

**Known limitation:** Position mapping (`from/to`) is calculated as `nodeStart + 1 + token.offsetStart`, which assumes flat text inside nodes. Heavily nested inline marks (bold inside italic inside a link) can shift offsets by a few characters. Fix path: walk ProseMirror's actual TextNode offsets instead of character math on extracted text. Not a blocker for current use.

---

### AI Service — Technical Architecture (`src/lib/ai.ts` — ~400 lines)

Multi-provider streaming AI engine supporting three backends and three roles.

**Providers:**
| Provider | Endpoint | Model | Auth |
|----------|----------|-------|------|
| Anthropic | `api.anthropic.com/v1/messages` | `claude-sonnet-4-20250514` | `sk-ant-*` API key |
| OpenAI | `api.openai.com/v1/chat/completions` | `gpt-4.1` | `sk-*` API key |
| Ollama | `127.0.0.1:11434/api/chat` | Configurable (default: `llama3.1:8b`) | None (local) |

All three implement SSE/NDJSON streaming with proper `AbortSignal` support, error handling, and graceful abort (partial text returned on cancel).

**Three-Role System:**
| Role | System Prompt | Purpose |
|------|--------------|---------|
| **Interface** | Direct user interaction — execute writing and thinking tasks | Primary chat partner |
| **Logic** | Validate structure, assumptions, consistency, contradictions | Background structural scan |
| **Copilot** | Anticipate next 2-4 useful actions, provide concrete next steps | Background action suggestions |

**Role routing:** Two modes controlled by `aiRoleRouting` setting:
- **Shared** — all roles use the same provider/model (default)
- **Split** — each role can be routed to a different provider/model independently

**Key functions:**
- `runAiChat(messages, systemPrompt, callbacks, signal, provider?, model?)` — general streaming chat
- `runAiRoleChat(role, messages, callbacks, signal, contextBlock?, extraPrompt?)` — role-aware chat with auto-routing
- `runAiCommand(command, blockContent, blockType, systemPrompt, callbacks, signal, workspaceContext?)` — promoted block commands

**Promoted Block Commands** (`src/lib/aiPrompts.ts`):
- `/PROBE` — structural integrity analysis (find exact hold/break point)
- `/EAST` — steelman strongest objection (version that makes serious physicist/theologian pause)
- `/CONNECT` — cross-domain structural bridge (shared logical architecture, not metaphorical similarity)

Each prompt is Theophysics framework-aligned with specific output format constraints.

**AI Runtime** (`src/lib/aiRuntime.ts` — ~100 lines):
- Event ring buffer (120 events max, localStorage-persisted)
- Deduplication: same role + kind + summary + provider + model within 8-minute cooldown window → suppressed
- Events: `{ id, role, kind, summary, provider, model, status, createdAt }`

---

### Inline AI Chat — Technical Architecture (`src/components/Editor/InlineAiChat.tsx` — ~260 lines)

The core interaction loop of FORGE. Select → Instruct → Done.

**How it works:**
1. User selects text in the editor (`Ctrl+I` or context menu)
2. Chat bubble appears at the DOM coordinates of the selection start
3. Context header shows: `[row,col]` grid coordinates, node type, active tags, active flags, truncated selection text
4. User types natural language instruction
5. `getSelectionContext()` builds `InlineContext`: selectedText, selectionFrom/To, gridRow, gridCol, nodeType, surroundingText (prev + current + next row), tags, flags
6. AI receives full context + instruction → streams response
7. Smart replace: if instruction contains "fix", "rewrite", "replace", "correct", "improve", "translate", "simplify", "expand" → AI result auto-replaces the selection in the editor
8. Manual options: Replace button, Copy button, Done button

**Quick action buttons:** Tag as..., Flag row, Explain, Fix grammar, Summarize, Link to...

- "Tag as..." prompts for tag name → calls `grid.addTag(row, col, tag)`
- "Flag row" prompts for flag → calls `grid.setFlag(row, flag)`
- Others pre-fill the instruction input

**Keyboard:** `Enter` submits, `Escape` closes, click outside closes.

---

### Rust Backend (`_FORGE_SOURCE/src-tauri/src/lib.rs` — 416 lines)
- `set_vault` / `get_vault_files` — vault selection and file scanning
- `read_note` / `write_note` / `create_note` / `create_folder` — full file CRUD
- `open_or_create_note_by_title` — wikilink resolution (recursive search)
- `rename_item` / `delete_item` — file management
- `connect_db` — PostgreSQL connection with credential fallback
- `run_python_sidecar` — executes Python AI scripts with JSON payload

### React Frontend (`_FORGE_SOURCE/src/`)
- `App.tsx` (580 lines) — main app: notebooks, file tree, editor, AI panel, settings, routing
- `ForgeEditor.tsx` (578 lines) — TipTap editor with grid integration, inline AI chat, context menu, autosave, `Ctrl+G` grid toggle, `Ctrl+I` inline chat, status bar with grid stats
- `InlineAiChat.tsx` (260 lines) — selection → instruct bubble (grid-aware, quick actions, smart replace)
- `AiPanel.tsx` — slide-out AI conversation panel (Ctrl+Shift+A)
- `BottomBar.tsx` — command bar with `/commands`
- `Sidebar.tsx` — file tree navigation with notebook switching
- `SettingsPage.tsx` — settings UI with AI provider config, per-role routing
- `LogicSheet.tsx` — spreadsheet-style logic view miniapp
- `TruthLayerWorkbench.tsx` — truth layer management miniapp

### Libraries (`_FORGE_SOURCE/src/lib/`)
- `grid.ts` (300 lines) — parallel addressable grid data structure with full query/mutation/serialization API
- `ai.ts` (400 lines) — three-provider streaming AI service with role routing
- `aiPrompts.ts` — /PROBE, /EAST, /CONNECT Theophysics-aligned prompts
- `aiRuntime.ts` (100 lines) — AI event logging with dedup ring buffer
- `markdown.ts` — markdown ↔ HTML conversion
- `noteMeta.ts` — note metadata extraction
- `pythonSidecar.ts` — Python execution bridge
- `settings.ts` — settings management with localStorage persistence
- `types.ts` — shared TypeScript types

### Hooks (`_FORGE_SOURCE/src/hooks/`)
- `useGrid.ts` (180 lines) — React hook: grid sync, debounced rebuild, metadata persistence, full query/mutation/highlight API

---

## WHAT NEEDS TO BE BUILT

### PRIORITY 1: ~~Grid Layer~~ ✅ COMPLETE

### PRIORITY 2: Cached Instruction Enforcement Engine

The inline AI chat works, but instructions are fire-and-forget. The next step is making them persistent and self-enforcing.

**Requirements:**
- When user gives an instruction via inline chat, store it as a `CachedInstruction`
- Each cached instruction has: pattern (what to match), action (what to do), scope (local/global), creation context
- On every document edit, the enforcement engine scans for pattern matches and auto-applies actions
- Three stored object types the system needs:

1. **Canonical Anchors** — selected text marked as canonical with UUID, label, grain size, scope (local/global), lock state
2. **Display Rules** — trigger (text match / semantic tag / domain) → color + shape + opacity + scope
3. **Expansion Macros** — abbreviation → expansion (e.g., LOW1 → full email address)

**What users can say (examples):**
- "This is the canonical Grace equation" → every G(t) gets checked against this
- "When I write Grace, highlight amber, circle shape" → display rule fires everywhere
- "This paragraph is load-bearing. Flag if anything contradicts it." → canonical anchor + contradiction watch
- "Anytime I write entropy in a physics context, link to E7.1" → contextual auto-link

**Files to create:**
- `src/lib/annotations.ts` — canonical anchors, display rules, expansion macros store
- `src/lib/instructionCache.ts` — cached instruction enforcement engine

---

### PRIORITY 3: Data Mirror Folders

**Requirements:**
- When user opens a vault/notebook, Forge creates a `_data/` mirror directory
- Structure mirrors the content folder exactly
- Generated outputs go to mirror, not content folder
- File watcher keeps them in sync
- UI shows both views (content | data) with toggle

**Rust backend additions needed:**
- `create_mirror` command — creates `_data/` structure
- `get_mirror_files` command — lists mirror contents
- `write_mirror_file` command — writes to mirror
- File watcher integration

**Files to create:**
- `src/lib/mirror.ts` — mirror folder logic
- `src/components/DataMirror/MirrorView.tsx` — mirror browser UI
- Extend `src-tauri/src/lib.rs` with mirror commands

---

### PRIORITY 4: YAML Global Engine

**Requirements:**
- `_engines/` folder in each vault
- Each engine: YAML config + optional Python script
- Forge scans for engines on vault open
- Settings page shows engines with on/off toggles
- Execution triggered by: manual, checkbox, save, schedule, or event

**Example engine YAML:**
```yaml
engine: tts
voice: "en-US-AriaNeural"
speed: 1.2
strip_links: true
output_format: mp3
output_to: mirror_folder
trigger: on_checkbox_complete
```

**Files to create:**
- `src/lib/engine.ts` — YAML engine discovery, parsing, execution
- `src/components/GlobalEngine/EngineManager.tsx` — engine config UI
- Default engines in `_engines/` directory

---

### PRIORITY 5: Version Control

**Requirements:**
- On every save, create a timestamped snapshot
- Store diffs, not full copies (for efficiency)
- Version browser UI (timeline view)
- Diff view between any two versions
- One-click rollback
- Grid changes trigger version snapshots

**Files to create:**
- `src/lib/versioning.ts` — version control logic
- `src/components/VersionControl/VersionBrowser.tsx` — timeline UI
- `src/components/VersionControl/DiffView.tsx` — diff comparison
- Extend `src-tauri/src/lib.rs` with snapshot commands

---

### PRIORITY 6: Data Ingestion Layer (Layer 0)

This comes BEFORE the knowledge graph. Any data source. Drop it in. Answer 3-5 questions. It's live.

**The flow:**
```
User drops Excel/CSV/JSON into FORGE
  → Setup wizard fires (3-5 questions max)
  → Data is ingested and indexed
  → Notes layer activates on top of it
  → AI partners start reading behind the scenes
  → Knowledge graph can consume it
```

**The 5-question wizard:**
1. What IS this? (Bible / Research papers / Stock prices / My notes / Other)
2. What's the primary key? (auto-detected, user confirms)
3. What do you want to DO with this? (Read, annotate, search, analyze, feed graph)
4. Any columns that are CANONICAL? (load-bearing, never overwrite)
5. What should the AI partners watch for? (plain language, or skip)

**Supported formats (v1):**
- Excel (.xlsx) — primary format, sheets become tabs
- CSV (.csv) — auto-detected delimiter
- JSON array — each object becomes a row
- Markdown folder — each .md file becomes a row
- PostgreSQL table — direct query, live sync
- Plain text — line-per-row, manual column mapping

**After ingestion, every row becomes:**
```json
{
  "id": "auto-generated FORGE UUID",
  "source": "KJV_Bible.xlsx",
  "primary_key": "JHN|1|1",
  "columns": { "Book": "John", "Chapter": 1, "Verse": 1, "Text": "..." },
  "notes": [],
  "anchors": [],
  "ai_flags": [],
  "graph_node": null
}
```

**Build order for ingestion:**
1. File drop handler — accept xlsx/csv/json, detect format
2. Setup wizard UI — 5-question flow, auto-detect primary key
3. Row renderer — display ingested data with notes column
4. Notes store — attach/read notes per row per source
5. AI-1 (Connector) — pattern match across sources
6. AI-2 (Challenger) — contradiction watch
7. AI-3 (Archivist) — gap detection
8. Graph bridge — expose scored rows to knowledge graph

---

### PRIORITY 7: Plugin Platform

**Goal:** Developer can build a plugin, drop it into `plugins/`, enable in Settings, see new commands/panels/dock actions immediately. No app rebuild.

**Architecture:**
- Extension host: sandboxed runtime (Web Worker / iframe / isolated ES module)
- Capability-based API: plugins request explicit permissions
- Single-writer guardrail: all note writes through `NoteGateway` queue
- Stable versioned plugin contract

**Plugin manifest:**
```json
{
  "id": "com.example.research-tools",
  "name": "Research Tools",
  "version": "0.1.0",
  "forgeApiVersion": "1.x",
  "entry": "main.js",
  "permissions": ["ui.command.register", "notes.read", "ai.chat"],
  "contributes": {
    "commands": [{ "id": "research.summarize", "title": "Summarize Selection" }],
    "panels": [{ "id": "research.panel", "title": "Research" }],
    "dock": [{ "id": "research.open", "title": "Research", "icon": "🔎" }]
  }
}
```

**Plugin API capabilities:**
- `notes.read` / `notes.write` — vault file access (write is queued + audited)
- `vault.search` — search across vault
- `ai.chat` / `ai.context.read` — AI interaction
- `ui.panel.register` / `ui.command.register` — UI contributions
- `launcher.open` / `files.open_path` — external resource access
- `network.http` — opt-in, domain allowlist

**Plugin build order:**
1. Plugin manifest schema + discovery + validation
2. Plugin manager UI (enable/disable, permissions view)
3. Command contribution + command palette integration
4. Panel contribution system
5. AI + vault APIs
6. Queued writes + audit log
7. GitHub installer
8. Signing/trust levels

---

### PRIORITY 8: UX Improvements (Fast Wins)

From the Simple First Roadmap:

**Phase 1: Stability + Navigation**
- Tauri `single-instance` — prevent duplicate app sessions
- Tauri `window-state` — layouts persist naturally
- `cmdk` command palette: open notebook, run launcher, switch AI provider

**Phase 2: Real TypingMind Feel**
- Chat folders + pinned chats + prompt snippets
- `assistant-ui` primitives for advanced chat UX
- Drag reorder via `dnd-kit` for chat folders, dock icons, quick places

**Phase 3: Obsidian Bridge (No Fighting)**
- Obsidian Local REST API integration for note ops when Obsidian is open
- Advanced URI for deep links (open workspace/file/heading)
- Write-mode toggle: `Forge Direct` vs `Via Obsidian API`
- "Active writer" indicator in status bar

**Recommended libraries:**
- `pacocoursey/cmdk` — command palette
- `assistant-ui/assistant-ui` — React chat primitives
- `bvaughn/react-resizable-panels` — stable 2/3/4 pane layouts
- `clauderic/dnd-kit` — drag reorder
- `coddingtonbear/obsidian-local-rest-api` — Obsidian bridge
- Tauri plugins: `store`, `window-state`, `single-instance`, `global-shortcut`

**Integration boundary — define one interface for note operations:**
```typescript
interface NoteGateway {
  listNotes(): Promise<NoteEntry[]>
  readNote(path: string): Promise<string>
  writeNote(path: string, content: string, mode: 'overwrite' | 'append'): Promise<void>
  patchNote(path: string, patchSpec: PatchSpec): Promise<void>
  openInObsidian(path: string, heading?: string): Promise<void>
}
// Implementations: FileSystemNoteGateway (current), ObsidianRestNoteGateway (next)
```

---

## THE CORE LOOP (CANONICAL)

**This is the rule. If you're building something that doesn't make this work better, you're building the wrong thing.**

```
1. SELECT   — any grain size: letter / word / sentence / block / paragraph
2. CHAT BOX — pops up inline, right where you are (Ctrl+I)
3. DECLARE  — say what it is, what it means, what to do with it
4. CACHED   — AI stores it losslessly, enforces it going forward
```

No forms. No schema design. No dropdowns. No "pick a type from this list."
You teach the system by talking to it, exactly like you'd explain it to a person.

**Steps 1-3 are live.** Step 4 (cached enforcement) is next priority.

---

## REPO STRUCTURE

```
Forge/
├── README.md                       ← THIS FILE (complete build manifest)
├── FORGE_BUILD_SPEC_MASTER.md      ← Original full build spec
├── .gitignore
│
├── _FORGE_SOURCE/                  ← THE APP (Tauri + React + Rust)
│   ├── src/                        ← React frontend
│   │   ├── components/
│   │   │   ├── Editor/
│   │   │   │   ├── ForgeEditor.tsx      ← ✅ Grid integrated, inline AI, context menu, Ctrl+G/Ctrl+I
│   │   │   │   ├── EditorToolbar.tsx
│   │   │   │   ├── InlineAiChat.tsx     ← ✅ Selection→Instruct bubble (grid-aware, quick actions)
│   │   │   │   ├── GridLayer.tsx        ← NEW: Grid visualization overlay (optional)
│   │   │   │   └── PromotedBlock*.ts/x
│   │   │   ├── DataMirror/             ← NEW: Mirror folder UI
│   │   │   ├── VersionControl/          ← NEW: Version browser UI
│   │   │   ├── GlobalEngine/            ← NEW: Engine config UI
│   │   │   ├── miniapps/               ← LogicSheet, TruthLayerWorkbench
│   │   │   └── [Sidebar, AiPanel, BottomBar, Settings, etc.]
│   │   ├── lib/
│   │   │   ├── grid.ts                  ← ✅ Parallel grid: build, query, mutate, serialize (~300 lines)
│   │   │   ├── ai.ts                    ← ✅ 3-provider streaming + 3-role routing (~400 lines)
│   │   │   ├── aiPrompts.ts             ← ✅ /PROBE, /EAST, /CONNECT prompts
│   │   │   ├── aiRuntime.ts             ← ✅ Event logging + dedup ring buffer (~100 lines)
│   │   │   ├── annotations.ts           ← NEW: anchors, display rules, macros
│   │   │   ├── instructionCache.ts      ← NEW: cached instruction enforcement
│   │   │   ├── mirror.ts               ← NEW: Data mirror logic
│   │   │   ├── versioning.ts           ← NEW: Version control
│   │   │   ├── engine.ts               ← NEW: YAML engine runner
│   │   │   ├── ingestion.ts            ← NEW: Data ingestion layer
│   │   │   └── [settings, types, markdown, pythonSidecar, noteMeta]
│   │   └── hooks/
│   │       └── useGrid.ts              ← ✅ React hook: sync, rebuild, metadata persistence (~180 lines)
│   ├── src-tauri/                      ← Rust backend
│   │   ├── src/lib.rs                  ← Add: mirror, version, engine commands
│   │   ├── Cargo.toml                  ← Dependencies
│   │   └── tauri.conf.json             ← App config
│   ├── scripts/                        ← Python sidecar, truth-layer sync
│   ├── docs/                           ← Technical specs (selection annotation, truth layer, co-partner)
│   └── [package.json, vite.config.ts, tsconfig.json, etc.]
│
├── FORGE_DOCS/                         ← Design docs, specs, roadmaps
│   ├── FORGE_BUILD_SPEC_MASTER.md
│   ├── FORGE_SELECTION_ANNOTATION_SPEC.md  ← READ THIS FIRST
│   ├── FORGE_SIMPLE_FIRST_ROADMAP.md
│   ├── FORGE_PLUGIN_PLATFORM_ARCHITECTURE.md
│   ├── FORGE_DATA_LAYER_SPEC.md
│   ├── FORGE_RELEASE_WORKFLOW.md
│   ├── FORGE_HANDOFF_FOR_PROGRAMMER.md
│   └── [+ more specs and notes]
│
└── FORGE_SEMANTIC/                     ← Semantic layer specs
    ├── BIBLE_SEMANTIC_EXPLORER_AI_HANDOFF.md
    ├── CO_PARTNER_QUESTION_SYSTEM.md
    ├── forge-hybrid-template/
    ├── dashboard-plan/
    └── semantic-workspace-excel-layer/
```

---

## SETUP & DEVELOPMENT

### Prerequisites
- [Node.js](https://nodejs.org/) v18+
- [Rust](https://www.rust-lang.org/tools/install) (latest stable)
- [Tauri CLI](https://tauri.app/) (`cargo install tauri-cli`)
- PostgreSQL (optional — for Truth Layer persistence, at 192.168.1.177:2665)
- Python 3.10+ (optional — for AI sidecar scripts)

### Install & Run
```bash
cd _FORGE_SOURCE
npm install
npm run tauri:dev
```

### Build for Production
```bash
cd _FORGE_SOURCE
npm run tauri:build
```
Binary lands in `_FORGE_SOURCE/src-tauri/target/release/`.

### Slash Commands (Bottom Bar)
| Command | Action |
|---------|--------|
| `/ai [prompt]` | Open AI panel / send prompt to Interface role |
| `/logic [prompt]` | Send to Logic role |
| `/copilot [prompt]` | Send to Copilot role |
| `/open [title]` | Open or create a note by title |
| `/link [title]` | Same as `/open` |
| `/logicsheet` | Switch to Logic Sheet view |
| `/truth` | Switch to Truth Layer Workbench |
| `/editor` | Switch back to editor |
| `/python [instruction]` | Run Python sidecar action |
| `/settings` | Open settings |
| `/app [id]` | Launch a registered mini-app |

### Keyboard Shortcuts
| Shortcut | Action |
|----------|--------|
| `Ctrl+G` | Toggle Grid View (side-by-side cell inspector) |
| `Ctrl+I` | Open Inline AI Chat on selection |
| `Ctrl+Shift+A` | Toggle AI Panel |
| `Ctrl+,` | Toggle Settings |

---

## WHAT NOT TO CHANGE

- Do NOT change the Tauri backend commands that already work
- Do NOT replace TipTap with a different editor (extend it)
- Do NOT change the file structure conventions
- Do NOT add Electron or any non-Tauri runtime
- Do NOT require additional software beyond what Tauri needs
- Do NOT run two independent write pipelines to the same note without lock/queue
- Do NOT change the grid's immutable mutation pattern (React state depends on it)
- Do NOT remove the from/to ProseMirror position bridge in GridCell (the inline AI chat depends on it)

---

## REFERENCE DOCS (Read Before Building)

Read these in order:
1. `FORGE_DOCS/FORGE_SELECTION_ANNOTATION_SPEC.md` — THE breakthrough spec (Core Loop)
2. `FORGE_BUILD_SPEC_MASTER.md` — Complete build specification
3. `FORGE_DOCS/FORGE_SIMPLE_FIRST_ROADMAP.md` — Phase plan and fast wins
4. `FORGE_DOCS/FORGE_PLUGIN_PLATFORM_ARCHITECTURE.md` — Engine/plugin architecture
5. `FORGE_DOCS/FORGE_DATA_LAYER_SPEC.md` — Data ingestion concept
6. `FORGE_SEMANTIC/forge-hybrid-template/FORGE_HYBRID_METHOD_BLUEPRINT.md` — Hybrid editor spec
7. `_FORGE_SOURCE/docs/FORGE_SELECTION_ANNOTATION_SPEC.md` — Selection spec (copy in source)
8. `_FORGE_SOURCE/docs/truth-layer/README.md` — Truth Layer architecture

---

## THE RULE

> The three-layer editor is the product. Everything else supports it. If you're building something that doesn't make "highlight → instruct → done" work better, you're building the wrong thing.

---

*POF 2828 | FORGE: Framework for Orchestrated Research, Growth, and Execution*  
*"Input → Memorize → Reform → Store or Reject."*  
*The forge is always hot.*
