# Application Services

This document defines the intended application service boundaries for Monster Control Center Linux.

It is documentation only.

This file does not implement Python code, service classes, backend protocols, Pydantic models, tests, fake backends, real backends, GUI code, ACPI/HID logic, privileged helper transport, shell commands, or hardware payloads.

The goal is to define what the application layer should coordinate before implementation begins.

## Purpose

The application layer coordinates user-facing use cases.

It sits between the GUI and the domain/backend protocol boundaries.

The application layer should answer questions such as:

- What happens when the user selects Performance?
- What happens when the user selects Quiet?
- What happens when the user requests Automatic fan mode?
- What happens when the user sets a keyboard static color?
- What happens when the application asks for read-only system status?
- What should happen when a feature is unsupported?

The application layer should express use-case flow, not hardware mechanics.

## Relationship to canonical docs

This document complements:

- `docs/architecture.md` for layer boundaries and dependency direction
- `docs/domain-models.md` for domain concepts and validation boundaries
- `docs/hardware-notes.md` for hardware research boundaries and safety status
- `docs/ai-workflow.md` for review expectations

This file must not duplicate hardware payload details from `docs/hardware-notes.md`.

This file must not define domain model fields in detail. Domain model boundaries live in `docs/domain-models.md`.

## Application layer responsibility

The application layer is responsible for:

- coordinating user-facing use cases
- accepting validated domain concepts
- calling backend protocol concepts
- sequencing safe operations
- handling unsupported capability decisions
- translating backend results into application-level outcomes
- keeping GUI logic separate from backend logic
- keeping hardware mechanics outside the GUI

Examples of application responsibility:

- apply a product profile
- choose the fan behavior associated with a product profile
- request a fan mode change through a backend boundary
- request RGB behavior through a backend boundary
- request read-only status through a backend boundary
- reject a requested behavior when the capability is unsupported
- return clear success, failure, or unsupported results to the GUI

The application layer should be testable without real hardware.

## What the application layer must not know

The application layer must not know about:

- PySide6 widgets
- GUI layout
- GUI event wiring
- Linux device paths
- `/proc/acpi/call`
- `/dev/hidraw*`
- `/dev/mem`
- raw ACPI method names
- raw SCMD integers
- raw PK04 cases
- raw HID report bytes
- AML table loading
- shell command strings
- `sudo`
- subprocess calls for hardware control
- systemd unit details
- polkit policy details
- privileged helper transport internals

The application layer must not perform hardware access directly.

It must not execute shell commands.

It must not open device files.

It must not construct ACPI or HID payloads.

It must not become a privileged helper client that exposes arbitrary hardware writes.

## Intended dependency flow

The intended flow is:

1. GUI collects user intent.
2. GUI calls an application service.
3. Application service uses domain concepts.
4. Application service calls backend protocol concepts.
5. Infrastructure later implements backend protocols.
6. Privileged operations, where needed, are isolated behind a narrow helper boundary.

The application layer should depend on abstractions, not concrete infrastructure implementations.

The application layer may know that it needs a fan backend or RGB backend concept.

It must not know whether the eventual implementation uses ACPI, HID, a privileged helper, fake state, or another infrastructure mechanism.

## Dependency direction

Application services depend on:

- domain models for typed, validated inputs
- backend protocols as abstract interfaces

Application services must not depend on:

- concrete infrastructure backend implementations
- PySide6
- hardware access code
- shell scripts

This allows the same application service to work with a fake backend during development and a real infrastructure backend during production.

## Service design principles

Application services should follow these principles:

- Use product language, not hardware language.
- Accept and return domain concepts where possible.
- Coordinate use cases; do not perform low-level hardware work.
- Depend on backend protocol concepts, not concrete infrastructure.
- Treat unsupported capabilities as expected states, not unexpected crashes.
- Keep methods narrow and understandable.
- Keep service behavior deterministic where possible.
- Do not hide dangerous behavior behind convenience methods.
- Do not expose generic command execution.
- Do not expose arbitrary ACPI or HID write operations.
- Keep application services independent from PySide6.

