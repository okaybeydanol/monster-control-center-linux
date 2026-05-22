# Hardware Notes

This document records the current hardware research boundaries for Monster Control Center Linux.

It is documentation only.

It does not implement fan control, RGB control, ACPI access, HID access, a privileged helper, runtime shell wrappers, or new hardware experiments.

The current commit scope is:

- document what is known
- document what is unknown
- document what must not be repeated
- keep hardware research separate from production application architecture

Prototype shell commands and hardware experiments are evidence. They are not the final runtime application architecture.

## Canonical project boundaries

The canonical architecture remains:

- GUI Layer
- Application Layer
- Domain Layer
- Infrastructure Layer
- Privileged Helper / Operating System / Hardware

The GUI must never perform privileged hardware operations directly.

The GUI must not:

- run `sudo`
- execute shell commands for hardware control
- write `/proc/acpi/call`
- open `/dev/mem`
- open `/dev/hidraw*`
- load AML tables
- send HID feature reports
- contain raw ACPI payload bytes
- contain raw HID payload bytes
- contain hardware protocol logic

The production path for hardware behavior is:

1. research fact
2. human-facing documentation
3. typed domain model
4. application service
5. backend protocol
6. infrastructure backend
7. privileged helper operation
8. tests and review

Hardware behavior must be documented before it is converted into production code.

## Safety labels

This document uses the following labels:

- **DEFINITE**: confirmed by prior working research.
- **INFERENCE**: likely based on observed behavior, but not enough for unsupported production claims.
- **UNKNOWN**: not established; do not guess.
- **DANGEROUS**: do not repeat without a separate isolated safety plan.

No ACPI payload, HID byte, fan offset, RGB effect packet, or recovery behavior should be invented.

## Current safety status

The project is not ready for production hardware writes.

Before any real fan or RGB implementation exists, the project still needs:

- typed domain models
- backend protocols
- fake/mock backends
- explicit unsupported-hardware behavior
- whitelisted infrastructure operations
- recovery documentation
- hardware safety review
- tests where possible

No hardware write should happen automatically at application startup.

No old shell prototype should be wrapped as the application backend.

No generic privileged command executor should be introduced.

## Known fan mode research

### Fan mode method

Status: **DEFINITE for prior prototype research**

Known method:

- `\_SB.WMI.SCMD`

Known fan mode command family:

- `SCMD 0x79`

Known prototype argument shape:

- `arg = 0x01000000 | mode`

Known prototype call shape:

- `\_SB.WMI.SCMD 0 0x79 0x01000000`
- `\_SB.WMI.SCMD 0 0x79 0x01000001`
- `\_SB.WMI.SCMD 0 0x79 0x01000002`
- `\_SB.WMI.SCMD 0 0x79 0x01000008`
- `\_SB.WMI.SCMD 0 0x79 0x01000009`

These are research facts, not production API design.

### Known SCMD 0x79 mode map

Status: **DEFINITE for prior prototype research**

| Prototype name | Mode value | Production status |
| --- | ---: | --- |
| `fan-auto` | `0` | candidate for Automatic |
| `fan-full` | `1` | candidate for Maximum |
| `fan-mid` | `2` | known prototype value; do not expose until product meaning is finalized |
| `fan-quiet` | `8` | candidate for Quiet |
| `fan-custom` | `9` | candidate for Custom Profile |

Recommended initial production concepts:

| Product concept | SCMD mode |
| --- | ---: |
| Automatic | `0` |
| Maximum | `1` |
| Quiet | `8` |
| Custom Profile | `9` |

The GUI profile names should be:

- Performance
- Entertainment
- Quiet

Do not use `LLM Mode` as a GUI profile name. Use `Performance` or `Performance Mode`.

### Fan mode readback

Status: **DEFINITE for prior prototype research**

Known readback method:

- `\_SB.PC00.LPCB.EC.ECMD`

Known D7 readback command shape:

- `{0x02,0x80,0xD7,0x00,0x00,0x00,0x00,0x00}`

Observed D7 examples:

| Observed state | D7 readback |
| --- | --- |
| Automatic | `{0x00, 0x00, 0x00, 0x02, 0x00, 0x00}` |
| Maximum/full | `{0x00, 0x00, 0x00, 0x10, 0x00, 0x00}` |
| Custom | `{0x00, 0x00, 0x01, 0x00, 0x00, 0x00}` |

This readback evidence is useful for later validation, but read-only hardware access is still not GUI-safe by default.

### Additional DSDT-observed modes

