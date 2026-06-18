---
name: idea-to-spec
description: Drive a loose idea for any tool or software project to spec-driven implementation artifacts — discuss and scope the concept, research current best options when the tech is unfamiliar, ask a few high-leverage questions, lock architecture decisions, write an extensive PRD and an epics breakdown, then codify each epic into OpenSpec changes (proposal → specs → design → tasks) using the openspec CLI. Use this skill whenever a user wants to turn an idea or feature into specs, write a PRD and epics, create or author OpenSpec changes, run a spec-driven workflow, scope a new tool or project, or break an idea into epics — even if they don't name "OpenSpec" or "PRD" explicitly. Domain-agnostic; works for CLIs, services, libraries, apps, pipelines, or hardware-adjacent software.
---

# Idea → Spec-Driven Development

Take a half-formed idea for *any* tool or software project and drive it to concrete, validated spec-driven artifacts: a PRD, an epics map, and a set of OpenSpec changes ready to implement. This skill encodes the **workflow and the tooling**, not any particular domain — apply it to whatever the user is building.

The end state you are producing:
- `docs/PRD.md` — the product requirements document
- `docs/epics.md` — epics mapped to OpenSpec changes, in dependency order
- `openspec/` — one validated OpenSpec change per epic (proposal, specs, design, tasks)

Work the steps below in order, but stay conversational and flexible — skip or compress steps that don't fit the user's project, and don't bureaucratically over-produce for a tiny tool.

## 0. Optional kickoff menu

Before diving in, offer the user a menu of optional add-ons and let them pick any combination (use `AskUserQuestion` with multi-select; "none" is a valid answer — go straight to step 1). Run the chosen ones at the point in the workflow where they fit best (noted per item), not necessarily up front.

1. **Generate an architecture diagram (Mermaid).** Produce a **Mermaid** diagram of components and data flow (`flowchart`/`graph` for runtime topology, optionally a `sequenceDiagram` for a key flow). Save it to `docs/architecture.md` and embed it in the PRD's System Architecture section. When selected, use Mermaid **instead of** the ASCII diagram in step 5 (Mermaid renders on GitHub). Best run during/after step 4, once components are settled. Minimal example:

   ```mermaid
   flowchart LR
     user[Client] --> api[API / Orchestrator]
     api --> svcA[Service A]
     api --> store[(Storage)]
   ```

2. **Generate a README and create a private git repository.** Write a `README.md` (one-line pitch, quickstart, project structure, status), then initialize git and publish to a **private** remote — never public. Run after the PRD/epics exist so the README can summarize them. Creating a remote is an outward-facing action: **confirm the repo name with the user first**, then:

   ```bash
   git init && git add -A && git commit -m "Initial scaffold: PRD, epics, OpenSpec changes"
   gh repo create <name> --private --source=. --push   # PRIVATE only
   ```

   If `gh` isn't available/authed, ask the user to run an interactive login (`! gh auth login`) or stop at the local commit and tell them the manual step.

3. **Research similar solutions / open-source projects for inspiration.** Before scoping, web-search for existing tools and OSS projects that solve a similar problem. Summarize: what already exists, what's worth borrowing (patterns, libraries, UX), and the gap the user's project should differentiate into. Feeds the PRD's Vision and Non-Goals. This overlaps with step 2 — when both are selected, do it as part of that research pass.

4. **Discuss the tech stack with the user.** Don't silently assume a stack. For each major component (e.g. backend, data store, model/runtime, frontend, packaging), present a **recommended choice plus 1–2 alternatives with a one-line trade-off**, anchored to the constraints from step 4 (platform, offline, cost, team familiarity). Then ask the user to confirm or adjust with a compact `AskUserQuestion` (one question per component that genuinely has a live trade-off; default the rest). Record the result — it feeds step 4's decisions and the `openspec/config.yaml` `context:` block. Best run alongside steps 3–4.

## 1. Discuss & scope the idea

Understand the concept conversationally before formalizing anything. Decompose the idea into its core components / stages / capabilities — the natural seams along which it would be built (e.g. ingestion, processing, storage, interface). Reflect this decomposition back to the user so you're aligned on the shape of the thing.

## 2. Ground with research when needed

If the idea leans on fast-moving or unfamiliar tech — model choices, libraries, frameworks, hardware, third-party APIs, pricing/licensing — do targeted web searches to verify the *current* best options **before** recommending. Do not rely on stale training knowledge for anything that changes quickly. Synthesize into a **decisive recommendation** (a clear pick with a one-line rationale), not an exhaustive survey. Research in parallel via subagents when several independent questions exist.

## 3. Clarify with the user

