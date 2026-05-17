# AI Workflow

This project uses AI tools as engineering assistants, not as uncontrolled code generators.

Monster Control Center Linux is both:

1. A real open-source Linux hardware-control desktop application.
2. A learning project for professional Python, Linux desktop architecture, Claude Code workflows, local LLM review workflows, and AI-assisted engineering discipline.

The project must be developed slowly, explicitly, and with senior-level engineering standards.

## Core rule

AI tools must not be used to bypass engineering discipline.

The project must not accept lower-quality code because it is faster to generate.

The highest rule is:

> Do not write junior-level or shortcut code just because it is easier.

Every implementation should be treated as production-grade unless it is explicitly marked as research or prototype code.

## Human developer context

The primary developer has senior software development experience, mainly with JavaScript and PHP ecosystems.

The developer is using this project to learn:

- Python
- Typed Python architecture
- Pydantic v2
- PySide6 desktop application development
- Linux desktop application structure
- Linux hardware-control boundaries
- ACPI concepts
- HID concepts
- Debian packaging basics
- Claude Code
- Claude Code Skills
- Local LLM review workflows
- AI review workflows
- Safe human/AI collaboration patterns

Explanations should be technical, precise, and respectful of an experienced developer background.

Do not explain like the developer is new to programming.

Do explain Python-specific, Linux-specific, desktop-specific, hardware-specific, and Claude Code-specific concepts carefully when they are introduced.

The preferred learning style is:

- Small steps
- Small commits
- Clear file responsibilities
- Explicit architectural boundaries
- Explain why each file exists
- Explain why each dependency exists
- Explain why each validation exists
- Explain why each layer exists
- Discuss tradeoffs before committing to a design
- Avoid hidden shortcuts
- Prefer senior-level code even if it is more complex

If something is unclear, the developer will ask.

## Engineering constitution

This project must always prefer:

- Clear architecture
- Typed models
- Explicit protocols
- Validated inputs
- Small commits
- Reviewable diffs
- Documented hardware behavior
- Safe privilege separation
- Testable application services
- Hardware access isolated away from the GUI
- Conservative handling of ACPI and HID behavior

This project must reject:

- GUI-level hardware access
- GUI-level `sudo`
- Untyped hardware payload dictionaries
- Generic command execution helpers
- Hidden shell-script behavior
- Large mixed commits
- Undocumented ACPI assumptions
- Undocumented HID assumptions
- Experimental scripts copied directly into production code
- Convenience shortcuts that weaken architecture
- AI-generated code that the developer does not understand

## Human developer role

The human developer owns the project.

Responsibilities:

- Decide what gets committed.
- Read and understand every meaningful change.
- Keep commit scope small.
- Reject shortcuts.
- Ask questions when unclear.
- Approve when hardware-facing behavior is ready to move forward.
- Keep research/prototype code separate from production code.
- Keep machine-specific local AI configuration out of the public repository.

AI tools may assist, but they do not own architectural decisions.

## ChatGPT role

ChatGPT is the main architect, teaching assistant, and decision support partner for this project.

Responsibilities:

- Lead the high-level architecture.
- Teach Python, Pydantic, PySide6, Linux desktop structure, and Claude Code usage.
- Propose small, reviewable code changes.
- Explain why each file, model, service, and boundary exists.
- Keep the project aligned with senior-level engineering standards.
- Prevent unsafe hardware shortcuts.
- Translate hardware research into production-ready architecture.
- Help decide when something belongs in documentation, domain, application, infrastructure, GUI, packaging, or AI workflow.
- Keep commits focused and understandable.
- Prefer explicit design discussion before implementation.

ChatGPT may propose code, but code should be introduced gradually and explained.

ChatGPT should not:

- Push the project too quickly.
- Hide complexity behind unexplained abstractions.
- Convert experimental shell scripts directly into production code.
- Encourage GUI code to execute privileged operations.
- Accept unsafe hardware assumptions.
- Use junior-level shortcuts for speed.
- Treat shell scripts as the final product architecture.
- Generate large unexplained patches.

