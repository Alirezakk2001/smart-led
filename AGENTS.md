# AGENTS.md

# LED RGB Controller — Agent Instructions

## 1. Project Overview

This repository contains one LED RGB Controller product with two coordinated parts:

```text
LED-RGB-Controller/
├── AGENTS.md
├── PROJECT_STATUS.md
├── hardware/
│   ├── AGENTS.md
│   ├── PROJECT_STATUS.md
│   └── ...
└── app/
    ├── AGENTS.md
    ├── PROJECT_STATUS.md
    └── ...
```

- `hardware/` — ESP32/device firmware and hardware implementation.
- `app/` — Kotlin Multiplatform + Compose Multiplatform application.
- The app and hardware are separate implementations connected through a shared device/protocol contract.

The current controller is based on ESP32-WROOM-32, WS2812B-class LEDs, and BLE.

---

## 2. Source of Truth

Before starting work:

1. Read the root `PROJECT_STATUS.md`.
2. Read the relevant child `AGENTS.md`.
3. Read the relevant child `PROJECT_STATUS.md`.
4. Inspect the existing implementation before changing it.

`PROJECT_STATUS.md` contains the current tasks and project progress. Do not duplicate task planning here.

If root and child instructions conflict, resolve the conflict explicitly and update the documentation.

---

## 3. Non-Negotiable Rules

1. Never hard-code exactly three LED lines.
2. Never hard-code a fixed LED count.
3. The number of LED lines is configurable.
4. LED count is configurable independently per line.
5. The device is authoritative for confirmed configuration/state after synchronization.
6. The app must not depend directly on ESP32-specific implementation details.
7. UI must not contain raw BLE/protocol logic.
8. Firmware must not contain UI assumptions.
9. Do not invent unspecified protocol fields, commands, UUIDs, packet formats, limits, or state semantics.
10. Meaningful implementation work must be tested/verified.
11. Commit only after successful verification.
12. Update the appropriate `PROJECT_STATUS.md` after meaningful progress.
13. Keep shared/root documentation synchronized with major architectural or contract changes.
14. Do not reintroduce removed roadmap features unless explicitly requested.

---

## 4. Architecture Boundaries

### App

The app uses:

- Kotlin Multiplatform
- Compose Multiplatform

Targets:

- Android
- iOS
- Windows
- Linux
- macOS

Preferred flow:

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
Device
```

The app should use device abstractions such as:

```text
Device
DeviceCapabilities
DeviceConfiguration
DeviceState
LedLine
LedLineState
Effect
EffectParameter
```

Avoid coupling the domain to names such as `Esp32Device` or `Esp32Line1`.

### Hardware

The current direction is:

```text
ESP32-WROOM-32
        │
       BLE
        │
   LED Controller
        │
   Line 1 ... Line N
        │
     WS2812B
```

Firmware should remain extensible, especially for effects, per-line state, BLE communication, configuration, validation, and persistence.

---

## 5. Dynamic Hardware Configuration

The app and hardware must support dynamic LED lines and LED counts.

### LED Lines

The number of lines is determined by device capabilities.

The app must generate its UI from the reported configuration/capabilities.

Never assume:

```text
Line 1
Line 2
Line 3
```

### LED Count

Each line may have its own LED count.

Example:

```text
Line 1 → 60
Line 2 → 120
Line 3 → 30
```

Hardware configuration should include, where supported:

- number of lines
- LED count per line
- LED type
- other device-exposed hardware parameters

A newly connected/unconfigured device should use a safe default, preferably one LED line.

---

## 6. Device State and Synchronization

The device is the authoritative source for confirmed state.

The app may maintain cached, optimistic, or last-known state, but must distinguish these from confirmed device state.

Preferred command lifecycle:

```text
User Action
 ↓
App Command
 ↓
Transport
 ↓
Device
 ↓
Validation
 ↓
Apply
 ↓
Confirmation / State Update
 ↓
Confirmed App State
 ↓
