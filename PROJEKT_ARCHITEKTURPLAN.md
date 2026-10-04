# Projekt-Architekturplan: Multiagent- & Subagent-System (OpenClaw Dashboard)

> **Generiert am**: 2026-10-04  
> **Projekt-Typ**: Zero-Dependency Go Observability Dashboard für OpenClaw Multiagent Runtime  
> **Ziel**: Bauplan für Verständnis und Weiterentwicklung des Multiagent-/Subagent-Stacks  
> **Sprache**: Deutsch

---

## 1. Executive Summary & Vision

- **Projektziel**: Das Multiagent-System von OpenClaw (Haupt-Agents, Bindings, Cron, Subagent-Spawns) im Dashboard vollständig, korrekt und bedienbar sichtbar machen.
- **Kernanforderungen**: Live-Hierarchie, Run-Historie, Limits, Runtime-Queue, Channel-Zuordnung; zero-dep; loopback.
- **Zielgruppe**: OpenClaw-Operatoren mit multi-agent, multi-channel Setups.

---

## 2. Code- & Tech-Stack

| Schicht | Entscheidung |
| :--- | :--- |
| Sprache | Go (Toolchain laut `go.mod`, CI 1.26.x) |
| Deps | **Keine** third-party Runtime-Deps |
| Frontend | Vanilla HTML/CSS/JS, `//go:embed` |
| Tests | `testing`, table-driven, `-race`, optional Playwright |
| Datenquellen | Dateien unter `~/.openclaw`, Gateway HTTP (`/healthz`,`/readyz`), CLI JSON |

Architekturmuster: **Collector-Pipeline + Facade** (`internal/app*` / root facade).

---

## 3. Design-System (UI Multiagent)

- Dark-Mode-first (Themes: Midnight, Nord, Catppuccin, …)
- Badges: live / recent / idle; status ok/err
- Tree-Nodes mit `--node-color` pro Agent
- Glass panels für Agent Cards, Limits, Bindings
- Tabs Today / 7d / 30d / all für Subagent-Runs

---

## 4. Dateisystem

```text
mammouth-dashboard/
├── internal/apprefresh/
│   ├── subagent_tasks.go          # CLI → subagentRuns
│   ├── refresh_config.go          # agents.list, subagentConfig, bindings
│   ├── refresh_sessions.go        # type, spawnedBy, hierarchy inputs
│   └── refresh.go                 # Orchestration collectDashboardData
├── web/index.html                 # Tree, Activity, Config Panels
├── docs/MULTIAGENT_SYSTEM.md      # Kanonische Spec
├── PROJEKT_ARCHITEKTURPLAN.md     # Dieser Bauplan
├── MASTER_TODO_LIST.html          # Phasen-Dashboard
└── PROJEKT_VORSCHAU.html          # Visuelle Vorschau
```

---

## 5. Frontend-Architektur (Multiagent)

| Surface | Daten | Dirty-Flag |
| :--- | :--- | :--- |
| Agent Tree | `sessions[]` + `spawnedBy` | `sessions` |
| Sub-Agent Activity | `subagentRuns*` | `subRuns` |
| Sub Tokens | `subagentUsage*` | `subTokens` |
| Agent Config / Limits | `agentConfig` | `agentConfig` |
| Runtime Health | `/api/system` tasks | SystemBar |

---

## 6. Backend- & Datenmodell

### agentConfig.subagentConfig

```json
{
  "maxConcurrent": 4,
  "maxSpawnDepth": 3,
  "maxChildrenPerAgent": 8
}
```

### subagentRuns[] Eintrag

```json
{
  "task": "Research X…",
  "agent": "main",
  "status": "succeeded",
  "durationSec": 42,
  "timestamp": "2026-10-04 08:15",
  "date": "2026-10-04",
  "error": ""
}
```

### Session-Hierarchie

```json
{
  "key": "agent:main:subagent:…",
  "type": "subagent",
  "spawnedBy": "agent:main:telegram:…",
  "active": true
}
```

---

## 7. Umsetzungs-Roadmap

### Phase PLAN (aktuell)

- [x] Bestandsaufnahme Collector/UI/CLI
- [x] Spec `docs/MULTIAGENT_SYSTEM.md`
- [x] Architekturplan + Todo-Dashboard

### Phase BUILD (P0/P1)

- [ ] Empty-vs-offline UI
- [ ] Status-Aggregates
- [ ] Activity Filter
- [ ] Deep Tree

### Phase TEST → LEARN → LOOP → DEPLOY

- [ ] Tests/Fixtures erweitern
- [ ] Review (Security read-only boundary)
- [ ] `make check` grün
- [ ] Changelog + README Screenshots aktualisieren
