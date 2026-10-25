# ADR-0010 — iOS development setup in the repository

Status: **Accepted** (maintainer decision, 2026-10-25) · Date: 2026-10-25

## Context

The iOS shell is phase 4 (WP-4.1), and the Apple hardware it needs — a
Mac, an iPhone and the Apple Developer Program (doc 08 I-11…I-13) — waits
for the OP-11 decision. Until then all development happens on Linux,
where Xcode cannot run. The maintainer decided that the repository shall
nevertheless be set up so that iOS app development can be done with it on
Apple hardware: a developer with a Mac clones the repository, installs the
documented tools and runs the app, without first restructuring anything.

ADR-0009 already builds and tests the shared libraries for iOS on every
pull request. What is missing is the app side: an Xcode project, the way
Xcode gets the Kotlin code, the Free and Paid editions (REQ-P-009, TDS-9.5)
and signing. Without a working app in CI this part would only exist on
paper until phase 4.

## Decision drivers

- From a clean checkout to the app running in the iOS simulator by
  following one README; no restructuring when phase 4 starts.
- iOS readiness is proved continuously: CI builds and tests the iOS app on
  every pull request, so it cannot decay unnoticed while nobody has a Mac.
- The project definition is plain text that can be written and reviewed
  on Linux; nobody edits Xcode's generated `project.pbxproj` by hand, and
  generated files cause no merge conflicts.
- Both UI options of WP-4.1 stay open: Compose Multiplatform or SwiftUI.
- Two editions with their own bundle ids from one source tree.
- No personal signing data in the repository; CI builds without signing.
- No dependency on CocoaPods, which is winding down.

## Options

**Xcode project**

| Option | Assessment |
|---|---|
| `.xcodeproj` created in Xcode and committed | The usual way, but it can only be created and safely edited in Xcode; `project.pbxproj` conflicts are hard to merge; impossible to set up from Linux now. |
| **XcodeGen `project.yml`** | A YAML spec from which `xcodegen` generates the `.xcodeproj`. Writable and reviewable on Linux; the generated project is not committed, so it never conflicts. Cost: one extra tool (`brew install xcodegen`, MIT). |
| Tuist (Swift manifests) | More capable (caching, modularisation) but a heavier toolchain than one app with two targets needs. |
| Swift package only (`Package.swift`) | Swift Package Manager does not describe an iOS app bundle with `Info.plist`, entitlements and signing. Not an option for the app. |

**How Xcode gets the Kotlin code**

| Option | Assessment |
|---|---|
| **Direct integration** — an Xcode build phase runs the Gradle task `embedAndSignAppleFrameworkForXcode` | The standard Kotlin Multiplatform route. The framework is always built from the current source for exactly the architecture and configuration Xcode builds; no extra tool. |
| CocoaPods Gradle plugin | Depends on CocoaPods; rejected. |
| Prebuilt XCFramework through Swift Package Manager | Right for publishing to other projects (ADR-0009 D8); for development inside the repository it adds a manual build step before every Xcode build. |

## Decision

**D1 — Layout.** The iOS shell lives in `app/` like the other shells, in
two parts:

```
app/
├── ios-kit/          Gradle module (Kotlin, iOS targets): the Kotlin side of the
│                     iOS shell; builds the one framework `ThingsMapperKit`
└── ios/              Xcode side (not a Gradle module)
    ├── project.yml          XcodeGen spec — the only project definition
    ├── ThingsMapper/        Swift sources (`@main` app), assets, Info.plist
    ├── ThingsMapperTests/   XCTest target
    ├── Configuration/       Base / Free / Paid .xcconfig, Local.xcconfig.example
    └── README.md            guide for developers on a Mac
```

`ios-kit` depends on the libraries of `lib/`. It produces the one framework
that the Xcode project links — one Kotlin runtime and shared types. That
framework exports the library API that Swift needs (SwiftUI shell) and/or
provides the Compose entry point `MainViewController()` (Compose shell). In
phase 4 it also holds the iOS dependency wiring and the iOS-only port
implementations that are not in `lib/`. This replaces the `lib/ios-umbrella`
module first described in ADR-0009 D7.

**D2 — Xcode project from XcodeGen.** `app/ios/project.yml` is committed.
The `.xcodeproj` is generated with `xcodegen generate` and ignored by git;
CI generates it the same way. After pulling a change to `project.yml`, a
Mac developer runs `xcodegen generate` again. The XcodeGen version is
pinned in the README and in CI.

