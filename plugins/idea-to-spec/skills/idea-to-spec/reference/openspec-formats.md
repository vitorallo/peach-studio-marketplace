# OpenSpec Artifact Formats (spec-driven schema)

Authoritative format reference for authoring OpenSpec changes. Verified against **openspec v1.2.0**, schema `spec-driven` (`proposal → specs → design → tasks`). Read this before writing any change; the formats are strict and some failures are silent.

All examples use neutral placeholder capabilities like `user-auth` and `data-export` — replace them with your project's real, kebab-case capability names.

## Table of contents

1. [Command reference](#1-command-reference)
2. [config.yaml and the context block](#2-configyaml-and-the-context-block)
3. [Change scaffold layout](#3-change-scaffold-layout)
4. [proposal.md](#4-proposalmd)
5. [specs/<capability>/spec.md](#5-specscapabilityspecmd)
6. [design.md](#6-designmd)
7. [tasks.md](#7-tasksmd)
8. [Validation gotchas](#8-validation-gotchas)

---

## 1. Command reference

```bash
openspec init --tools claude --force        # scaffold openspec/ and config.yaml (non-interactive)
openspec new change <name> --description "<text>"   # create openspec/changes/<name>/ (README + .openspec.yaml)
openspec instructions <artifact> --change <name>    # enriched authoring guidance + template for one artifact
                                                    # <artifact> ∈ proposal | specs | design | tasks
openspec validate <name> --strict           # validate one change strictly (run after each change)
openspec validate --changes --strict         # validate all changes
openspec list                                # list active changes (--specs lists specs, --json for tooling)
openspec show <name> [--json]                # show a change or spec
openspec status --change <name>              # artifact completion status for a change
openspec view                                # interactive TUI dashboard of specs + changes
openspec archive <name> -y                   # archive an implemented change into main specs
```

`openspec init --tools` accepts a comma-separated list of AI tools (`claude`, `cursor`, `codex`, `gemini`, etc.), or `all` / `none`. `--force` auto-cleans legacy files without prompting.

`openspec archive` flags: `-y/--yes` skips confirmation, `--skip-specs` for infra/tooling/doc-only changes that don't touch specs, `--no-validate` to skip validation (not recommended).

`openspec instructions <artifact> --change <name>` is the most useful command while authoring — it prints the task, the output path, detailed section-by-section instructions, and a fill-in template tailored to that change. Run it whenever unsure about a specific artifact.

---

## 2. config.yaml and the context block

`openspec init` creates `openspec/config.yaml`. The `context:` block is shown to the AI when generating artifacts, so fill it with the architecture/deployment decisions you locked earlier — this grounds every generated artifact in the real project.

```yaml
schema: spec-driven

context: |
  Tech stack: <languages, frameworks, key libraries>
  Deployment model: <where it runs, how it's packaged/distributed>
  Conventions: <commit style, code layout, naming, testing approach>
  Domain: <one-line description of what the project is>

# Optional per-artifact rules:
# rules:
#   proposal:
#     - Keep proposals under 500 words
#   tasks:
#     - Break tasks into chunks of max 2 hours
```

---

## 3. Change scaffold layout

`openspec new change <name> --description "..."` creates:

```
openspec/changes/<name>/
├── README.md          # title + the description you passed
└── .openspec.yaml     # schema: spec-driven  +  created: <date>
```

You then author the four artifacts so the change becomes:

```
openspec/changes/<name>/
├── proposal.md
├── specs/
│   └── <capability>/
│       └── spec.md
├── design.md          # include for significant changes; optional for trivial ones
└── tasks.md
```

One `spec.md` per capability listed in the proposal's Capabilities section.

---

## 4. proposal.md

Establishes **WHY** the change is needed (the "how" belongs in design.md). Keep it to 1–2 pages.

Required sections:

- `## Why` — 1–2 sentences on the problem/opportunity. Why now?
- `## What Changes` — bullet list of concrete changes. Mark breaking changes with **BREAKING**.
- `## Capabilities` — the contract between proposal and specs. Critical: every capability listed here needs a corresponding spec file.
  - `### New Capabilities` — each `- \`kebab-name\`: description` becomes `specs/<kebab-name>/spec.md`.
  - `### Modified Capabilities` — existing capabilities whose *requirements* change (not just implementation). Use names that already exist under `openspec/specs/`. Leave empty if none.
- `## Impact` — affected code, APIs, dependencies, systems.

Example:

```markdown
## Why

Users have no way to recover their data if they switch tools, which blocks adoption by teams with compliance requirements.

## What Changes

- Add a CSV export endpoint for user-owned records
- Add an "Export" action to the account screen
- **BREAKING** Remove the legacy XML dump endpoint

## Capabilities

### New Capabilities
- `data-export`: Export user-owned data in portable formats

### Modified Capabilities
- `user-auth`: Export requires a re-authenticated session

## Impact

- New `/api/v1/export` route; touches auth middleware and the records store
- Adds a streaming CSV dependency
```

---

## 5. specs/<capability>/spec.md

Defines **WHAT** the system does, as a *delta* against existing specs. One file per capability. Use the exact kebab-case name from the proposal (new capabilities) or the existing spec folder name (modified capabilities).

### Delta-operation headers (`##`)

- `## ADDED Requirements` — brand-new requirements.
- `## MODIFIED Requirements` — changed behavior. **Repeat the ENTIRE updated requirement block** (header through all scenarios); partial content loses detail at archive time. Header text must match the existing requirement (whitespace-insensitive).
- `## REMOVED Requirements` — deprecated requirements. Each MUST include **Reason** and **Migration**.
- `## RENAMED Requirements` — name-only changes, using FROM:/TO: format.

### Requirement and scenario format

- Each requirement: `### Requirement: <name>` followed by normative text using **SHALL / MUST** (avoid "should"/"may").
- Every requirement MUST have **at least one scenario**.
- Each scenario uses **exactly four hashtags**: `#### Scenario: <name>` with `- **WHEN** ...` / `- **THEN** ...` bullet lines.

> **CRITICAL:** Scenarios must use exactly four hashtags (`####`). Using three hashtags, or writing scenarios as plain bullets, makes them **fail validation silently** — the requirement looks scenario-less and the change won't validate. This is the most common authoring mistake.

Example:

```markdown
## ADDED Requirements

### Requirement: User can export their data
The system SHALL allow an authenticated user to export all of their own records in CSV format.

#### Scenario: Successful export
- **WHEN** an authenticated user triggers an export
- **THEN** the system streams a CSV file containing all records the user owns

#### Scenario: Unauthenticated request
- **WHEN** an unauthenticated client requests an export
- **THEN** the system responds with 401 and exports nothing

## REMOVED Requirements

### Requirement: Legacy XML dump
**Reason**: Replaced by the CSV export capability
**Migration**: Call `/api/v1/export` instead of `/api/v0/dump`
```

---

## 6. design.md

Explains **HOW** to implement. Include it for cross-cutting or architecturally significant changes; it's optional for trivial ones.

Include design.md when any apply: change spans multiple modules/services, introduces a new architectural pattern, adds a significant external dependency, changes the data model meaningfully, has security/performance/migration complexity, or has ambiguity worth resolving before coding.

Sections:

- `## Context` — background, current state, constraints, stakeholders.
- `## Goals / Non-Goals` — what this design achieves and explicitly excludes.
- `## Decisions` — key technical choices, each with rationale (*why X over Y?*) and **alternatives considered**.
- `## Risks / Trade-offs` — known limitations; format each as `[Risk] → Mitigation`.
- `## Migration Plan` — (optional) deploy steps and rollback strategy.
- `## Open Questions` — (optional) unresolved decisions.

---

## 7. tasks.md

The implementation checklist. The apply phase **parses checkbox format literally**, so the format must be exact.

- Group tasks under numbered `## N. <Group name>` headings.
- Each task is a checkbox: `- [ ] N.M <description>` (note the space inside the brackets).
- Order tasks by dependency — what must happen first.
- Keep each task small enough to finish in one session and individually verifiable.

```markdown
## 1. Setup

- [ ] 1.1 Add the streaming CSV dependency
- [ ] 1.2 Create the export module skeleton

## 2. Core implementation

- [ ] 2.1 Implement the data-export query over user-owned records
- [ ] 2.2 Wire the `/api/v1/export` route through auth middleware
- [ ] 2.3 Stream results as CSV

## 3. Verification

- [ ] 3.1 Add tests for the success and unauthenticated scenarios
```

> Tasks that don't use the exact `- [ ]` form won't be tracked during apply.

---

## 8. Validation gotchas

- **Four-hashtag scenarios.** `#### Scenario:` exactly. Wrong hashtag count = silent failure (see §5).
- **Every requirement needs ≥1 scenario.** A requirement with no valid scenario fails strict validation.
- **MODIFIED requires full content.** Copy the entire existing requirement block, then edit — don't paste a fragment.
- **REMOVED requires Reason + Migration.** Both fields are mandatory.
- **Capabilities ↔ spec files must match.** Every capability named in proposal.md needs a `specs/<name>/spec.md`, and vice versa.
- **Task checkboxes are literal.** `- [ ] N.M ...` with the space; numbered group headings.
- **Always run `openspec validate <name> --strict`** after authoring a change, and fix until clean before moving on.
