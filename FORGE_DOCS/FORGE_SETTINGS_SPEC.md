# FORGE Settings System Specification
## v0.1 — 2026-03-13

> **Goal:** Every setting a user needs to make FORGE feel like their tool, organized so they find things intuitively, with sensible defaults so they never HAVE to touch settings unless they want to.

---

## CURRENT STATE

The settings page (`SettingsPage.tsx`) has three sections:
1. **General** — autosave delay slider, top prompt bar toggle
2. **AI Layers** — provider picker, role routing, Ollama model, per-role config, workspace context toggle
3. **Mini Apps** — add/remove web app launchers

**Missing:** Everything else. Editor config, appearance, file handling, keyboard shortcuts, vault settings, block/zone defaults, plugin management, database connections, backup/export, accessibility.

---

## SETTINGS ARCHITECTURE

### Organization (sidebar navigation, like Obsidian)

```
Settings
├── General
├── Editor
├── Appearance
├── Files & Vaults
├── Keyboard Shortcuts
├── AI Configuration
├── Blocks & Zones
├── Database & Connections
├── Plugins & Engines
├── Backup & Export
└── About
```

### Storage

Settings persist in localStorage (`forge_settings_v1`). Vault-specific overrides stored in `<vault>/_forge/settings.json`. Global settings apply everywhere; vault settings override where specified.

**Scope indicators:** Each setting shows a small badge: `[Global]` or `[Vault]`. Vault-level overrides are optional — if not set, global applies.

---

## 1. GENERAL

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| Language | Select | System default | Interface language |
| Autosave Delay | Slider (0.5s–10s) | 2.0s | Delay after typing before auto-saving |
| Confirm Before Delete | Toggle | On | Ask before deleting files/notes |
| Show Startup Tips | Toggle | On | Show tip of the day on launch |
| Check for Updates | Toggle | On | Auto-check for FORGE updates |
| Telemetry | Toggle | Off | Anonymous usage analytics (opt-in only) |
| Default New Note Location | Select | Vault root / Same folder / Specified folder | Where new notes are created |
| Default Attachment Location | Select | Vault root / Same folder / Subfolder `_attachments` | Where dropped files go |

---

## 2. EDITOR

### Display
| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| Readable Line Length | Toggle | On | Limit line width for readability |
| Max Line Width | Slider (60–120 chars) | 80 | Maximum characters per line (when readable length on) |
| Show Line Numbers | Toggle | Off | Line numbers in gutter |
| Show Indentation Guides | Toggle | On | Vertical lines showing indent depth |
| Fold Headings | Toggle | On | Allow collapsing content under headings |
| Fold Indented Blocks | Toggle | On | Allow collapsing lists and indented content |
| Show Word Count | Toggle | On | Word count in status bar |
| Show Character Count | Toggle | Off | Character count in status bar |
| Show Reading Time | Toggle | Off | Estimated reading time in status bar |

### Behavior
| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| Spellcheck | Toggle | On | Enable browser spellcheck |
| Spellcheck Language | Select | English (US) | Language(s) for spellcheck |
| Auto-pair Brackets | Toggle | On | Auto-close `( [ { " '` |
| Auto-pair Markdown | Toggle | On | Auto-close `** __ \`\`` |
| Smart Lists | Toggle | On | Auto-continue and indent lists |
| Tab Behavior | Select | Indent / Tab character | What Tab key does |
| Tab Width | Select (2/4/8) | 4 | Spaces per tab/indent |
| Paste HTML as Markdown | Toggle | On | Convert pasted HTML to markdown |
| Default View | Select | Editing / Reading | Default mode for new notes |
| Vim Mode | Toggle | Off | Vim key bindings |

### Grid Layer
| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| Grid Enabled | Toggle | On | Enable the addressable grid layer |
| Grid Rebuild Debounce | Slider (50ms–500ms) | 150ms | Delay before grid rebuild after edit |
| Show Grid Stats in Status Bar | Toggle | On | Show `{rows}r/{cells}c` in status bar |
| Auto-Tag on Flag | Toggle | Off | When flagging a row, auto-prompt for a tag too |
| Metadata Persistence | Select | Per-note JSON / Per-vault DB | Where grid metadata (tags, flags) is stored |