Ask a **small** number of high-leverage, multiple-choice questions — only decisions that genuinely require the user and that materially change the design. Typical ones: form factor / packaging, target platform or hardware, and v1 scope (what's in vs. deferred). For everything else, pick sensible defaults and state them. Resist the urge to over-ask; a few sharp questions beat a long questionnaire.

## 4. Lock architecture & deployment decisions

Record the firm choices so downstream artifacts are grounded: where it runs, how it's packaged/distributed, key constraints (latency, offline, privacy, cost), the major components, and how they communicate. Write these down explicitly — they become the PRD's architecture section and the `context:` block in OpenSpec config.

## 5. Write the PRD → `docs/PRD.md`

Write an extensive PRD. Treat the following as a **template to adapt**, not a rigid checklist — drop sections that don't apply, expand the ones that matter:

- Overview & Vision
- Goals / Non-Goals
- Personas & Use Cases
- User Stories & Flows
- Functional Requirements
- Non-Functional Requirements
- System Architecture (include a diagram of components and data flow — an **ASCII diagram**, or a **Mermaid** diagram if kickoff option 1 was selected)
- Dependencies / Models & Licensing
- API Surface
- Data & Storage
- Risks & Mitigations
- Milestones & Acceptance Criteria

## 6. Break into epics → `docs/epics.md`

Slice the PRD into epics, each one small enough to become a single OpenSpec change. Produce a **table** mapping each epic → its OpenSpec change name → its capability spec name (kebab-case) → dependencies → goal. Then list the **dependency-ordered implementation sequence** so it's clear what gets built first.

| Epic | OpenSpec change | Capability spec | Depends on | Goal |
|------|-----------------|-----------------|------------|------|
| Authentication | `add-user-auth` | `user-auth` | — | Let users sign in securely |
| Data export | `add-data-export` | `data-export` | `add-user-auth` | Export user data as CSV |

## 7. Codify into OpenSpec changes

Use the `openspec` CLI (schema `spec-driven`, verified with v1.2.0) to turn each epic into a change. The spec-driven flow authors four artifacts per change in this order: **proposal → specs → design → tasks**.

Core commands:

```bash
openspec init --tools claude --force      # scaffolds openspec/ + openspec/config.yaml
openspec new change <name> --description "<one-line goal>"   # creates openspec/changes/<name>/
openspec instructions <artifact> --change <name>   # prints enriched per-artifact authoring guidance
openspec validate <name> --strict         # validate a change; run after authoring each one
openspec list                             # list changes (--specs to list specs)
openspec view                             # interactive dashboard of specs and changes
openspec archive <name> -y                # archive a change once implemented
```

After `init`, open `openspec/config.yaml` and fill in the `context:` block (tech stack, deployment model, conventions from step 4) so generated artifacts are grounded in the real project. Then for each epic: `openspec new change`, author the four artifacts, and `openspec validate <name> --strict` until clean.

**The exact artifact formats matter** — scenarios in particular fail validation *silently* if you use the wrong number of hashtags, and the apply phase parses task checkboxes literally. Do not author from memory: before writing each change, read **`reference/openspec-formats.md`** for the precise format of proposal.md, specs/<capability>/spec.md, design.md, and tasks.md, plus the validation gotchas. When in doubt about a specific artifact, also run `openspec instructions <artifact> --change <name>` to get the CLI's own enriched guidance and template.

## 8. Implement each change — with discipline

Implement the changes in the dependency order from `docs/epics.md`. For **every** change, hold to this discipline (don't skip it for "small" features):

1. **Document as you build.** Update `docs/` (PRD/feature docs/README) in the same pass as the code — never "later". New feature or changed behavior ⇒ the docs change with it.
2. **Write proper tests and make them pass.** Add real tests for the new behavior and actually run them until green before moving on. Don't mark a task done on untested code. Keep tests dependency-light so they run anywhere (e.g. on the dev box and inside the target container/CI).
3. **Browser-test anything with a UI.** For web/UI features, verify real interaction with **Playwright** (or the **Claude Chrome extension**), not just unit tests — load the page, drive the flow, assert what the user sees.
4. **Check off `tasks.md`** as each item lands, then **`openspec archive <name>`** once a change is fully implemented and its checkboxes are complete, so it folds into the project's main specs and clears the active queue.

Treat docs + passing tests (+ browser check for UI) as the definition of done for a change, not optional extras.

## Reference files

- `reference/openspec-formats.md` — verbose, authoritative format spec for all four OpenSpec artifacts (proposal, specs, design, tasks), delta operations, the four-hashtag scenario rule, config.yaml `context:`, and the full command reference. Read this before authoring any OpenSpec change.