## Claude Code role

Claude Code is primarily a review assistant for this project.

Claude Code may be used to:

- Review diffs before commits.
- Check architecture boundaries.
- Check typing issues.
- Check missing tests.
- Check unsafe hardware access.
- Check oversized commit scope.
- Check whether a change belongs in the current commit.
- Suggest improvements without taking over implementation.
- Run explicitly requested review workflows.
- Later, run project skills for repeatable review tasks.

Claude Code should usually be prompted with review-only instructions.

Example review instruction:

```text
Review this diff only.

Do not edit files.
Do not rewrite the implementation.
Do not create new files.
Do not apply patches.

Check for:
- architecture violations
- typing issues
- missing tests
- unsafe hardware access
- oversized commit scope
- unclear naming
- hidden shortcuts

Return:
- blockers
- warnings
- optional suggestions
- whether this commit is safe to commit
```

Claude Code should not be used as the primary code generator for this project.

Claude Code may write code only when the developer explicitly asks for a very small, narrow, reviewable change.

## Local MiniMax-M2.7 UD-Q4_K_XL role

A local MiniMax-M2.7 model may be used as a long-running local review assistant through Claude Code-compatible local configuration.

The local MiniMax-M2.7 workflow is intended for times when the developer is away from active coding, such as overnight, during long breaks, or while doing unrelated work.

The local MiniMax-M2.7 role is:

- Long-horizon repository review.
- Architecture review.
- Security review.
- Hardware safety review.
- Test gap review.
- Documentation consistency review.
- Packaging risk review.
- Cross-file consistency review.

The local MiniMax-M2.7 role is not:

- Primary implementation.
- Automatic patch generation.
- Unreviewed code editing.
- Hardware command execution.
- A replacement for human understanding.
- A reason to make larger commits.

Local MiniMax-M2.7 may use the same project skills as Claude Code.

The recommended usage pattern is:

```text
Human developer
  ↓
Small focused commit candidate
  ↓
Claude Code or local MiniMax-M2.7 review
  ↓
Developer reads the review
  ↓
Developer decides
  ↓
Commit
```

For long overnight review, local MiniMax-M2.7 should be asked to produce a report, not edit files.

Example local long-review instruction:

```text
Review the entire repository as a read-only reviewer.

Do not edit files.
Do not create files.
Do not apply patches.
Do not run hardware commands.
Do not call sudo.
Do not access /proc/acpi/call, /dev/mem, HID devices, or privileged system paths.

Check:
- architecture violations
- layer boundary issues
- missing tests
- missing documentation
- unsafe hardware assumptions
- unclear public APIs
- oversized modules
- naming problems
- future packaging risks
- GUI/infrastructure coupling
- generic command execution risks

Return:
- executive summary
- blockers
- warnings
- suggested future commits
- files that need human review
```

Machine-specific local model configuration must not be committed to the public repository.

Allowed locations for local-only configuration:

```text
~/.claude/settings.json
~/.claude/settings.local.json
personal notes outside the repository
```

Disallowed in the public repository:

```text
personal model paths
personal API tokens
machine-specific home directory paths
private local inference settings
```

If the project later documents local model usage, it should use generic examples and clearly mark them as optional.

## AI tool hierarchy

The intended workflow is:

```text
Human developer
  ↓
ChatGPT as main architect and teacher
  ↓
Developer writes or applies a small change
  ↓
Claude Code or local MiniMax-M2.7 reviews the diff
  ↓
Developer decides
  ↓
Commit
```

The important point is that AI review does not replace developer ownership.

## Claude slash command and brevity policy

Claude Code supports slash commands and custom commands, but this project should not depend on unverified third-party slash commands.

Some public workflows mention commands or skills such as:

```text
/caveman
/ghost
/deep think
ultrathink
```

These may be useful in personal workflows, but they must not become required project infrastructure unless they are reviewed and intentionally adopted.

Project policy:

- Do not depend on unverified influencer commands.
- Do not assume a command exists unless it is visible in the local Claude Code session or documented by the installed skill/plugin.
- Do not encode third-party command assumptions into project documentation.
- Prefer project-owned skills for repeatable behavior.
- Prefer explicit review instructions over magical command names.

