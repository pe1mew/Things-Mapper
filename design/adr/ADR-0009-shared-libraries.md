# ADR-0009 — Shared code as libraries in a build of their own

Status: **Accepted** (maintainer decision, 2026-10-25) · Date: 2026-10-25

## Context

ADR-0002 puts all logic in Kotlin Multiplatform modules that the Android,
Linux desktop / Raspberry Pi and — later — iOS shells share. Up to TDS
rev 0.6 these library modules and the application shells were planned as
modules of **one** Gradle build under `app/`. The maintainer decided that
the parts shared between Linux, Android and iOS shall be **developed
separately, tested as libraries and reused** by the shells.

In a single build nothing structural keeps a library from depending on a
shell. Testing one library also configures the whole build, including the
Android application, which fails without an Android SDK — a real barrier
on the Linux-only workstation where the desktop and kiosk work happens.
For iOS, Kotlin code reaches a SwiftUI app only as a framework, and
several separately exported Kotlin frameworks do not share a runtime or
types.

## Decision drivers

- Libraries build and pass their tests without any shell, and without an
  Android SDK on a Linux-only host.
- One test suite proves the same behaviour on every target (JVM for
  Linux, Android, iOS).
- Different platform implementations of the same port (secret store,
  settings, position source) are verified against the same expectations.
- Changes to a library's public API are deliberate and visible in review.
- One maintainer: a change that crosses library and shell stays one pull
  request; no release ceremony between them.
- Both iOS options of WP-4.1 stay open: a Compose Multiplatform shell and
  a SwiftUI shell on the libraries.
- A path to reuse outside this project (for example the protocol
  adapters) without restructuring.

## Options

| Option | Assessment |
|---|---|
| A. One Gradle build; libraries and shells as modules (TDS rev 0.6) | Simplest. Boundaries only by convention; a library can reach into a shell; testing a library configures the whole build including the Android application. |
| **B. One repository, two Gradle builds — `lib/` and `app/` — joined as a composite build** | `lib/` is a standalone build that knows no shell; `app/` includes it with `includeBuild("../lib")`, and Gradle substitutes the library coordinates with the library projects, so the shells always build against the current library source. One pull request for cross-cutting changes. `lib/` can later move to its own repository unchanged. Cost: a little more build configuration (shared convention plugins and version catalogue for two builds). |
| C. Libraries in their own repository, published to a Maven repository | Strongest separation and ready for outside consumers. For one maintainer: version bumps, a release and two pull requests for every change that crosses the line, and shells that lag behind the libraries. |
| D. A repository per library | C multiplied by eight; rejected. |

## Decision

**D1 — Two builds (option B).**
- `lib/` is a standalone Gradle build with the libraries `core`, `data`,
  `net`, `map`, `ui`, `ads`, `support` and `testing`, plus
  `ios-umbrella` in phase 4 (D7).
- `app/` is the build of the shells `android`, `desktop` and
  `tools/replay`. It includes `lib/` as a composite build and depends on
  the coordinates `nl.pe1mew.thingsmapper:<library>`.
- The convention plugins (`build-logic/`), the version catalogue
  (`gradle/libs.versions.toml`) and the Gradle wrapper sit at the
  repository root and are used by both builds, so both use the same
  versions.
- The root settings file includes both builds, so the IDE opens the
  repository root as one project.

**D2 — One direction.** `app/` depends on `lib/`, never the reverse.
Inside `lib/` the dependency rule of TDS §2.1 holds: `core` depends on no
other library; the infrastructure libraries, `ads` and `support` depend on
`core`; `ui` depends on those; `testing` provides fakes and contract suites
for `core`'s ports.

**D3 — What goes where.**
- Code belongs in `lib/` when more than one shell uses it, or when it is
  platform-independent logic.
- Platform implementations of a port live in the per-target source sets of
  the library that implements the port: `jvmMain` for Linux and the Pi,
  `androidMain`, `iosMain`. Example: the secret store, with Android
  Keystore, Secret Service / file, and later Keychain implementations.
- A shell keeps only its lifecycle, dependency wiring, packaging and code
  that no other shell can use: the Android service and Car App service,
  the systemd unit, the gpsd and NMEA sources.

