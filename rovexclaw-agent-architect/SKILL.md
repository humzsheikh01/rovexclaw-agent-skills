---
name: rovexclaw-agent-architect
description: Complete blueprint and engineering guide for building an autonomous, self-learning multi-agent AI workstation (RovexClaw architecture) with multi-round tool-calling, pre-flight action execution, reasoning model isolation, key rotation, vector memory, auto-skill creation, and live messaging bridges (WhatsApp/Telegram/Discord).
---

# RovexClaw Multi-Agent System Architect

Use this skill whenever building or upgrading an autonomous multi-agent AI backend/workstation in Node.js + TypeScript + Express + WebSockets + SQLite.

---

## 1. Core Architectural Pillars

### A. Real LLM Routing (Zero Canned Interceptors)
- **Rule:** Never intercept user prompts with static `if/else` keyword templates. Every message must route through the configured LLM provider with full conversation history and tool-calling.
- **Exception (Pre-Flight Action-First):** For unambiguous real-world side effects (e.g. sending a WhatsApp message to an explicit phone number like `"92314... pr Hello bhejo"`), execute the deterministic action *before* the LLM call, inject the completed result into the system prompt (`PRE-EXECUTED ACTIONS`), and guard the tool loop against duplicate execution.

### B. Multi-Round Autonomous Tool Loop
1. Parse inline tool invocations using a strict token format such as `[[TOOL:tool_name {"arg":"val"}]]`.
2. Support **up to 5 sequential tool rounds** (`for (let round = 0; round < 5; round++)`) so read/inspect/search tools feed live observations back into the LLM for self-correction and retries.
3. Broadcast live stage progress over WebSockets (`chat:activity` with `stage: 'thinking' | 'tool' | 'done'`) so the frontend can show a clean collapsed activity pill.
4. Always sanitize final assistant output by stripping `<think>...</think>` blocks, unclosed `<think>` tails, and raw `[[TOOL:...]]` tags before persisting or rendering.
5. **Synthesis Fallback:** If a reasoning model exhausts tokens on `<think>` and returns empty `content`, trigger a lightweight synthesis call using the completed tool results + reasoning tail so the user never receives an empty bubble.

### C. Provider Hub & Reasoning Model Handling
- Maintain a provider pool with multi-key rotation and automatic failover on rate limits or auth errors.
- Bump `max_tokens` to at least `4096` for reasoning models (`nemotron`, `deepseek-r1`, `o1/o3`, etc.).
- Separate `<think>...</think>` into a dedicated `reasoning` field in `LlmCompletionResult` so `content` stays clean.

### D. Self-Learning Memory & Auto-Skill Engine
1. **Instant Rule Persistence:** Detect preference/instruction phrases (`remember`, `always`, `never`, `yaad rakh`, `hamesha`, etc.) and persist them immediately to `learned-rules` in SQLite/vector memory before the LLM call.
2. **Progressive Skill Disclosure:** Load skill index (`slug` list) into the system prompt, and inject full skill markdown bodies (`skills/*.md`) on demand when the user's intent matches.
3. **Auto-Skill Creation on Approval:** When the user approves a workflow (`"haan theek hai"`, `"ab behtar hai"`, `"approved"`, `"good"`), invoke `create_skill_from_session` (with a post-LLM fallback if the model omits the tool tag) to write a new reusable `.md` skill to `skills/`.
4. **Safe Self-Upgrade:** File mutations (`patch_workspace_file`, `create_component`, `edit_skill`) must create a snapshot before writing and expose `rollback_snapshot`.

### E. Live Messaging Bridges (WhatsApp, Telegram, Discord)
- **WhatsApp (`@whiskeysockets/baileys`):**
  - On `connection === 'open'`, extract `sock.user?.id`, strip `:device` and `@s.whatsapp.net`, auto-whitelist the owner's own number, and broadcast `bridge_status` over WebSockets.
  - Normalize incoming JIDs (`@s.whatsapp.net`, `@lid`, `:deviceId`, leading `0` -> country code) before comparing against the authorized whitelist.
- **Telegram:** Validate bot token via `getMe`, strip `@` from username, and generate clean `https://t.me/<bot>?start=<token>` QR deep links.

---

## 2. Standard Directory Layout

```text
backend/src/
  db/sqlite.ts                  # Persistent SQLite tables (chat, memory, state, profile)
  memory/vectorStore.ts         # Semantic/layered memory & learned-rules store
  services/
    agentOrchestrator.ts        # Main prompt router, pre-flight, 5-round tool loop, synthesis
    providerHub.ts              # Multi-provider LLM client, key rotation, <think> extractor
    pluginLoader.ts             # Dynamic agent & skill loader from disk
    safeFsTools.ts              # Snapshot-backed file read/patch/write & skill CRUD
    whatsappBridge.ts           # Baileys socket, QR, JID normalizer, sendWhatsAppMessage
    telegramBridge.ts           # Telegram bot & QR deep link pairing
    discordBridge.ts            # Discord bot/webhook bridge
    browserEngine.ts            # Playwright Chromium automation & screenshot stream
    wsHub.ts                    # WebSocket broadcaster (chat:message, chat:activity, logs)
```

---

## 3. Verification Checklist
1. `npm run build` passes with `0` TypeScript errors.
2. No hardcoded keyword-only responses exist in `agentOrchestrator.ts`.
3. `<think>` tags never leak into rendered assistant messages.
4. Direct messaging prompts (`923... pr Hello bhejo`) execute `sendWhatsAppMessage` exactly once with clean extracted text.
5. Approval phrases persist a new `.md` skill into `skills/`.
