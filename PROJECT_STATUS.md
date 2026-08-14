# PROJECT_STATUS.md

# LED RGB Controller — Overall Project Status

## 1. Project Summary

This document tracks the overall state of the complete LED RGB Controller project.

The project consists of:

```text
LED-RGB-Controller/
├── hardware/
└── app/
```

Current hardware:

- ESP32-WROOM-32
- WS2812B-class addressable RGB LEDs
- BLE communication

Application:

- Kotlin Multiplatform
- Compose Multiplatform
- Android
- iOS
- Windows
- Linux
- macOS

The application is intentionally hardware-independent.

---

# 2. Product Vision

Create a polished cross-platform controller for configurable addressable RGB LED installations.

The product should allow users to:

- connect to a controller
- configure LED lines
- configure LED count per line
- control power
- control brightness
- select effects
- configure effect parameters
- control individual lines or all lines
- see a live LED preview
- experience a subtle ambient UI derived from LED state
- manage devices
- configure hardware

Future:

- Scheduler
- Music Reactive

---

# 3. Overall Architecture

Target architecture:

```text
┌───────────────────────────────────────────┐
│                  APP                      │
│                                           │
│ KMP + Compose Multiplatform               │
│ Android / iOS / Windows / Linux / macOS │
│                                           │
│ UI → Presentation → Domain → Repository  │
└─────────────────────┬─────────────────────┘
                      │
                    BLE
                      │
┌─────────────────────▼─────────────────────┐
│             DEVICE CONTRACT               │
│                                           │
│ Capabilities / Configuration / State      │
│ Commands / Responses / Notifications      │
└─────────────────────┬─────────────────────┘
                      │
┌─────────────────────▼─────────────────────┐
│                HARDWARE                   │
│                                           │
│ Current: ESP32-WROOM-32                   │
│                                           │
│ Dynamic LED Lines                         │
│ WS2812B-class addressable RGB LEDs        │
└───────────────────────────────────────────┘
```

---

# 4. Current Status

## Product Definition

Status: **Defined**

The core product concept and UX direction have been established.

## Hardware

Status: **Core implementation substantially developed**

The hardware/firmware side has already been developed substantially, with the current project using ESP32 and WS2812B LEDs and BLE control.

Exact current implementation status is maintained in:

```text
hardware/PROJECT_STATUS.md
```

## Application

Status: **Architecture / planning phase**

The app product design, screen structure, visual direction, and architectural requirements are defined.

Implementation is the next major phase after the App ↔ Hardware Contract is finalized.

## App ↔ Hardware Contract

Status: **Pending final specification**

BLE services, characteristics, packet format, commands, responses, state synchronization semantics, and versioning still need to be explicitly coordinated.

---

# 5. Hardware Configuration Model

The product is not limited to three lines.

The user can configure:

```text
Line Count = N
```

and for every line:

```text
Line N
 └── LED Count = X
```

Example:

```text
Line 1 → 60
Line 2 → 120
Line 3 → 60
Line 4 → 30
```

The actual maximum values must come from device capabilities.

---

# 6. Default / Initial Setup

Preferred default:

```text
1 LED line
```

Initial setup flow:

```text
Connect
 ↓
Discover device
 ↓
Read capabilities
 ↓
Read configuration
 ↓
If unconfigured:
    configure line count
    configure LED count per line
    configure required hardware fields
 ↓
Validate
 ↓
Apply
 ↓
Persist
 ↓
Read back
 ↓
Enter Dashboard
```

A device that is already configured should not force the setup flow again.

---

# 7. Device Configuration Ownership

Preferred model:

```text
Device = authoritative configuration
App = cache / editor / UI
```

The device should retain its hardware configuration when supported.

This allows another app installation or computer to connect and discover the actual configuration.

---

# 8. Device Capabilities

Conceptual capability model:

```text
DeviceCapabilities
├── maxLines
├── maxLedsPerLine
├── supportedLedTypes
├── supportedEffects
├── supportsScheduler
├── supportsMusicReactive
├── protocolVersion
└── firmwareVersion
```