**D4 — Targets.**
- Every library builds for `jvm`, `android`, `iosArm64` and
  `iosSimulatorArm64`. `net` offers only HTTP on iOS until WP-4.1 adds
  MQTT.
- Apple targets build on macOS hosts only; the Kotlin Gradle plugin skips
  them elsewhere.
- The Android target is enabled where an Android SDK is configured. On a
  Linux-only workstation the libraries therefore build and test for `jvm`.
- CI always has both the Android SDK and macOS, so every pull request
  still builds and tests every target (REQ-NF-042).

**D5 — Tested as libraries.**
- **Common tests on every target.** Tests in `commonTest` run as JVM
  tests (Linux), as Android host tests and as iOS simulator tests.
  Android-only behaviour, such as the Keystore, adds instrumented tests on
  the emulator.
- **Contract suites.** `testing` contains an abstract test suite for each
  port that has more than one implementation (for example
  `SecretStoreContract`, `SettingsStoreContract`,
  `PositionSourceContract`). Every implementation runs that suite from its
  own test source set — in-memory fakes included — so all implementations
  behave the same.
- **Public API control.** Explicit API mode (TDS §3) and a checked-in ABI
  dump per library that CI compares, so a change to a public declaration
  fails CI until the dump is updated in the same pull request.
- **Coverage gate.** The coverage gate of REQ-NF-041 applies to the
  library build.
- **A consumer without UI.** The replay tool uses only `core` and `net`
  and shows that the libraries work without a shell.

**D6 — CI.** CI is split into two workflows, both run on every pull
request:
- `lib.yml` runs on Ubuntu (lint, ABI check, JVM and Android host tests,
  coverage) and on a macOS runner (iOS simulator tests). GitHub-hosted
  macOS runners are free for public repositories, so iOS behaviour is
  tested before the Apple hardware of OP-11 is bought.
- `app.yml` builds and tests the shells against the current library
  source.

**D7 — iOS consumption.**
- A Compose Multiplatform iOS shell links the libraries directly as part
  of the Kotlin build.
- A SwiftUI shell gets them through `lib/ios-umbrella`. That module exports
  the libraries as **one** XCFramework — one Kotlin runtime and shared
  types — and Xcode consumes it through Swift Package Manager as a binary
  target.
- The public API that Swift will see avoids Kotlin-only constructs.
  Whether flows and `suspend` functions reach Swift through wrappers in
  the umbrella module or through Kotlin's Swift export is decided in
  WP-4.1.

**D8 — Versions and publication.**
- `lib/` carries its own version (SemVer, in step with the 0.x
  pre-releases until 1.0).
- Inside the repository the shells always build against the library
  source, so no published version is needed.
- Publishing to a Maven repository starts only when an outside consumer
  appears; `lib/` then moves to its own repository (option C) without
  restructuring.

## Consequences

- WP-2.1 creates the root wrapper, `build-logic/`, the shared version
  catalogue, the two builds, `lib.yml` with its macOS job, and `app.yml`.
- TDS §3 lists per module whether it belongs to `lib/` or `app/`. The
  shells' README stays in `app/`; the libraries get `lib/README.md`.
- New requirement REQ-NF-045 records the maintainer's rule. REQ-P-005 (no
  platform API in common code, iOS compile in CI) is now met with iOS
  *tests* on every pull request.
- New test cases TC-LB-01…04 and the test environment CI-iOS. The pull
  request checklist gains "every port implementation runs its contract
  suite".
- The composite build makes the library source the single truth for the
  shells. The price is that a library change can break a shell in the same
  pull request — `app.yml` catches it there, which is intended.
- If the repository ever becomes private, macOS minutes cost money. The
  iOS job then moves to a nightly run.

## Links

[ADR-0002](ADR-0002-technology-stack.md),
[05 TDS §2.1, §3, §11](../05-tds.md),
[02 Requirements REQ-NF-045, REQ-P-005](../02-requirements.md),
[06 Test plan §4, §6.23](../06-test-plan.md),
[07 Implementation plan WP-2.1, WP-4.1](../07-implementation-plan.md).
