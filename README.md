# d-eng-team
An AI Agent plugin to manage software engineering team AI workflow

# Glossary

- **the plugin**: this plugin
- **the project** / **target project**: the repo/project that installs this plugin
- **the user**: the human who is working on the **project**
- **the workflow**: the way that this plugin works.

# Initiative

Manage an AI engineering team for the target project, with:

- Document as memory.
- Different roles with separate sessions, hence separated contexts and memories as well.
- Agents with a high turnover rate, don't just rely on the session / context memory.


# Ideology

- Skill **onboarding**: gets the repo ready for this workflow, ensures that the `AGENTS.md`/`CLAUDE.md` contains necessary instruction for this workflow
- Document structure:
  * Three levels of design documents:
    - **Highlevel design**:
      * Key business flow
      * Modules boundaries and relationships
    - **Internal design**:
      * Module logical design
      * Module _internal_ usage, for who works on this module to reference.
    - **Interface design**:
      * API spec
      * detail flow chart
      * Exceptional handling
      * Module _external_ usage, for those who works with this module to reference.
  * An instruction / indexing system for agent to reference. A new agent without any context and memory, can follow this trail to understand the project and start to work on specific task with minimum effort/context.
    1. When a new agent gets a problem statement, it reads the **Highlevel design** to understand the project and which module to work with.
    2. Read the **Internal design** to understand further and work on the solution
    3. For cross module dependencies and interactions, refer to target module's **Interface design**
    4. Update **Internal design** and **Interface design** of the module it's working on
    5. If modification / support from other module is required, trigger **architect** to coordinate.
  * A knowledge base with index. Agent can get knowledge on demand. For example, title: Bug in foo() function in base library, link to a document that describe this bug in detail and the solution to avoid it. Agent only keep the title in the context, only during troubleshooting or using this function will the agent load the detail into the context.


# Documents



| Document            | Owner            | Descriptions                                                                                                  |
|---------------------|------------------|---------------------------------------------------------------------------------------------------------------|
| Initiative          | Product Owner    | Target and end-goal of the project. A north star of the project.                                              |
| Prioritised backlog | Product Owner    | Ordered list of features and requirements by business priority                                                |
| Business constraints | Product Owner   | Non-functional constraints from a business perspective (budget, timeline, compliance, etc.)                   |
| Use cases           | Business analyst | - Use cases for business scenarios<br>- Includes acceptance criteria<br>- Includes the exceptional scenarios |
| Detail requirements | Business analyst | Supplementary document of **Use cases**. Provide detail requirement of Use cases                              |
| ADR                 | Architect        | - Architecture decision record, record every key architecture decisions                                       |
| Highlevel design    | Architect        | - Highlevel design of the project<br>- Focus on highlevel flow and the boundary of modules                     |
| Internal design     | Engineer         | - Detail logic of the module<br>- Internal usage reference for engineers working on this module               |
| Interface design    | Engineer         | - API spec, flow charts, exception handling<br>- External usage reference for engineers working with this module |
| Test cases          | QA               | - Test case<br>- User test cases and instructions                                                             |
| Manuals             | Architect        | - `Usage.md`<br>- `User Manuals.md`                                                                           |
| Index               | Librarian        | - index of all documents<br>- provide index for agent to reference                                            |


# Roles

- Product Owner
  * **Scope**: product level
  * **Input**: user input (vision, goals, problem statements)
  * **Output**: Initiative document, prioritised backlog, business constraints
  * **Behaviour**: challenges scope and priority from a business value perspective — asks "what's the ROI?", "who are the users?", "what's the priority?". Delegates requirements structuring to BA.
- Business Analyst
  * **Scope**: requirements level
  * **Input**: Initiative and direction from Product Owner, or direct user input
  * **Output**: use cases, acceptance criteria, detail requirements
  * On completion: fires `requirements-ready` event
- Architect
  * **Scope**: project level
  * **Input**: requirements and user stories from BA — **or direct user input**
  * **Note**: Architect is also a direct user entry point. The user may bring a problem statement or feature request straight to Architect without going through PO/BA. If requirements are clear and actionable, Architect proceeds directly. If requirements are unclear or incomplete, Architect always delegates to PO/BA first — it never asks the user for clarification itself.
  * **Phase 1 — Design**:
    - Output: Highlevel design, ADRs
    - On completion: fires `solution-ready` event to Librarian, then stops and awaits user review
    - The user reviews the design and may request changes. Architect revises and fires `solution-ready` again. This loop continues until the user is satisfied.
  * **Phase 2 — Implementation** (explicit user trigger only):
    - Begins only when the user explicitly instructs Architect to proceed
    - Architect reads the approved Highlevel design, breaks it into module-level tasks, and spawns one Developer per task
    - Each task assignment includes: module name, task description, design doc references, and cross-module constraints
- Developer
  * **Scope**: module level — one Developer per assigned task
  * **Input**: task assignment from Architect (module, task, design doc references, constraints)
  * **Output**: implementation, updated Internal design and Interface design
  * On completion: fires `development-done` event
