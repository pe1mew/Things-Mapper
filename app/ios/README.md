# app/ios/ — iOS app (Xcode side)

Empty in phase 1. Work package WP-2.1 creates the skeleton app described
here ([ADR-0010](../../design/adr/ADR-0010-ios-development-setup.md),
[TDS-9.7](../../design/05-tds.md)). From then on, CI builds and tests it on
every pull request. The full iOS shell follows in phase 4 (WP-4.1).

This guide is for developers working on a Mac. Everything else in the
repository builds on Linux; this folder needs Xcode.

## What is here (from WP-2.1)

```
app/ios/
├── project.yml           XcodeGen spec — the only project definition
├── ThingsMapper/         Swift sources (@main SwiftUI app), Assets.xcassets, Info.plist
├── ThingsMapperTests/    XCTest target: calls into ThingsMapperKit
└── Configuration/
    ├── Base.xcconfig     shared settings; ends with #include? "Local.xcconfig"
    ├── Free.xcconfig     bundle id nl.pe1mew.thingsmapper,      THINGSMAPPER_EDITION = free
    ├── Paid.xcconfig     bundle id nl.pe1mew.thingsmapper.paid, THINGSMAPPER_EDITION = paid
    └── Local.xcconfig.example   copy to Local.xcconfig (git-ignored) for signing
```

The Kotlin code reaches the app as one framework, `ThingsMapperKit`. The
Gradle module `app/ios-kit` builds it from the shared libraries in
[`lib/`](../../lib/). You don't build it by hand: a build
phase of each app target runs Gradle when Xcode builds.

## Prerequisites

- A Mac with Apple silicon and a current macOS.
- Xcode — the version CI uses is selected in `.github/workflows/app.yml`.
- JDK 21, for Gradle: `brew install --cask temurin@21`. Xcode's build
  phase finds it with `/usr/libexec/java_home -v 21`.
- [Homebrew](https://brew.sh) and XcodeGen: `brew install xcodegen` —
  use the version CI pins in `.github/workflows/app.yml`.
- Optional: IntelliJ IDEA or Android Studio with the Kotlin Multiplatform
  plugin, for editing the Kotlin code.
- Not needed: the Android SDK (the Android target is skipped without it)
  and CocoaPods.

## First run in the simulator

```bash
git clone git@github.com:pe1mew/Things-Mapper.git
cd Things-Mapper/app/ios
xcodegen generate
open ThingsMapper.xcodeproj
```

Choose the scheme `ThingsMapper` (Free edition) or `ThingsMapperPaid`
(Paid edition) and an iPhone simulator, then Run. The first build takes
several minutes: Gradle and Kotlin/Native download their toolchains.

## Running on your own iPhone

```bash
cp Configuration/Local.xcconfig.example Configuration/Local.xcconfig
```

Set `DEVELOPMENT_TEAM` in `Local.xcconfig` to your team id; Xcode then
signs automatically. The file is git-ignored — signing data never goes
into the repository. A free Apple ID can sign for your own device within
Xcode's limits. App Groups, TestFlight and the App Store need the Apple
Developer Program (doc 08 I-13).

## Day to day

- After pulling a change to `project.yml`, run `xcodegen generate` again.
  The `.xcodeproj` is generated and never committed.
- Kotlin changes need no extra step: building in Xcode rebuilds
  `ThingsMapperKit`.
- Build from the command line the way CI does:

  ```bash
  xcodebuild -project ThingsMapper.xcodeproj -scheme ThingsMapper -destination 'generic/platform=iOS Simulator' CODE_SIGNING_ALLOWED=NO build
  ```

- Library tests for iOS run with Gradle from the repository root; see
  [`lib/README.md`](../../lib/README.md).

## Never commit

The generated `ThingsMapper.xcodeproj`, `Configuration/Local.xcconfig`,
`DerivedData/`, `xcuserdata/` and provisioning profiles. All of these are
in `.gitignore`. If `git status` shows one of them, fix `.gitignore`
instead of committing it.