Brevity is still important.

Instead of requiring a third-party brevity command, project skills should include concise output rules.

Example concise review output rule:

```text
Be concise.
No motivational filler.
No generic praise.
No long preamble.
Return only blockers, warnings, optional suggestions, and safe-to-commit status.
```

Deep reasoning is also important, but it should be requested explicitly for complex decisions.

Example deep review instruction:

```text
Think carefully before answering.
Do a layered review of architecture, typing, tests, hardware safety, and commit scope.
Return a concise final report.
```

## Skills-first policy

This project should prefer Claude Code Skills over custom subagents for repeated review workflows.

Review roles such as Python quality review, hardware safety review, desktop UI review, packaging review, and diff review should initially be implemented as project skills.

Skills are preferred because they represent reusable instructions, checklists, procedures, references, and review playbooks.

A skill is the correct abstraction when the project needs to say:

```text
When reviewing this kind of change, follow this exact checklist.
When checking hardware code, load these safety rules.
When reviewing Python code, apply these typing and architecture expectations.
When reviewing PySide6 code, check GUI/application boundary rules.
```

A subagent is not needed for that at the beginning.

## Mandatory review scope map

AI reviewers must treat the following project areas as mandatory review scopes.

These are not passive documentation topics.

They are review obligations.

```text
Architecture boundary changes
  → review-diff skill
  → python-quality-review skill if Python code is touched

Domain model changes
  → python-quality-review skill
  → testing checklist

Application service changes
  → review-diff skill
  → python-quality-review skill
  → testing checklist

GUI changes
  → desktop-ui-review skill
  → architecture boundary checklist

Infrastructure changes
  → review-diff skill
  → python-quality-review skill
  → hardware-safety-review skill if hardware or system paths are touched

ACPI changes
  → hardware-safety-review skill
  → hardware research checklist
  → recovery path checklist

HID/RGB changes
  → hardware-safety-review skill
  → hardware research checklist
  → payload validation checklist

Privileged helper changes
  → hardware-safety-review skill
  → security checklist
  → packaging-review skill if systemd/polkit/deb files are touched

Packaging changes
  → packaging-review skill
  → security checklist if permissions/services/polkit are touched

Release changes
  → release-review skill
  → documentation consistency checklist
```

If a commit touches more than one mandatory review scope, the reviewer must say whether the commit should be split.

## Planned project skills

Future project skills may live under:

```text
.claude/skills/review-diff/SKILL.md
.claude/skills/python-quality-review/SKILL.md
.claude/skills/hardware-safety-review/SKILL.md
.claude/skills/desktop-ui-review/SKILL.md
.claude/skills/packaging-review/SKILL.md
.claude/skills/release-review/SKILL.md
```

These should be added gradually.

Do not add all skills at once.

Each skill should be introduced only when the project has a real repeated workflow for it.

## Skill design principles

Project skills should follow these rules:

- Keep `SKILL.md` focused.
- Use `SKILL.md` as the entry point.
- Move detailed checklists to supporting files.
- Move long references to supporting files.
- Avoid broad tool permissions.
- Avoid hidden side effects.
- Avoid writing code from a review skill.
- Prefer review output over automatic edits.
- Keep hardware-facing skills conservative.
- Keep each skill narrow.

Example skill structure:

```text
.claude/skills/review-diff/
├── SKILL.md
├── checklists/
│   ├── architecture.md
│   ├── typing.md
│   ├── testing.md
│   └── safety.md
└── examples/
    └── review-output.md
```

The main `SKILL.md` should say what the skill does and when to load supporting files.

Detailed supporting files should be loaded only when relevant.

## Skill invocation policy

Some skills may be automatically invoked by Claude Code when relevant.

Other skills should be manual-only.

Manual-only skills are appropriate when the workflow has side effects or should run only when explicitly requested.

For this project:

