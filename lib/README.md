# lib/

Shared Kotlin Multiplatform libraries of Things Mapper. Empty in phase 1.

The code that the Linux, Android and iOS shells share is developed here as
libraries, in a Gradle build of its own that knows nothing of the shells
([ADR-0009](../design/adr/ADR-0009-shared-libraries.md), REQ-NF-045):

- Every library builds for `jvm` (Linux desktop and Raspberry Pi),
  `android`, `iosArm64` and `iosSimulatorArm64`.
- Common tests run on every target.
- Every platform implementation of a port runs the same contract suite
  from `testing`.

The module layout is defined in the technical design specification
([design/05-tds.md](../design/05-tds.md), section "Module structure"). It is
created in phase 2, work package WP-2.1 of the
[implementation plan](../design/07-implementation-plan.md):

```
lib/
├── settings.gradle.kts
├── core/            domain, ports, use cases, protocol adapters, rules, exporters
├── data/            SQLDelight storage, settings, secret store (per-platform implementations)
├── net/             MQTT and HTTP clients, TLS, connectivity
├── map/             map model, projection, tile cache, renderer
├── ui/              Compose Multiplatform: screens, view models, theme
├── ads/             sponsor ad slot and policy (Free edition)
├── support/         funding data, start screen and support policies
└── testing/         fixtures, fakes, contract test suites for the ports
```

The iOS app links the libraries through one framework, `ThingsMapperKit`,
which the shell module `app/ios-kit` builds
([ADR-0010](../design/adr/ADR-0010-ios-development-setup.md)).

Build and test the libraries on their own from this folder with the
wrapper at the repository root. On a host without an Android SDK, the
Android target is skipped; the Apple targets are built on macOS only. CI
runs all targets on every pull request.
