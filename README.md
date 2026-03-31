# PeekAClaw

> A standalone, open-sourceable UI module that visualizes AI agents as cute cartoon characters with live thought bubbles.

**Status:** Concept / Design phase  
**Created:** 2026-03-30  
**Author:** Khan (idea) + Hanna (spec)

---

## Concept

Watch your AI agents think. Each agent appears as an animated cartoon character. Their thought bubbles start empty — hover to see their live stream of reasoning, tool calls, and actions. Browse the full history of past agents on a timeline and click any of their brains to replay their thoughts.

Built as a standalone module — embeddable in Fry Lab or run independently. Designed to be open sourced and useful for any multi-agent AI system.

---

## Core Features

### Live Agent View
- Each active agent rendered as an animated cartoon character
- Idle animation when waiting, thinking animation when processing
- Empty thought bubble above each head by default
- **Hover** over a thought bubble → fills with live streamed reasoning, tool calls, text output
- Hanna (main agent) is visually distinct from subagents

### Historical Timeline
- Bottom scrollable row showing all past agents in chronological order
- Each agent shown as a small avatar with start/end timestamps
- **Click a past agent's brain** → opens a replay panel showing their full stream of thoughts and actions
- Filter/search past agents by task, date, model, label

### Thought Bubble Content
- Extracted from OpenClaw `.jsonl` session files
- Shows: thinking blocks, tool calls (with args), tool results, assistant text
- Live streaming for active agents (polling or WebSocket)
- Historical replay for past agents (read from file)

---

## Aesthetic Direction

**Cute, cartoony, warm** — Studio Ghibli meets Animal Crossing. Not cold/corporate.

- Characters are round, expressive, slightly chibi proportions
- Hanna: main character, slightly larger, warm orange/navy palette
- Subagents: smaller, distinct colors per agent type (research=teal, coding=green, etc.)
- Thought bubbles: classic comic-style, soft rounded rectangle, light background
- Timeline: horizontal scrollable strip, past agents faded/desaturated
- Active agents: subtle glow/pulse to indicate they're alive

**Color palette:** Orange (#ff6b1a) and navy (#0d1220) as primary, with cartoon-softened pastels for agent variety.

---

## Technical Architecture

### Rendering
- **PixiJS** or **WebGL via Three.js** for smooth canvas-based animation
- Alternatively: **React + Framer Motion** for simpler implementation (less smooth but easier to build)
- Target: 60fps, no jank, seamless transitions

### Data Layer
- **Source:** OpenClaw session `.jsonl` files at `~/.openclaw/agents/main/sessions/`
- **Live data:** Poll session file for active agent, or tap into OpenClaw gateway WebSocket if available
- **Historical:** Read completed `.jsonl` files, parse message stream
- **Session format:**
  ```json
  { "type": "message", "timestamp": "...", "message": { "role": "assistant", "content": [...] } }
  ```
  Content blocks: `{ "type": "thinking", "thinking": "..." }`, `{ "type": "text", "text": "..." }`, `{ "type": "toolCall", ... }`

### Agent Discovery
- Active agents: `openclaw subagents list` or polling `~/.openclaw/agents/main/sessions/sessions.json`
- Past agents: enumerate all `.jsonl` files in sessions directory
- Subagent relationships: parse `parentId` fields in session messages

### Module Structure
```
peekaclaw/
  src/
    components/
      AgentCharacter.jsx     — animated cartoon character
      ThoughtBubble.jsx      — hover-to-reveal thought stream
      AgentTimeline.jsx      — horizontal scrollable history
      ThoughtReplay.jsx      — full replay panel for past agents
    lib/
      sessionParser.js       — parse OpenClaw .jsonl format
      agentPoller.js         — live polling for active sessions
    assets/
      sprites/               — character sprite sheets
  public/
    index.html
  package.json
```

### Open Source Plan
- No OpenClaw-specific code in core components — data adapter pattern
- Anyone with a multi-agent system can plug in their own session format
- MIT license
- Ship as: npm package + standalone HTML build

---

## Implementation Phases

### Phase 1 — Static Prototype
- Hard-coded character positions
- Read a single session file and render thought bubble content
- No animation yet, just layout proof

### Phase 2 — Animated Characters
- Sprite-based character animation (idle, thinking states)
- Thought bubble hover interaction
- Single active agent

### Phase 3 — Multi-Agent + Timeline
- Multiple simultaneous agents
- Historical timeline from session directory
- Click-to-replay

### Phase 4 — Live Streaming
- Real-time update as active session file grows
- WebSocket integration if OpenClaw supports it
- Polish, performance optimization

### Phase 5 — Open Source Release
- Clean up OpenClaw-specific dependencies
- Write data adapter docs
- Publish to npm + GitHub

---

## Questions to Resolve

- [ ] What animation library? PixiJS (performance) vs Framer Motion (ease of development)?
- [ ] Character design — hand-drawn sprites or CSS/SVG shapes?
- [ ] Does OpenClaw expose a WebSocket for live session data, or do we poll?
- [ ] Should thought bubbles show raw session content or AI-summarized versions?
- [ ] How to handle very long thought streams (virtualization)?
- [ ] Standalone port (e.g. 7475) or embedded as Fry Lab panel?

---

## Origin

PeekAClaw was born from [Hanna](https://openclaw.ai) — a personal AI agent built on [OpenClaw](https://openclaw.ai). Watching Hanna and her subagents work in parallel made us want to *see* what they were thinking. This is that project.

## Related Projects
- [OpenClaw](https://openclaw.ai) — the personal AI agent framework PeekAClaw was designed for
- OpenClaw session format docs: `~/.npm-global/lib/node_modules/openclaw/docs/` (if you have OpenClaw installed)

---

*"Watch your AI think. It's weirder and more beautiful than you'd expect."*