- Review skills may be automatically suggested.
- Commit/release skills should be manual-only.
- Hardware safety review may be invoked automatically when hardware files are touched.
- Any skill that runs commands should be manual-only until proven safe.
- Any skill that touches hardware must not run automatically.

Recommended default for early skills:

```text
Review-only.
No file edits.
No command execution unless explicitly requested.
No hardware access.
```

## Planned skill responsibilities

### review-diff

Purpose:

- General diff review.
- Commit scope review.
- Architecture boundary review.
- Decide whether more specialized review is needed.

Should check:

- Is the commit too large?
- Does the change belong in this commit?
- Does the diff mix unrelated concerns?
- Does the change violate layer boundaries?
- Does the change need tests?
- Does the change need documentation?
- Does the change touch hardware, packaging, or privileged boundaries?
- Which specialized skills should be applied?

### python-quality-review

Purpose:

- Python typing and code quality review.

Should check:

- Type hints.
- Mypy strict compatibility.
- Ruff compatibility.
- Naming.
- Import structure.
- Pydantic model design.
- Avoidance of dynamic untyped dictionaries.
- Testability.
- Correct use of protocols/interfaces.
- Clear error handling.
- Small, readable modules.
- Layer-appropriate dependencies.

### hardware-safety-review

Purpose:

- ACPI, HID, root, systemd, polkit, `/sys`, `/proc`, `/dev`, and privileged helper safety review.

Should check:

- Is the behavior based on verified research?
- Is unsupported hardware rejected safely?
- Is there a recovery path?
- Are payloads typed and validated?
- Does the GUI avoid direct hardware access?
- Is hardware write behavior documented?
- Is there any generic command execution path?
- Are experimental scripts separated from production code?
- Is the privileged helper boundary respected?

### desktop-ui-review

Purpose:

- PySide6 GUI architecture review.

Should check:

- GUI only collects user intent.
- GUI calls application services.
- GUI does not import ACPI/HID/system backends directly.
- GUI does not use sudo.
- GUI does not execute shell commands.
- Widgets are small and focused.
- Pages are separated clearly.
- Presentation logic is not mixed with hardware logic.

### packaging-review

Purpose:

- Linux packaging and distribution review.

Should check:

- Debian package structure.
- Desktop entry.
- App metadata.
- Systemd service files.
- Polkit policy.
- File permissions.
- Post-install behavior.
- Removal behavior.
- No unsafe automatic hardware writes during install.

### release-review

Purpose:

- Pre-release quality review.

Should check:

- Version bump.
- Changelog.
- README accuracy.
- Known limitations.
- Packaging state.
- Test state.
- Hardware safety notes.
- Upgrade/downgrade behavior.
- Recovery instructions.
- Whether public claims match implemented features.

## Subagents policy

Custom subagents should not be introduced at the beginning of the project.

Subagents may be considered later only when a task needs isolated context, parallel exploration, or large-volume codebase research that would pollute the main conversation.

Examples where a subagent may become appropriate later:

- Searching a large codebase while keeping the main session clean.
- Running a read-only architecture survey.
- Comparing multiple implementation strategies in parallel.
- Investigating packaging options without mixing exploratory logs into the main session.
- Investigating hardware research logs in a separate context and returning only a summary.

Subagents must not become uncontrolled implementers.

If custom subagents are introduced later, they should be:

- Narrow.
- Read-only by default.
- Clearly scoped.
- Explicitly named.
- Limited in tool access.
- Used for exploration or review, not primary implementation.
- Preloaded only with the skills they need.

Subagents should not be added just because they are available.

## CLAUDE.md policy

`CLAUDE.md` should be short and high-signal.

It should contain rules that Claude Code must know at the start of every session.

It should not become a giant project manual.

Good content for `CLAUDE.md`:

- Project purpose.
- Core architecture rule.
- Commit discipline.
- AI role boundaries.
- Hardware safety warning.
- Main commands.
- What not to do.

Bad content for `CLAUDE.md`:

- Long hardware research dumps.
- Full ACPI tables.
- Full HID protocol notes.
- Long review checklists.
- Detailed packaging procedures.
- Large code examples.
- Machine-specific local model configuration.