- QA
  * **Scope**: module level
  * **Input**: reads Internal design and Interface design from `docs/internal/` and `docs/interface/`; reads acceptance criteria from BA's Use cases document in `docs/`
  * **Output**: test cases, test results, GitHub issues (for bugs)
  * On completion: fires `test-cases-ready` event
- Librarian
  * **Scope**: project level
  * **Trigger**: lifecycle events — not called directly by other roles or the user
  * **Output**: updated Index, and may spawn other orchestrators to advance the workflow
  * **Behaviour**: event-driven workflow coordinator. Each lifecycle event tells Librarian what just completed; Librarian indexes the new documents and decides whether to advance the workflow by spawning the next orchestrator.
  * **Events**:
    - `requirements-ready` — BA has produced requirements. Librarian indexes them, then spawns Architect to begin design.
    - `solution-ready` — Architect has produced or revised the Highlevel design. Librarian indexes it and notifies the user to review. No automatic progression — implementation begins only on explicit user instruction.
    - `development-done` — Developer has completed a module. Librarian indexes updated design docs, then spawns QA for that module.
    - `test-cases-ready` — QA has completed test cases. Librarian indexes them and reports status.
  * **Triggering Librarian**: any role may fire an event on completion — event firing is distinct from process spawning and requires no orchestrator privilege. Librarian reacts to the event independently.


| role          | is orchestrator | counterparts             | Document scope                                           |
| ------------- | --------------- | ------------------------ | -------------------------------------------------------- |
| Product Owner | Yes             | BA, Architect            | Initiative                                               |
| BA            | No              | Product Owner, Architect | Use cases<br>Detail requirements                         |
| Architect     | Yes             | Everyone                 | Highlevel design<br>ADR<br>Manuals                       |
| Developer     | No              | Architect, QA            | Internal design<br>Interface design                           |
| QA            | No              | Developer, BA            | Test cases                                               |
| Librarian     | Yes             | Everyone                 | Index                                                    |


# End-to-end workflow

```
User
 │
 ├─(requirements unclear)─► /po  →  PO spawns BA  →  BA produces requirements
 │                                  └─ fires: requirements-ready
 │                                       └─ Librarian indexes  →  spawns Architect (design phase)
 │
 └─(requirements clear)──► /architect
                              │
                         [Design phase]
                              Architect produces Highlevel design + ADRs
                              └─ fires: solution-ready
                                   └─ Librarian indexes  →  notifies user to review
                                        │
                                   ┌────┴────────────────────────┐
                                   │  User reviews design         │
                                   │  ├─ changes requested  ──►  Architect revises
                                   │  │                           └─ fires: solution-ready (loop)
                                   │  └─ satisfied  ─────────►  User: "proceed with implementation"
                                   └─────────────────────────────┘
                                        │
                         [Implementation phase — explicit user trigger]
                              Architect assigns module tasks to Developers (one per module)
                              └─ each Developer completes task
                                   └─ fires: development-done
                                        └─ Librarian indexes  →  spawns QA for that module
                                             └─ QA produces test cases
                                                  └─ fires: test-cases-ready
                                                       └─ Librarian indexes  →  notifies user
```


# Session architecture

Each role runs as a separate `claude` CLI process, giving true context and memory isolation. Roles communicate through files. The current implementation is synchronous — the caller blocks until the spawned process completes. Async is a future upgrade path.

## Spawning a role

Orchestrators use the Bash tool to launch a new `claude` process, injecting the target role's skill as the system prompt:

```bash
claude -p "From: Architect
To: Developer
---
<request body>" \
  --system-prompt "$(cat .claude/skills/developer.md)" \
  > team/outbox/developer-$(date +%s).md
```

The response file is the callee's full reply, written in the response format below. The caller reads it and continues.

## Nesting rule

Spawning is governed by two tiers:

**Tier 1 — Librarian (event-driven top-level coordinator)**
Librarian may spawn any role including full orchestrators. It sits above the normal call hierarchy, activated only by lifecycle events. Because it is never called by another agent, it cannot create cycles.

**Tier 2 — All other orchestrators (PO, Architect)**
When a non-Librarian orchestrator calls another orchestrator, the callee runs in non-orchestrator mode and cannot spawn further. This caps that branch at depth 2.

The caller enforces non-orchestrator mode by appending to the system prompt:

```bash
ROLE_SKILL=$(cat .claude/skills/architect.md)

claude -p "..." \
  --system-prompt "${ROLE_SKILL}

You are running as a delegated agent. Do not spawn new claude processes."
```

Effective call graph:

