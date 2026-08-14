# AGENTS.md

# LED RGB Controller — Root Project Instructions

## 1. Project Identity

This is the root repository/documentation for the complete LED RGB Controller project.

The project contains two coordinated parts:

```text
LED-RGB-Controller/
├── hardware/
└── app/
```

- `hardware/` contains the embedded/controller firmware and hardware-related implementation.
- `app/` contains the cross-platform KMP + Compose Multiplatform application.

The two parts are separate implementations but one product.

The root-level documentation defines the shared product architecture, contracts, constraints, development workflow, and coordination rules.

---

# 2. Product Vision

The project is a configurable addressable-RGB LED controller.

The current hardware implementation is based on:

- ESP32-WROOM-32
- WS2812B-class addressable RGB LEDs
- BLE communication
- multiple independently controllable LED lines

The software must not permanently depend on the current ESP32 implementation.

The long-term architecture is:

```text
                    ┌─────────────────────┐
                    │   Cross-platform    │
                    │        App          │
                    │ KMP + Compose MP    │
                    └──────────┬──────────┘
                               │
                              BLE
                               │
                    ┌──────────▼──────────┐
                    │  Device Protocol    │
                    │ / Device Contract   │
                    └──────────┬──────────┘
                               │
                    ┌──────────▼──────────┐
                    │ Current Controller   │
                    │   ESP32-WROOM-32     │
                    └──────────┬──────────┘
                               │
                ┌──────────────┼──────────────┐
                │              │              │
             Line 1         Line 2         Line N
                │              │              │
             WS2812B        WS2812B        WS2812B
```

The application should remain usable if the controller implementation changes later.

---

# 3. Non-Negotiable Product Rules

1. Never hard-code exactly three LED lines.
2. Never hard-code a fixed LED count.
3. The number of LED lines is configurable.
4. LED count is configurable independently for each line.
5. The hardware/device is authoritative for actual configuration/state after synchronization.
6. The app must not be coupled to ESP32-specific implementation details.
7. UI must not contain raw BLE/protocol logic.
8. Hardware firmware must not contain UI assumptions.
9. Protocol details must be explicitly agreed between app and hardware.
10. Do not invent unspecified protocol fields, commands, UUIDs, packet formats, limits, or state semantics.
11. Every meaningful implementation task must be tested/verified.
12. Commit only after successful verification.
13. Update the appropriate `PROJECT_STATUS.md` after meaningful progress.
14. Keep root documentation synchronized with major architectural changes.
15. Do not reintroduce removed roadmap items unless explicitly requested.

---

# 4. Repository Boundaries

The intended structure is:

```text
LED-RGB-Controller/
│
├── AGENTS.md
├── PROJECT_STATUS.md
│
├── hardware/
│   ├── AGENTS.md
│   ├── PROJECT_STATUS.md
│   └── ...
│
└── app/
    ├── AGENTS.md
    ├── PROJECT_STATUS.md
    └── ...
```

Root documents describe:

- shared product requirements
- app/hardware contract
- cross-project constraints
- integration rules
- roadmap
- overall status

Child documents describe implementation-specific rules.

If root and child documentation conflict:

1. resolve the conflict explicitly;
2. do not silently choose one;
3. update the documents after the decision.

---

# 5. Hardware — Current Direction

The current controller is based on:

```text
ESP32-WROOM-32
        │
       BLE
        │
   LED Controller
        │
        ├── Line 1 → WS2812B
        ├── Line 2 → WS2812B
        └── Line N → WS2812B
```

The current firmware is being developed with an object-oriented, extensible architecture.

Effects should be easy to add without rewriting the entire controller.

Hardware-side concerns include:

- LED initialization
- LED output
- brightness
- color
- effects
- per-line state
- BLE communication
- device configuration
- state management
- validation
- persistence where required
- future Scheduler support
- future Music Reactive support

---

# 6. Hardware Independence

The application must use a device abstraction.

The current device is ESP32, but the application domain must understand concepts such as:

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

rather than:

```text
Esp32Device
Esp32Line1
Esp32Line2
...
```

ESP32-specific code belongs behind platform/device implementation boundaries.

---

# 7. Dynamic LED Lines

The user can configure the number of LED lines.

Valid range is determined by the connected device capabilities.

Examples:

```text
1 line
2 lines
3 lines
4 lines
...
N lines
```

There must be no application-wide assumption that three lines exist.

Hardware must expose its supported limits.

The app must build its UI from the reported configuration.

---

# 8. Dynamic LED Count

Every line can have its own LED count.

Example:

```text
Line 1 → 60 LEDs
Line 2 → 120 LEDs
Line 3 → 30 LEDs
```

LED count is hardware configuration, not a daily runtime control.

It should be configured through:

```text
App
 → Settings
   → Hardware
     → LED Configuration
```

and persisted by the device where supported.

---

# 9. Initial Configuration

A newly connected/unconfigured device should have a safe default.

Preferred default:

```text
1 LED line
```

Initial setup can collect:

- number of lines
- LED count for each line
- LED type or hardware parameters if required
- other device-specific configuration exposed by capabilities

Preferred flow:

```text
Connect
 ↓
Read capabilities
 ↓
Read configuration
 ↓
If unconfigured → Setup
 ↓
Validate
 ↓
Apply
 ↓
Persist
 ↓
Read back / synchronize
```

A preconfigured device should skip unnecessary setup.

---

# 10. Device Capabilities

The device should expose capabilities sufficient for the app to configure itself dynamically.

Conceptual model:

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

The actual fields are subject to the final App ↔ Hardware Contract.

The app must not assume that all devices support every future feature.

---

# 11. Device State

The app and hardware should share a clear concept of current state.

Potential state:

```text
power
brightness
selected/active line state
color
effect
effect parameters
per-line state
```

The exact model must be finalized in the protocol contract.

State must be synchronized after connection and after relevant commands.

---

# 12. State Authority

After synchronization:

```text
Device state = authoritative hardware state
```

The app may maintain:

- cached state
- optimistic/pending state
- last-known state

but must distinguish these from confirmed device state.

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
App Confirmed State
 ↓
UI
```

A successful BLE write must not automatically mean the requested state is confirmed.

---

# 13. BLE / Protocol Boundary

BLE is a transport layer.

Protocol is a separate layer.

Recommended:

```text
UI
 ↓
Presentation
 ↓
Domain
 ↓
Repository
 ↓
Device Protocol
 ↓
BLE Transport
 ↓
Hardware
```

The protocol must define, after agreement:

- service/characteristic layout
- discovery
- commands
- responses
- notifications
- configuration
- runtime state
- errors
- acknowledgements
- packet framing
- payload limits
- versioning
- capability negotiation

Do not implement unknown protocol behavior by assumption.

---

# 14. App Technology

The application is:

- Kotlin Multiplatform
- Compose Multiplatform

Targets:

- Android
- iOS
- Windows
- Linux
- macOS

Compose Multiplatform is the shared UI framework. Current official documentation confirms shared UI support for Android, iOS, and desktop, including Windows, Linux, and macOS. citeturn0search1turn0search4

When setting up/upgrading the project, use the current compatible Kotlin/Compose/Gradle/Xcode versions rather than copying obsolete templates. Compose Multiplatform versions have their own compatibility matrix. citeturn0search3

---

# 15. App Architecture

Preferred direction:

```text
Compose UI
    ↓
Presentation / ViewModels
    ↓
Domain / Use Cases
    ↓
Repository Interfaces
    ↓
Data / Protocol
    ↓
Transport
    ↓
Device
```

Suggested conceptual modules/packages:

```text
core/
  common/
  model/
  domain/

data/
  protocol/
  repository/
  local/
  transport/

feature/
  dashboard/
  effects/
  devices/
  lines/
  setup/
  settings/
  scheduler/
  music/

platform/
  android/
  ios/
  desktop/
```

Exact structure may evolve.

Do not create unnecessary abstraction layers just for architectural appearance.

---

# 16. App Screens

## Dashboard

Primary control surface.

Includes:

- connection state
- device identity
- power
- brightness
- line selection
- effect
- effect parameters
- color where supported
- speed where supported
- LED preview

This screen receives the strongest Ambient UI treatment.

---

## Effects

Contains:

- available effects
- effect preview
- effect selection
- dynamic parameters
- supported controls

Effects must be data-driven/extensible.

---

## Devices

Contains:

- scanning
- discovered devices
- connect
- disconnect
- device status
- device identity
- rename/forget where supported

The app should conceptually support multiple devices even if V1 focuses on one active device.

---

## Lines

Controls must be generated dynamically:

```text
[All] [1] [2] [3] ... [N]
```

No fixed line count.

---

## Hardware Configuration

Contains:

- number of lines
- LED count per line
- LED type/configuration
- other hardware-specific settings exposed by capabilities

This is intentionally not a daily-use control surface.

---

## Settings

Possible sections:

```text
Appearance
Animation
Ambient UI
Device
Hardware
Advanced
About
```

Settings must be visually calm.

---

## Scheduler — Future

Reserved for:

- time
- days
- actions
- power
- brightness
- effect

Do not implement until explicitly requested.

---

## Music Reactive — Future

Reserved for:

- audio input
- beat/spectrum analysis
- sensitivity
- reactive effects

Do not implement until explicitly requested.

---

# 17. Removed Roadmap Features

The following are intentionally removed from the roadmap:

- WiFi
- OTA
- Presets
- MQTT
- Web UI
- IR Remote

Do not add them back unless explicitly requested.

---

# 18. Visual Design Direction

The app visual identity is:

> **Ambient Dark UI**

Goals:

- dark
- minimal
- modern
- ambient
- reactive
- restrained

Avoid:

- excessive neon
- cyberpunk clutter
- heavy gaming UI
- constant RGB cycling
- distracting animations

The LED is the visual focus, not the decorative UI.

---

# 19. Base Colors

Suggested centralized tokens:

```text
Background       #0B0D10
Surface          #12151A
Surface Elevated #181C22

