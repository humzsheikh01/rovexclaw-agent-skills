# RovexClaw Agent Skills

Production-tested, reusable AI agent skills extracted from building **[RovexClaw](https://github.com/humzsheikh01/rovexclaw-trading-workstation)** — covering both **autonomous multi-agent backend architecture** and **macOS Obsidian agentic frontend design**.

---

## Included Skills

| Skill | Folder | Description |
| :--- | :--- | :--- |
| **RovexClaw Agent Architect** | [`rovexclaw-agent-architect/`](./rovexclaw-agent-architect/SKILL.md) | Complete blueprint for multi-round tool-calling, pre-flight action-first execution, reasoning model `<think>` isolation, API key rotation, self-learning memory, auto-skill creation, and WhatsApp/Telegram/Discord bridges. |
| **Obsidian Agentic Frontend** | [`obsidian-agentic-frontend/`](./obsidian-agentic-frontend/SKILL.md) | Complete macOS Obsidian dark-mode UI system with collapsed-by-default `Thinking… • N steps` activity pills, structured Markdown rendering with code copy buttons, collapsible War-Room drawers, monochrome Lucide `1.5px` icons, and mobile PWA responsiveness. |

---

## Installation

### Option 1: Antigravity (Global Skills)
Copy the skill folders into your Antigravity skills directory:

```bash
# Windows (PowerShell)
git clone https://github.com/humzsheikh01/rovexclaw-agent-skills.git
Copy-Item -Recurse .\rovexclaw-agent-skills\rovexclaw-agent-architect "$env:USERPROFILE\.gemini\config\skills\"
Copy-Item -Recurse .\rovexclaw-agent-skills\obsidian-agentic-frontend "$env:USERPROFILE\.gemini\config\skills\"
```

```bash
# macOS / Linux
git clone https://github.com/humzsheikh01/rovexclaw-agent-skills.git
cp -r rovexclaw-agent-skills/rovexclaw-agent-architect ~/.gemini/config/skills/
cp -r rovexclaw-agent-skills/obsidian-agentic-frontend ~/.gemini/config/skills/
```

### Option 2: Project-Level (RovexClaw / Claude Code / Cursor)
Copy the `SKILL.md` files into your project's `skills/` or `.agents/skills/` folder so your workspace agents load them automatically via progressive disclosure.

---

## Related Project
- **[RovexClaw Workstation](https://github.com/humzsheikh01/rovexclaw-trading-workstation)** — Full-stack multi-agent AI workstation built using these exact patterns.
