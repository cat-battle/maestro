# Maestro Architecture

> Multi-model AI orchestration system — Opus conducts, sub-agents perform.

## Overview

Maestro is a hierarchical AI orchestration system where a central "Maestro" agent (running on a powerful model like Opus) coordinates multiple specialized "sub-agents" (running on models optimized for specific tasks) to complete complex projects.

```
                    ┌─────────┐   ┌─────────────┐   ┌─────────┐
                    │   CLI   │   │ Web Portal  │   │  Chat   │
                    └────┬────┘   └──────┬──────┘   └────┬────┘
                         │               │               │
                         └───────────────┼───────────────┘
                                         ▼
                              ┌─────────────────────┐
                              │      Maestro        │
                              │     (Opus 4.6)      │
                              └──────────┬──────────┘
                                         │
                    ┌────────────────────┼────────────────────┐
                    ▼                    ▼                    ▼
             ┌────────────┐       ┌────────────┐       ┌────────────┐
             │ Sub-agent  │       │ Sub-agent  │       │ Sub-agent  │
             │ Sonnet 4.5 │       │ Haiku 4.5  │       │ Gemini 3.0 │
             └────────────┘       └────────────┘       └────────────┘
```

## Core Principles

- **Maestro orchestrates, never implements** — Pure coordination role
- **Right model for the right job** — Cost/speed/capability optimization
- **Human-in-the-loop** — Configurable approval gates
- **Fractal design** — Sub-agents can be Maestros themselves (nested orchestration)

---

## Architectural Decisions

### 1. Decision Engine

**Approach:** Hybrid rules + learning

- **Bootstrap phase:** Rules-based heuristics for initial task decomposition
- **Learning phase:** Accumulate patterns from completed tasks
- **Shared learning:** Learnings can be shared between Maestro instances

**Storage:** JSON/YAML files

```yaml
# Example learning entry
task_pattern: "user authentication"
recommended_agent: "sonnet-4.5"
avg_duration_minutes: 12
success_rate: 0.94
dependencies: ["database", "session-management"]
common_failures:
  - "forgot password reset flow"
  - "email verification integration"
notes: |
  Authentication tasks work best when broken into:
  1. Core auth logic (Sonnet)
  2. UI components (Haiku)
  3. Email integration (Sonnet)
```

---

### 2. Communication Protocol

**Approach:** Hub-and-spoke (Maestro-as-relay)

- All communication flows through Maestro
- Sub-agents never communicate directly (v1)
- Git repositories serve as shared workspaces
- Supports single-repo and multi-repo projects

**Flow:**
```
Maestro → Sub-agent: "Work on /src/auth/ and commit when done"
Sub-agent → Maestro: Commits + completion signal
```

**Multi-repo support:**
```
Maestro
├── Sub-agent A → works in repo: frontend/
├── Sub-agent B → works in repo: backend-api/
├── Sub-agent C → works in repo: shared-lib/
└── Integrator  → works across repos
```

**Future consideration:** Direct agent-to-agent channels for tightly coupled tasks

---

### 3. State Management

**Approach:** File-based, global directory

**Location:** `~/.maestro/`

```
~/.maestro/
├── config.yaml           # Global configuration
├── learnings/            # Shared learnings (cross-project)
│   ├── patterns.yaml
│   └── agent-performance.yaml
├── active/               # Current session (one at a time)
│   ├── state.json        # Task state
│   ├── tasks/            # Individual task details
│   └── artifacts/        # Produced outputs
└── archive/              # Completed sessions
    ├── 2026-02-20-stock-ticker/
    └── 2026-02-19-auth-system/
```

**State file example:**
```json
{
  "projectId": "stock-ticker-app",
  "status": "running",
  "startedAt": "2026-02-20T19:40:00Z",
  "workingDir": "/path/to/repo",
  "tasks": {
    "task-001": { "status": "done", "agent": "sonnet-4.5" },
    "task-002": { "status": "running", "agent": "haiku-4.5" },
    "task-003": { "status": "queued", "dependsOn": ["task-001", "task-002"] }
  }
}
```

**Design choices:**
- One active session at a time (simplicity for v1)
- Archive completed sessions for history/learning
- Human-readable files for debugging
- Git-trackable state

**Future consideration:** Multi-session support with UUIDs

---

### 4. Integration Layer

**Approach:** Explicit integrator sub-agent

After sub-agents complete their individual tasks, Maestro spawns a dedicated "integrator" agent:

**Integrator responsibilities:**
- 🔗 **Connect** — imports, dependencies, wiring
- 🩹 **Resolve** — naming conflicts, API mismatches
- 📦 **Package** — entry points, configs, requirements.txt
- ✅ **Verify** — basic sanity checks

**Flow:**
```
Sub-agents complete work
         ↓
Maestro spawns Integrator
         ↓
Integrator receives:
  - Summary of all artifacts
  - Architecture context
  - Integration requirements
         ↓
Integrator produces:
  - Glue code
  - Entry points
  - Unified configuration
  - Integration report
         ↓
Maestro reviews integration status
```

**Think of it as:** The "Tech Lead" agent who reviews everyone's PRs and makes sure they work together.

---

### 5. Interface Design

**Approach:** Human-in-the-loop with configurable approval gates

**Five approval gates:**

```
User Request
    ↓
┌─────────────────────────┐
│ 🚦 GATE 1: Plan Review  │  "Here's how I'll break this down. Approve?"
└─────────────────────────┘
    ↓
┌─────────────────────────┐
│ 🚦 GATE 2: Assignment   │  "Assigning auth→Sonnet, UI→Haiku. OK?"
└─────────────────────────┘
    ↓
  [Sub-agents work...]
    ↓
┌─────────────────────────┐
│ 🚦 GATE 3: Milestone    │  "Backend complete. Review before frontend?"
└─────────────────────────┘
    ↓
  [More work...]
    ↓
┌─────────────────────────┐
│ 🚦 GATE 4: Integration  │  "Ready to integrate. Proceed?"
└─────────────────────────┘
    ↓
┌─────────────────────────┐
│ 🚦 GATE 5: Completion   │  "Done! Review final output?"
└─────────────────────────┘
```

**Policy levels:**

| Policy | Active Gates | Use Case |
|--------|--------------|----------|
| **Strict** | All 5 gates | Critical projects, learning the system |
| **Balanced** | Plan + Integration + Completion | Day-to-day work |
| **Autonomous** | Completion only | Trusted, routine tasks |
| **YOLO** | None (notify only) | Batch jobs, low-risk |

**At each gate, user can:**
- ✅ **Approve** — continue to next phase
- ✏️ **Modify** — adjust plan/assignments
- ❌ **Reject** — stop or redo
- 💬 **Comment** — add guidance for next phase

---

## Future Considerations

- **Nested Maestros** — Sub-agent can itself be a Maestro for complex subsystems
- **Multi-session** — Multiple parallel orchestrations with UUID tracking
- **Direct agent channels** — P2P communication for tightly coupled tasks
- **Vector embeddings** — Semantic search over learnings at scale
- **Integration tests** — Automated validation gates

---

## Interfaces

Maestro supports multiple interaction surfaces:

- **CLI** — Command-line task submission and monitoring
- **Web Portal** — Kanban-style task visualization
- **Chat** — Conversational interface (Telegram, etc.)

All interfaces communicate with the same Maestro core.

---

## License

MIT
