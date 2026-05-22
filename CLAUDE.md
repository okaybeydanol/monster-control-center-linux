# CLAUDE.md

## Project purpose

Monster Control Center Linux is an open-source Linux desktop control center for Monster/Clevo laptops.

The project is both a real hardware-control desktop application and a learning project for professional Python, PySide6, Pydantic v2, Linux hardware boundaries, and Claude Code review workflows.

Do not lower engineering quality for speed.

## Commands

```text
git diff                 # review before committing
git status --short       # check working tree
git log --oneline -5     # verify after push
```

## Current stage

The project is in documentation, architecture, and safety-boundary setup.

Current priority:

- keep hardware research separate from production code
- document assumptions before implementation
- define safe architecture boundaries
- prepare review workflows before hardware-facing code

Do not implement Python domain models, PySide6 GUI, ACPI/HID backends, or privileged helpers until the relevant boundaries are documented and reviewed.

## Canonical docs

- `docs/architecture.md` — application architecture
- `docs/ai-workflow.md` — AI workflow and Claude Code role
- `docs/hardware-notes.md` — hardware research boundaries and safety status

Do not duplicate content from these files here.

## Architecture boundaries

The GUI must only collect user intent and call application services.
The GUI must never run sudo, execute shell commands, or contain hardware logic.

Full layer rules and dependency direction live in `docs/architecture.md`.

## Hardware safety rules

Before any hardware-facing code, follow the production path in `docs/hardware-notes.md`.

No hardware write, ACPI payload, HID payload, fan curve, or RGB behavior without that documented path.

## Shell prototype policy

Prior shell commands and experiments are evidence, not architecture.

Do not wrap prototype scripts as the production backend.
Do not copy experimental ACPI, HID, or AML behavior into runtime code without converting it into typed, bounded, reviewed application architecture.

## AI assistant role

Claude Code is a review assistant for this project, not a primary code generator.

For exact role boundaries, permitted actions, and review checklists, see `docs/ai-workflow.md`.

The human developer owns all commits and architectural decisions.

## Workflow

Prefer the Claude Code best-practice flow:

1. explore
2. plan
3. implement
4. verify
5. commit

For unclear or multi-file tasks, use plan mode first.
For small, obvious documentation edits, a direct change is acceptable.

Keep prompts specific: name the target files, state what must not change, state the intended commit message, state verification expectations.

## Commit discipline

Use small, reviewable commits. A commit should change one concept at a time.

Good examples:

```text
docs(hardware): add initial research boundaries
docs(claude): add project guidance
docs(domain): outline model boundaries
```

Bad examples:

```text
wip
fix stuff
fan rgb gui backend together
```

## Documentation-first policy

When in doubt, document the boundary before writing code.

Preferred progression:

1. documentation
2. domain concepts
3. fake/mock backend contracts
4. application service boundaries
5. tests
6. real infrastructure
7. privileged helper
8. packaging

Hardware-facing implementation must not lead the design.

## Context management

Keep Claude Code sessions focused. Use a fresh session or `/clear` between unrelated tasks.

When compacting, preserve: current goal, modified files, intended commit message, safety constraints, commands already run, verification status.

## Future Claude Code extensions

Use `CLAUDE.md` only for short, always-relevant project guidance.

Reusable review workflows should later live under `.claude/skills/`. Do not add them now.

Do not add hooks, MCP, plugins, or custom subagents until there is a clear, specific need.