Large or procedural content belongs in:

```text
Large human-facing documentation belongs in:

docs/

Short always-loaded Claude Code project rules belong in:

CLAUDE.md

Reusable Claude Code workflows belong in:

.claude/skills/
.claude/skills/<skill-name>/references/
.claude/skills/<skill-name>/checklists/
```

## Claude settings policy

Project-level Claude settings may be introduced later under:

```text
.claude/settings.json
```

Project-level settings may define:

- Safe permissions.
- Hook configuration.
- Shared project behavior.
- Project-specific Claude Code defaults.

Local-only settings should remain uncommitted:

```text
.claude/settings.local.json
~/.claude/settings.json
```

Local settings may contain:

- Machine-specific paths.
- Personal preferences.
- Experimental permissions.
- Local model endpoint settings.
- Temporary hook experiments.

Do not commit secrets.

Do not commit personal tokens.

Do not commit local model paths.

Do not commit machine-specific home directory paths.

Do not commit private local inference settings.

## Hooks policy

Hooks are for deterministic automation.

Hooks should be introduced only after the relevant manual workflow is understood.

Possible future hook use cases:

- Run Ruff after Python edits.
- Run Mypy before commit.
- Run Pytest before commit.
- Block GUI-level `sudo`.
- Block GUI imports of infrastructure hardware modules.
- Warn when hardware files change without documentation updates.
- Warn when ACPI/HID code is modified without hardware safety review.
- Block generic command execution helpers.
- Notify when Claude Code is waiting for user input.

Hooks should be:

- Small.
- Readable.
- Deterministic.
- Documented.
- Easy to disable while debugging.
- Added one at a time.

Hooks should not:

- Hide important behavior.
- Perform hardware writes.
- Run dangerous commands.
- Become complex AI-driven automation too early.
- Replace human review.

Hooks are not a substitute for architecture.

## MCP policy

MCP is not required at the beginning of this project.

MCP may be considered later only if there is a real need to connect AI tooling to an external system.

Possible future MCP use cases:

- Issue tracker integration.
- Documentation search integration.
- CI/build status integration.
- Package repository automation.
- Hardware database lookup.

MCP should not be added just because it exists.

Current policy:

```text
No MCP until there is a clear external integration need.
```

## Plugins policy

Claude Code plugins may be useful later for sharing skills, commands, hooks, and settings across projects.

This project should not start with a plugin.

A plugin may be considered later if the project creates reusable AI workflow assets that should be shared outside this repository.

Current policy:

```text
No custom plugin until project skills stabilize.
```

## AI-generated code policy

AI-generated code must meet the same standard as human-written code.

AI-generated code is not acceptable if:

- It is not understood.
- It is too broad.
- It mixes layers.
- It is untyped without reason.
- It bypasses tests.
- It hides hardware behavior.
- It weakens privilege separation.
- It introduces generic command execution.
- It changes hardware behavior without documentation.
- It uses convenience shortcuts instead of proper architecture.

AI-generated code may be acceptable if:

- It is small.
- It is reviewed.
- It is understood.
- It fits the current commit scope.
- It follows the architecture.
- It is typed.
- It has or does not require tests for a clear reason.
- It does not touch unsafe hardware boundaries without explicit review.

## General review checklist

Before important commits, review should check:

- Is this commit too large?
- Does this change belong to the current layer?
- Does the change mix unrelated concerns?
- Does GUI code call system commands?
- Does GUI code know ACPI or HID details?
- Does GUI code use sudo?
- Does domain code depend on infrastructure?
- Does application code import PySide6?
- Are new domain rules validated?
- Are safety-sensitive assumptions documented?
- Are tests needed for this change?
- Are names clear and production-ready?
- Is the code typed?
- Is the code compatible with the project architecture?
- Is this still a learning-friendly step?
- Would a future maintainer understand the boundary?

## Mandatory hardware safety review

Any change touching fan, RGB, ACPI, HID, `/sys`, `/proc`, `/dev`, `/dev/mem`, root permissions, systemd, polkit, or privileged helpers requires extra review.

