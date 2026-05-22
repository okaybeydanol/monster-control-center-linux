# Domain Models

This document outlines the intended domain model boundaries for Monster Control Center Linux.

It is documentation only.

This file does not implement Python code, Pydantic models, enums, tests, backend protocols, fake backends, GUI code, ACPI/HID logic, privileged helpers, shell wrappers, or hardware payloads.

The goal is to define what the domain layer should represent before implementation begins.

## Purpose

The domain layer describes the core concepts of the application in a typed, validated, hardware-agnostic form.

It should answer questions such as:

- What is a profile?
- What is a fan mode?
- What is a fan curve?
- What is a valid RGB color?
- What capabilities does a device expose?
- What values are allowed before a request leaves the application/domain boundary?

The domain layer should make invalid application states difficult to represent.

It should describe what the application means, not how a specific ACPI, HID, shell, Linux, or privileged operation is performed.

## Relationship to canonical docs

This document complements:

- `docs/architecture.md` for layer boundaries and dependency direction
- `docs/ai-workflow.md` for AI review expectations
- `docs/hardware-notes.md` for hardware research boundaries and safety status

This file must not duplicate hardware payload details from `docs/hardware-notes.md`.

The domain layer should consume only documented, reviewed concepts. It must not become a hardware research dump.

## Domain layer responsibility

The domain layer is responsible for:

- named application concepts
- typed value objects
- validation rules
- safe profile data shapes
- capability descriptions
- safe state representation
- clear errors for unsupported or invalid values

The domain layer should be stable, testable, and independent from the desktop UI, operating system, hardware access, and privileged helper implementation.

Examples of domain responsibility:

- defining known product-facing profile names
- representing fan modes as semantic values
- representing fan curve points
- validating fan duty percentage ranges
- validating RGB component ranges
- representing whether a feature is supported
- preventing duplicate fan curve temperatures
- preventing unsupported brightness or lighting behavior from being silently accepted

Validation is not optional. It is the domain layer's primary contribution to hardware safety.

## What the domain layer must not know

The domain layer must not know about:

- PySide6
- windows, widgets, tabs, or buttons
- Linux file paths
- `/proc`
- `/sys`
- `/dev`
- `/dev/mem`
- `acpi_call`
- HID device paths
- HID report bytes
- ACPI method names
- ACPI package shapes
- AML tables
- shell commands
- `sudo`
- subprocess calls
- systemd
- polkit
- Debian packaging
- privileged helper transport details

The domain layer must not perform I/O.

It must not read hardware state.

It must not write hardware state.

It must not execute commands.

It must be fully testable without hardware, without root, and without a running desktop environment.

## Design principles

Domain models should follow these principles:

- Model product concepts, not implementation shortcuts.
- Use semantic names instead of raw hardware values.
- Reject invalid values early.
- Keep hardware-specific translation outside the domain layer.
- Prefer explicit unsupported states over silent fallback behavior.
- Make future hardware support possible without weakening validation.
- Keep models small and focused.
- Keep validation close to the value it protects.
- Avoid untyped dictionaries for core concepts.
- Avoid leaking prototype shell naming into product concepts.
- Avoid exposing unknown firmware or prototype values as stable product concepts.

The domain layer expresses what the user wants and what the application considers valid. Infrastructure later translates reviewed domain concepts into hardware-specific behavior.

## Pydantic v2 policy

Pydantic v2 is the expected validation tool for domain models where structured validation is needed.

This document does not add Pydantic implementation.

Future Pydantic usage should be introduced in small, reviewable commits.

Expected future uses include:

- validating fan curve point values
- validating fan curve ordering
- validating RGB component ranges
- validating built-in profile data
- validating capability declarations
- rejecting unsupported or unknown feature requests

Pydantic models must not contain hardware access logic.

Pydantic validators must not call infrastructure code.

Pydantic validators must not read `/proc`, `/sys`, `/dev`, HID devices, ACPI methods, shell commands, or privileged helpers.

## Initial candidate domain concepts

### ProfileKind

Represents the high-level user-facing profile selected by the user.

Initial expected product-facing values:

- Performance
- Entertainment
- Quiet

Rules:

