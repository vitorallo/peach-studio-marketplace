# PEACH STUDIO Marketplace

[Claude Code](https://docs.anthropic.com/en/docs/claude-code) plugins by [PEACH STUDIO](https://www.peachstudio.be). Cybersecurity, AI security, and automation tools.

## Add the marketplace

```bash
/plugin marketplace add vitorallo/peach-studio-marketplace
```

This works in Claude Code CLI, Claude Desktop, and the VS Code extension.

## Available Plugins

| Plugin | Category | Description |
|--------|----------|-------------|
| [`ps-spec`](https://github.com/vitorallo/ps-spec) | Automation | Epic-driven development on OpenSpec: PRD → epics → one change per epic → branch → code → test plan → doc → merge, with four human checkpoints, a rules guard and secure-coding rule packs |
| [`idea-to-spec`](https://github.com/vitorallo/idea-to-spec) | Automation | Turn an idea into a PRD, epics, and validated OpenSpec changes (spec-driven workflow). *Superseded by `ps-spec`.* |
| [`ps-vulnscan`](https://github.com/vitorallo/ps-vulnscan) | Cybersecurity | Rule-driven security scanning: detect the stack, model the app's authorization, scan against matching secure-coding rule packs, one report with CVSS |
| [`business-profiler`](https://github.com/vitorallo/business-profiler) | Cybersecurity | Threat intelligence, attack surface assessment, and strategic sales targeting (7 skills) |

### Install a plugin

```bash
/plugin install ps-spec@peach-studio
/plugin install idea-to-spec@peach-studio
/plugin install ps-vulnscan@peach-studio
/plugin install business-profiler@peach-studio
```

### ps-spec

Epic-driven development on [OpenSpec](https://github.com/Fission-AI/OpenSpec), from an idea or a PRD to shipped, tested, documented features — one epic = one OpenSpec change = one git branch = one feature doc.

```
PRD → [CP1] → EPICS (stories) → [CP2] → plan epic → [CP3] → git branch → code
      → test / fix loop → review & clean up → doc → [CP4] → archive + merge → next epic
```

| Skill | What it does |
|-------|-------------|
| `ps-spec` | `ps-spec next` detects the phase from repo state and runs it: `init` (openspec + custom schema + hooks + secure-coding rules) · `prd` · `epics` · `plan E0N` · `code` · `test` · `doc` · `close`. Four human checkpoints (PRD, epics, plan, validate) with an autonomous agentic coding loop between them. Adds a `test-plan.md` artifact with interactive cases (Playwright / Chrome / http / console), an Observability and a Security section in every design, tasks ticked as they land, `doc/<feature>.md` as project memory. |

What makes it different from writing artifacts by hand: a **rules guard** hook blocks any artifact write until `openspec instructions` (which carries your `config.yaml` rules) has been fetched in the session, and `init` installs **secure-coding rule packs** (OWASP Top 10 2025, MCP/AI/agent/RAG, per-language and per-framework, from [TikiTribe/claude-secure-coding-rules](https://github.com/TikiTribe/claude-secure-coding-rules)) into `.claude/rules/security/`, path-scoped so they cost context only when relevant.

**Prerequisites:** Node ≥ 20.19 and git; the [`openspec`](https://github.com/Fission-AI/OpenSpec) CLI (≥ 1.13) is installed by `init` if missing. Full docs: [ps-spec README](https://github.com/vitorallo/ps-spec#readme). For the [pi coding agent](https://github.com/earendil-works/pi): [`ps-spec-pi`](https://github.com/vitorallo/ps-spec-pi) (`pi install git:github.com/vitorallo/ps-spec-pi`).

### idea-to-spec

> Superseded by **ps-spec**, which covers the same idea → PRD → epics → OpenSpec flow and continues through implementation, testing and docs. Kept for existing users.

Spec-driven development workflow: take a half-formed idea for any tool or software and drive it to concrete, validated artifacts.

| Skill | What it does |
|-------|-------------|
| `idea-to-spec` | Discuss & scope → research current options → write `docs/PRD.md` → break into `docs/epics.md` → codify each epic into a validated [OpenSpec](https://github.com/Fission-AI/OpenSpec) change (proposal → specs → design → tasks). Optional kickoff menu: Mermaid architecture diagram, README + private git repo, prior-art research, tech-stack discussion. |

**Prerequisites:** the [`openspec`](https://github.com/Fission-AI/OpenSpec) CLI (`npm i -g openspec`, v1.2.0+) for the OpenSpec steps. The PRD/epics steps work without it.

### ps-vulnscan

Rule-driven security scanning. The rule packs are chosen for the stack the project actually uses, and the scan runs against them.

```
stack ID → app/auth model → rule packs [you confirm] → scan & confirm → one report
```

| Skill | What it does |
|-------|-------------|
| `ps-vulnscan` | Detects languages, frameworks and AI surface from the manifests; models identities, roles, tenancy and authorization into `security/MODEL.md`; proposes the matching rule packs and downloads them from [claude-secure-coding-rules](https://github.com/TikiTribe/claude-secure-coding-rules); reviews the source against them; writes `security/findings/<target>-security-report.md` with severity, CVSS v3.1, CWE and remediation per finding. |

Leads with broken access control — IDOR/BOLA, missing function-level authorization, cross-tenant leaks — the class no rule file can decide without knowing the app's ownership model, which is why the model comes before the scan. Then injection, SSRF, deserialization, session and JWT flaws, mass assignment, upload and traversal, committed secrets.

One human checkpoint: you see the proposed packs and the engine choice, and nothing is downloaded until you approve. Rule-driven grep and source review by default; semgrep OSS optional. Nothing is executed — no requests, no exploitation, no PoCs.

**Prerequisites:** git (for the one-time rules clone). semgrep is optional: `brew install semgrep` or `pipx install semgrep`.

**On pi:** [`ps-vulnscan-pi`](https://github.com/vitorallo/ps-vulnscan-pi) — same method, with the checkpoint enforced by a pi extension (`pi install git:github.com/vitorallo/ps-vulnscan-pi`).

Deeper work — chained attack scenarios, PoCs, coordinated disclosure — is [`peach-vulnhunt`](https://github.com/vitorallo/peach-vulnhunt).

### business-profiler

Threat intelligence and sales targeting. 7 skills:

| Skill | What it does |
|-------|-------------|
| `full-profile` | Complete assessment: recon + business intel + threats + regulatory + report + PDF |
| `threat-profile` | Threat actors, TTPs, incidents, risk analysis |
| `attack-surface` | Infrastructure reconnaissance and exposure mapping |
| `threat-actors` | MITRE ATT&CK actor lookup by sector, country, or name |
| `ot-ics-assessment` | OT/ICS/SCADA focused assessment |
| `incident-lookup` | Security breach and incident research |
| `sales-targeting` | Strategic account targeting: 6-part sales report with 38-service catalog |

**Prerequisites:** Python 3.10+ with a virtual environment. See the [business-profiler README](https://github.com/vitorallo/business-profiler) for full setup instructions.

```bash
# After installing the plugin, set up the venv
cd ~/.claude/plugins/cache/business-profiler  # or wherever the plugin is cached
python3 scripts/setup.py --venv
source .venv/bin/activate
```

## Troubleshooting

### "Permission denied (publickey)" during install

The installer clones via SSH by default. If you don't have SSH keys for GitHub, run this first:

```bash
git config --global url."https://github.com/".insteadOf "git@github.com:"
```

### SSH host key prompt hangs the installer

If the install hangs asking about GitHub's host key authenticity, you can't type `yes` inside the plugin UI. Add the key beforehand:

```bash
ssh-keyscan github.com >> ~/.ssh/known_hosts
```

### Install fails in Claude Desktop

Claude Desktop runs in a sandbox that may not have git access. Install via Claude Code CLI instead, or clone the plugin locally and point Claude Desktop to the directory.

## About

PEACH STUDIO is a research lab working on AI-powered security tooling. We build plugins that do the tedious parts of threat analysis, compliance research, and sales prep so you can focus on the parts that need a human.

More plugins on the way.

## License

MIT

---

[peachstudio.be](https://www.peachstudio.be)
