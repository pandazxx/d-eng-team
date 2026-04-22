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
  * A knowledge base that works as a **note dropbox**: any role may drop a note into `team/kb/entries/` at any time — no orchestrator privilege required. Notes record anything worth sharing: bugs, gotchas, decisions, useful patterns. Librarian periodically scans the dropbox, merges duplicate entries, and updates the index. Stale indexing is acceptable. Agents query the KB by loading `team/kb/index.md` (titles and links only, low context cost), then loading a specific entry only when relevant to the current task.


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
| KB entry            | Any role         | A note dropped into `team/kb/entries/`. Records bugs, gotchas, decisions, or useful patterns worth sharing across sessions. |
| KB index            | Librarian        | Merged, deduplicated index of all KB entries. Agents load this first; entry files on demand.                  |
| Index               | Librarian        | Index of all project documents (design docs, requirements, test cases). Provides the agent onboarding trail.  |


# Roles

- Product Owner
  * **Scope**: product level
  * **Phase 1 — Requirements**:
    - Input: user input (vision, goals, problem statements)
    - Output: Initiative document, prioritised backlog, business constraints
    - Behaviour: challenges scope and priority from a business value perspective — asks "what's the ROI?", "who are the users?", "what's the priority?". Spawns BA to structure requirements.
  * **Phase 2 — UAT**:
    - Triggered by Librarian after `test-cases-ready`
    - Input: reads Implementation against Initiative document and Use cases
    - Output: UAT result — pass or fail with specific gaps
    - Pass: notifies user that the cycle is complete and ready for release pipeline
    - Fail: spawns BA to update requirements, loop restarts from design
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
    - On all tasks complete: Architect notifies the user; Librarian handles the per-module QA and UAT flow via lifecycle events
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
  * **Behaviour**: event-driven workflow coordinator and KB curator. Each lifecycle event tells Librarian what just completed; Librarian indexes the new documents and decides whether to advance the workflow by spawning the next orchestrator. On every trigger, Librarian also scans `team/kb/entries/` for new or updated notes, merges duplicates, and updates the KB index.
  * **Events**:
    - `requirements-ready` — BA has produced requirements. Librarian indexes them, then spawns Architect to begin design.
    - `solution-ready` — Architect has produced or revised the Highlevel design. Librarian indexes it and notifies the user to review. No automatic progression — implementation begins only on explicit user instruction.
    - `development-done` — Developer has completed a module. Librarian indexes updated design docs, then spawns QA for that module.
    - `test-cases-ready` — QA has completed test cases for a module. Librarian indexes them, then spawns PO to perform UAT for that module.
    - `uat-passed` — PO has validated the module against business requirements. Librarian indexes the result and notifies the user. CI/CD and release are pipeline actions triggered by the user via tagging or PR merging — not managed by any role.
    - `uat-failed` — PO found gaps against business requirements. Librarian spawns BA to update requirements; the design-implement-test loop restarts.
  * **Triggering Librarian**: any role fires an event by including `Event: <name>` in its response. The receiving orchestrator reads the field and spawns Librarian with the event as the prompt. Event firing requires no orchestrator privilege — only the final spawning of Librarian does.


| role          | is orchestrator | counterparts  | Document scope                                                  |
| ------------- | --------------- | ------------- | --------------------------------------------------------------- |
| Product Owner | Yes             | BA, Librarian | Initiative<br>Prioritised backlog<br>Business constraints       |
| BA            | No              | PO            | Use cases<br>Detail requirements                                |
| Architect     | Yes             | Librarian, BA | Highlevel design<br>ADR<br>Manuals                              |
| Developer     | No              | Architect     | Internal design<br>Interface design                             |
| QA            | No              | Librarian     | Test cases                                                      |
| Librarian     | Yes             | Everyone      | KB index<br>Index                                               |


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
                                   ┌────┴──────────────────────────────┐
                                   │  User reviews design               │
                                   │  ├─ changes requested  ──►  Architect revises
                                   │  │                           └─ fires: solution-ready (loop)
                                   │  └─ satisfied  ─────────►  User: "proceed with implementation"
                                   └───────────────────────────────────┘
                                        │
                         [Implementation phase — explicit user trigger]
                              Architect assigns module tasks to Developers (one per module)
                              └─ each Developer completes task
                                   └─ fires: development-done
                                        └─ Librarian indexes  →  spawns QA for that module
                                             └─ QA produces test cases + results
                                                  └─ fires: test-cases-ready
                                                       └─ Librarian indexes  →  spawns PO for UAT
                                                            └─ PO validates against Initiative + Use cases
                                                                 ├─ uat-passed  →  Librarian notifies user
                                                                 │                 └─ User triggers pipeline
                                                                 │                    (tag / PR merge → CI/CD → release)
                                                                 └─ uat-failed  →  Librarian spawns BA
                                                                                   └─ requirements updated, loop restarts
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
Event: <event-name>          (optional)
---
<response body>
```

- **Status**:
  - **Done**: task complete. Response body is the result.
  - **Stuck**: task blocked by insufficient information or an issue outside the callee's scope. Response body describes the blockage. The caller must resolve it (clarify, coordinate, or escalate to the user) and re-invoke.
- **Event** (optional): the lifecycle event this role fires on completion. Any role may include this field. The orchestrator that receives the response is responsible for reading it and spawning Librarian with the event name as the trigger.

## Event firing

Roles signal lifecycle completion via the `Event:` field in their response — no orchestrator privilege required. The receiving orchestrator spawns Librarian after reading the response:

```bash
# orchestrator reads response, sees Event: requirements-ready
claude -p "Event: requirements-ready
Summary: <what was produced>" \
  --system-prompt "$(cat .claude/skills/librarian.md)"
```

Librarian processes the event, indexes new documents, and decides whether to advance the workflow.

| Role | Event fired |
|---|---|
| BA | `requirements-ready` |
| Architect (design phase) | `solution-ready` |
| Developer | `development-done` |
| QA | `test-cases-ready` |
| PO (UAT) | `uat-passed` or `uat-failed` |

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
