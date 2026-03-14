# FORGE Block & Zone Layout System
## Specification v0.1 — 2026-03-13

> **One sentence:** Every page in FORGE is a spatial composition of typed blocks arranged in drawable zones — you sketch the layout by splitting regions, then assign each region a block type with a keyboard shortcut.

---

## 1. THE TWO PRIMITIVES

### Zones (the containers)
A zone is a rectangular region of the page. The page starts as one zone (full width, full height). You split zones by drawing lines — horizontal splits create top/bottom, vertical splits create left/right. Zones can be split recursively. Each leaf zone holds one or more blocks.

### Blocks (the content)
A block is a typed piece of content that lives inside a zone. There are seven block types. That's it. Everything in FORGE is one of these seven things.

---

## 2. THE SEVEN BLOCK TYPES

### Type 1: TEXT — `Ctrl+Alt+T`
The default. What TipTap already does.

**Subtypes (scroll/arrow to select after shortcut):**
| Subtype | What it does |
|---------|-------------|
| **Header** | H1–H6. Options: center, left, right. Collapsible (fold open/closed). |
| **Paragraph** | Plain prose. The workhorse. |
| **Bullet List** | Unordered list. Pick from style library (disc, circle, dash, arrow, checkbox, custom icon). |
| **Numbered List** | Ordered list. Pick style (1/2/3, a/b/c, i/ii/iii, roman). |
| **Blockquote** | Indented quote block. Optional attribution field. |
| **Callout** | Colored box with icon + title + body (info, warning, tip, note, custom). |
| **Code** | Syntax-highlighted code block with language selector. |
| **Toggle** | Collapsible content — heading visible, body folds. |

**Style library:** Each subtype can have multiple visual styles (fonts, spacing, colors, borders). User picks a style number or name. Styles are CSS-driven and customizable per vault. "Pick style 99" means the 99th style in your library for that subtype. Styles are stored in `_engines/styles/` as YAML + CSS pairs.

**Expand/Contract behavior:** Any text block can be collapsed (shows just the first line or header) or expanded (shows full content). State persists per block. Keyboard: `Ctrl+.` to toggle.

---

### Type 2: MEDIA — `Ctrl+Alt+M`

**Subtypes (scroll to select):**
| Subtype | Options |
|---------|---------|
| **Image** | Resizable: yes/no. Text wrap: around/above-below/none. Alignment: left/center/right/float. Caption: yes/no. Alt text field. |
| **Video** | Standard embed size. Autoplay: yes/no. Loop: yes/no. Source: file/URL/embed code. |
| **Audio** | Playable inline. Downloadable: yes/no. Waveform visualization: yes/no. |
| **PDF** | Embedded viewer. Page navigation. Scrollable. |
| **File** | Generic file attachment. Download link + icon + filename + size. |

**Resizable behavior:** Drag corners/edges to resize. Hold Shift for proportional. Double-click to reset to original.

**Text wrap modes:**
- **Around** — text flows around the media (magazine style, the thing nobody does well)
- **Above/Below** — media takes full zone width, text before and after
- **None** — media is inline, breaks the text flow
- **Float** — media floats to left or right edge, text wraps opposite side

---

### Type 3: CARD — `Ctrl+Alt+C`
A structured data display with a defined layout. This is the Jane Jacobs entity from that screenshot.

**Card anatomy:**
```
┌──────────────────────────────┐
│  [Image/Avatar]   Title      │
│                   Subtitle   │
│  Tags: ████ ████ ████        │
├──────────────────────────────┤
│  Description / Bio / Summary │
│  (markdown-enabled text)     │
├──────────────────────────────┤
│  [Action 1]  [Action 2]  [Action 3]  │
└──────────────────────────────┘
```

**Fields:**
- **Image/Avatar** — optional, left-aligned or top-centered
- **Title** — required, the name/heading
- **Subtitle** — optional, one line
- **Tags** — colored tag pills
- **Body** — markdown text, expandable
- **Actions** — 1–5 buttons (configurable: link, command, AI action, navigate)

**Card schemas (presets, user can create more):**
| Schema | Fields | Use case |
|--------|--------|----------|
| **Person** | Photo, Name, Role, Bio, Tags, Links | Contacts, authors, researchers |
| **Book** | Cover, Title, Author, Summary, Rating, Tags | Reading list, library |
| **Concept** | Icon, Name, Definition, Related, Tags | Glossary, framework terms |
| **Paper** | Title, Authors, Abstract, DOI, Tags, Status | Research tracking |
| **Verse** | Reference, Text, Cross-refs, Tags, Notes | Bible/scripture study |
| **Project** | Icon, Name, Status, Description, Links, Tags | Project tracking |
| **Custom** | User-defined fields | Anything else |

