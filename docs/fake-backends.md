# Fake Backends

This document defines the fake-first backend strategy for Monster Control Center Linux.

It is documentation only.

This file does not implement Python code, protocol classes, fake backend classes, real backend classes, service code, Pydantic models, tests, GUI code, ACPI/HID logic, privileged helpers, shell wrappers, systemd/polkit configuration, or hardware payloads.

The goal is to explain why fake backends come before real hardware backends and what safety boundaries they must preserve.

## Purpose

Fake backends are the first safe implementation approach for backend protocol boundaries in this project.

They exist to make the following possible before real hardware integration begins:

- develop and verify application service behavior
- develop and iterate on GUI behavior
- write future tests against application services
- model unsupported capability behavior explicitly
- establish the correct backend protocol shape
- validate that the architecture works end-to-end without hardware

Fake backends are not throwaway hacks.

They are the first controlled expression of what a backend boundary must provide. Writing a fake backend forces the protocol boundary to be concrete, explicit, deterministic, and honest about what it does and does not support.

## Relationship to canonical docs

This document complements:

- `docs/architecture.md` for layer boundaries and dependency direction
- `docs/domain-models.md` for domain concept boundaries
- `docs/application-services.md` for application service use cases and backend protocol concepts
- `docs/hardware-notes.md` for hardware research boundaries and safety status
- `docs/ai-workflow.md` for review expectations

Fake backend design follows from the application service boundaries documented in `docs/application-services.md`.

Fake backends must not introduce hardware behavior that is not documented in `docs/hardware-notes.md`.

This file must not duplicate hardware payload details from `docs/hardware-notes.md`.

This file must not define concrete Python protocol APIs.

Backend protocol boundaries may be documented separately before implementation begins.

## Why fake backends come before real backends

Monster Control Center Linux is a hardware-control project.

That makes early direct hardware integration risky.

Real hardware backends require:

- confirmed hardware research
- typed domain models
- application service boundaries
- documented backend protocol boundaries
- validated payload builders
- hardware safety review
- privileged helper boundaries where needed
- tests or review checks where possible

Fake backends require none of that hardware access.

They implement the same backend boundary using safe in-memory behavior. This allows the rest of the application to be built, reviewed, and iterated on while hardware integration is still being prepared.

The fake-first approach means:

- the GUI does not wait for hardware to be ready
- application services can be exercised without a supported laptop
- future tests can run without root or hardware
- unsupported capabilities are modeled from the beginning
- backend protocol concepts are validated before hardware writes are introduced
- shell prototypes are less likely to become production architecture
- ACPI/HID logic is not added under pressure just to make the UI work

A feature should work against a fake backend before it is promoted toward real infrastructure.

Real hardware backends should not be the first backend implementation. They should replace or extend an already proven fake backend boundary after the semantic contract is clear.

## Application service to backend protocol boundary

The intended layering is:

    Application Service
      depends on Backend Protocol
                       |
              implemented by
                 /         \
        Fake Backend     Real Backend
        in-memory        infrastructure / privileged helper

Application services depend on backend protocol concepts.

They do not know whether the current implementation is fake or real.

This is intentional.

Fake backends and real backends should stand behind the same conceptual boundary. The application service must not behave differently depending on which backend is present, except when a backend explicitly reports that a capability is unsupported.

Application services must not contain checks shaped like:

- if fake mode, do one thing
- if real mode, do another thing

The correct shape is:

- application services depend on backend protocol concepts
- the application startup or test setup decides which backend implementation is supplied
- service behavior remains semantic and hardware-agnostic

## Fake backend responsibility

A fake backend is responsible for:

- providing safe backend behavior without hardware access
- implementing backend boundary behavior with in-memory state
- modeling supported and unsupported capabilities
- returning clear outcomes for requested behavior
- making service behavior inspectable
- supporting GUI development before real hardware exists
- supporting future service and integration-style tests
- helping reviewers understand expected backend behavior

A fake backend should behave like a controlled simulation of the backend boundary, not like a shell script wrapper.

Fake backends must not become a hidden infrastructure layer.

## What fake backends must not do

Fake backends must not perform real hardware access.

They must not:

- access ACPI interfaces
- access HID devices
- write to `/proc/acpi/call`
- read from `/proc`, `/sys`, `/dev`, or `/dev/mem`
- open `/dev/hidraw*` paths
- construct or transmit hardware payloads
- execute shell commands
- call `sudo`
- use subprocess for hardware control
- contact a privileged helper
- depend on systemd or polkit
- depend on Linux-specific hardware interfaces
- assume a supported laptop is present
- assume hardware state is reachable

A fake backend must run correctly on any machine, without root, without Monster/Clevo hardware, and without a running desktop environment.

Fake backends must not construct or store raw ACPI/HID payloads as if they were production behavior.

Fake backends must not expose generic command execution.

Fake backends must not become a shortcut around the architecture.

