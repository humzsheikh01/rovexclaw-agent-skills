---
name: obsidian-agentic-frontend
description: Complete UI/UX design system and implementation skill for macOS Obsidian dark-mode agentic chat interfaces, featuring collapsed-by-default Thinking/Tool activity pills, clean Markdown rendering with code copy buttons, collapsible War-Room drawers, monochrome Lucide 1.5px iconography, and mobile-first PWA responsiveness.
---

# macOS Obsidian Agentic Frontend Design

Use this skill when designing or refactoring AI chat interfaces, agent workstations, and mobile-responsive PWA frontends in React + TypeScript + Tailwind CSS.

---

## 1. Visual Design System (macOS Obsidian)

### Color Palette & Surfaces
- **Primary Canvas:** Deep obsidian `#09090b` (`bg-zinc-950`)
- **Elevated Cards / Bubbles:** `#111114` / `bg-zinc-900/80` with `1px solid` subtle borders (`border-zinc-800/80`)
- **User Message Bubble:** Clean high-contrast pill (`bg-zinc-100 text-zinc-950` or `bg-zinc-800 text-zinc-100`)
- **Text Hierarchy:** Primary `text-zinc-100`, Secondary `text-zinc-400`, Muted `text-zinc-500`
- **No Neon / No Clutter:** Avoid bright gradients, glowing borders, or colorful emojis inside UI chrome and AI bubbles.

### Iconography Rules
- Use **Lucide icons** exclusively with **`strokeWidth={1.5}`** in monochrome `text-zinc-400` / `text-zinc-200`.
- Never place raw Unicode emojis in status pills, headers, or tool-call badges.

---

## 2. Chat UX & Thinking/Tool Collapse Pattern

A clean agentic chat must **never** dump raw chain-of-thought or noisy backend tool logs directly into the message body.

### A. Live Streaming Activity Pill (While Generating)
- Listen to WebSocket `chat:activity` events (`stage: 'thinking' | 'tool' | 'done'`).
- Render a compact, **collapsed-by-default** `<details>` pill above the composer/stream:
  - Summary shows a subtle spinner + current step label (e.g. `Thinking…` or `Running send_whatsapp_message…`) + step count (`• 2 steps`).
  - Expanding reveals the chronological `LiveThinkingStep[]` list.
  - Clear `liveSteps` automatically when `stage === 'done'` or `chat:message` arrives.

### B. Persisted Message Bubble (`MessageBubble.tsx`)
- **Tool & Thinking Accordion:** Render `Worked for X.Xs • N steps` inside a **closed-by-default** `<details>` element (never add the `open` attribute by default).
- **Clean Content First:** Strip any leftover `<think>` tags or raw tool JSON on the frontend (`stripRawToolJson`) as a defense-in-depth safeguard.
- **War-Room / Multi-Agent Debate Drawer:** Keep multi-agent debate transcripts inside a collapsed `<details>` drawer so the main answer stays scannable.

---

## 3. Markdown Rendering (`MarkdownContent.tsx`)

Never render assistant replies in a plain `whitespace-pre-wrap` `<div>`. Always use a structured Markdown renderer supporting:
- Headings (`#`, `##`, `###`) with tight vertical rhythm
- Bold (`**text**`), Italic (`*text*`), Inline Code (`` `code` ``)
- Bullet & numbered lists with clean indentation
- Fenced code blocks (` ```lang ... ``` `) with a language badge and one-click **Copy** button
- Clickable external/file links (`target="_blank" rel="noopener noreferrer"`)

---

## 4. Mobile PWA & Responsive Layout

- **Viewport & Safe Areas:** Support iOS/Android PWA standalone mode (`viewport-fit=cover`, `env(safe-area-inset-bottom)`).
- **Compact Header:** Single-row top bar with active Agent selector, Model selector pill, and sidebar toggle drawer button.
- **Zero Horizontal Overflow:** Constrain code blocks and tables with `overflow-x-auto max-w-full` and `break-words` inside message bubbles so mobile screens never scroll horizontally.
- **Sticky Bottom Composer:** Input bar with attachment/action `+` button, auto-growing textarea, and send/voice button anchored cleanly above the mobile keyboard.

---

## 5. Verification Checklist
1. `npm run build` (`tsc -b && vite build`) passes with `0` errors.
2. Live `Thinking…` pill and `Worked for X.Xs • N steps` accordion are both collapsed by default.
3. Markdown headings, bold text, lists, and code blocks with copy buttons render cleanly on desktop and mobile (`390px` viewport).
4. Zero raw `<think>` tags or raw `[[TOOL:...]]` strings appear in the UI.