**D3 — Direct integration.** Each app target has a build phase that runs
before compilation. It sets `JAVA_HOME` with `/usr/libexec/java_home -v 21`
and runs `ios-kit`'s `embedAndSignAppleFrameworkForXcode` through the
Gradle wrapper at the repository root, passing the edition.
- Xcode's user-script sandboxing is switched off for these targets
  (`ENABLE_USER_SCRIPT_SANDBOXING = NO`), because the script reads the
  repository.
- The framework is static.
- CocoaPods is not used; Swift package dependencies, should any be needed,
  are declared in `project.yml`.

**D4 — Editions as two targets.** Two app targets build from the same
sources:

| Target | Edition | Bundle id |
|---|---|---|
| `ThingsMapper` | Free | `nl.pe1mew.thingsmapper` |
| `ThingsMapperPaid` | Paid | `nl.pe1mew.thingsmapper.paid` — suffix still open, TDS-OP-4 |

The build setting `THINGSMAPPER_EDITION` (`free` / `paid`) is passed to
Gradle and becomes Kotlin's build-time `Edition` constant (TDS-9.5). The
App Group for the Free → Paid import (FDS-17.5) is added in WP-4.1, when a
developer account exists.

**D5 — Signing stays out of the repository.**
- `Configuration/Base.xcconfig` ends with `#include? "Local.xcconfig"`.
- Each developer copies `Local.xcconfig.example` to the git-ignored
  `Local.xcconfig` and sets their own `DEVELOPMENT_TEAM`; signing is
  automatic.
- CI builds for the simulator with `CODE_SIGNING_ALLOWED=NO`.
- Release signing (phase 4) comes from CI secrets, as for Android.

**D6 — A skeleton from WP-2.1.** WP-2.1 creates `ios-kit` and the Xcode
project as a minimal app:
- one SwiftUI screen that shows the version and a value computed by
  `core`, which proves the Swift → Kotlin link;
- an XCTest target that calls into `ThingsMapperKit`.

The macOS job of `app.yml` generates the project, builds both targets for
the simulator and runs the tests on every pull request (TC-IO-01/02).

**D7 — Toolchain on the Mac.** The README in `app/ios/` lists:
- macOS on Apple silicon;
- Xcode — the current release that the pinned Kotlin version supports;
- JDK 21, Homebrew and XcodeGen;
- for the Kotlin code, IntelliJ IDEA or Android Studio with the Kotlin
  Multiplatform plugin.

The Android SDK is not needed (ADR-0009 D4). The iOS deployment target is
set in `project.yml`: the lowest version that the pinned Kotlin and Compose
Multiplatform support, chosen in WP-2.1 and reviewed in WP-4.1.

## Consequences

- A developer with a Mac can start on day one:
  1. clone the repository;
  2. install the tools of D7;
  3. run `xcodegen generate`;
  4. open the project and run it.

  Before OP-11 nobody in the project can run that path; CI and TC-IO-03 on
  the first Mac cover it.
- On Linux, `project.yml` and the Swift sources can be edited but not
  built; the macOS CI job is the feedback loop until a Mac is available.
- WP-2.1 grows: besides the two builds it now carries the iOS skeleton and
  the early `.deb` packaging check. Neither costs money: GitHub's macOS and
  arm64 runners are free for public repositories.
- New requirement REQ-P-010; new section TDS-9.7; module `ios-kit`
  (TDS-3.12, replacing `ios-umbrella`); test cases TC-IO-01…03; XcodeGen
  in the TDS dependency table.
- The pinned Kotlin version and the Xcode version on the CI runner must
  stay compatible. CI selects the Xcode version explicitly, and upgrades
  happen at milestone boundaries, like the other toolchain versions
  (risk R-5).

## Links

[ADR-0009](ADR-0009-shared-libraries.md), [ADR-0007](ADR-0007-editions-and-monetisation.md),
[05 TDS §3, §9.5, §9.7, §11](../05-tds.md),
[02 Requirements REQ-P-009, REQ-P-010](../02-requirements.md),
[06 Test plan §6.24](../06-test-plan.md),
[07 Implementation plan WP-2.1, WP-4.1](../07-implementation-plan.md),
[08 Costs and funding I-11…I-13, OP-11](../08-costs-and-funding.md).