Status: **UNKNOWN**

Additional SCMD values were observed in DSDT logic:

| Mode value | Observed D7 bit/value |
| ---: | ---: |
| `5` | `0x01` |
| `6` | `0x04` |
| `7` | `0x20` |

Do not expose these in production UI until deliberately verified.

## Known fan curve / PK04 research

### PK04 method

Status: **DEFINITE for prior prototype research**

Known method:

- `\_SB.DCHU.PK04`

Known signature:

- `Method (PK04, 3, Serialized)`

Known behavior:

- `Arg2[0]` is dereferenced into an internal buffer.
- `R000` selects the PK04 case.
- Input data fields are read from specific buffer regions.
- Output data is returned through an internal output buffer.

Direct text-based `acpi_call` package/buffer attempts failed with:

- `AE_AML_OPERAND_TYPE`

Conclusion:

- PK04 expects ACPI-native Buffer/Package objects in a shape that direct text `acpi_call` did not reliably construct.

### PK04 case map

Status: **DEFINITE for cases 0x28-0x2B**

| PK04 case | Meaning | Status |
| ---: | --- | --- |
| `0x28` | read fan block A | **DEFINITE** |
| `0x29` | write fan block A | **DEFINITE** |
| `0x2A` | read fan block B | **DEFINITE** |
| `0x2B` | write fan block B | **DEFINITE** |
| `0x2C` | not safely understood | **UNKNOWN / DANGEROUS** |

Production rule:

- Do not use PK04 case `0x2C` until it has separate research, safety review, and a documented recovery plan.

### Dynamic SSDT wrapper research

Status: **DEFINITE for prior prototype research**

Dynamic SSDT wrappers worked because they created ACPI-native Buffer and Package objects before calling PK04.

Important boundary:

- Dynamic table loading is not automatically production-safe.
- Do not copy AML wrapper logic directly into runtime application code.
- If this approach is ever used, it must be hidden behind a narrow, reviewed backend/helper boundary.

## Fan curve prototype notes

### FCD0

Status: **DANGEROUS / DO NOT REPEAT**

FCD0 is historical failure evidence only.

It must not be used as production restore logic.

An early default restore attempt used the wrong table/source split and produced nonsensical decoded fan data, including examples such as:

- `20°C` entries
- `200%` duty
- `9020 RPM`
- `20564 RPM`
- `255°C`

### FCD1

Status: **CONTROLLED BASELINE / DEFAULT RESTORE PROTOTYPE**

FCD1 is useful evidence, but it is not final production restore logic.

Known expected regions from prior validation:

| Region | Values |
| --- | --- |
| A duty | `[0, 41, 46, 51, 61, 71, 76, 81, 86, 91]` |
| A temp | `[40, 45, 50, 60, 70, 75, 80, 85, 90, 100]` |
| B duty | `[0, 36, 46, 51, 61, 71, 76, 81, 86, 91]` |
| B temp | `[35, 45, 50, 60, 70, 75, 80, 85, 90, 100]` |

Important ambiguity:

- FCD1 baseline/default restore decoded max RPM as `4900`.
- Windows Control Center reset clean curve decoded max RPM as `5600`.

Do not resolve this by guessing.

Keep this ambiguity documented until deliberately tested.

### FCL2

Status: **WORKING AGGRESSIVE / PERFORMANCE CURVE PROTOTYPE**

Production name:

- `Performance Mode`

Do not use:

- `LLM Mode`

Known FCL2 duty values:

| Region | Values |
| --- | --- |
| A duty | `[0, 50, 60, 70, 78, 85, 90, 95, 100, 100]` |
| B duty | `[0, 45, 60, 70, 78, 85, 90, 95, 100, 100]` |

Known FCL2 RPM table:

| Point | RPM |
| ---: | ---: |
| 0 | `0` |
| 1 | `1680` |
| 2 | `2240` |
| 3 | `2520` |
| 4 | `2800` |
| 5 | `3360` |
| 6 | `3920` |
| 7 | `4480` |
| 8 | `5040` |
| 9 | `5600` |

Known FCL2 decoded curve:

| Point | Temperature | Duty | RPM |
| ---: | ---: | ---: | ---: |
| 0 | `35°C` | `0%` | `0` |
| 1 | `45°C` | `45%` | `1680` |
| 2 | `50°C` | `60%` | `2240` |
| 3 | `60°C` | `70%` | `2520` |
| 4 | `70°C` | `78%` | `2800` |
| 5 | `75°C` | `85%` | `3360` |
| 6 | `80°C` | `90%` | `3920` |
| 7 | `85°C` | `95%` | `4480` |
| 8 | `90°C` | `100%` | `5040` |
| 9 | `100°C` | `100%` | `5600` |