Text Primary     #F5F7FA
Text Secondary   #A8AFBA
Text Disabled    #626974
```

Do not scatter these values throughout the code.

---

# 20. Dynamic Accent

The UI accent is derived from the actual LED state.

Conceptually:

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

Examples:

```text
Red LED    → red accent
Blue LED   → blue accent
Purple LED → purple accent
```

The system must adjust the resulting colors for readability/contrast.

---

# 21. Ambient Background

Dashboard and Effects may use a very subtle background influenced by the current LED state.

Possible implementation:

```text
radial gradient
+
blur
+
very low opacity
+
slow transition
```

Starting design tokens:

```text
Ambient Background ≈ 3%
Glow ≈ 6%
```

These are starting values, not immutable constants.

The user should feel the lighting atmosphere without consciously seeing a large animated gradient.

---

# 22. Multiple Lines and Multiple Colors

If lines have different colors/effects:

```text
Line 1 → Red
Line 2 → Blue
Line 3 → Purple
Line 4 → Green
```

the app can aggregate those colors into an ambient palette.

Transitions should be:

- slow
- smooth
- low contrast
- non-distracting

The background must not rapidly cycle through line colors.

---

# 23. Effect-Aware Ambient UI

Examples:

```text
Static
→ stable ambient

Pulse
→ subtle slow intensity movement

Rainbow
→ slow color drift

Fire
→ warm slow movement

Ocean
→ slow blue/cyan movement
```

Ambient animation must be significantly calmer than the actual LED effect.

---

# 24. Calm Utility UI

The following screens should not have strong dynamic effects:

- Settings
- Hardware Configuration
- Devices
- Advanced
- About

They should emphasize:

- readability
- stable surfaces
- clear hierarchy
- low motion

Ambient can be reduced or disabled there.

---

# 25. LED Preview

The app should visually represent the configured LED hardware.

Example:

```text
Line 1: ● ● ● ● ● ● ●
Line 2: ● ● ● ● ● ● ●
Line 3: ● ● ● ● ● ● ●
```

The preview must dynamically support:

- line count
- LED count
- color
- effect
- brightness
- power

LED visuals should use a bright core plus subtle glow.

---

# 26. Animation System

Three categories:

### Micro UI

Approximately:

```text
150–250ms
```

### Ambient

Approximately:

```text
2–8 seconds
```

### Effect Preview

Driven by the effect but calmer than hardware output.

Use smooth easing.

Avoid unnecessary linear animations.

Support Reduced Motion.

---

# 27. Themes

Support:

- System
- Dark
- Light

Dark is the primary experience.

Suggested light base:

```text
Background #F6F7F9
Surface    #FFFFFF
Text       #15181D
```

Ambient effects should be much weaker in Light mode.

---

# 28. Responsive Design

### Mobile

- touch-first
- compact layouts
- mobile navigation

### Tablet

- adaptive/multi-column layouts

### Desktop

- sidebar
- multi-column layouts
- resizable window
- keyboard/mouse support

Do not create unrelated platform-specific copies of the same screen.

---

# 29. Accessibility

Must consider:

- text contrast
- touch targets
- keyboard navigation
- focus states
- Reduced Motion
- state communication that does not depend only on color
- clear connection status

---

# 30. Persistence

## Device-side

Potential:

- line count
- LED count
- hardware configuration
- device configuration

## App-side

Potential:

- remembered devices
- theme
- ambient preference
- reduced motion
- last-known state
- UI preferences

Never confuse local cache with device-confirmed state.

---

# 31. Error Handling

Handle explicitly:

- scan failure
- connection failure
- disconnect
- timeout
- invalid configuration
- command rejection
- unsupported feature
- protocol mismatch
- firmware incompatibility
- device error

User-facing messages should be understandable and actionable.

---

# 32. Versioning

The shared contract should account for:

```text
App Version
Firmware Version
Protocol Version
Capability Version
```

The app must not assume the device uses the newest protocol.

The device must not assume the app supports every feature it knows about.

---

# 33. Testing and Verification

Testing is a mandatory part of the development workflow.

For every meaningful task:

```text
Implement
 ↓
Write/update appropriate tests
 ↓