- Use product-facing names.
- Do not use prototype names such as `LLM Mode`.
- Do not encode ACPI or HID values here.
- Do not expose raw firmware modes here.
- Do not add `Custom` here until the relationship between product profile, fan mode, and custom fan curve is documented.

`Custom Profile` is currently better treated as a fan mode or fan profile concept, not as a top-level `ProfileKind`, until service boundaries are documented.

### FanMode

Represents a semantic fan control mode.

Possible initial concepts:

- Automatic
- Maximum
- Quiet
- Custom Profile

Rules:

- FanMode must not expose raw SCMD values.
- Raw ACPI mode integers belong in infrastructure translation, not domain.
- Unknown firmware-observed modes must not be added just because they exist.
- Additional modes require documentation and review first.
- Product naming should remain user-facing and not leak prototype script names.

### FanCurvePoint

Represents one point in a fan curve.

Expected conceptual fields:

- temperature
- duty percentage
- optional RPM value only if the product design needs a read/display concept

Rules:

- Temperature must be in a documented supported range.
- Duty percentage must be between 0 and 100.
- RPM must not be guessed.
- RPM must not be treated as a required write target until the FCD1 versus Windows reset RPM ambiguity is resolved.
- A point must not contain ACPI table offsets, PK04 fields, or raw buffer positions.

### FanCurve

Represents an ordered collection of fan curve points.

Rules:

- Points must be sorted by temperature.
- Duplicate temperatures should be rejected.
- Empty curves should be rejected.
- Curve length should be validated according to documented product constraints.
- Hardware-specific table layout must not appear in this model.
- The model should describe the curve as the product understands it, not as PK04 stores it.

### FanProfile

Represents a named fan behavior profile.

Expected conceptual fields may later include:

- profile name or profile kind
- fan mode
- optional fan curve
- metadata needed for user-facing display

Rules:

- Built-in profiles should be data-driven and validated.
- A profile must not contain raw ACPI method names.
- A profile must not contain raw SCMD values.
- A profile must not contain PK04 case numbers.
- A profile must not describe recovery procedures.
- FanProfile should be implemented after FanMode, FanCurve, and application service boundaries are stable.

### RgbColor

Represents an RGB color.

Expected conceptual fields:

- red
- green
- blue

Rules:

- Each component must be between 0 and 255.
- Named colors may exist as convenience values later.
- RGB values are domain-safe only as color components, not as HID report bytes.
- The domain model must not contain HID command prefixes, report IDs, report lengths, keyboard key IDs, or lightbar payload bytes.

### RgbBrightness

Represents brightness for an RGB-capable device.

Current status:

- Brightness is a valid product concept.
- Exact supported brightness values and payload mapping are not yet production-confirmed.
- Keyboard brightness mapping is UNKNOWN.
- Lightbar brightness mapping is UNKNOWN.

Rules:

- Brightness must not be implemented as arbitrary integer guessing.
- Do not claim an `0..4` range as production-supported until the mapping is documented and reviewed.
- If brightness mapping is unknown for a device, the model or capability layer must represent that it is unsupported.
- Do not assume Windows UI brightness levels are production-safe until mapped and documented.

### KeyboardZone

Represents a semantic keyboard lighting zone.

Possible future concepts:

- whole keyboard
- left zone
- middle zone
- right zone
- per-key

Current status:

- Whole-keyboard static color is the safest early product concept.
- Exact zone-to-key-id mapping is UNKNOWN.
- Per-key protocol details are hardware-specific.

Rules:

- Do not invent key groups.
- Do not expose zone concepts until supported by documented behavior.
- Per-key IDs are hardware detail and must not leak into this model unless represented through a reviewed abstraction.
- KeyboardZone should be deferred until zone mapping is documented.

### LightbarProfile

Represents lightbar lighting behavior.

Possible future concepts:

- off
- static red
- static color
- effect profile

Current status:

- Lightbar red/on/off behavior is documented as prototype evidence.
- Arbitrary lightbar RGB is UNKNOWN.
- Lightbar effects are UNKNOWN.
- Lightbar brightness mapping is UNKNOWN.

Rules:

- Do not model arbitrary lightbar RGB as supported until exact payload behavior is documented.
- Do not model effects as supported until exact behavior is documented.
- Do not model brightness as supported until exact behavior is documented.
- The model should be capability-aware.
- LightbarProfile should be deferred until the hardware behavior is better understood.

