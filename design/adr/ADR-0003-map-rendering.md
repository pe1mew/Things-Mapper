# ADR-0003 — Map rendering: own raster renderer with `DrawTarget`, MapComposeMP fallback

Status: **Accepted (gated by spike WP-2.4)** · Date: 2026-09-11

## Context

The map must show raster tiles (OpenStreetMap and custom XYZ), the
ttnmapper.org coverage overlay, tens of thousands of coloured points and
lines, gateway markers and the own position — on Compose (phone, desktop,
kiosk) **and** on an Android Auto `Surface` (app-drawn map in
`MapWithContentTemplate`, REQ-F-120), and later on iOS. Performance targets:
≥ 30 fps on a phone and ≥ 15 fps on a Raspberry Pi 4 at 800×480 with
20 k points and 20 k lines (REQ-NF-003). The original used osmdroid with one
overlay object per point (EVAL-18).

## Decision drivers

- One map implementation for all shells including the car surface.
- No hard dependency on OpenGL on the Pi (software rendering must work).
- Modest feature needs: slippy tiles, overlays, gestures, follow mode — no
  vector styling, no 3D, no routing.
- Apache-2.0/MIT-compatible dependencies only.

## Options

| Option | Fit |
|---|---|
| osmdroid | Android only; overlay model is the problem we are replacing. |
| MapLibre Native / MapLibre Compose | Full vector/raster GL engine; Compose bindings for Android, iOS, desktop; API declared unstable; GL requirement and unclear aarch64 Linux desktop artefacts; cannot draw into the car surface without extra work; heavy for the need. |
| **MapComposeMP** (p-lr) | Compose Multiplatform tile map with markers, paths, clustering, rotation; Apache-2.0; stable; works on Android, desktop, iOS, wasm. Compose-only: no way to render to an Android `Surface` for the car; per-marker composables are not designed for 20 k points (paths yes). |
| WebView + Leaflet/MapLibre GL JS | Heavy on Pi 4; no car path; two runtimes. |
| **Own raster renderer** behind a `DrawTarget` abstraction | Small, fully under control, same code on Compose `DrawScope`, Android `Canvas` (car) and later CoreGraphics; batched primitive drawing scales to the target counts; software rendering friendly. Cost: implementing projection, tile pyramid loading with placeholders, disk cache, gestures/fling, hit-testing — an estimated 2–3 weeks; risk of subtle rendering bugs. |

## Decision

Implement an **own lightweight raster-tile renderer** (TDS §7) with a
minimal `DrawTarget` interface and two implementations (Compose, Android
Canvas). The decision is **gated by the spike WP-2.4**: if the benchmark
scene (20 k points, 20 k lines, 200 markers) does not reach ≥ 30 fps on the
reference phone and ≥ 15 fps on a Pi 4 within the time box (3 weeks), the
Compose shells switch to **MapComposeMP** for map display while the car
keeps the own renderer (which then only needs the car's smaller feature
set), and the tile cache/projection code is shared where possible.

## Consequences

- `map` module with projection, tile source/cache/loader, layers with
  world-coordinate buffers and LOD binning, renderer, `MapView` composable.
- Golden-image tests (TC-MA-11) guard rendering regressions.
- Attribution, User-Agent and cache discipline are implemented once
  (REQ-NF-061, NF-006).
- If the fallback is triggered, two renderers coexist (Compose map via
  MapComposeMP, car via own); the domain layer model (`MapModel`, layers)
  stays the same to limit divergence.

## Links

[05 TDS §7 and §13](../05-tds.md), [07 Implementation plan WP-2.4](../07-implementation-plan.md),
[04 GUI design](../04-gui-design.md).
