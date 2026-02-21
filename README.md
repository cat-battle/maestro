# 🎼 Maestro

> Multi-model AI orchestration system — Opus conducts, sub-agents perform.

## What is Maestro?

Maestro is a hierarchical AI orchestration system where a central "Maestro" agent coordinates multiple specialized "sub-agents" to complete complex projects.

```
         User Request: "Build a stock ticker app"
                            ↓
                    ┌───────────────┐
                    │   Maestro     │  ← Plans, delegates, integrates
                    │  (Opus 4.6)   │
                    └───────┬───────┘
                            │
          ┌─────────────────┼─────────────────┐
          ↓                 ↓                 ↓
    ┌──────────┐      ┌──────────┐      ┌──────────┐
    │ Sonnet   │      │ Haiku    │      │ Gemini   │
    │ Backend  │      │ UI/CSS   │      │ API int. │
    └──────────┘      └──────────┘      └──────────┘
          │                 │                 │
          └─────────────────┼─────────────────┘
                            ↓
                    ┌───────────────┐
                    │  Integrator   │  ← Combines everything
                    └───────────────┘
                            ↓
                      Working App ✅
```

## Why Maestro?

| Problem | Maestro's Solution |
|---------|-------------------|
| Single AI models get overwhelmed on large tasks | Break down into sub-tasks for specialized agents |
| Expensive models for simple tasks | Right model for the right job (cost optimization) |
| No visibility into AI decision-making | Human-in-the-loop approval gates |
| AI outputs don't integrate well | Dedicated integrator agent |
| No learning from past projects | Shared learning across sessions |

## Key Features

- **🎭 Multi-model orchestration** — Opus, Sonnet, Haiku, Gemini working together
- **🧠 Learns over time** — Remembers what works, shares across projects
- **👁️ Human-in-the-loop** — Configurable approval gates (Strict → YOLO)
- **🔗 Smart integration** — Dedicated agent for combining outputs
- **📁 Git-native** — Works with your repos as shared workspace
- **🪆 Fractal design** — Maestros can delegate to other Maestros

## Status

🚧 **Architecture Phase** — Design decisions documented, ready for implementation.

See [ARCHITECTURE.md](ARCHITECTURE.md) for detailed design documentation.

## Roadmap

### Design (Complete)
- [x] Core architecture (5 foundational decisions)
- [x] CLI/TUI interface design
- [x] Agent configuration design

### Implementation (Upcoming)
- [ ] MVP scope definition
- [ ] Implementation stack decision
- [ ] Core Maestro orchestration
- [ ] Sub-agent adapters (Claude, Gemini, etc.)
- [ ] CLI with TUI dashboard
- [ ] State management (`~/.maestro/`)
- [ ] Learning system
- [ ] Integration agent

## License

MIT
