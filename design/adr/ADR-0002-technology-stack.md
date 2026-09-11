# ADR-0002 — Technology stack: Kotlin Multiplatform + Compose Multiplatform

Status: **Accepted** · Date: 2026-09-11 (confirmed by the maintainer)

## Context

The remake must run on Android phones, Android Auto, Linux desktop and
Raspberry Pi touch kiosks, and keep an iOS shell possible without a rewrite
(REQ-P-001…005). The original is Kotlin/Android with all logic fused to
Android singletons (EVAL-01). The maintainer's aim is to evolve the original
rather than to start from a different ecosystem.

## Decision drivers

- One platform-independent core with a real test suite (REQ-NF-040, 041).
- First-class Android (foreground service, Car App Library) without bridges.
- Runs on Raspberry Pi 4/5 (aarch64 Linux) with touch, ideally without a
  GPU driver dependency.
- iOS reachable later from the same core.
- Continuity with the Kotlin code base and skills.

## Options

| Option | Pros | Cons |
|---|---|---|
| **Kotlin Multiplatform (core) + Compose Multiplatform (UI)** | Direct evolution of the Kotlin app; Android is native (no plugin layer for foreground service or Car App Library); Compose for Desktop supports Linux x86-64 and aarch64; iOS target for the core and Compose iOS are stable; single language, single build system; large library ecosystem (Ktor, SQLDelight, kotlinx). | JVM on the Pi (memory ≈ 150–250 MB, cold start seconds); Compose Desktop needs X11/XWayland and, on the Pi, possibly software rendering; no established multiplatform map widget (see ADR-0003); car UI is templated anyway. |
| Flutter / Dart | Single UI toolkit incl. Linux arm64 (`flutter-pi`) and iOS; `flutter_map` for raster tiles; fast UI iteration. | Nothing reused from the Kotlin app; Android Auto and the foreground service need native Kotlin code behind platform channels; two languages; GPL derived-work argument becomes purely behavioural. |
| Qt / QML (C++ or Python) | Best embedded-Linux/Pi story (EGLFS, no X11), mature touch UI, small footprint. | Android distribution and Car App Library integration are awkward; iOS possible but niche; full rewrite; licensing (LGPL/commercial) considerations for static builds. |
| Native per platform (Kotlin Android + GTK/Rust desktop) | Optimal per platform. | Two or three implementations of the same logic; contradicts REQ-NF-040. |
| Web (Tauri/Electron + Leaflet) | Quick map UI. | Heavy on Pi 4; no path to Android Auto; background operation on Android poor. |

## Decision

Kotlin Multiplatform for `core`, `data`, `net`, `map` and `ui`; Compose
Multiplatform for the phone, desktop and kiosk UI; the Android shell in
Kotlin with the Car App Library for Android Auto; a JVM desktop shell for
Linux x86-64 and aarch64. The core is kept free of platform APIs and is
compiled for `iosArm64` in CI where possible (REQ-P-005).

## Consequences

- Module and port structure of the TDS; expect/actual for MQTT, HTTP engine,
  secure storage, position sources, connectivity.
- The map is either an own renderer or a Compose-only library (ADR-0003).
- Pi builds are produced natively on aarch64 runners (cross-compilation of
  the JVM image is not supported by jpackage); `SKIKO_RENDER_API=SOFTWARE`
  is the safe default until the spike proves GL.
- JVM memory and start-up on the Pi are budgeted (REQ-NF-001, NF-004) and
  measured (TC-P-01/04).
- Versions of Kotlin, Compose Multiplatform and AGP are pinned per milestone.

## Links

[05 TDS §1–3](../05-tds.md), [02 Requirements REQ-C-004](../02-requirements.md).