Run tests/build/static checks
 ↓
Fix failures
 ↓
Run verification again
 ↓
Update status
 ↓
Commit
```

### Hardware testing

Where applicable:

- compile
- static checks
- unit tests
- protocol tests
- configuration validation
- effect tests
- device-level/manual hardware tests

### App testing

Where applicable:

- common/domain unit tests
- state transformation tests
- protocol encoder/decoder tests
- capability/configuration validation tests
- color engine tests
- ambient calculation tests
- ViewModel tests
- repository tests
- UI tests
- platform-specific tests

### Integration testing

App and hardware integration should verify:

- discovery
- connection
- capability read
- configuration read
- configuration write
- state read
- runtime commands
- acknowledgements/state updates
- reconnect synchronization
- invalid commands
- unsupported operations
- protocol mismatch behavior

Do not mark a feature complete solely because it compiles.

---

# 34. Git Workflow

Use small, logical commits.

Required workflow:

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

Do not commit known-broken changes.

Do not mix unrelated refactors with feature work.

Never commit:

- secrets
- API keys
- private keys
- generated build artifacts
- machine-specific files
- local IDE state

---

# 35. Codex Workflow

When working in the root project:

1. Read root `AGENTS.md`.
2. Read root `PROJECT_STATUS.md`.
3. Read the relevant child `AGENTS.md`.
4. Read the relevant child `PROJECT_STATUS.md`.
5. Inspect existing code before changing it.
6. Determine whether the change affects the app, hardware, or shared contract.
7. Make the smallest coherent change.
8. Write/update tests.
9. Run verification.
10. Fix failures.
11. Run verification again.
12. Update status documentation.
13. Commit only after successful verification.

If a change affects the App ↔ Hardware Contract, update both sides' documentation.

---

# 36. Cross-Project Change Rule

Changes that affect communication or shared behavior must be treated as contract changes.

Examples:

- new command
- changed command payload
- new effect
- renamed effect
- new effect parameter
- configuration field
- line count behavior
- LED count limits
- device capability
- state field
- error code
- protocol version

Required process:

```text
Define contract
 ↓
Update root documentation
 ↓
Update hardware documentation
 ↓
Update app documentation
 ↓
Implement hardware
 ↓
Test hardware
 ↓
Implement app
 ↓
Test app
 ↓
Integration test
 ↓
Commit
```

Do not update only one side and silently assume compatibility.

---

# 37. App ↔ Hardware Contract

The following is intentionally **pending final definition**:

### Device discovery

- device identity
- device type
- capabilities

### Configuration

- line count
- LED count per line
- LED type
- hardware limits
- configuration persistence

### Runtime state

- power
- brightness
- color
- effect
- effect parameters
- per-line state

### Commands

- configuration
- runtime controls
- state query
- synchronization

### Notifications

- state updates
- configuration updates
- device errors

### BLE

- services
- characteristics
- read/write/notify
- packet format
- framing
- acknowledgements
- MTU/payload strategy

### Compatibility

- protocol version
- firmware version
- capability negotiation

No values should be invented until the contract is explicitly agreed.

---

# 38. Future Extensibility

The architecture must make it easy to add:

- new LED effects
- new effect parameters
- additional line counts
- different LED counts
- new controller hardware
- new transports
- Scheduler
- Music Reactive

without rewriting the core application.

---

# 39. Definition of Done

A cross-project feature is complete when:

- requirements are implemented
- relevant tests exist
- tests pass
- builds/checks pass
- hardware behavior is verified where applicable
- app behavior is verified where applicable
- App ↔ Hardware compatibility is verified for contract changes
- documentation is updated
- status is updated
- commit is created after successful verification

---

# 40. Root Project Principle

The project should evolve as one product with two independently maintainable implementations:

```text
             PRODUCT
                │
       ┌────────┴────────┐
       │                 │
   HARDWARE             APP
       │                 │
 ESP32/device       KMP + CMP
       │                 │
       └────── Contract ─┘
                │
               BLE
```

The contract between these sides is more important than any individual implementation.

When implementation details change, preserve the product contract and user experience wherever possible.


## Git Commit Identity

- All Git commits created by the agent MUST use the following author identity:
  - Name: `codex`
  - Email: `codex@mail.com`
- Before creating a commit, ensure the Git author identity is configured correctly.
- Do not use the user's personal Git name or email for agent-created commits.

## Task Management

- Keep tasks small, focused, and reasonably short.
- Do NOT create large tasks that combine multiple independent features, changes, or responsibilities.
- If a task is expected to be long or complex, split it into multiple smaller, logical tasks.
- Each task should have a clear and limited objective and should ideally be independently testable.
- Complete and test each small task before moving to the next one.
- Avoid unnecessary scope expansion within a task.
