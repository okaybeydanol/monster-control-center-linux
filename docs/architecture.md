# Architecture

Monster Control Center Linux is designed as a layered desktop application.

The main rule is simple:

> The GUI must never perform privileged hardware operations directly.

Hardware access is isolated behind explicit application services, infrastructure backends, and eventually a privileged helper process.

## Architecture overview

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

Each layer has a clear responsibility.

## GUI Layer

The GUI layer contains the PySide6 desktop interface.

Its responsibilities are:

* Render windows, pages, tabs, buttons, inputs, and status views.
* Collect user intent.
* Display application state.
* Call application services.

The GUI layer must not:

* Execute shell commands directly.
* Call ACPI methods directly.
* Open HID devices directly.
* Use sudo directly.
* Write to system files directly.
* Contain hardware protocol logic.

Correct example:

```text
User clicks "Performance"
→ GUI calls ProfileService.apply_profile(ProfileName.PERFORMANCE)
```

Wrong example:

```text
User clicks "Performance"
→ GUI executes sudo command
→ GUI writes ACPI payload
```

The GUI is a presentation layer, not a hardware controller.

## Application Layer

The application layer contains use cases.

It answers questions such as:

* What should happen when the user selects Performance mode?
* What should happen when the user applies a fan curve?
* What should happen when the user changes keyboard RGB color?
* What should happen when the app asks for hardware status?

Its responsibilities are:

* Coordinate domain models and infrastructure backends.
* Enforce use-case flow.
* Keep GUI logic separate from hardware logic.
* Provide readable methods for the GUI.

Example use cases:

```text
apply_performance_profile()
apply_entertainment_profile()
apply_quiet_profile()
set_keyboard_static_color()
set_lightbar_static_color()
read_fan_status()
```

The application layer should not know low-level ACPI or HID byte details unless those details are represented through infrastructure interfaces.

## Domain Layer

The domain layer contains typed business objects.

It describes the concepts of the application without depending on Linux, ACPI, HID, PySide6, or shell scripts.

Examples:

```text
FanMode
FanCurvePoint
FanCurve
FanProfile
RgbColor
RgbBrightness
KeyboardZone
LightbarProfile
DeviceCapability
```

The domain layer should use Pydantic models where validation matters.

Examples of validation:

* Fan temperature must be within a safe range.
* Fan duty percent must be between 0 and 100.
* RGB values must be between 0 and 255.
* Brightness must be within supported device limits.
* Fan curve points must be sorted by temperature.
* Fan curve points must not contain duplicate temperatures.

The domain layer must not:

* Import PySide6.
* Execute commands.
* Read `/proc`, `/sys`, `/dev`, or `/dev/mem`.
* Know about `acpi_call`.
* Know about HID device paths.
* Know about systemd, polkit, or Debian packaging.

The domain layer should be easy to test.

## Infrastructure Layer

The infrastructure layer contains real system integrations.

Examples:

```text
ACPI backend
HID backend
System probe backend
Configuration storage
Privileged helper client
Linux sensor reader
```

Its responsibilities are:

* Implement hardware access boundaries.
* Translate domain/application requests into low-level operations.
* Hide operating-system details from the application layer.
* Provide safe, testable interfaces.

Examples:

```text
AcpiFanBackend.apply_curve(profile)
AcpiFanBackend.set_auto_mode()
AcpiFanBackend.set_full_speed_mode()
HidRgbBackend.set_keyboard_static_color(color, brightness)
HidRgbBackend.set_lightbar_static_color(color, brightness)
```

The infrastructure layer may know about:

* ACPI methods.
* HID payloads.
* Device paths.
* Kernel interfaces.
* `acpi_call`.
* systemd.
* polkit.
* file permissions.

The infrastructure layer must still avoid unsafe shortcuts. Dangerous operations should be explicit and documented.

## Privileged Helper

Some operations require elevated permissions.

The long-term design should use a dedicated privileged helper instead of allowing the GUI to run commands as root.

The privileged helper is responsible for:

* Receiving a small set of allowed requests.
* Validating request payloads.
* Performing privileged hardware operations.
* Returning structured results.
* Rejecting unsupported or unsafe operations.

The helper should not expose a generic command execution API.

Correct design:

```text
apply_fan_profile(profile_name)
set_fan_mode(mode)
set_keyboard_rgb_static(color, brightness)
set_lightbar_rgb_static(color, brightness)
```

Wrong design:

```text
run_shell_command(command)
write_any_acpi_payload(payload)
write_any_hid_payload(payload)
```

The helper boundary exists to prevent accidental or malicious misuse.

## Dependency direction

Dependencies should point inward.

Allowed examples:

```text
GUI imports application services.
Application imports domain models and backend protocols.
Infrastructure imports domain models and implements backend protocols.
Tests import domain/application/infrastructure modules.
```

Disallowed examples:

```text
Domain imports GUI.
Domain imports infrastructure.
Application imports PySide6 widgets.
GUI imports low-level ACPI payload builders.
GUI imports HID write code.
```

The domain layer should remain the most stable part of the system.

## Backend protocols

Application services should depend on protocols/interfaces, not concrete hardware implementations.

Example:

```text
FanService
  depends on FanBackend protocol

AcpiFanBackend
  implements FanBackend protocol
```

This allows the project to support:

* Mock backends for development.
* Fake backends for tests.
* Real ACPI backends for supported hardware.
* Future hardware-specific backends.

This also lets the GUI be developed before all hardware operations are finalized.

## Configuration and profiles

Built-in profiles should be represented as data, not hardcoded inside GUI widgets.

Examples:

```text
Performance profile
Entertainment profile
Quiet profile
Keyboard RGB defaults
Light bar RGB defaults
```

Profile data may initially live in repository-managed files such as:

```text
src/monster_control_center/data/fan_profiles.yaml
src/monster_control_center/data/rgb_profiles.yaml
```

The application layer can load and validate those files using domain models.

## Hardware safety rules

Hardware behavior must be documented before it is converted into production code.

Rules:

* Do not guess ACPI payloads.
* Do not guess HID payloads.
* Do not convert experimental scripts into production code blindly.
* Do not hide dangerous behavior behind generic helper functions.
* Do not write root-level code from the GUI.
* Do not mix hardware discovery scripts with application runtime code.
* Do not implement a feature until the expected hardware behavior is documented.

Experimental scripts may exist during research, but production code must be typed, bounded, reviewed, and documented.

## Testing strategy

The project should test from the inside outward.

Initial priority:

```text
Domain model validation
Application service behavior with fake backends
Profile loading and validation
Infrastructure payload builders
GUI smoke tests
Privileged helper request validation
```

Hardware integration tests should be separate from normal unit tests because they depend on a supported laptop and elevated permissions.

## Commit discipline

Each commit should be small and reviewable.

Good examples:

```text
docs(architecture): define application layers
feat(domain): add fan mode enums
feat(domain): add fan curve point model
test(domain): validate fan curve point bounds
feat(fan): define fan backend protocol
feat(gui): add main window shell
```

Bad examples:

```text
add everything
wip
fix stuff
fan rgb gui backend all together
```

A commit should usually change one concept at a time.

## Current design status

The architecture is intentionally defined before implementation.

At this stage, the project should avoid writing real hardware control code until the domain model, service boundaries, and safety rules are in place.
