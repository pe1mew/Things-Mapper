# ADR-0004 — Car platform: Android Auto via the Car App Library, POI category, IoT fallback

Status: **Accepted** · Date: 2026-09-11 (target confirmed by the maintainer)

## Context

"Android car" in the project goal can mean Android Auto (the phone projects
a driving-optimised UI to the head unit) or Android Automotive OS (the app
runs on the car's own Android). Coverage mapping while driving is a core
use case; the driver needs a glanceable map and the key numbers, and must
not type.

## Decision drivers

- Reach: Android Auto works with the user's existing phone in most cars
  sold in the last decade; AAOS is limited to specific car models.
- Rules: both are only reachable through the Android for Cars App Library
  templates for driving use; the app must fit an approved category and pass
  Google's review for Play distribution.
- The map must be drawn by the app (own data), which the library allows only
  for some categories.

## Options

| Option | Assessment |
|---|---|
| **Android Auto with the Car App Library, POI category, `MapWithContentTemplate`** | POI apps may use `MapWithContentTemplate` (app-drawn map via `SurfaceCallback`, requires the `androidx.car.app.MAP_TEMPLATES` permission and Car App API level 7). Gateways-as-points-of-interest is a defensible category fit but not guaranteed to pass review (OP-3). |
| Android Auto, IoT category | Templates without a map (`GridTemplate`, `ListTemplate`): indicators and recent packets only; very likely to pass review; less useful while driving. |
| Android Auto, Navigation category | Full map templates, but the app is not a turn-by-turn navigation app and would not meet the navigation quality requirements; rejected. |
| Android Automotive OS build | Same templates; separate build and review; needs an AAOS device or emulator; deferred. |
| No car integration (phone in a holder, drive view) | Zero effort; already covered by SCR-03; does not meet the project goal. |

## Decision

Build the car experience for **Android Auto** with the Car App Library
(API level 7+): a `MapWithContentTemplate` screen in the **POI** category
with the app-drawn map and an indicator pane, plus a `ListTemplate` for
profile selection. Implement the **IoT-category fallback**
(`GridTemplate`/`ListTemplate`, no map) behind a build flag so that the car
experience can ship even if the POI submission is refused. Android
Automotive OS is a later packaging variant of the same car module.

## Consequences

- Manifest: `CarAppService` with category `androidx.car.app.category.POI`,
  `minCarApiLevel 7`, `MAP_TEMPLATES` permission; `automotive_app_desc.xml`.
- The map renderer must draw to an Android `Surface` (ADR-0003).
- The car UI never asks for text input; profiles come from the phone.
- Testing on the Desktop Head Unit in CI-adjacent manual runs and on a real
  head unit before 1.0 (TC-CA, TC-F-04).
- Play requires an explicit Android Auto review; plan for review time and
  the fallback decision (WP-3.3).

## Links

[02 Requirements §6.13, REQ-C-005](../02-requirements.md), [03 FDS FDS-14](../03-fds.md),
[04 GUI design SCR-40/41](../04-gui-design.md), [05 TDS §9.2](../05-tds.md).