Application services should make the safe path the easy path.

## Backend protocol concepts

Backend protocol concepts are the boundary between application use cases and infrastructure implementations.

This document only names the conceptual backend boundaries.

It does not implement protocols.

Possible future backend protocol concepts include:

- `FanBackend`
- `RgbBackend`
- `SystemStatusBackend` or `SystemProbeBackend`

These names are conceptual and may be refined during implementation.

Backend protocols should expose semantic operations.

They should not expose arbitrary hardware writes.

Allowed conceptual shape:

- set a known fan mode
- apply a validated fan curve
- set a validated keyboard static color
- set a supported lightbar behavior
- read supported status information

Forbidden conceptual shape:

- run any shell command
- write any ACPI payload
- write any HID payload
- write any file as root
- open any device path from application code
- pass raw hardware byte arrays from the application layer

Concrete backend implementations are not part of this document.

Fake backend strategy will be documented separately before backend implementation begins.

## Candidate service boundaries

### ProfileService

`ProfileService` coordinates high-level product profile selection.

Possible future responsibilities:

- apply Performance profile
- apply Entertainment profile
- apply Quiet profile
- decide which lower-level fan and RGB use cases belong to a profile
- reject profile application if required capabilities are unsupported
- return clear application-level results to the GUI

Rules:

- ProfileService should use `ProfileKind`-style domain concepts.
- ProfileService must not contain raw ACPI values.
- ProfileService must not contain raw HID values.
- ProfileService must not know hardware payload formats.
- ProfileService must not execute profile scripts.
- ProfileService must not directly access hardware.

Important boundary:

- A product profile is not the same thing as a raw firmware mode.
- Profile-to-hardware translation belongs behind service/backend boundaries after documentation and review.

### FanService

`FanService` coordinates fan-related use cases.

Possible future responsibilities:

- request Automatic fan mode
- request Maximum fan mode
- request Quiet fan mode
- request Custom Profile fan mode
- apply a validated fan curve when that feature becomes supported
- read fan-related status when supported
- reject unsupported fan behavior clearly

Rules:

- FanService should use domain concepts such as `FanMode`, `FanCurve`, and `FanCurvePoint`.
- FanService must not expose raw SCMD values.
- FanService must not expose PK04 cases.
- FanService must not construct ACPI calls.
- FanService must not write `/proc/acpi/call`.
- FanService must not execute shell commands.
- FanService must not assume all Monster/Clevo models support the same behavior.

FanService may coordinate a backend protocol concept such as `FanBackend`.

The backend protocol may later be implemented by a fake backend first, then by reviewed infrastructure.

### RgbService

`RgbService` coordinates RGB-related use cases.

Possible future responsibilities:

- set keyboard-wide static RGB color
- turn supported lighting devices off
- apply supported lightbar behavior
- reject unsupported brightness, zone, effect, or arbitrary lightbar RGB requests
- expose clear unsupported-feature results to the GUI

Rules:

- RgbService should use domain concepts such as `RgbColor`, future brightness concepts, and future capability concepts.
- RgbService must not expose HID report bytes.
- RgbService must not know `/dev/hidraw*` paths.
- RgbService must not construct HID feature reports.
- RgbService must not assume keyboard zones are known before they are documented.
- RgbService must not assume arbitrary lightbar RGB is supported before it is documented.
- RgbService must not execute shell commands.

RgbService may coordinate a backend protocol concept such as `RgbBackend`.

The backend protocol may later be implemented by a fake backend first, then by reviewed infrastructure.

### SystemStatusService or SystemProbeService

A read-only system status/probe service may later coordinate status and monitoring use cases.

Possible future responsibilities:

- read supported fan status
- read supported thermal status
- read supported device capability status
- read supported system information for display
- report unavailable or unsupported readings clearly