Cards are stored as YAML frontmatter + markdown body. They render as the visual card but are plain text underneath.

---

### Type 4: TABLE — `Ctrl+Alt+D` (D for Data)

**Two modes:**

**Simple Table** — what TipTap already supports. Rows and columns, editable cells, headers.

**Database View** — Notion-style. This is the one that matters.
- **Table view** — spreadsheet grid with sortable/filterable columns
- **Board view** — kanban columns (group by a status/tag field)
- **Gallery view** — cards in a grid (each row renders as a mini-card)
- **List view** — compact single-line per row
- **Calendar view** — rows placed on dates (requires a date field)

**Column types:** Text, Number, Date, Checkbox, Select, Multi-select, URL, Email, Person, Relation, Formula, Rollup, File, Created/Modified time.

**Data source:** Inline (stored in the note), or linked (pulls from PostgreSQL, CSV, or another FORGE database).

---

### Type 5: GRAPH — `Ctrl+Alt+G`
A bounded knowledge graph visualization.

**Modes:**
- **Local graph** — nodes connected to the current note
- **Custom query** — show nodes matching a tag, flag, or search
- **Full vault** — the whole graph (filterable)
- **Subgraph** — manually selected nodes

**Interactions:** Click node to navigate. Drag to rearrange. Right-click for context menu (open, link, tag, remove). Zoom/pan. Force-directed layout with options (tree, radial, hierarchical).

**Data:** Pulls from the grid layer (links between notes) and any explicit graph edges stored in note metadata.

---

### Type 6: CODE — `Ctrl+Alt+X` (X for eXecute)
An executable code block with live output.

**Supported languages:** Python (via sidecar), JavaScript (in-browser), SQL (against PostgreSQL), Shell (via Tauri).

**Layout:** Code on top, output below. Or side-by-side. Output can be: text, table, chart, image, error.

**Execution:** Run button, or `Ctrl+Enter`. Output persists until re-run. Variables persist within a page session (like Jupyter cells).

---

### Type 7: EMBED — `Ctrl+Alt+E`
A live view of something external.

**Types:**
- **URL** — iframe embed (YouTube, Maps, Figma, etc.)
- **Query** — live PostgreSQL query result (auto-refreshes)
- **API** — REST endpoint response, formatted
- **Dashboard widget** — a mini-component from the Theophysics dashboard
- **Another note** — transclusion (live preview of another FORGE note)
- **Calendar/Agenda** — date-based view

---

## 3. ZONE LAYOUT SYSTEM

### Creating zones
The page starts as one zone. To split:

**Method 1: Keyboard**
- `Ctrl+Alt+|` (pipe) — vertical split at cursor position (left/right)
- `Ctrl+Alt+-` (dash) — horizontal split at cursor position (top/bottom)

**Method 2: Drag**
- Drag from the zone edge inward to create a split
- A guide line appears showing where the split will land
- Release to confirm

**Method 3: Preset layouts**
- `Ctrl+Alt+L` opens layout picker
- Presets: Single column, Two columns (50/50, 33/67, 67/33), Three columns, Grid (2×2), Sidebar+Main, Header+Body+Footer

### Zone properties
Each zone has:
- **Width ratio** — percentage of parent (adjustable by dragging divider)
- **Min width** — prevents zone from collapsing below usable size
- **Padding** — internal spacing
- **Background** — optional color/gradient
- **Border** — optional

### Zone operations
- **Merge** — select two adjacent zones → merge into one
- **Swap** — select two zones → swap their contents
- **Delete** — remove zone, adjacent zone expands to fill
- **Resize** — drag the divider between zones

### Responsive behavior
On narrow viewports (mobile, small window), zones stack vertically in document order. The spatial layout is a progressive enhancement — the content is always readable in a single column.

---

## 4. DATA MODEL

### Zone tree (stored per page)
```typescript
interface Zone {
  id: string;
  split?: {
    direction: 'horizontal' | 'vertical';
    ratio: number;          // 0.0–1.0, position of the split
  };
  children?: [Zone, Zone];  // exactly two children if split
  blocks?: Block[];         // leaf zones have blocks, not children
  style?: ZoneStyle;
}

interface ZoneStyle {
  padding?: string;
  background?: string;
  border?: string;
  minWidth?: number;
  minHeight?: number;
}
```