UI
```

A successful BLE write does not automatically mean the requested state is confirmed.

---

## 7. BLE and Protocol

BLE is the transport layer; the protocol is a separate layer.

The final App ↔ Hardware Contract must explicitly define, when agreed:

- discovery and device identity
- capabilities
- configuration
- runtime state
- commands
- responses
- notifications
- errors
- acknowledgements
- packet framing/payload limits
- versioning
- capability negotiation
- BLE services/characteristics

Do not implement unknown protocol behavior by assumption.

Changes affecting shared communication or behavior are contract changes.

Examples:

- new/changed command
- payload change
- effect or parameter change
- configuration field
- line/LED limits
- capability
- state field
- error code
- protocol version

For contract changes, update the shared/root documentation and both app/hardware documentation before or alongside implementation.

---

## 8. Product/UI Requirements

The product is a configurable addressable-RGB LED controller.

Visual direction:

> Ambient Dark UI

The UI should be:

- dark
- minimal
- modern
- ambient
- reactive
- restrained

The LED is the visual focus.

Important requirements:

- accent color follows the actual LED state
- dashboard/effects may use a subtle ambient background based on LED state
- multiple LED colors may contribute to a calm ambient palette
- settings/configuration/devices/about should remain visually calm
- LED preview must dynamically reflect line count, LED count, color, effect, brightness, and power
- support Reduced Motion
- support System, Dark, and Light themes
- desktop, tablet, and mobile layouts should adapt rather than duplicating screens
- accessibility must not rely only on color

Do not overbuild UI effects or add unnecessary abstraction.

---

## 9. Removed Roadmap Features

These features are intentionally removed:

- WiFi
- OTA
- Presets
- MQTT
- Web UI
- IR Remote

Do not add them back unless explicitly requested.

Reserved future features:

- Scheduler
- Music Reactive

Do not implement future features until explicitly requested.

---

## 10. Testing and Verification

Testing is mandatory.

For every meaningful task:

```text
Implement
 ↓
Write/update tests
 ↓
Run tests/build/static checks
 ↓
Fix failures
 ↓
Run verification again
 ↓
Update PROJECT_STATUS.md
 ↓
Commit
```

Use appropriate verification for the change.

### Hardware

Where applicable:

- compile
- static checks
- unit/protocol tests
- configuration validation
- effect tests
- manual/device testing

### App

Where applicable:

- domain/state tests
- protocol encoder/decoder tests
- configuration/capability validation
- repository/ViewModel tests
- UI tests
- platform-specific tests

### Integration

For contract changes, verify as applicable:

- discovery
- connection
- capability/configuration synchronization
- runtime commands
- acknowledgements/state updates
- reconnect synchronization
- invalid/unsupported operations
- protocol mismatch behavior

A feature is not complete just because it compiles.

---

## 11. Git Workflow

Use small, logical commits.

```text
Task
 ↓
Implementation
 ↓
Test
 ↓
Verification
 ↓
Status update
 ↓
Commit
```

Rules:

- Do not commit known-broken changes.
- Do not mix unrelated refactors with feature work.
- Do not commit secrets, keys, generated build artifacts, or machine-specific files.

### Commit Identity

All Git commits created by the agent MUST use:

```text
Name:  codex
Email: codex@mail.com
```

Before committing, verify/configure the Git author identity.

Never use the user's personal Git identity for agent-created commits.

---

## 12. Task Management

- Keep tasks small, focused, and reasonably short.
- Do not create large tasks combining multiple independent features or responsibilities.
- If a task is long or complex, split it into smaller logical tasks.
- Each task should have a clear, limited objective and should ideally be independently testable.
- Complete and test each task before moving to the next one.
- Avoid unnecessary scope expansion.

Task details and current task planning belong in `PROJECT_STATUS.md`, not in this file.

---

## 13. Definition of Done

A meaningful feature/change is complete when:

- requirements are implemented
- relevant tests exist
- tests/checks pass
- hardware behavior is verified where applicable
- app behavior is verified where applicable
- contract compatibility is verified for contract changes
- documentation/status is updated
- the change is committed after successful verification

The project should evolve as one product with independently maintainable hardware and app implementations connected by a clear contract.
