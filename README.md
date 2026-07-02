# wad-speckit — Spec Kit demo for WeAreDevelopers 2026

An empty showcase repository for the talk **“Best Practices for AI-Assisted Development of Distributed Systems”** (Przemysław Ładyński, WeAreDevelopers 2026).

This repository **does not contain a production application** — only [GitHub Spec Kit](https://github.com/github/spec-kit) infrastructure: templates, prompts, and agents for **spec-driven development (SDD)**. On stage we demonstrate how an engineer coordinates AI agents through **specifications in the repository**, instead of one-off prompts like “build me a CRM like Dynamics”.

## Presentation context

In distributed systems, the bottleneck is no longer raw model code generation, but:

- architectural decisions,
- specifications and tests,
- repository structure,
- technology stack choices.

The agent works in the repo — you **steer** it through:

| Artifact | Role | Where in this repo |
|----------|------|-------------------|
| **Rules** | What is allowed / forbidden (MUST / MUST NOT) | `.cursor/rules/`, team policies |
| **Skills** | How to use frameworks and tools correctly | agent skills (e.g. Cursor Skills) |
| **Specs** | What the system should do and **how** it should be built | `.specify/`, `specs/` (after running the workflow) |

This project demonstrates the **Specs** layer — from project constitution, through requirements and technical plan, down to atomic implementation tasks.

---

## Requirements

- **Python 3.10+**
- **Git**
- **uv** (Python package manager by Astral) — installation below
- An IDE agent with Spec Kit support — this repo is configured for **GitHub Copilot** (`.github/agents/`, `.github/prompts/`)

---

## Installing Spec Kit yourself

### 1. Install `uv`

**Windows (PowerShell — recommended):**

```powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

**Alternative (pip):**

```powershell
pip install uv
```

After installing via `pip` on Windows, scripts usually land in:

```text
%APPDATA%\Python\Python314\Scripts\
```

If `uvx` is not recognized, add that folder to your user PATH and restart the terminal:

```powershell
[Environment]::SetEnvironmentVariable(
  "Path",
  $env:Path + ";$env:APPDATA\Python\Python314\Scripts",
  "User"
)
```

Verify:

```powershell
uv --version
uvx --version
```

### 2. Initialize Spec Kit in a new project

In your target repository directory:

```powershell
uvx --from git+https://github.com/github/spec-kit.git specify init .
```

Optionally choose an agent integration (instead of the default):

```powershell
uvx --from git+https://github.com/github/spec-kit.git specify init . --integration copilot
# others: claude, cursor, gemini, opencode, windsurf, ...
```

After `init`, you will get among other things:

```text
.specify/
  memory/constitution.md    # project constitution (top-level principles)
  templates/                # spec / plan / tasks templates
  scripts/                  # helper scripts (PowerShell)
.github/
  agents/                   # speckit.* agent definitions
  prompts/                  # slash-command prompts
```

---

## Spec Kit workflow — from architecture to tasks

Full SDD cycle (see `.specify/workflows/speckit/workflow.yml`):

```text
/speckit.constitution  →  /speckit.specify  →  /speckit.plan  →  /speckit.tasks  →  /speckit.implement
        ↑                        ↑                  ↑                 ↑
   project principles        WHAT we build      HOW we build      atomic tasks
   architecture,             requirements,      stack, structure   with file paths
   MUST/MUST NOT             user stories       API contracts      and checkpoints
```

Optional before planning: `/speckit.clarify`. After tasks: `/speckit.analyze`, `/speckit.checklist`.

Run commands in the agent chat (Copilot in VS Code / Cursor with the right integration) as **slash commands**, e.g. `/speckit.constitution`.

---

## Example 1: Constitution — architecture, technologies, principles

**Goal:** Before the agent writes a single line of code, establish **non-negotiable** rules — the “stack = rules file” idea from the talk.

Run in the agent:

```text
/speckit.constitution

Project: weather-demo (weather app from the presentation).
Principles:
- Architecture: one frontend + two independent backends (auth/history + weather API).
- Frontend: React + TypeScript, Vite, REST communication.
- Auth backend: .NET 8 minimal APIs, in-memory store for demo (no real database).
- Weather backend: Node.js + TypeScript, integration with an external Weather API.
- Observability: OpenTelemetry in all services (traces + structured logging).
- Tests: every service has contract and integration tests before merge.
- MUST: local changes in one service must not require edits in others without updating the contract in specs/contracts/.
- MUST NOT: shared monorepo package with domain logic hidden behind abstractions — prefer explicit HTTP contracts.
- Priority: agent-operability and change locality (small PRs, clear directory structure).
```

**Outcome:** `.specify/memory/constitution.md` with concrete principles instead of placeholders. The agent references it during planning and implementation.

---

## Example 2: Feature specification — WHAT the system should do

**Goal:** Describe business requirements **without** jumping straight to code — user stories, Given/When/Then scenarios, success criteria.

```text
/speckit.specify

Weather app for end users:
- User can register and log in (email + password).
- After login, user sees current weather for a selected city (default: Warsaw).
- User can search for another city and save it in a history of the last 10 searches.
- History is per user and visible after re-login.
- On external weather API failure, the UI shows a message and allows retry.
- P95 end-to-end response (click → data on screen) < 2 s under normal weather API conditions.

Out of scope for v1: multi-day forecast, push notifications, SSO.
```

**Outcome:** directory `specs/<number>-weather-app/` with `spec.md` (template: `.specify/templates/spec-template.md`):

- user stories with priorities P1, P2, P3,
- acceptance scenarios,
- functional requirements (FR-xxx),
- measurable success criteria (SC-xxx).

---

## Example 3: Technical plan — HOW to build it

**Goal:** Turn the specification into **architecture, stack, repo structure, and contracts**.

```text
/speckit.plan

Implement the weather-app spec according to the constitution.
Frontend in apps/web, auth backend in services/auth-api, weather in services/weather-api.
OpenAPI contracts in specs/<feature>/contracts/.
Docker Compose to run all three services locally.
```

**Outcome:** in `specs/<feature>/` you get among others:

| File | Contents |
|------|----------|
| `plan.md` | summary, technical context, directory structure |
| `research.md` | technical decisions and rejected alternatives |
| `data-model.md` | entities (User, SearchHistory, WeatherSnapshot) |
| `contracts/` | OpenAPI / inter-service schemas |
| `quickstart.md` | how to run locally |

Sample **Technical Context** section in `plan.md`:

```markdown
**Language/Version**: TypeScript 5.x (frontend + weather-api), C# / .NET 8 (auth-api)
**Primary Dependencies**: React, Vite, ASP.NET Core, Express, OpenTelemetry SDK
**Storage**: in-memory (auth/history — demo), no weather persistence
**Testing**: Vitest (web), xUnit (auth), Vitest + supertest (weather)
**Target Platform**: Docker Compose (dev), Linux containers (prod)
**Project Type**: distributed web app (1 FE + 2 BE)
```

---

## Example 4: Implementation tasks — atomic steps for the agent

**Goal:** Break the plan into **small, independent tasks** with file paths — ready for `/speckit.implement` or pair-programming.

```text
/speckit.tasks
```

The agent generates `specs/<feature>/tasks.md` from `.specify/templates/tasks-template.md`. Sample **expected shape** (abbreviated):

```markdown
## Phase 1: Setup (Shared Infrastructure)

- [ ] T001 Create structure apps/web, services/auth-api, services/weather-api
- [ ] T002 [P] Configure docker-compose.yml with three services and internal network
- [ ] T003 [P] Add OpenTelemetry bootstrap in each service

## Phase 2: Foundational (Blocking Prerequisites)

- [ ] T004 Define auth OpenAPI contract in specs/001-weather-app/contracts/auth-api.yaml
- [ ] T005 Define weather BFF contract in specs/001-weather-app/contracts/weather-api.yaml
- [ ] T006 [P] Generate API client in apps/web from contracts/weather-api.yaml

## Phase 3: User Story 1 — Registration and login (P1) 🎯 MVP

- [ ] T007 [US1] POST /auth/register endpoint in services/auth-api/Program.cs
- [ ] T008 [US1] POST /auth/login + JWT in services/auth-api/
- [ ] T009 [P] [US1] Login form in apps/web/src/pages/Login.tsx
- [ ] T010 [US1] Integration test: register → login → token in services/auth-api/tests/

**Checkpoint**: User can log in and receive a session — weather not required yet.

## Phase 4: User Story 2 — Current weather (P1)

- [ ] T011 [US2] Weather API client in services/weather-api/src/weatherClient.ts
- [ ] T012 [US2] GET /weather/current?city= with JWT authorization
- [ ] T013 [P] [US2] Dashboard view in apps/web with API error handling
```

ID conventions:

- `[P]` — task can run in parallel,
- `[US1]`, `[US2]` — link to user story in `spec.md`,
- every task points to **specific files**.

Implementation:

```text
/speckit.implement

Start with Phase 1 and Phase 2. Stop at the US1 checkpoint.
```

---

## Example 5: One sentence (full workflow)

For a quick conference demo, use the built-in workflow:

```text
# in an agent that supports Spec Kit workflows
Run the full SDD cycle for: "Weather app — login, city history, current weather"
```

The workflow (`specify → review → plan → tasks → implement`) is defined in `.specify/workflows/speckit/workflow.yml`.

---

## Repository structure

```text
wad-speckit/
├── .specify/           # Spec Kit engine (templates, constitution, scripts)
├── .github/
│   ├── agents/         # speckit.constitution, speckit.specify, ...
│   └── prompts/        # slash-command prompts
├── .vscode/
└── README.md           # this file
```

After running the workflow, a `specs/` directory with feature documentation will appear — **it is not here yet** because this is intentionally an empty demo project.

---

## Mapping to slide topics

| Presentation topic | How to demo in this repo |
|--------------------|--------------------------|
| “Point the right way” instead of slop | `/speckit.constitution` + `/speckit.plan` |
| Rules / Skills / Specs | Rules and Skills live outside the repo; **Specs** — this project |
| Distributed architecture (FE + 2 BE) | weather-demo examples in the sections above |
| AI-modifiability, change locality | small phases in `tasks.md`, contracts in `contracts/` |
| Assigning tasks to agents | `tasks.md` → `/speckit.implement` |
| Evaluation (evals / graders) | separate stage — rules and specs can be tested on known tasks |

---

## Useful links

- [GitHub Spec Kit](https://github.com/github/spec-kit)
- [Spec-Driven Development](https://github.com/github/spec-kit/blob/main/spec-driven.md)
- Talk: *Best Practices for AI-Assisted Development of Distributed Systems* — Przemysław Ładyński, WeAreDevelopers 2026

---

## License

This project is released under the [MIT License](LICENSE). You may use, copy, modify, merge, publish, distribute, sublicense, and sell it without restriction.

The `.specify/` and `.github/` scaffolding was generated with [GitHub Spec Kit](https://github.com/github/spec-kit); see the Spec Kit repository for its license terms.