Exact fields are pending protocol design.

The app must construct its UI from capabilities and configuration rather than hard-coded assumptions.

---

# 9. Runtime State

Conceptual device state:

```text
DeviceState
├── power
├── brightness
└── lines
      ├── color
      ├── effect
      └── effectParameters
```

Exact state semantics remain pending protocol definition.

---

# 10. App Screen Plan

## Dashboard

Primary screen.

Planned controls:

- connection status
- device identity
- power
- brightness
- line selection
- effect
- dynamic effect parameters
- color where supported
- speed where supported
- LED preview

Visual intensity: **high**

---

## Effects

Planned:

- effect list
- effect cards
- preview
- effect selection
- dynamic parameters

Visual intensity: **high**

---

## Devices

Planned:

- scan
- device list
- connect
- disconnect
- device information
- rename/forget where supported

Visual intensity: **low/moderate**

---

## Lines

Dynamic:

```text
All
1
2
3
...
N
```

The number of lines comes from device configuration.

---

## Hardware Configuration

Planned:

- line count
- LED count per line
- LED type/configuration
- hardware-specific fields

Visual intensity: **low**

---

## Settings

Planned sections:

```text
Appearance
Animation
Ambient UI
Device
Hardware
Advanced
About
```

Visual intensity: **low**

---

# 11. Future Features

## Scheduler

Status: **Reserved / not implemented**

## Music Reactive

Status: **Reserved / not implemented**

Only these two features remain on the future roadmap.

---

# 12. Removed Features

Explicitly removed from roadmap:

- WiFi
- OTA
- Presets
- MQTT
- Web UI
- IR Remote

Status: **Do not implement unless explicitly re-added.**

---

# 13. UI Design Direction

Name:

**Ambient Dark UI**

Design principles:

```text
Dark
Minimal
Ambient
Reactive
Restrained
```

The LED state is the visual source of personality.

The application itself should not look like a generic RGB gaming dashboard.

---

# 14. Dark Theme Tokens

Suggested:

```text
Background       #0B0D10
Surface          #12151A
Surface Elevated #181C22

Text Primary     #F5F7FA
Text Secondary   #A8AFBA
Text Disabled    #626974
```

These should become centralized design tokens during implementation.

---

# 15. Dynamic Accent

The primary UI accent is derived from the current LED state.

Concept:

```text
LED State
 ↓
Color Engine
 ↓
Accent
Accent Soft
Accent Strong
Glow
Ambient
```

The system must ensure sufficient readability and contrast.

---

# 16. Ambient Background

Dashboard/Effects screens can use a subtle color atmosphere.

Concept:

```text
LED colors
 ↓
Ambient palette
 ↓
Blur / radial gradient
 ↓
Low-opacity background
 ↓
Slow transition
```

Initial targets:

```text
Background ambient ≈ 3%
Glow ≈ 6%
```

These are starting design values.

---

# 17. Multiple LED Lines

If lines use different colors:

```text
Line 1 → Red
Line 2 → Blue
Line 3 → Purple
```

the background can subtly blend between those colors.

Transitions should be nearly imperceptible and slow.

No rapid RGB cycling.

---

# 18. Effect-Aware Ambient

Conceptual behavior:

```text
Static
→ stable glow

Pulse
→ subtle intensity movement

Rainbow
→ slow color drift

Fire
→ warm movement

Ocean
→ blue/cyan movement
```

The UI must always be calmer than the hardware effect.

---

# 19. Calm Utility Screens

Settings and configuration screens must not have strong animated effects.

Affected screens:

- Settings
- Hardware Configuration
- Devices
- Advanced
- About

Use stable surfaces and readable hierarchy.

---

# 20. LED Preview

Dynamic preview:

```text
Line 1: ● ● ● ● ● ●
Line 2: ● ● ● ● ● ●
Line N: ● ● ● ● ● ●
```

Must support:

- dynamic line count
- dynamic LED count
- color
- effect
- brightness
- power

Visual model:

```text
LED core
+
subtle glow
```

---

# 21. Responsive Layout

## Mobile

