# DEBUG TASK — Claude Debugging Codex's Code

## What Happened
The source code in `_FORGE_SOURCE/` was written by Codex AI (originally in Forge-v1-Codex repo). It has been placed here for you (Claude) to debug, fix, and improve.

The original Claude code is preserved on the `main` branch. This swapped code is on the `codex-code-debug` branch.

## Your Job
1. Get this code to compile and run cleanly (`npm install && npm run dev`)
2. Fix any TypeScript errors, missing imports, broken references
3. Test each component actually works in the browser
4. Report what you fixed

## What Codex Built (that you need to debug)
This unique file didn't exist in your original version:
- `src/lib/instructionCache.ts` — instruction caching with dedup, localStorage, action type inference

## Key Architectural Differences From Your Version
- `App.tsx` is 589 lines (yours was 674) — fewer panels, simpler state
- `App.tsx` has `data_mirror` as a CenterView option (yours didn't)
- `useGrid.ts` uses event-driven subscriptions via Map (yours used setInterval polling) — Codex's approach is BETTER here, keep it
- `setCell()` returns boolean for success/failure — yours returned void, Codex's is better
- `GridLayer.tsx` uses `<button>` elements for cells — yours used `<span>`, Codex's is better for accessibility
- `GridLayer.tsx` uses `nodeId` for React keys — yours used `index`, Codex's is more stable
- `InlineAiChat.tsx` is simpler — no declare vs AI modes, no annotation browser
- No CommandPalette, no ChatSidebar, no VersionBrowser, no EngineManager, no PluginManager, no IngestionView

## What's Missing (That Your Version Had)
These components don't exist in this codebase:
- `ChatSidebar.tsx` + `chatStore.ts`
- `CommandPalette.tsx`
- `DataIngestion/IngestionView.tsx` + `ingestion.ts`
- `GlobalEngine/EngineManager.tsx`
- `PluginManager/PluginManagerView.tsx` + `plugins.ts`
- `PromptSnippets.tsx`
- `SortableList.tsx`
- `VersionControl/VersionBrowser.tsx` + `versioning.ts`

## Extra Credit (After Debugging)
Port your declarative `parseInstruction()` system and two-mode InlineAiChat into Codex's version — but keep Codex's `instructionCache.ts` wired up. The combination would give declaration parsing + instruction caching.

## What Codex Did Better (Keep These)
- Event-driven grid subscriptions (useGrid.ts) — do NOT replace with polling
- Boolean return from setCell() — do NOT change to void
- Button semantics in GridLayer — do NOT switch back to spans
- instructionCache.ts — production-quality, keep as-is

## Next Steps (After Debug Pass)
Refer to `FORGE_BUILD_SPEC_MASTER.md` — Priority order:
1. Verify Grid Layer works end-to-end
2. Verify Inline AI Chat selection → instruct → done loop
3. Verify Data Mirror creates shadow folders
4. Add the missing components back (Version Control, Engine Manager, etc.)

## Rules
- Do NOT delete files you don't understand — investigate first
- Do NOT rewrite from scratch — fix what's here
- Keep the existing architecture decisions (Tauri v2, TipTap, React 19)
- Log every fix you make

---
*Swapped by Opus | POF 2828 | March 20, 2026*
