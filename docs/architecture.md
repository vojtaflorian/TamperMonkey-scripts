# Architecture

Single-file Tampermonkey userscript (`asana-improvements.user.js`) injected into `app.asana.com/*`.
No build step, no deps — runs as one IIFE (`"use strict"`).

## Layout (top → bottom of file)
- **Metadata header** — `// ==UserScript==` block: version, update/download URLs, `@grant` (GM_*), `@connect` Gemini.
- **CONFIG** — central config: `ui`, `features` (toggles), `ai` (model/base/storageKey/limits), `selectors`, `storage`.
- **Utilities** — `escapeHtml`, `applyCompletedVisibility`, `toggleCompleted`.
- **Features (DOM)** — `updateDueDates`, `autoExpand`, `injectToggleUI`, `injectStyles`.
- **AI core (Gemini)** — `getApiKey`, `callGemini`, `parseAiJson`, `gatherTaskContext`, `withLoading`.
- **AI buttons** — `makeTopbarBtn`/`injectAiButtons` (topbar) + `makeInlineAiBtn`/`injectInlineAiButtons` (subtask-section pills).
- **AI prompts** — `buildBreakdownPrompt`, `buildEnhancePrompt`.
- **AI flow** — `handleBreakdownClick`, `handleEnhanceClick`, `showApprovalModal`, `writeDescription`, `createSubtasksSequentially`.
- **Init** — `MutationObserver` (300ms debounce) re-runs all injectors; `init()` on load.

## AI data flow
`gatherTaskContext()` (DOM scrape) → `buildXPrompt()` → `callGemini()` (GM_xmlhttpRequest) → `parseAiJson()` → `showApprovalModal()` → user approves → `writeDescription()` (ProseMirror) + `createSubtasksSequentially()`.

## External integration
Google Gemini `generativelanguage.googleapis.com` (`gemini-2.5-flash`). API key in GM storage. Enhance uses Google Search grounding (`tools:[{google_search:{}}]`).