Hardware-related review must check:

- Is the behavior based on verified research?
- Is the behavior documented?
- Is the operation bounded?
- Is the payload typed or validated?
- Can unsupported hardware be rejected safely?
- Is there a recovery path?
- Is the GUI isolated from privileged operations?
- Is this experimental code or production code?
- Is there a risk of writing to the wrong device?
- Is there a risk of leaving the system in an unsafe state?
- Is the operation reversible?
- Is the commit small enough to review safely?

Experimental hardware scripts must not be silently converted into runtime application code.

## Mandatory hardware research review

Hardware research may involve scripts, logs, dumps, ACPI calls, HID calls, and controlled experiments.

Research artifacts should be separated from production code.

Research may live in:

```text
notes/
research/
docs/hardware-notes.md
docs/acpi-fan-notes.md
docs/rgb-notes.md
```

Production code must not depend on random research scripts.

Before research becomes production code, it must be converted into:

- Typed models.
- Explicit service methods.
- Backend interfaces.
- Validated payload builders.
- Safe infrastructure code.
- Tests where possible.
- Documentation.
- Recovery notes where relevant.

Reviewers must check whether a hardware change is still research or has been promoted into production code.

If research is promoted into production code without documentation, the review must block the commit.

## Mandatory hardware implementation boundary review

The project architecture is:

```text
GUI Layer
  ↓
Application Layer
  ↓
Domain Layer
  ↓
Infrastructure Layer
  ↓
Privileged Helper / Operating System / Hardware
```

The GUI must never perform privileged hardware operations directly.

The application layer should coordinate use cases.

The domain layer should validate concepts.

The infrastructure layer should implement real system boundaries.

The privileged helper should perform only a small set of allowed privileged operations.

The privileged helper must not expose a generic shell command API.

Reviewers must check this boundary whenever a change touches:

- GUI code.
- Application services.
- Infrastructure backends.
- Hardware payload builders.
- Privileged helper code.
- Packaging files related to services, permissions, or polkit.
- Documentation that claims hardware behavior.

Boundary violations are blockers.

## Commit workflow

Preferred workflow:

```text
1. Discuss the next small step.
2. Define the file responsibility.
3. Write or edit the file.
4. Explain what changed.
5. Run the smallest useful check.
6. Review git diff.
7. Optionally ask Claude Code or local MiniMax-M2.7 for review.
8. Commit.
9. Push.
10. Check git log.
```

Commits should be small and named with Conventional Commits.

Good examples:

```text
docs(ai-workflow): document AI review policy
chore(ai): add basic CLAUDE.md
chore(ai): add review diff skill
feat(domain): add fan mode enum
test(domain): validate fan point bounds
feat(gui): add main window shell
```

Bad examples:

```text
wip
fix
update
add app
fan rgb gui backend all together
```

A commit should normally answer one question:

```text
What single thing did this change introduce?
```

## AI workflow roadmap

Planned AI workflow evolution:

```text
1. docs(ai-workflow): document AI review policy
2. chore(ai): add basic CLAUDE.md
3. chore(ai): add review-diff skill
4. chore(ai): add python quality review skill
5. chore(ai): add hardware safety review skill
6. chore(ai): add desktop UI review skill
7. chore(ai): add safe Claude settings
8. chore(ai): add simple hooks only after skills are stable
9. chore(ai): add subagents only if isolated context becomes necessary
10. chore(ai): add MCP only if a real external integration need appears
```

This roadmap may change as the project evolves.

The important rule is:

```text
Do not add AI workflow machinery before its purpose is clear.
```

## Current AI workflow status

At the current stage:

- No `CLAUDE.md` exists yet.
- No project skills exist yet.
- No hooks are active.
- No subagents are configured.
- No MCP integration is configured.
- No Claude project settings are committed.
- Local MiniMax-M2.7 configuration is personal and must remain outside the repository.

This file is the first AI workflow policy document.

The next likely AI workflow step is:

```text
chore(ai): add basic CLAUDE.md
```

That step should happen only after the initial repository documentation is stable.