### DeviceCapability

Represents what the current hardware/device supports.

Expected future concepts may include:

- supports fan mode switching
- supports fan curve readback
- supports fan curve writing
- supports keyboard static RGB
- supports keyboard brightness
- supports keyboard zones
- supports lightbar red/on/off
- supports lightbar arbitrary RGB
- supports lightbar effects

Rules:

- Capabilities must be explicit.
- Unknown must not be treated as supported.
- Unsupported hardware should produce clear application behavior.
- Capability detection belongs outside the domain layer.
- Capability representation can be a domain concept.
- A capability model must not contain raw device paths, USB/HID payloads, ACPI method names, shell commands, or privileged helper details.
- DeviceCapability should be deferred until the hardware detection and fake backend strategy are documented.

## v0 candidate models

The first domain implementation phase should stay small, safe, and hardware-agnostic.

Good candidates for the first implementation phase:

- ProfileKind
- FanMode
- FanCurvePoint
- FanCurve
- RgbColor

These concepts are useful because they have clear semantics, clear validation boundaries, and do not require unknown hardware behavior to be resolved first.

The initial implementation should focus on validation and semantics, not hardware execution.

## Deferred concepts

The following should be deferred until more design or hardware behavior is documented:

| Concept | Reason |
| --- | --- |
| RgbBrightness | brightness payload mapping is UNKNOWN |
| KeyboardZone | zone-to-key-id mapping is UNKNOWN |
| LightbarProfile | arbitrary RGB, brightness, and effects are UNKNOWN |
| DeviceCapability | hardware detection and fake backend strategy are not documented yet |
| FanProfile | FanCurve and application service boundaries should be stable first |

Other deferred concepts:

- detailed keyboard zone mapping
- per-key keyboard models
- arbitrary lightbar RGB profiles
- RGB effect models
- advanced fan curve editor models
- model-specific hardware identity rules
- ACPI payload models
- HID payload models
- privileged helper request/response models
- low-level sensor models tied to Linux paths
- packaging or system service models

Some of these may eventually exist, but not in the first domain implementation phase.

ACPI and HID payload models may belong in infrastructure, not domain.

Privileged helper request models may belong at a helper boundary, not in the core domain layer, depending on final design.

## Validation expectations

Future domain validation should include:

- known product profile names only
- known fan modes only
- fan duty percentage between 0 and 100
- fan temperature within a documented supported range
- fan curve points sorted by temperature
- no duplicate fan curve temperatures
- no empty fan curves
- RGB components between 0 and 255
- no unsupported brightness values
- no unsupported RGB capabilities
- no silent fallback from unsupported to supported behavior
- no unknown hardware mode exposed as stable product behavior

Validation should fail clearly.

Validation must not perform hardware access.

Validation must not call infrastructure services.

Validation must not mutate system state.

## Hardware payload exclusion rule

Domain models must not contain hardware payloads.

Do not put the following in domain models:

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
- shell command strings
- `sudo` commands
- systemd unit details
- polkit policy details

Hardware payload construction belongs behind infrastructure boundaries after documentation, protocols, fake backends, and review.

## ACPI/HID byte boundary

ACPI and HID bytes are not domain language.

The domain may say:

- apply Performance profile
- use Automatic fan mode
- use Maximum fan mode
- use Quiet fan mode
- set keyboard static color to RGB value
- report that lightbar arbitrary RGB is unsupported
- report that keyboard brightness is unsupported

The domain must not say:

- call this ACPI method with this integer
- write this PK04 case
- send this HID feature report
- use this `/dev/hidraw` path
- write this byte sequence
- load this AML table
- write this shell command

Translation from domain concepts to ACPI/HID behavior belongs in infrastructure and privileged helper boundaries.

## Documentation-to-code path

Before domain code is added, the intended model should be documented here.

Before hardware-facing code is added, the behavior must also be documented in `docs/hardware-notes.md`.

The preferred path is:

1. document the concept
2. define domain model boundary
3. add typed domain model
4. add validation tests
5. define application service use case
6. define backend protocol
7. add fake/mock backend
8. later add reviewed infrastructure implementation
9. later add privileged helper where needed

Do not skip from hardware research directly to runtime code.
