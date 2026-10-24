# app/

Application source of Things Mapper. Empty in phase 1.

The module layout is defined in the technical design specification
([design/05-tds.md](../design/05-tds.md), section "Module structure") and
is created in phase 2, work package WP-2.1 of the
[implementation plan](../design/07-implementation-plan.md):

```
app/
├── settings.gradle.kts
├── gradle/libs.versions.toml
├── core/            Kotlin Multiplatform: domain, protocol adapters, pipeline, rules
├── data/            Kotlin Multiplatform: SQLDelight storage, settings, secrets ports
├── net/             Kotlin Multiplatform: MQTT and HTTP ports and expect/actual clients
├── map/             Kotlin Multiplatform: map model, tile cache, renderer
├── ui/              Compose Multiplatform: screens, view models, theme
├── android/         Android application (phone/tablet) + Android Auto car service
├── desktop/         Compose for Desktop application (Linux x86-64 and aarch64 / Raspberry Pi)
└── tools/replay/    Developer tool: replays recorded uplinks and GNSS tracks
```