---

## 3. APPEARANCE

### Theme
| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| Color Scheme | Select | Adapt to system / Light / Dark | Base color mode |
| Accent Color | Color picker | `#e8a912` (forge ember) | Primary accent color used throughout |
| Theme | Select + Browse | FORGE Default | Community themes (CSS-based) |
| Custom CSS | File path | None | Path to custom CSS snippet file |

### Fonts
| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| Interface Font | Font picker | System default | Font for menus, panels, UI |
| Editor Font | Font picker | System default | Font for note editing |
| Monospace Font | Font picker | `JetBrains Mono` / system mono | Font for code blocks, grid view |
| Font Size | Slider (10–24px) | 16px | Editor font size |
| Line Height | Slider (1.0–2.5) | 1.6 | Line spacing in editor |
| Zoom Level | Slider (75%–150%) | 100% | Overall app zoom |

### Interface
| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| Show Sidebar | Toggle | On | Show file tree sidebar |
| Sidebar Position | Select | Left / Right | Which side the file tree is on |
| Sidebar Width | Slider (180–400px) | 240px | Default sidebar width |
| Show Status Bar | Toggle | On | Bottom status bar |
| Show Top Prompt Bar | Toggle | On | Top-of-window prompt bar |
| Show Ribbon | Toggle | Off | Vertical icon toolbar (Obsidian-style) |
| Window Frame | Select | FORGE frame / Native / Hidden | Title bar style |
| Tab Bar Visible | Toggle | On | Show tab bar for open notes |
| Translucent Background | Toggle | Off | Window transparency effect (dark mode) |

---

## 4. FILES & VAULTS

### Vault Management
| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| Active Vault | Display + Switch | — | Currently open vault with switch button |
| Recent Vaults | List | — | Quick switch between recent vaults |
| New Note Extension | Select | `.md` / `.txt` | Default extension for new notes |
| Show All File Types | Toggle | Off | Show non-markdown files in tree |

### Links
| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| Link Format | Select | Wikilinks `[[]]` / Markdown `[]()` | Default internal link format |
| Auto-Update Links on Rename | Toggle | On | Fix links when file is renamed |
| New Link Style | Select | Shortest path / Relative / Absolute | How auto-generated links are formatted |

### Trash & Deletion
| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| Deleted Files | Select | System trash / FORGE trash (`.trash/`) / Permanent | What happens on delete |
| Confirm File Deletion | Toggle | On | Ask before deleting |

### Data Mirror
| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| Enable Data Mirror | Toggle | On | Auto-create `_data/` mirror directories |
| Mirror Location | Select | Sibling `_data/` / Custom path | Where mirror folders live |
| Auto-Clean Mirror | Toggle | Off | Remove mirror files when source is deleted |

---

## 5. KEYBOARD SHORTCUTS

A searchable, rebindable list of all commands and their shortcuts. Filterable to show only assigned or only unassigned.

### Default shortcuts (user can rebind any of these)

**Editor**
| Shortcut | Command |
|----------|---------|
| `Ctrl+S` | Force save |
| `Ctrl+Z` | Undo |
| `Ctrl+Shift+Z` | Redo |
| `Ctrl+B` | Bold |
| `Ctrl+I` | Italic (or Inline AI Chat if text selected) |
| `Ctrl+U` | Underline |
| `Ctrl+K` | Insert link |
| `Ctrl+.` | Toggle fold/expand |
| `Ctrl+/` | Toggle comment |

