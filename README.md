# Monster Control Center Linux

Open-source Linux control center for Monster/Clevo laptops.

This project aims to provide a native Linux desktop application for controlling laptop performance profiles, fan behavior, keyboard RGB lighting, and light bar RGB features on supported Monster/Clevo-based devices.

> Project status: pre-alpha.  
> The project is currently in early architecture and hardware research stages.

## Goals

- Provide a native Linux desktop control center for supported Monster/Clevo laptops.
- Expose performance profiles such as Performance, Entertainment, and Quiet.
- Support fan mode switching and custom fan curve profiles.
- Support keyboard RGB and light bar RGB controls.
- Keep hardware access isolated from the GUI.
- Build the project with a professional, typed, testable Python architecture.
- Package the application for Linux users, starting with Debian-based distributions.

## Non-goals

- This project is not a Windows Control Center replacement.
- This project does not aim to support every laptop model at the beginning.
- The GUI will not execute privileged hardware commands directly.
- Shell scripts are not the final application architecture.
- Unsafe hardware access will not be hidden behind convenience abstractions.

## Planned application architecture

The project will follow a layered architecture:

```text
GUI Layer
  PySide6 desktop interface.

Application Layer
  Use cases such as applying a performance profile or changing RGB color.

Domain Layer
  Typed Pydantic models such as fan profiles, fan curves, RGB colors, and device capabilities.

Infrastructure Layer
  ACPI, HID, system probing, configuration files, and operating-system integration.

Privileged Helper
  A dedicated boundary for operations requiring elevated permissions.
```

The GUI must never directly perform privileged hardware operations. It should call application services, and those services should delegate hardware work to infrastructure backends or a privileged helper.

## Planned profile model

The initial user-facing profile tabs are:

- Performance
- Entertainment
- Quiet

The exact backend behavior for each profile will be implemented gradually and verified against real hardware behavior.

## Planned RGB scope

Initial RGB support will focus on:

- Keyboard static RGB color
- Keyboard brightness
- Light bar static RGB color
- Light bar brightness

Additional RGB effects such as breathing, wave, slide, reactive, boot effect, and sleep timer may be added later after the base implementation is stable.

## Development approach

This is both a real open-source hardware-control project and a learning project.

The project will be developed in small, reviewable commits. Each commit should have a focused purpose, such as adding metadata, documenting architecture, defining one domain model, adding one test group, or introducing one GUI shell component.

Example commit style:

```text
chore(project): add pyproject metadata
docs(readme): add project overview
docs(architecture): define application layers
feat(domain): add fan mode enums
test(domain): validate fan point bounds
feat(gui): add main window shell
```

## Safety principles

Hardware control code must be written carefully.

Important rules:

- Do not guess ACPI or HID payloads.
- Do not mix experimental scripts with production code.
- Do not let the GUI call privileged commands directly.
- Do not hide unsafe operations behind generic helper functions.
- Keep hardware-specific behavior documented.
- Prefer typed models, explicit boundaries, and tests.

## Technology choices

Planned core technologies:

- Python
- PySide6
- Pydantic v2
- Ruff
- Mypy
- Pytest
- Hatchling

## License

MIT
