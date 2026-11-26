---
name: multi-agent-team-builder
description: Complete architecture and implementation guide for multi-agent bot creation, Chief-to-SubAgent orchestration, dynamic hiring and firing via natural language, fast decision routing, delegation, and institutional multi-agent debate pipelines.
---

# Multi-Agent Bot Creation & Chief Orchestration

Use this skill whenever building or extending a system where a **Chief Agent** dynamically hires, fires, routes to, and coordinates specialized **Sub-Agent Bots** stored on disk.

---

## 1. How Multi-Agent Bots Work (Core Lifecycle)

Every agent in the system is defined as a persistent disk record (`agents/*.json` or `.md` card) loaded dynamically via `loadAgentsFromDisk()` — never hardcoded only in memory.

### Agent Definition Schema (`AgentDefinition`)
- **`id`**: URL-safe unique slug (e.g. `chief-agent`, `technical-analyst`, `macro-sentiment`).
- **`name` & `role`**: Human-readable title and specialty.
- **`tier`**: `'chief' | 'research' | 'risk' | 'execution'` (protects `chief-agent` from accidental deletion).
- **`providerId` & `modelId`**: Optional per-agent model override (falls back to global `state.activeProviderId`).
- **`systemPrompt`**: Role-specific instructions injected when delegated to.
- **`autonomyLevel`**: `'ask_first' | 'auto'` — governs whether actions queue an approval card or run directly.
- **`allowedModules`**: Scoped capabilities (e.g. `['/skills', '/mcp']`).
- **`status`**: Live runtime map (`idle`, `thinking`, `executing`, `halted`, `error`) broadcast over WebSockets (`agent:status`).

---

## 2. Dynamic Hiring & Firing Through the Chief Agent

When the user tells the Chief Agent *"hire 2 agents"* or *"hire a Technical Analyst and Risk Manager"*, the Chief executes `[[TOOL:hire_agents {...}]]` inside the orchestrator loop:

### Hiring Flow (`hireBatchAgents(count, customNames?)`)
1. Load existing agents from disk (`loadAgentsFromDisk()`) and build `existingIds` / `existingNames` sets to prevent duplicate clones.
2. **Custom Named Agents:** If `names` are provided, slugify each unique name (incrementing `-2`, `-3` on slug collisions), assign tier/avatar/systemPrompt, and call `saveAgentToDisk(agent)`.
3. **Pool-Based Hiring:** If only `count` is provided, pick unused templates from `SPECIALIST_POOL` up to `count` and persist each to disk.
4. **Live UI Sync:** Broadcast `wsHub.broadcast('roster_updated', { agents: getLiveAgents() })` so the org chart / sidebar updates immediately without a page reload.

### Firing Flow (`executeFireAgents({ mode, count, targets })`)
- Always filter out `tier === 'chief'` and `id === 'chief-agent'` so the Chief can never be fired.
- Support 3 modes: `'all'`, `'count'` (slice last `N` sub-agents), or `'names'` (match `@id` or substring).
- Delete files with `deleteAgentFromDisk(id)`, clear `liveAgentStatus`, and broadcast `agent:deleted` + `roster_updated`.

---

## 3. Routing & Chief-to-SubAgent Coordination

### A. Fast Decision Router (`routeChannelMessage`)
For channels in `auto` routing mode:
1. **Explicit `@mention` Wins:** `@technical-analyst ...` routes directly to that bot.
2. **Fast Heuristic Match:** Keyword checks (`chart`/`smc`/`fvg` -> technical, `cpi`/`fomc`/`news` -> news, `dxy`/`macro` -> macro, `lot`/`drawdown`/`risk` -> risk) route in `<1ms`.
3. **LLM / Lead Fallback:** Falls back to the channel's lead agent (`chief-agent`) if no specialist matches.

### B. Direct Delegation (`[[TOOL:delegate_to_agent {"agentId": "...", "task": "..."}]]`)
1. Look up the target agent in `getLiveAgents()`.
2. Set its live status to `'thinking'` (`setAgentStatus(target.id, 'thinking')`).
3. Call `runChatCompletionWithRotation(target.providerId || state.activeProviderId, [{ role: 'system', content: target.systemPrompt }, { role: 'user', content: task }], target.modelId)`.
4. Reset status to `'idle'`, record a `ToolCallCard` (`mesh.delegate_<id>`), and append the sub-agent's sanitized response.

### C. Institutional Debate Pipeline (`[[TOOL:run_institutional_debate {...}]]`)
- Runs 4-analyst synthesis (Technical, Macro, Sentiment, Economic Calendar), followed by multi-round **Bull vs. Bear** adversarial debate, producing a structured **CIO Investment Plan** (`entry`, `sl`, `tp1..tp3`) gated by a **3-Tier Risk Committee** (which creates an approval card when `safeMode` is off).

---

## 4. Shared Team Memory & Progressive Skills
- **Live Roster Injection:** Always inject the live DB/disk roster (`liveRoster.map(...)`) into the Chief's system prompt so the Chief never hallucinates that agents are missing or present when they aren't.
- **Shared Memory & Skills:** All hired sub-agents share the same SQLite `vectorStore` (`learned-rules`, `agent-journal`) and `skills/*.md` index, so teaching the Chief a rule immediately benefits every newly hired specialist.