### Block (stored per zone)
```typescript
interface Block {
  id: string;
  type: 'text' | 'media' | 'card' | 'table' | 'graph' | 'code' | 'embed';
  subtype?: string;         // e.g., 'header', 'image', 'person', 'database'
  config: BlockConfig;      // type-specific configuration
  content: any;             // the actual content (markdown, image path, query, etc.)
  style?: string;           // style name/number from the style library
  gridEnabled?: boolean;    // whether this block participates in the grid layer
}

// Type-specific configs
interface TextBlockConfig {
  subtype: 'header' | 'paragraph' | 'bullet' | 'numbered' | 'blockquote' | 'callout' | 'code' | 'toggle';
  headerLevel?: 1 | 2 | 3 | 4 | 5 | 6;
  alignment?: 'left' | 'center' | 'right';
  collapsed?: boolean;
  listStyle?: string;       // style name from library
}

interface MediaBlockConfig {
  subtype: 'image' | 'video' | 'audio' | 'pdf' | 'file';
  src: string;
  resizable?: boolean;
  textWrap?: 'around' | 'above-below' | 'none' | 'float-left' | 'float-right';
  caption?: string;
  altText?: string;
  downloadable?: boolean;
}

interface CardBlockConfig {
  schema: string;           // 'person' | 'book' | 'concept' | 'paper' | 'verse' | 'project' | 'custom'
  fields: Record<string, any>;
  actions?: CardAction[];
  image?: string;
}

interface CardAction {
  label: string;
  type: 'link' | 'command' | 'ai' | 'navigate';
  target: string;
}

interface TableBlockConfig {
  mode: 'simple' | 'database';
  view?: 'table' | 'board' | 'gallery' | 'list' | 'calendar';
  source?: 'inline' | 'postgres' | 'csv' | 'forge-db';
  columns?: ColumnDef[];
  sortBy?: string;
  filterBy?: string;
  groupBy?: string;
}

interface GraphBlockConfig {
  mode: 'local' | 'query' | 'full' | 'subgraph';
  query?: string;
  layout?: 'force' | 'tree' | 'radial' | 'hierarchical';
  depth?: number;
  filter?: string;
}

interface CodeBlockConfig {
  language: 'python' | 'javascript' | 'sql' | 'shell';
  outputLayout?: 'below' | 'side';
  autoRun?: boolean;
}

interface EmbedBlockConfig {
  subtype: 'url' | 'query' | 'api' | 'dashboard' | 'note' | 'calendar';
  src: string;
  refreshInterval?: number;  // seconds, 0 = manual
}
```

---

## 5. KEYBOARD SHORTCUT MAP

### Block creation (in a zone)
| Shortcut | Block Type | Then... |
|----------|-----------|---------|
| `Ctrl+Alt+T` | Text | Arrow keys to scroll subtypes: Header → Paragraph → Bullet → Numbered → Blockquote → Callout → Code → Toggle |
| `Ctrl+Alt+M` | Media | Arrow keys: Image → Video → Audio → PDF → File |
| `Ctrl+Alt+C` | Card | Arrow keys: Person → Book → Concept → Paper → Verse → Project → Custom |
| `Ctrl+Alt+D` | Table/Data | Arrow keys: Simple → Database (then view picker) |
| `Ctrl+Alt+G` | Graph | Arrow keys: Local → Query → Full → Subgraph |
| `Ctrl+Alt+X` | Code | Arrow keys: Python → JavaScript → SQL → Shell |
| `Ctrl+Alt+E` | Embed | Arrow keys: URL → Query → API → Dashboard → Note → Calendar |

### After subtype selection
- `Enter` — confirm and create the block
- `Tab` — cycle through configuration options (alignment, wrap mode, etc.)
- `Escape` — cancel

### Zone operations
| Shortcut | Action |
|----------|--------|
| `Ctrl+Alt+\|` | Split zone vertically |
| `Ctrl+Alt+-` | Split zone horizontally |
| `Ctrl+Alt+L` | Open layout preset picker |
| `Ctrl+Alt+Backspace` | Delete current zone |
| `Ctrl+.` | Collapse/expand current block |