- compact
- touch-first
- mobile navigation

## Tablet

- adaptive columns

## Desktop

- sidebar
- multi-column
- resizable
- keyboard/mouse friendly

All should share the same design system.

---

# 22. Themes

Supported:

- System
- Dark
- Light

Dark is primary.

Light:

```text
Background #F6F7F9
Surface    #FFFFFF
Text       #15181D
```

Ambient effects are reduced in Light mode.

---

# 23. Animation

### Micro

```text
150–250ms
```

### Ambient

```text
2–8s
```

### Effect Preview

Effect-aware but calmer than hardware.

Reduced Motion must be supported.

---

# 24. Accessibility

Must support consideration of:

- contrast
- touch target sizes
- keyboard navigation
- focus
- Reduced Motion
- non-color-only state communication
- connection state clarity

---

# 25. Persistence

## Device

Potential:

- line count
- LED count
- hardware configuration
- device configuration

## App

Potential:

- remembered devices
- theme
- ambient setting
- reduced motion
- last-known state
- UI preferences

Device-confirmed data must be distinguishable from app cache.

---

# 26. App Architecture Status

Target:

```text
Compose UI
 ↓
Presentation / ViewModels
 ↓
Domain / Use Cases
 ↓
Repository
 ↓
Protocol
 ↓
Transport
 ↓
Hardware
```

No final module/package structure has been frozen yet.

---

# 27. Hardware Architecture Status

The hardware implementation is currently based on ESP32-WROOM-32 and WS2812B-class addressable LEDs.

The firmware should remain:

- modular
- object-oriented
- effect-extensible
- configuration-driven
- BLE-controlled

Exact implementation details remain documented under:

```text
hardware/
```

---

# 28. Cross-Project Contract — Pending

This is the next major technical phase.

Must define:

## Device Discovery

- identity
- device type
- capabilities

## Configuration

- line count
- LED count per line
- LED type
- limits
- persistence

## Runtime State

- power
- brightness
- colors
- effects
- effect parameters
- per-line state

## Commands

- state query
- runtime changes
- configuration changes

## Responses

- acknowledgements
- state
- configuration
- errors

## Notifications

- state changes
- configuration changes
- errors

## BLE

- services
- characteristics
- read/write/notify
- packet framing
- payload/MTU strategy

## Compatibility

- protocol version
- firmware version
- capability negotiation

---

# 29. Contract Rule

Any change to one of the following is a cross-project contract change:

- command
- response
- notification
- state field
- configuration field
- capability
- effect
- effect parameter
- line behavior
- LED count limits
- error
- protocol version

Such a change must be documented and coordinated in both projects.

---

# 30. Synchronization Model

Target:

```text
Connect
 ↓
Capabilities
 ↓
Configuration
 ↓
Current State
 ↓
UI
```

Runtime:

```text
User Action
 ↓
App Command
 ↓
BLE / Protocol
 ↓
Device Validation
 ↓
Apply
 ↓
Confirmation / State Update
 ↓
App State
 ↓
UI
```

---

# 31. Disconnection Model

When disconnected:

- show clear connection state
- retain last-known state where useful
- allow local preview where possible
- distinguish pending/local changes
- do not falsely claim synchronization

On reconnect:

```text
Capabilities
 ↓
Configuration
 ↓
State
```

must be refreshed.

---

# 32. Error Model

Expected categories:

- discovery failure
- connection failure
- timeout
- invalid configuration
- unsupported command
- command rejection
- device error
- protocol mismatch
- firmware incompatibility

User-facing errors should be clear and actionable.

---

# 33. Versioning

The system must account for:

```text
App Version
Firmware Version
Protocol Version
Capability Version
```

Compatibility must be checked before unsupported operations.

---

# 34. Testing Strategy

Testing is mandatory.

For every meaningful task:

```text
Implement
 ↓
Write/update tests
 ↓
Run tests
 ↓
Fix
 ↓
Run again
 ↓
Update status
 ↓
Commit
```

Required coverage areas include:

### Hardware

