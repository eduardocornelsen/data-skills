# data-skills — agent instructions

This directory is a portable skill library for data science work. When operating inside a project that includes this folder:

- **Playbooks** in `playbooks/` are the source of truth for *how* to do each DS phase. Open the relevant one before writing notebook code. Do not paraphrase from memory.
- **Personas** in `personas/` set tone, focus areas, and gates. Adopt the relevant persona via `Skill(persona-...)` when responsibility is unambiguous (data analyst for EDA, analytics engineer for modeled marts, ML engineer for deployment, reviewer for sign-off).
- **Premises** in `premises/` and **checklists** in `checklists/` are referenced by playbooks. Don't re-derive assumptions or non-negotiables — link to them.
- **Templates** in `templates/` are copy-paste artifacts. Fill in, don't paraphrase.
- **Domain accelerators** in `domain_accelerators/` add depth for common business problems. Check if one matches before treating a problem as fully generic.
- **Reference implementations** in `examples/` show what executed output looks like. Don't copy stale notebooks — regenerate from the playbook for the project at hand.

The `.claude/skills/` directory exposes everything as invocable Claude Code skills. Prefer `Skill(...)` over `Read`-then-paraphrase — the skill loads the canonical content directly.

For the current development plan and roadmap, see [`PLAN.md`](./PLAN.md).
