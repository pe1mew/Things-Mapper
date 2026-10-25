# app/

Application shells of Things Mapper. Empty in phase 1.

The shells are thin. They wire the shared libraries of [`lib/`](../lib/)
together and own the platform lifecycle and packaging. `app/` is a Gradle
build of its own that includes `lib/` as a composite build
([ADR-0009](../design/adr/ADR-0009-shared-libraries.md)), so the shells
always build against the current library source. The module layout is
defined in the technical design specification
([design/05-tds.md](../design/05-tds.md), section "Module structure"). It is
created in phase 2, work package WP-2.1 of the
[implementation plan](../design/07-implementation-plan.md):

```
app/
├── settings.gradle.kts   includeBuild("../lib")
├── android/              Android application (phone/tablet) + Android Auto car service
├── desktop/              Compose for Desktop application (Linux x86-64 and aarch64 / Raspberry Pi)
├── ios-kit/              Kotlin side of the iOS app: builds the framework ThingsMapperKit
├── ios/                  Xcode side of the iOS app (XcodeGen project; not a Gradle module)
└── tools/replay/         Developer tool: replays recorded uplinks and GNSS tracks
```

The iOS app needs a Mac with Xcode; see [`ios/README.md`](ios/README.md)
and [ADR-0010](../design/adr/ADR-0010-ios-development-setup.md). The
desktop window and the iOS app exist as skeletons from WP-2.1, so CI
packages and builds them on every pull request.

The Gradle wrapper, the version catalogue (`gradle/libs.versions.toml`)
and the convention plugins (`build-logic/`) sit at the repository root and
are shared with `lib/`.