**Navigation**
| Shortcut | Command |
|----------|---------|
| `Ctrl+P` | Command palette |
| `Ctrl+O` | Quick open (note switcher) |
| `Ctrl+Shift+A` | Toggle AI panel |
| `Ctrl+,` | Settings |
| `Ctrl+\` | Toggle sidebar |
| `Ctrl+Tab` | Next tab |
| `Ctrl+Shift+Tab` | Previous tab |
| `Ctrl+W` | Close tab |

**FORGE-Specific**
| Shortcut | Command |
|----------|---------|
| `Ctrl+G` | Toggle grid view |
| `Ctrl+I` (with selection) | Inline AI chat |
| `Ctrl+Alt+T` | Create text block |
| `Ctrl+Alt+M` | Create media block |
| `Ctrl+Alt+C` | Create card block |
| `Ctrl+Alt+D` | Create data/table block |
| `Ctrl+Alt+G` | Create graph block |
| `Ctrl+Alt+X` | Create code block |
| `Ctrl+Alt+E` | Create embed block |
| `Ctrl+Alt+\|` | Split zone vertical |
| `Ctrl+Alt+-` | Split zone horizontal |
| `Ctrl+Alt+L` | Layout presets |

Implementation: All shortcuts stored in `settings.shortcuts` as `Record<string, string>`. The hotkey page shows the command name, current binding, and a "record new shortcut" button.

---

## 6. AI CONFIGURATION

### Provider
| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| Primary Provider | Select | Ollama / Anthropic / OpenAI | Default AI backend |
| Anthropic API Key | Password field | — | `sk-ant-*` key (stored in localStorage, never synced) |
| OpenAI API Key | Password field | — | `sk-*` key (stored in localStorage, never synced) |
| Ollama Model | Text | `llama3.1:8b` | Local model name |
| Ollama Endpoint | Text | `http://127.0.0.1:11434` | Ollama server URL (for remote Ollama) |

### Role Routing
| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| Routing Mode | Select | Shared / Split | Whether all roles share one provider or each gets its own |
| Interface Role | Provider + Model | (inherits primary) | Provider and model for Interface role |
| Logic Role | Provider + Model | (inherits primary) | Provider and model for Logic role |
| Copilot Role | Provider + Model | (inherits primary) | Provider and model for Copilot role |

### Behavior
| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| Include Workspace Context | Toggle | On | Send current note context to AI |
| Context Window Size | Select | Small (2K) / Medium (8K) / Large (32K) | How much context to include |
| Background Scan Debounce | Slider (2s–30s) | 6s | Delay before Logic/Copilot fire after edit |
| Background Scan Enabled | Toggle | On | Whether Logic and Copilot roles auto-fire |
| Max Response Length | Slider (256–4096 tokens) | 2048 | Max tokens per AI response |
| Streaming | Toggle | On | Stream tokens as they arrive |
| AI Temperature | Slider (0.0–1.5) | 0.7 | Creativity/randomness of AI responses |

### Inline AI Chat
| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| Auto-Replace on Transform | Toggle | On | Auto-replace selection for "fix/rewrite/improve" type instructions |
| Show Quick Actions | Toggle | On | Show Tag/Flag/Explain/Fix/Summarize buttons in bubble |
| Bubble Position | Select | Below selection / Above selection / Auto | Where the chat bubble appears |

---

## 7. BLOCKS & ZONES

### Block Defaults
| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| Default Text Style | Select from style library | Clean (style 1) | Default visual style for new text blocks |
| Default Card Schema | Select | Custom | Default schema when creating cards |
| Default Code Language | Select | Python | Default language for new code blocks |
| Default Image Wrap | Select | Above-below | Default text wrap for images |
| Default Image Resizable | Toggle | On | Whether new images are resizable by default |
| Default Table Mode | Select | Simple / Database | Default for new table blocks |

### Zone Defaults
| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| Default Layout | Select from presets | Single column | Layout for new pages |
| Zone Padding | Slider (0–32px) | 8px | Default internal padding for zones |
| Show Zone Borders | Toggle | Off | Show dotted borders around zones in edit mode |
| Snap to Grid | Toggle | On | Zone splits snap to percentages (25%, 33%, 50%, etc.) |
| Minimum Zone Width | Slider (100–300px) | 150px | Smallest zone allowed |

### Style Library
| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| Style Directory | Path | `_engines/styles/` | Where YAML+CSS style definitions live |
| Refresh Styles | Button | — | Rescan style directory |
| Manage Styles | Link | — | Opens style browser/editor |

---