This is prototype evidence only. It does not authorize production fan curve writes yet.

## Known RGB / HID research

### Keyboard RGB device

Status: **DEFINITE for prior prototype static RGB research**

Known USB/HID identity:

- vendor/product: `048d:8910`
- observed name: `ITE Device(829x)`

Prototype fallback path:

- `/dev/hidraw6`

Production rule:

- Do not hard-code `/dev/hidraw6`.
- Production must discover and validate the target device safely.

Known keyboard RGB report shape:

- `[0xcc, 0x01, key_id, R, G, B, 0x00]`

Human-readable shape:

- `cc 01 key R G B 00`

Known meaning:

| Byte | Meaning |
| --- | --- |
| `0xcc` | report/command family |
| `0x01` | per-key color command |
| `key_id` | keyboard key identifier |
| `R` | red component, `0..255` |
| `G` | green component, `0..255` |
| `B` | blue component, `0..255` |
| `0x00` | flag byte used by prototype |

Known named prototype colors:

| Name | RGB |
| --- | --- |
| red | `ff 00 00` |
| green | `00 ff 00` |
| blue | `00 00 ff` |
| white | `ff ff ff` |
| off / black | `00 00 00` |

Known behavior:

- A list of many keyboard key IDs exists.
- The prototype looped over all known key IDs and sent the same RGB color to each key.
- This is enough evidence for later keyboard-wide static RGB design.

Current status:

| Feature | Status |
| --- | --- |
| keyboard static RGB | **DEFINITE for prototype** |
| keyboard arbitrary RGB | **INFERENCE from payload shape; must validate `R/G/B` as `0..255`** |
| keyboard brightness `0..4` | **UNKNOWN** |
| keyboard left/middle/right zone mapping | **UNKNOWN** |
| keyboard effects | **UNKNOWN** |

### Lightbar RGB device

Status: **DEFINITE for prior prototype red/on/off research**

Known USB/HID identity:

- vendor/product: `048d:8911`
- observed name: `ITE Device(8911)`

Prototype fallback path:

- `/dev/hidraw1`

Production rule:

- Do not hard-code `/dev/hidraw1`.
- Production must discover and validate the target device safely.

Known report length:

- 17-byte padded HID feature report

Known red/on payloads:

- `cd e2 03 02 00 00 00 00 00 00 00 00 00 00 00 00 00`
- `cd e2 02 05 00 00 00 00 00 00 00 00 00 00 00 00 00`

Known off payloads:

- `cd e2 03 01 00 00 00 00 00 00 00 00 00 00 00 00 00`
- `cd e2 02 01 00 00 00 00 00 00 00 00 00 00 00 00 00`

Current status:

| Feature | Status |
| --- | --- |
| lightbar red/on/off | **DEFINITE for prototype** |
| lightbar arbitrary RGB | **UNKNOWN** |
| lightbar brightness `0..4` | **UNKNOWN** |
| lightbar effects | **UNKNOWN** |

Hard rule:

- Do not invent HID bytes.
- Do not guess effect packets.
- Do not assume arbitrary RGB for the lightbar until exact payloads are known.

## Safe-ish / read-only research operations

The following operations were useful as research evidence:

- fan mode readback
- D7 readback
- C0-0C readback
- fan curve readback
- sensor reads
- PK04 read wrappers

Important boundary:

- Read-only is not automatically GUI-safe.
- Read-only hardware access may still require root or risky interfaces.
- Production must still isolate this behavior behind backend/helper boundaries.

## Controlled write prototypes

The following write prototypes were useful evidence:

- SCMD `0x79` fan mode switching
- PK04 FCD1 baseline write prototype
- PK04 FCL2 performance curve write prototype
- keyboard static RGB HID feature reports
- lightbar red/on/off HID feature reports

Production status:

- useful evidence
- not final runtime design
- not application architecture
- not permission to add production writes yet

## Dangerous / do not repeat

Do not repeat:

- early wrong default restore behavior that produced nonsensical decoded fan data
- hard restore attempts that caused RAM/fan table area to appear zeroed or corrupted
- mid-boost restore attempts that produced corrupted readback such as `127°C` or zero tables
- any FCD0-based restore behavior
- any raw `/dev/mem` write
- any arbitrary ACPI payload write
- any unknown PK04 `0x2C` use
- any random HID byte guessing
- any GUI direct `sudo` or `subprocess` hardware control
- any startup fan write
- any startup RGB write
- any generic privileged command executor
- any direct conversion of prototype shell scripts into runtime application code