### Navigation
| Shortcut | Action |
|----------|--------|
| `Ctrl+Alt+Arrow` | Move focus between zones |
| `Tab` (in zone selector mode) | Cycle through zones |

---

## 6. STYLE LIBRARY

Styles are stored in `_engines/styles/` as YAML + CSS:

```yaml
# _engines/styles/bullet-modern.yaml
name: Modern Bullets
type: text.bullet
css: bullet-modern.css
preview: "• Clean, spaced, subtle indent guides"
```

```css
/* _engines/styles/bullet-modern.css */
.forge-bullet-modern li {
  padding: 4px 0;
  border-left: 2px solid var(--forge-accent);
  padding-left: 12px;
  margin-left: 8px;
}
.forge-bullet-modern li::marker {
  color: var(--forge-accent);
}
```

Users pick styles by number or name in the block config popup. The style library is extensible — drop a YAML+CSS pair into the folder, it appears in the picker.

---

## 7. RELATIONSHIP TO EXISTING SYSTEMS

| Existing system | How blocks interact with it |
|----------------|---------------------------|
| **Grid Layer** (grid.ts) | Text blocks are grid-enabled by default. Every word in a text block gets `[row, col]` coordinates. Other block types can opt into grid addressing. |
| **Inline AI Chat** (InlineAiChat.tsx) | Works inside any text block. At the zone level, AI can rearrange blocks, change types, modify configs. |
| **TipTap editor** | Text blocks ARE TipTap instances. Each text block in a zone is its own TipTap editor (or a single TipTap with node views for other block types). |
| **Markdown storage** | Blocks serialize to extended markdown. Text = standard markdown. Cards = YAML frontmatter + body. Tables = markdown tables or CSV. Zones = HTML comments marking layout boundaries. |
| **Data Mirror** | Generated outputs (charts, exports, AI results) go to `_data/` mirror. Source content stays clean. |

---

## 8. BUILD ORDER

1. **Zone tree data model** — `src/lib/zones.ts` — the split tree, serialize/deserialize
2. **Zone renderer** — `src/components/Layout/ZoneRenderer.tsx` — recursive rendering of the split tree with draggable dividers
3. **Block registry** — `src/lib/blocks.ts` — block type definitions, configs, defaults
4. **Block creation UI** — keyboard shortcut handler + subtype picker popup
5. **Text block** — already exists (TipTap), wire into zone system
6. **Media block** — image/video/audio with resize + wrap
7. **Card block** — schema system, field rendering, action buttons
8. **Table block (simple)** — extend existing TipTap table
9. **Table block (database)** — Notion-style views (table, board, gallery, list, calendar)
10. **Graph block** — knowledge graph renderer (D3 or similar)
11. **Code block** — executable with output pane
12. **Embed block** — iframe + query + API
13. **Style library** — YAML+CSS loading, style picker
14. **Layout presets** — preset zone arrangements
15. **Responsive stacking** — mobile/narrow viewport behavior

---

## 9. FILES TO CREATE

| File | Purpose |
|------|---------|
| `src/lib/zones.ts` | Zone tree: split, merge, serialize, query |
| `src/lib/blocks.ts` | Block type registry, configs, defaults, validation |
| `src/lib/blockStyles.ts` | Style library loader (YAML+CSS from `_engines/styles/`) |
| `src/components/Layout/ZoneRenderer.tsx` | Recursive zone layout renderer |
| `src/components/Layout/ZoneDivider.tsx` | Draggable divider between zones |
| `src/components/Layout/LayoutPresets.tsx` | Preset layout picker (`Ctrl+Alt+L`) |
| `src/components/Blocks/TextBlock.tsx` | TipTap instance in a zone |
| `src/components/Blocks/MediaBlock.tsx` | Image/video/audio with resize + wrap |
| `src/components/Blocks/CardBlock.tsx` | Schema-driven card renderer |
| `src/components/Blocks/TableBlock.tsx` | Simple + database views |
| `src/components/Blocks/GraphBlock.tsx` | Knowledge graph visualization |
| `src/components/Blocks/CodeBlock.tsx` | Executable code with output |
| `src/components/Blocks/EmbedBlock.tsx` | URL/query/API/note embed |
| `src/components/Blocks/BlockCreator.tsx` | Shortcut handler + subtype picker |

---

*POF 2828 | FORGE Block & Zone Layout System*
*"Draw the container. Tell it what it is. Done."*