Current status:

- This service should be treated as read-only.
- It should be deferred until the status/probe boundary is documented more carefully.
- It must not become a hidden hardware-control path.

Rules:

- The service must not write hardware state.
- The service must not execute privileged commands directly.
- The service must not read arbitrary system paths directly from application code.
- The service must not assume every sensor is available on every machine.
- The service must not make monitoring behavior responsible for fan or RGB control.

Possible naming:

- `SystemStatusService` if the service represents user-facing status.
- `SystemProbeService` if the service represents hardware/system capability probing.

The final name should be chosen when the read-only boundary is better documented.

## Capability and unsupported behavior

Unsupported behavior is a normal application state.

Application services should not silently fall back from unsupported behavior to another behavior.

Examples:

- If keyboard brightness is unsupported, report it as unsupported.
- If lightbar arbitrary RGB is unsupported, report it as unsupported.
- If fan curve writing is unsupported, report it as unsupported.
- If hardware detection is inconclusive, do not assume support.

The application layer should provide clear outcomes that the GUI can display.

Possible conceptual outcomes:

- success
- unsupported
- invalid request
- backend unavailable
- failed with recoverable error
- failed with safety-critical error

These are conceptual outcomes only.

This document does not define result classes or error types.

## Profile application boundary

Applying a product profile may eventually coordinate multiple lower-level use cases.

For example, a future Performance profile may involve fan behavior and possibly RGB behavior.

However, profile application must remain bounded.

Profile application must not become:

- a shell script runner
- a generic command batch executor
- an arbitrary ACPI/HID payload dispatcher
- a hidden startup hardware writer
- a place for undocumented hardware assumptions

Each profile action should be expressed through documented domain concepts and backend protocol concepts.

If a profile requires unsupported behavior, the application should report that clearly.

## Read-only does not mean boundary-free

Read-only operations still need boundaries.

A service that reads status must not bypass architecture just because it does not write hardware.

Read-only behavior may still involve:

- privileged paths
- device files
- unstable sensor availability
- model-specific assumptions
- infrastructure-specific parsing

Therefore, read-only system status and hardware probing should still go through backend protocol concepts and infrastructure implementations.

The GUI must not read system paths directly.

The application layer must not hide direct system access.

## Fake backend relationship

Fake backends are expected before real hardware backends.

This document defines the application service side of that boundary.

A later backend strategy document should define:

- why fake backends come before real backends
- how fake backend state should behave
- how fake backends support GUI development
- how fake backends support tests
- how fake backends model unsupported capabilities
- how fake backends avoid ACPI/HID access entirely

Application services should be designed so they can run against fake backends before any real hardware integration exists.

## Hardware safety boundary

Application services must not contain hardware payloads.

Do not put the following in application services:

- raw ACPI method names
- raw SCMD integers
- raw PK04 case numbers
- fan table byte offsets
- raw HID report prefixes
- raw HID report lengths
- raw keyboard key IDs as device protocol values
- raw lightbar payload bytes
- `/dev/hidraw*` paths
- `/proc/acpi/call` paths
- `/dev/mem` paths
- shell command strings
- `sudo` commands
- systemd unit details
- polkit policy details
- privileged helper transport internals

Hardware payload construction belongs in infrastructure boundaries after documentation, protocols, fake backends, tests or review checks, and hardware safety review.

## Service-to-code path

Before application service code is added, the intended use cases should be documented here.

Before backend implementation is added, the backend strategy should be documented separately.

Before hardware-facing code is added, the behavior must also be documented in `docs/hardware-notes.md`.

The preferred path is:

1. document domain concepts
2. define application service boundaries
3. document backend strategy
4. define backend protocol boundaries
5. document fake/mock backend behavior
6. add application service implementation
7. add validation and service tests
8. later add reviewed infrastructure implementation
9. later add privileged helper where needed

Do not skip from hardware research directly to application runtime code.
