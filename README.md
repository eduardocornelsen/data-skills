██████╗  █████╗ ████████╗ █████╗           
██╔══██╗██╔══██╗╚══██╔══╝██╔══██╗          
██║  ██║███████║   ██║   ███████║          
██║  ██║██╔══██║   ██║   ██╔══██║          
██████╔╝██║  ██║   ██║   ██║  ██║          
╚═════╝ ╚═╝  ╚═╝   ╚═╝   ╚═╝  ╚═╝          
                                           
███████╗██╗  ██╗██╗██╗     ██╗     ███████╗
██╔════╝██║ ██╔╝██║██║     ██║     ██╔════╝
███████╗█████╔╝ ██║██║     ██║     ███████╗
╚════██║██╔═██╗ ██║██║     ██║     ╚════██║
███████║██║  ██╗██║███████╗███████╗███████║
╚══════╝╚═╝  ╚═╝╚═╝╚══════╝╚══════╝╚══════╝

# data-skills

Composable Claude Code skills that turn an agent into a senior data science team. Plug-and-play for any project — add as a git submodule and Claude can invoke the playbooks and personas directly.

## What's in here

| Folder | Purpose |
|---|---|
| `playbooks/` | Step-by-step instructions for each phase of a DS project (problem framing, data contracts, data modeling, EDA, hypothesis testing, FE, training, evaluation, inferencing, monitoring, experimentation) |
| `personas/` | Role-specific system prompts (data analyst, analytics engineer, ML engineer, reviewer, etc.) |
| `premises/` | Reusable checklists of statistical assumptions for tests and models |
| `checklists/` | Short "real DS would not skip" guards (leakage, multiple-testing, reproducibility, data quality tests) |
| `templates/` | Copy-paste scaffolds (problem statements, model cards, experiment designs, metric definitions) |
| `domain_accelerators/` | Deeper guided instructions for common business problems (ad-click, churn, forecasting, fraud, recommenders) |
| `examples/reference_implementations/` | Full working end-to-end examples |

## Quickstart

### Add as a git submodule

```bash
git submodule add <repo-url> data-skills
git submodule update --init --recursive
```

### Invoke a playbook or persona via Claude Code

Claude Code auto-discovers skills under `data-skills/.claude/skills/`. Examples:

- `Skill(eda)` — load the EDA playbook
- `Skill(hypothesis-testing)` — load the hypothesis testing playbook
- `Skill(persona-data-analyst)` — adopt the data analyst persona
- `Skill(persona-analytics-engineer)` — adopt the analytics engineer persona

## Status

In active development. See [`PLAN.md`](./PLAN.md) for the roadmap and current sequencing.

## License

MIT — see [`LICENSE`](./LICENSE).