## Recovery notes

Windows Control Center reset is emergency recovery knowledge only.

It is not normal product design.

If fan table state becomes corrupted, the observed recovery path is:

1. reboot
2. boot Windows
3. open Windows Control Center
4. reset/default fan profile
5. reboot into Linux
6. verify fan state and fan curve

Known Windows reset clean curve:

| Point | Temperature | Duty | RPM |
| ---: | ---: | ---: | ---: |
| 0 | `35°C` | `0%` | `0` |
| 1 | `40°C` | `36%` | `1680` |
| 2 | `45°C` | `41%` | `2240` |
| 3 | `50°C` | `46%` | `2520` |
| 4 | `60°C` | `51%` | `2800` |
| 5 | `65°C` | `61%` | `3360` |
| 6 | `70°C` | `66%` | `3920` |
| 7 | `80°C` | `71%` | `4480` |
| 8 | `90°C` | `81%` | `5040` |
| 9 | `100°C` | `91%` | `5600` |

Do not rely on Windows recovery as a reason to take unsafe risks.

## Production validation implications

Before production fan curve writes exist, the project must be able to:

- validate temperature points
- validate duty values
- validate RPM values if RPM data is represented
- require sorted fan curve points
- reject duplicate temperatures
- reject unsupported hardware
- require explicit user action for writes
- avoid startup writes
- perform readback where possible
- log hardware operations
- document recovery behavior
- avoid unknown table fields
- avoid generic ACPI writers
- avoid generic HID writers

Before production RGB writes exist, the project must be able to:

- discover supported HID devices safely
- reject unsupported devices
- validate RGB values as `0..255`
- validate brightness only after exact brightness mapping is confirmed
- whitelist known operations
- avoid random effect packets
- avoid startup RGB writes
- avoid hard-coded `/dev/hidraw*` paths
- avoid generic HID payload writers

## Unknowns

The following are intentionally unresolved:

- exact semantic difference between PK04 block A and block B across all profiles
- whether PK04 `0x2C` is needed for some production writes
- exact names and meaning for SCMD modes `5`, `6`, and `7`
- lightbar arbitrary RGB payload
- keyboard brightness `0..4` payload mapping
- lightbar brightness `0..4` payload mapping
- keyboard zone grouping
- RGB effect payloads
- cross-model hardware compatibility
- FCD1 `4900` max RPM vs Windows reset `5600` max RPM final baseline meaning
- unsupported-hardware behavior across other Monster/Clevo models

Do not resolve unknowns by guessing.

## Compatibility boundary

The primary target hardware family is:

- Monster / Clevo laptops

The project must not overclaim support for every Monster/Clevo model until detection, validation, and unsupported-hardware behavior exist.

Early implementation should be conservative and explicit about supported hardware.

## Documentation-to-code promotion checklist

A hardware behavior may move toward production only after this checklist is satisfied:

- the behavior is documented
- the behavior is classified as DEFINITE or intentionally limited INFERENCE
- unsafe assumptions are named
- unsupported hardware behavior is designed
- recovery behavior is documented where relevant
- domain models can represent the operation safely
- application services expose semantic use cases
- backend protocols avoid raw arbitrary writes
- infrastructure implements only whitelisted operations
- privileged helper operations are narrow and validated
- tests or review checks exist where possible
- hardware safety review has been performed

## Review requirements

Any future change touching the following areas requires hardware safety review:

- fan
- RGB
- ACPI
- HID
- `/sys`
- `/proc`
- `/dev`
- `/dev/mem`
- root permissions
- systemd
- polkit
- privileged helper
- Debian package permissions
- hardware docs
- payload builders

A reviewer should block the change if it:

- suggests GUI-level hardware access
- introduces generic command execution
- invents ACPI payloads
- invents HID payloads
- hides unsafe behavior behind convenience helpers
- promotes research directly into runtime code
- mixes unrelated commit scope
- skips documentation for hardware behavior

## Current non-goals

This document does not add:

- Python implementation
- PySide6 implementation
- ACPI backend
- HID backend
- privileged helper
- runtime shell scripts
- fan write code
- RGB write code
- new hardware experiments
- copied AML runtime logic

The next production-safe direction is still documentation, domain modeling, fake/mock backends, and reviewable service boundaries before real hardware writes.