## Minimal fake vs stateful fake

There are two useful fake backend styles.

### Minimal fake

A minimal fake returns fixed success, fixed failure, or fixed unsupported responses.

This can be useful for very early smoke tests or simple UI wiring.

However, minimal fakes are limited because they do not prove much about state transitions.

For example, a minimal fake may report that a fan mode request succeeded without remembering what the current fan mode became.

### Stateful fake

A stateful fake maintains internal state that changes in response to backend protocol calls.

For example:

- when an application service requests Maximum fan mode, the fake backend stores that semantic mode
- when an application service reads fan status, the fake backend returns the stored semantic mode
- when an application service sets a keyboard static color, the fake backend stores that RGB color
- when the GUI displays current state, it reflects the stored fake state

Stateful fakes are preferred for this project.

They allow the full use-case flow to be exercised and observed without hardware.

A stateful fake makes it possible to verify that:

- application service calls produce expected state changes
- GUI behavior can reflect backend state
- unsupported capability reporting reaches the GUI as expected
- tests can inspect backend state safely
- read-only flows do not accidentally become write flows

Both minimal and stateful fakes must still satisfy the hardware exclusion rules in this document.

## Deterministic and inspectable state

Fake backend behavior should be deterministic.

Given the same initial fake state and the same application request, the fake backend should produce the same outcome.

Inspectable state is important because it allows future tests and reviewers to answer questions such as:

- Did the application request the expected semantic operation?
- Did unsupported behavior remain unsupported?
- Did the selected profile update expected state?
- Did an RGB request update only the simulated RGB state?
- Did a read-only status request avoid write behavior?
- Did the service avoid hidden hardware assumptions?

Fake backend state should be simple, explicit, and easy to reset.

Fake backend state must remain domain-level or service-level state.

It must not contain raw ACPI, HID, Linux path, shell, firmware, or privileged helper details.

## Unsupported capability modeling

Fake backends must model unsupported capabilities explicitly.

Unknown or unsupported features must not be silently accepted, silently ignored, or silently converted into guessed behavior.

When an application service requests an unsupported operation, the fake backend should return an explicit unsupported result that the application service can surface to the GUI.

Features that must be modeled as unsupported in the initial fake backend design, based on the current hardware research status in `docs/hardware-notes.md`, include:

- keyboard brightness control because payload mapping is UNKNOWN
- keyboard zone control because zone-to-key-id mapping is UNKNOWN
- lightbar arbitrary RGB because payload behavior is UNKNOWN
- lightbar brightness because payload mapping is UNKNOWN
- lightbar effects because payload behavior is UNKNOWN
- fan curve writing unless deliberately enabled by documented fake capability profile and review
- model-specific hardware behavior unless the model behavior is documented
- unsupported sensor readings
- unavailable system status fields

Features that may be modeled as supported with plausible fake state include:

- fan mode switching between known semantic modes
- keyboard-wide static RGB color
- lightbar known prototype behaviors such as red/on/off, only when deliberately enabled in a fake capability profile
- read-only fan or thermal status, only when the status boundary is documented

The fake backend must not claim to support a feature that `docs/hardware-notes.md` marks as UNKNOWN or DANGEROUS.

## Fake backend profiles

A fake backend may later support named fake capability profiles.

Conceptually, fake profiles could represent states such as:

- no supported hardware
- fan mode support only
- keyboard static RGB support only
- lightbar known behavior support only
- read-only system status only
- development profile with several safe simulated capabilities enabled

These are conceptual examples only.

This document does not define fake profile data structures or APIs.

Fake profiles should be used to exercise application behavior safely, not to claim real hardware support.

## Candidate fake backend areas

The following fake backend areas are likely useful later.

These names are conceptual and may be refined during implementation.

### FakeFanBackend

A fake fan backend may later simulate fan-related backend behavior using in-memory state.

Possible conceptual responsibilities:

- store the current semantic fan mode
- return the stored semantic fan mode when status is requested
- reject unsupported fan modes clearly
- simulate fan curve support only when deliberately enabled by a documented fake capability profile
- reject fan curve writing when the fake capability profile does not support it
- provide safe simulated fan status if needed

Rules:

- It must not expose raw SCMD values.
- It must not expose PK04 cases.
- It must not contain ACPI method names.
- It must not contain fan table offsets.
- It must not simulate undocumented hardware payloads.
- It must not claim real fan behavior.

Initial fake state should reflect a safe semantic default, such as Automatic fan mode.

### FakeRgbBackend

A fake RGB backend may later simulate RGB-related backend behavior using in-memory state.

Possible conceptual responsibilities:

- store the current keyboard static color
- store the current supported lightbar state
- return stored values when status is requested
- report keyboard brightness as unsupported
- report keyboard zones as unsupported
- report lightbar arbitrary RGB as unsupported
- report lightbar brightness as unsupported
- report lightbar effects as unsupported

Rules:

- It must not expose HID report bytes.
- It must not contain `/dev/hidraw*` paths.
- It must not contain raw keyboard key IDs as protocol details.
- It must not contain lightbar payload bytes.
- It must not simulate unknown lightbar arbitrary RGB payloads.
- It must not claim real RGB behavior.

Initial fake state should reflect a safe semantic default, such as all lighting off or a known safe static color.

### FakeSystemStatusBackend or FakeSystemProbeBackend

A fake system status or probe backend may later simulate read-only status behavior using safe in-memory state.

Possible conceptual responsibilities:

- provide safe simulated status values
- return plausible fixed or configurable temperature readings
- return plausible fixed or configurable fan speed readings
- report unavailable readings clearly
- represent supported and unsupported status fields
- avoid implying any specific hardware sensor path

Current boundary:

- This backend area should remain read-only.
- It must not become a hidden fan or RGB control path.
- It must not read real system paths.
- It must not execute hardware or sensor commands.
- It must not imply that every sensor exists on every machine.

Possible naming:

- `FakeSystemStatusBackend` if the focus is user-facing status.
- `FakeSystemProbeBackend` if the focus is capability/system probing.

The final naming should follow the service boundary chosen later.

## Fake backend and GUI development

Fake backends allow GUI development without hardware risk.

The GUI should be able to call application services and display outcomes while the service uses fake backend behavior underneath.

This supports early UI work such as:

- profile selection flows
- unsupported feature messages
- RGB color selection flows
- read-only status panels
- error and unavailable-state display

The GUI must still not know whether the backend is fake or real.

The GUI must still not access hardware directly.

## Fake backend and application service development

Application services should be designed so they can run against fake backends first.

This allows service behavior to be reviewed before infrastructure exists.

A service should be able to coordinate semantic operations and receive clear backend outcomes without knowing how real hardware will eventually work.

This helps prevent application services from depending on concrete infrastructure details.

## Fake backend and future tests

Fake backends should support future tests.

They can help test:

- service behavior
- unsupported capability handling
- safe state transitions
- profile coordination
- read-only status behavior
- GUI-service integration at a safe boundary
- error display and unavailable states

Fake backend tests must not require root.

Fake backend tests must not require Monster/Clevo hardware.

Fake backend tests must not require a desktop environment unless specifically testing GUI behavior.

Fake backend tests must not require ACPI, HID, privileged helper access, or real sensor availability.

## Fake backend and real backend coexistence

The project should support running with either a fake backend or a real backend through dependency injection or configuration, not through conditional logic scattered across application services.

Application services must not contain checks shaped like:

- if fake mode, do this
- else, do that

The correct approach is:

- application services always depend on backend protocol concepts
- the concrete backend is supplied by startup configuration, composition root, or test setup
- application service code does not change between fake and real backends

## Transition to real backends

A feature should not move from fake backend behavior to real infrastructure backend behavior until the relevant criteria are satisfied.

Required preparation includes:

- hardware behavior documented in `docs/hardware-notes.md` with DEFINITE status where relevant
- domain boundaries documented in `docs/domain-models.md`
- application service boundaries documented in `docs/application-services.md`
- backend protocol boundaries documented before implementation
- fake backend behavior implemented and reviewed
- unsupported behavior modeled explicitly
- review or test safety checks prepared
- hardware safety review performed
- no unknown ACPI/HID behavior treated as supported
- no direct shell prototype promotion
- no generic privileged command executor

Real infrastructure backends should be added only after the semantic contract is already clear.

A real backend must not be introduced as a large mixed commit alongside GUI, domain, service, or fake backend changes.

## Backend boundary review

When a future change touches backend code, review should check:

- whether the backend accesses hardware directly
- whether the backend executes shell commands
- whether the backend opens device files
- whether the backend constructs ACPI or HID payloads
- whether the fake backend claims to support a feature that is UNKNOWN in `docs/hardware-notes.md`
- whether the backend bypasses the application service layer
- whether the backend exposes generic command execution
- whether fake state remains semantic and inspectable
- whether unsupported capabilities are explicit
- whether the backend change is small and reviewable

Boundary violations in backend code are blockers, not warnings.

## Documentation-to-code path

Before a fake backend is implemented, the backend protocol concept it implements should be defined.

Before a backend protocol is implemented, the application service use cases that depend on it should be documented.

Before real backend code is added, fake backend behavior should exist first.

Before hardware-facing code is added, the behavior must also be documented in `docs/hardware-notes.md`.

The preferred path is:

The preferred path is:

1. document domain concepts
2. define application service boundaries
3. document fake backend strategy
4. define backend protocol boundaries
5. implement domain models
6. define and implement backend protocols
7. implement fake backends
8. implement application services against fake backends
9. later implement reviewed infrastructure backends
10. later implement privileged helper where needed

Do not implement a real infrastructure backend before the fake backend boundary is proven correct.