- firmware compilation
- effect logic
- configuration validation
- protocol encoding/decoding
- device state
- BLE command handling where testable

### App

- domain
- state transformation
- configuration validation
- capabilities
- protocol encoding/decoding
- color calculations
- ambient calculations
- ViewModels
- repositories
- UI

### Integration

- discovery
- connect
- capability sync
- configuration sync
- configuration update
- runtime commands
- acknowledgements
- notifications
- reconnect
- invalid commands
- unsupported features
- protocol compatibility

---

# 35. Git Workflow

Required:

```text
Task
 ↓
Implementation
 ↓
Test
 ↓
Verification
 ↓
PROJECT_STATUS update
 ↓
Commit
```

Do not commit known-broken code.

Keep commits focused and logical.

---

# 36. Current Milestones

## Milestone 1 — Hardware Foundation

Status: **Substantially complete**

Current focus:

- ESP32-WROOM-32
- WS2812B
- BLE
- effects
- configurable hardware architecture

Detailed status:

```text
hardware/PROJECT_STATUS.md
```

---

## Milestone 2 — App Product Definition

Status: **Complete**

Defined:

- KMP/CMP
- target platforms
- screens
- dynamic lines
- dynamic LED counts
- device model
- capabilities
- visual identity
- ambient UI
- LED preview
- settings structure
- testing workflow

---

## Milestone 3 — App ↔ Hardware Contract

Status: **Next**

Need to finalize:

- BLE architecture
- services
- characteristics
- commands
- responses
- state
- configuration
- errors
- versioning
- capability negotiation

---

## Milestone 4 — App Foundation

Status: **Pending**

Planned:

- create/verify KMP/CMP project
- configure five targets
- base architecture
- dependency boundaries
- design system
- navigation
- device abstractions
- test infrastructure

---

## Milestone 5 — Device Communication

Status: **Pending**

Planned:

- transport abstraction
- BLE implementation
- discovery
- connection
- synchronization
- protocol implementation

---

## Milestone 6 — Dashboard

Status: **Pending**

Planned:

- LED preview
- power
- brightness
- line selection
- effect
- dynamic parameters
- ambient UI
- dynamic accent

---

## Milestone 7 — Effects

Status: **Pending**

Planned:

- effect list
- effect metadata
- parameter model
- effect editor
- preview

---

## Milestone 8 — Hardware Configuration

Status: **Pending**

Planned:

- line count
- LED count
- capabilities
- setup flow
- persistence
- validation

---

## Milestone 9 — Devices / Settings

Status: **Pending**

Planned:

- device management
- appearance
- animation
- ambient settings
- hardware settings
- about

---

## Milestone 10 — Integration / Stabilization

Status: **Pending**

Planned:

- hardware/app integration tests
- reconnect behavior
- error handling
- compatibility
- performance
- responsive layouts
- accessibility
- packaging

---

# 37. Definition of Done

The overall product is complete for a feature when:

- implementation is finished
- tests are written
- tests pass
- relevant builds/checks pass
- hardware is verified where required
- app is verified where required
- integration is verified for contract changes
- documentation is updated
- status is updated
- commit is created only after successful verification

---

# 38. Current Immediate Next Step

Before substantial application implementation:

> **Finalize the App ↔ Hardware Contract.**

This is intentionally the next phase because it determines:

- device model
- BLE layer
- protocol layer
- state model
- configuration model
- effect model
- error model
- synchronization strategy
- compatibility/versioning

Do not guess these details in either project.

---

# 39. Overall Status Summary

```text
Product Vision              ██████████  Defined
Hardware Foundation         ████████░░  Substantially complete
App UX / UI Design          ██████████  Defined
App Architecture            ███████░░░  Planned
App Implementation          ░░░░░░░░░░  Pending
BLE Protocol                ░░░░░░░░░░  Pending contract
App/Hardware Integration    ░░░░░░░░░░  Pending
Scheduler                   ░░░░░░░░░░  Future
Music Reactive              ░░░░░░░░░░  Future
```

The project is currently at the boundary between **product/architecture definition** and **App ↔ Hardware contract design**.