```
User
 └── Product Owner (orchestrator) ← requirements flow
      ├── BA              (non-orchestrator)
      └── Architect       (called as non-orchestrator)
           └── [no further spawning]

User
 └── Architect (orchestrator) ← direct entry point
      ├── Developer       (non-orchestrator)
      ├── QA              (non-orchestrator)
      └── BA              (non-orchestrator)

[Event] ── requirements-ready / solution-ready / development-done / test-cases-ready
 └── Librarian (event-driven, tier-1 orchestrator)
      ├── Architect       (full orchestrator)
      │    ├── Developer  (non-orchestrator)
      │    └── QA         (non-orchestrator)
      └── Product Owner   (full orchestrator)
           └── BA         (non-orchestrator)
```

Max effective depth: Event → Librarian → Orchestrator → Non-orchestrator (3 levels, always bounded).

# Orchestration

Orchestrators command other roles via file-based message passing. Request and response files are written to `team/outbox/`.

## Task assignment (Architect → Developer)

A task assignment is a request with a structured body:

```
From: Architect
To: Developer
---
Module: <module name>
Task: <what to implement>
Design refs:
  - docs/highlevel/<file>.md
  - docs/internal/<module>.md
Constraints: <cross-module dependencies or interface contracts to respect>

<additional context>
```

Architect spawns one Developer process per task. In the current sync implementation, tasks are assigned sequentially. Parallel assignment is a future upgrade.

## Request format

```
From: <from role>
To: <target role>
---
<request body>
```

## Response format

```
From: <from role>
To: <to role>
Status: <Done | Stuck>
---
<response body>
```

- **Done**: task complete. Response body is the result.
- **Stuck**: task blocked by insufficient information or an issue outside the callee's scope. Response body describes the blockage. The caller must resolve it (clarify, coordinate, or escalate to the user) and re-invoke.

# How this plugin works

## Plugin mechanism

This plugin is implemented as **Claude Code skills + `CLAUDE.md` injection**. No npm, no CLI, no separate installer. Claude Code is the runtime.

Skills are markdown files (`.claude/skills/<name>.md`) invoked as `/skill-name` inside Claude Code. The plugin ships role skills (`/ba`, `/architect`, `/developer`, `/qa`, `/librarian`) and one lifecycle skill (`/onboarding`).

## Installation

The entire plugin is distributed as a **single self-contained skill file**. To install:

```bash
curl -o .claude/skills/onboarding.md https://raw.githubusercontent.com/<org>/d-eng-team/main/onboarding.md
```

Then inside Claude Code:

```
/onboarding
```

That's it. The `/onboarding` skill bootstraps everything else — no second network call required. All role skill templates and document templates are embedded in the single file. Claude materialises them locally on first run.

## The `/onboarding` skill

`/onboarding` is the single entry point for the full plugin lifecycle:

| Mode | Command | Behaviour |
|---|---|---|
| Auto-detect | `/onboarding` | Install if fresh project, upgrade if already installed |
| Install | `/onboarding install` | Force full install |
| Upgrade | `/onboarding upgrade` | Overwrite plugin-owned files, preserve user content |
| Doctor | `/onboarding doctor` | Report only — no writes |

### Doctor as the foundation

Doctor defines the canonical "correctly installed" state. Install and upgrade both build on it:

- **Install** = doctor → create everything *missing*
- **Upgrade** = doctor → overwrite everything *outdated*
- **Doctor** = doctor → report only

### Doctor checklist

| Check | Expected state | Owned by |
|---|---|---|
| Role skill files | `.claude/skills/{ba,architect,developer,qa,librarian}.md` exist | Plugin |
| Doc folders | `docs/highlevel/`, `docs/internal/`, `docs/interface/`, `docs/templates/`, `team/kb/` exist | Plugin |
| `CLAUDE.md` block | `<!-- d-eng-team -->` block present and at current version | Plugin |
| KB index | `team/kb/index.md` exists | Plugin (scaffold only) |
| Document templates | One template per document type in `docs/templates/` | Plugin |

### Idempotency rules

| File category | Install | Upgrade |
|---|---|---|
| Role skill files (`.claude/skills/`) | Write | Always overwrite |
| Document templates (`docs/templates/`) | Write | Overwrite if unchanged, prompt if modified |
| `CLAUDE.md` block | Append | Diff and update block in-place |
| KB index (`team/kb/index.md`) | Scaffold empty | Never touch |
| User design docs (`docs/`, `team/`) | Never touch | Never touch |

## Plugin structure

```
onboarding.md          ← single curl target; contains all embedded templates
```

After `/onboarding install`, the project gains:

```
.claude/
  skills/
    onboarding.md      ← lifecycle skill (install / upgrade / doctor)
    ba.md              ← Business Analyst role skill
    architect.md       ← Architect role skill
    developer.md       ← Developer role skill
    qa.md              ← QA role skill
    librarian.md       ← Librarian role skill
docs/
  templates/           ← one template per document type
  highlevel/
  internal/
  interface/
team/
  kb/
    index.md           ← knowledge base index
```




# Notes

- Developer should focus on specific module. With full knowledge of the module, knowledge about the interfaces related to this module, the shape of the project and the high level design instruction given by architect.
- Architect with full knowledge of the system design, full knowledge about the module boundaries, breaks down the requirement and design into module level.