## 8. DATABASE & CONNECTIONS

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| PostgreSQL Host | Text | `192.168.1.177` | Database server address |
| PostgreSQL Port | Number | `2665` | Database port |
| PostgreSQL Database | Text | `theophysics` | Default database name |
| PostgreSQL User | Text | `Yellowkid` | Database username |
| PostgreSQL Password | Password | — | Database password (stored locally, never synced) |
| Test Connection | Button | — | Verify database connectivity |
| Auto-Connect on Launch | Toggle | Off | Connect to DB when FORGE opens |

---

## 9. PLUGINS & ENGINES

### Engines
| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| Engine Directory | Path | `_engines/` | Where YAML engine configs live |
| Refresh Engines | Button | — | Rescan engine directory |
| Engine List | Toggle list | — | Each discovered engine with on/off toggle + settings icon |

### Mini Apps
| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| Mini App List | Add/Remove | — | Web apps with name + URL (existing feature) |

### Plugins (future)
| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| Plugin Directory | Path | `plugins/` | Where plugins live |
| Enable Community Plugins | Toggle | Off (restricted mode) | Safety gate for third-party code |
| Installed Plugins | Toggle list | — | Each plugin with enable/disable + settings + uninstall |

---

## 10. BACKUP & EXPORT

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| Auto-Backup | Toggle | Off | Periodic vault backup |
| Backup Interval | Select | Daily / Weekly / Monthly | How often to back up |
| Backup Location | Path | System default | Where backups are stored |
| Export Format | Select | Markdown / HTML / PDF / DOCX | Default export format |
| Include Grid Metadata in Export | Toggle | On | Include tags, flags, links in exported files |
| Export Vault | Button | — | Full vault export to zip |
| Import Vault | Button | — | Import from zip/folder |

---

## 11. ABOUT

| Item | Content |
|------|---------|
| Version | `1.1.0` |
| Build | Tauri 2 + React 19 + TypeScript |
| Author | David Lowery / POF 2828 |
| License | — |
| Changelog | Link to release notes |
| Reset All Settings | Button (with confirmation) |
| Open Config Folder | Button |
| Debug Info | Collapsible panel: OS, Tauri version, Node version, Rust version, vault path, DB status |

---

## UI DESIGN NOTES

1. **Sidebar navigation** — settings categories listed vertically on the left, content on the right. Like Obsidian, VS Code, or any modern desktop app. NOT tabs across the top.

2. **Search bar** — top of settings page, filters all settings by keyword. Type "font" → shows only font-related settings across all categories.

3. **Sections within pages** — grouped with subtle borders and section headers. Each setting is a single row: label on left, control on right.

4. **Reset buttons** — each section has a "Reset to defaults" link. Individual settings have a reset icon on hover.

5. **Vault override indicator** — settings that have vault-level overrides show a small `[Vault]` badge. Click to clear the override and revert to global.

6. **Responsive** — settings page works at narrow widths (sidebar collapses to hamburger menu).

---

## DATA MODEL UPDATE

