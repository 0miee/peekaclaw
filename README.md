# PeekAClaw

> A standalone, open-sourceable UI module that visualizes AI agents as cute cartoon characters with live thought bubbles.

**Status:** Concept / Design phase  
**Created:** 2026-03-30  
**Author:** Khan (idea) + Hanna (spec)

---

## Concept

Watch your AI agents think. Each agent appears as an animated cartoon character. Their thought bubbles start empty — hover to see their live stream of reasoning, tool calls, and actions. Browse the full history of past agents on a timeline and click any of their brains to replay their thoughts.

Built as a standalone module — embeddable in any dashboard or run independently. Designed to work with any agentic system via a simple adapter interface.

---

## Supported Agent Frameworks

PeekAClaw uses a framework-agnostic data model. Adapters translate each framework's native format into a common session structure.

### Planned Adapters (in build order)

| Framework | Format | Status |
|-----------|--------|--------|
| **Claude Code** | JSONL (`~/.claude/projects/`) | Phase 1 — start here |
| **Hermes** | SQLite (`~/.hermes/state.db`) | Phase 2 |
| **OpenClaw** | JSONL (`~/.openclaw/agents/main/sessions/`) | Phase 3 |
| *Any framework* | Write your own adapter | Open spec |

### Why Claude Code first?
Simplest format to parse — flat JSONL, clear message types, well-structured. Good foundation before tackling SQLite and OpenClaw's more complex session graph.

---

## Common Data Model

Every adapter outputs this structure:

```ts
interface AgentSession {
  id: string
  framework: "claude-code" | "hermes" | "openclaw" | string
  startedAt: Date
  endedAt?: Date
  parentSessionId?: string   // for subagents
  model?: string
  label?: string             // task description / title
  messages: AgentMessage[]
}

interface AgentMessage {
  id: string
  role: "user" | "assistant" | "tool"
  timestamp: Date
  content: MessageContent[]
  parentId?: string
}

type MessageContent =
  | { type: "text"; text: string }
  | { type: "thinking"; thinking: string }
  | { type: "tool_call"; name: string; args: object }
  | { type: "tool_result"; toolName: string; result: string }
```

---

## Adapter Specs

### Claude Code Adapter
- **Source:** `~/.claude/projects/<project>/` — one JSONL file per session
- **Subagents:** `<session_dir>/subagents/agent-*.jsonl`
- **Key fields:** `type`, `message.role`, `message.content`, `timestamp`, `parentUuid`, `sessionId`
- **Message types to handle:** `user`, `assistant`, `queue-operation` (skip), `result`
- **Thinking blocks:** `content[].type === "thinking"`
- **Tool calls:** `content[].type === "tool_use"` with `name` + `input`

### Hermes Adapter
- **Source:** `~/.hermes/state.db` (SQLite)
- **Tables:** `sessions`, `messages`
- **Key fields:** `messages.role`, `messages.content`, `messages.timestamp`, `messages.tool_calls`, `sessions.id`
- **Live data:** Poll DB for new rows, or watch WAL file

### OpenClaw Adapter
- **Source:** `~/.openclaw/agents/main/sessions/*.jsonl`
- **Key fields:** `type: "message"`, `message.role`, `message.content`, `timestamp`, `id`, `parentId`
- **Content blocks:** `thinking`, `text`, `tool_use`, `tool_result`
- **Active session:** `sessions.json` in same directory

---

## Core Features

### Live Agent View
- Each active agent rendered as an animated cartoon character
- Idle animation when waiting, thinking animation when processing
- Empty thought bubble above each head by default
- **Hover** over a thought bubble fills with live streamed reasoning, tool calls, text output
- Main agent is visually distinct from subagents

### Historical Timeline
- Bottom scrollable row showing all past agents in chronological order
- Each agent shown as a small avatar with start/end timestamps
- **Click a past agent's brain** opens a replay panel showing their full stream of thoughts and actions
- Filter/search past agents by task, date, model, label

### Thought Bubble Content
- Shows: thinking blocks, tool calls (with args), tool results, assistant text
- Live streaming for active agents (polling or WebSocket)
- Historical replay for past agents (read from file/DB)

---

## Aesthetic Direction

**Cute, cartoony, warm** — Studio Ghibli meets Animal Crossing. Not cold/corporate.

- Characters are round, expressive, slightly chibi proportions
- Main agent: slightly larger, warm orange/navy palette
- Subagents: smaller, distinct colors per agent type (research=teal, coding=green, etc.)
- Thought bubbles: classic comic-style, soft rounded rectangle, light background
- Timeline: horizontal scrollable strip, past agents faded/desaturated
- Active agents: subtle glow/pulse to indicate they're alive

**Color palette:** Orange (#ff6b1a) and navy (#0d1220) as primary, with cartoon-softened pastels for agent variety.

---

## Technical Architecture

### Rendering
- **PixiJS** or **WebGL via Three.js** for smooth canvas-based animation
- Alternatively: **React + Framer Motion** for simpler implementation
- Target: 60fps, no jank, seamless transitions

### Module Structure
```
peekaclaw/
  src/
    adapters/
      claude-code.js      -- Claude Code JSONL adapter
      hermes.js           -- Hermes SQLite adapter
      openclaw.js         -- OpenClaw JSONL adapter
      base.js             -- Adapter interface / common model
    components/
      AgentCharacter.jsx  -- animated cartoon character
      ThoughtBubble.jsx   -- hover-to-reveal thought stream
      AgentTimeline.jsx   -- horizontal scrollable history
      ThoughtReplay.jsx   -- full replay panel for past agents
    lib/
      agentPoller.js      -- live polling for active sessions
    assets/
      sprites/            -- character sprite sheets
  public/
    index.html
  package.json
```

---

## Implementation Phases

### Phase 1 — Claude Code Static Prototype
- Claude Code adapter: parse JSONL, extract messages + thinking blocks
- Hard-coded character positions
- Render thought bubble content from a single session file
- No animation yet, just layout proof
- **Goal:** See a real Claude Code session rendered as thought bubbles

### Phase 2 — Hermes Adapter + Animation
- Hermes SQLite adapter
- Sprite-based character animation (idle, thinking states)
- Thought bubble hover interaction
- Single active agent

### Phase 3 — OpenClaw + Multi-Agent + Timeline
- OpenClaw adapter
- Multiple simultaneous agents
- Historical timeline from session directory
- Click-to-replay

### Phase 4 — Live Streaming
- Real-time update as active session file/DB grows
- WebSocket integration where supported
- Polish, performance optimization

### Phase 5 — Open Source Release
- Clean adapter interface documentation
- Example adapter for custom frameworks
- Publish to npm + GitHub

---

## Questions to Resolve

- [ ] Animation library: PixiJS (performance) vs Framer Motion (ease)?
- [ ] Character design: hand-drawn sprites or CSS/SVG shapes?
- [ ] Should thought bubbles show raw content or AI-summarized versions?
- [ ] How to handle very long thought streams (virtualization)?
- [ ] Standalone port (e.g. 7475) or embedded as Fry Lab panel?

---

## Origin

PeekAClaw was born from watching Hanna — a personal AI agent — and her subagents work in parallel. We wanted to *see* what they were thinking. This is that project.

## Related Projects
- [OpenClaw](https://openclaw.ai) — personal AI agent framework
- [Hermes](https://github.com/NousResearch/hermes-agent) — open source agent framework by Nous Research
- [Claude Code](https://anthropic.com) — Anthropic's CLI coding agent

---

*"Watch your AI think. It's weirder and more beautiful than you'd expect."*
