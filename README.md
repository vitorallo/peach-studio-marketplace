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
| [`business-profiler`](https://github.com/vitorallo/business-profiler) | Cybersecurity | Threat intelligence, attack surface assessment, and strategic sales targeting (7 skills) |

### Install a plugin

```bash
/plugin install business-profiler@peach-studio
```

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

## About

PEACH STUDIO is a research lab working on AI-powered security tooling. We build plugins that do the tedious parts of threat analysis, compliance research, and sales prep so you can focus on the parts that need a human.

More plugins on the way.

## License

MIT

---

[peachstudio.be](https://www.peachstudio.be)