```typescript
interface ForgeSettings {
  // General
  language: string;
  autosaveDelayMs: number;
  confirmBeforeDelete: boolean;
  showStartupTips: boolean;
  checkForUpdates: boolean;
  telemetry: boolean;
  defaultNoteLocation: 'vault-root' | 'same-folder' | 'specified';
  defaultNoteFolder?: string;
  defaultAttachmentLocation: 'vault-root' | 'same-folder' | 'subfolder';
  defaultAttachmentFolder?: string;

  // Editor — Display
  readableLineLength: boolean;
  maxLineWidth: number;
  showLineNumbers: boolean;
  showIndentGuides: boolean;
  foldHeadings: boolean;
  foldIndents: boolean;
  showWordCount: boolean;
  showCharCount: boolean;
  showReadingTime: boolean;

  // Editor — Behavior
  spellcheck: boolean;
  spellcheckLang: string;
  autoPairBrackets: boolean;
  autoPairMarkdown: boolean;
  smartLists: boolean;
  tabBehavior: 'indent' | 'tab';
  tabWidth: 2 | 4 | 8;
  pasteHtmlAsMarkdown: boolean;
  defaultView: 'editing' | 'reading';
  vimMode: boolean;

  // Editor — Grid
  gridEnabled: boolean;
  gridRebuildDebounce: number;
  showGridStats: boolean;
  autoTagOnFlag: boolean;
  metadataPersistence: 'per-note' | 'per-vault';

  // Appearance — Theme
  colorScheme: 'system' | 'light' | 'dark';
  accentColor: string;
  theme: string;
  customCssPath?: string;

  // Appearance — Fonts
  interfaceFont: string;
  editorFont: string;
  monospaceFont: string;
  fontSize: number;
  lineHeight: number;
  zoomLevel: number;

  // Appearance — Interface
  showSidebar: boolean;
  sidebarPosition: 'left' | 'right';
  sidebarWidth: number;
  showStatusBar: boolean;
  topPromptBarEnabled: boolean;
  showRibbon: boolean;
  windowFrame: 'forge' | 'native' | 'hidden';
  tabBarVisible: boolean;
  translucentBackground: boolean;

  // Files & Vaults
  newNoteExtension: '.md' | '.txt';
  showAllFileTypes: boolean;
  linkFormat: 'wikilinks' | 'markdown';
  autoUpdateLinks: boolean;
  newLinkStyle: 'shortest' | 'relative' | 'absolute';
  deletedFiles: 'system-trash' | 'forge-trash' | 'permanent';
  enableDataMirror: boolean;
  mirrorLocation: 'sibling' | 'custom';
  mirrorCustomPath?: string;
  autoCleanMirror: boolean;

  // Keyboard Shortcuts
  shortcuts: Record<string, string>;

  // AI Configuration
  aiProvider: 'ollama' | 'anthropic' | 'openai';
  aiRoleRouting: 'shared' | 'split';
  ollamaModel: string;
  ollamaEndpoint: string;
  aiRoles: Record<AiRole, AiRoleConfig>;
  aiUseWorkspaceContext: boolean;
  aiContextWindowSize: 'small' | 'medium' | 'large';
  aiBackgroundScanDebounce: number;
  aiBackgroundScanEnabled: boolean;
  aiMaxResponseTokens: number;
  aiStreaming: boolean;
  aiTemperature: number;
  aiAutoReplaceOnTransform: boolean;
  aiShowQuickActions: boolean;
  aiBubblePosition: 'below' | 'above' | 'auto';

  // Blocks & Zones
  defaultTextStyle: string;
  defaultCardSchema: string;
  defaultCodeLanguage: string;
  defaultImageWrap: 'around' | 'above-below' | 'none' | 'float-left' | 'float-right';
  defaultImageResizable: boolean;
  defaultTableMode: 'simple' | 'database';
  defaultLayout: string;
  zonePadding: number;
  showZoneBorders: boolean;
  snapToGrid: boolean;
  minZoneWidth: number;
  styleDirectory: string;

  // Database
  pgHost: string;
  pgPort: number;
  pgDatabase: string;
  pgUser: string;
  pgAutoConnect: boolean;

  // Plugins & Engines
  engineDirectory: string;
  pluginDirectory: string;
  communityPluginsEnabled: boolean;
  miniApps: MiniApp[];

  // Backup
  autoBackup: boolean;
  backupInterval: 'daily' | 'weekly' | 'monthly';
  backupLocation: string;
  defaultExportFormat: 'markdown' | 'html' | 'pdf' | 'docx';
  includeGridMetaInExport: boolean;
}
```

---

## BUILD ORDER

1. **Expand `ForgeSettings` type** in `types.ts` with all new fields
2. **Update `settings.ts`** with defaults and parsing for every new field
3. **Rebuild `SettingsPage.tsx`** with sidebar navigation + 11 sections
4. **Add search/filter** to settings page
5. **Keyboard shortcut editor** — record new binding, conflict detection
6. **Theme system** — CSS variable overrides, theme file loading
7. **Font picker** — list system fonts, preview
8. **Vault-level overrides** — `_forge/settings.json` per vault, merge logic

---

*POF 2828 | FORGE Settings System*
*"If they can't configure the basics, they'll never reach the powerful stuff."*
