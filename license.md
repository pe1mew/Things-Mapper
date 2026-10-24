# License Information

## This repository: GNU GPL v3.0

Everything in this repository — the design documents under `design/`, the
application source that will live under `app/`, the user documentation under
`documentation/` and the build and packaging files — is licensed under the
**GNU General Public License, version 3** (SPDX: `GPL-3.0-only`), with the
[additional permission](#additional-permission-gplv3-section-7) below.
See [LICENSE](LICENSE) for the full text of the GPL.

- New work: Copyright (c) 2026 Remko Welling (PE1MEW).
- Portions derived from TTN Mapper Android v3: Copyright (c) JP Meijers and
  the ttnmapper-android-v3 contributors.

### Why GPL-3.0, and why "only"

Things Mapper is a remake of [ttnmapper-android-v3](https://github.com/ttnmapper/ttnmapper-android-v3),
which is distributed under the GNU GPL v3.0 (its `LICENSE` file is the
unmodified GPLv3 text; the sources carry no per-file notices and no
"or any later version" grant). The design documents evaluate and describe the
original in detail, and the tests reuse its recorded protocol messages, so
the project adopted the original's licence: GPL-3.0, **version 3 only**,
because the upstream grant does not include "or any later version".

Since 2026-10-24 the application code under `app/` is a **clean
implementation**: it is written from the design documents (FDS, TDS), and no
source code is copied from the original. Protocol formats, the ttnmapper.org
upload contract and the mapping rules are used as functional
specifications. The material that does come from the original — excerpts in
the evaluation document and the recorded test messages — stays under the
plain GPL-3.0 and is never part of a distributed application. This is what
allows the additional permission below. The full reasoning is recorded in
[design/adr/ADR-0001-license.md](design/adr/ADR-0001-license.md).

Practical consequences for contributors and users:

- Source files carry an SPDX header `SPDX-License-Identifier: GPL-3.0-only`
  followed by the line `Additional permission: see license.md (GPLv3 section 7)`.
- Contributions are accepted under the same licence and additional
  permission (inbound = outbound); contributors do not copy code from
  ttnmapper-android-v3 into `app/`.
- Runtime dependencies must be GPL-3.0-compatible (Apache-2.0, MIT, BSD,
  LGPL, MPL-2.0 are; proprietary SDKs are not), with the single exception
  of store and platform vendor libraries under permission (a) — never
  advertising, analytics or tracking libraries. See the dependency table
  in the TDS.
- Binary distributions (APK, Android Auto, `.deb`, Raspberry Pi images) must
  offer the corresponding source, which this public repository does.

### Additional permission (GPLv3 section 7)

In force since 2026-10-24 (licence route L-A of
[ADR-0007](design/adr/ADR-0007-editions-and-monetisation.md)).

> **Additional permission under GNU GPL version 3 section 7**
>
> As copyright holders of Things Mapper we grant the following additional
> permissions:
>
> **(a) Store and platform libraries.** You may combine Things Mapper with
> libraries that are published by the vendor of an operating system or of an
> application store for use by applications on that platform — for example
> Google Play services and the Google Play Billing Library — and convey the
> combination, even though those libraries are not available under
> GPL-compatible terms. This permission does not extend to advertising,
> analytics, tracking or crash-reporting libraries.
>
> **(b) Application stores.** You may convey Things Mapper in object code form
> through application stores whose terms of service impose restrictions on
> recipients that the GPL would otherwise not allow (for example the Apple
> App Store), provided that the Corresponding Source is available to
> everyone under the terms of this licence, free of charge.
>
> The libraries in (a) remain under their own terms; these permissions grant
> no rights to them. When you convey a copy of Things Mapper you may, at your
> option, remove these additional permissions from that copy (GPLv3 section
> 7). These permissions apply to the parts of Things Mapper of which we are
> the copyright holders. They do not apply to material taken from
> ttnmapper-android-v3 — the excerpts in the evaluation document and the
> recorded test messages in the test-only directory
> `testing/upstream-fixtures/` — which is used under the plain GPL-3.0 and is
> never part of a distributed application.

Why: the paid and Free editions are distributed through Google Play and —
later — the Apple App Store, and the `play` build variants use Google Play
services for location. Advertisements come from the project's own sponsor
catalogue, which needs no third-party library; ad-network and analytics
SDKs are deliberately excluded. The permission is possible because the
application code is a clean implementation (no source code copied from
ttnmapper-android-v3, see below). This is the maintainer's assessment, not
legal advice; it should be reviewed before the first App Store submission.

## Third-party components and derived work

| Component | Author / copyright | Licence | Role in Things Mapper |
|---|---|---|---|
| [ttnmapper-android-v3](https://github.com/ttnmapper/ttnmapper-android-v3) | JP Meijers and contributors | GPL-3.0 | Git submodule; study reference and **origin of this repository's licence**; its behaviour and rules are specified in the design documents; its recorded test messages are test-only fixtures (plain GPL-3.0); no source code is copied into `app/` |
| [TTN Mapper ingress-api](https://github.com/ttnmapper/ingress-api) | JP Meijers | MIT | Protocol reference for the measurement upload (`/android/v4`); no code included |
| [TTN Mapper documentation](https://docs.ttnmapper.org) | JP Meijers and contributors | MIT | Source of the mapping rules (accuracy, HDOP, experiments, gateway location) |
| [The Things Stack](https://github.com/TheThingsNetwork/lorawan-stack) API definitions | The Things Network Foundation / The Things Industries | Apache-2.0 | Reference for the TTS v3 MQTT uplink JSON; Things Mapper hand-writes a minimal model, no generated code is vendored |
| [ChirpStack](https://github.com/chirpstack/chirpstack) API definitions | Orne Brocaar | MIT | Reference for the ChirpStack v4 MQTT uplink event JSON |
| Kotlin, Kotlin Multiplatform, kotlinx libraries | JetBrains and contributors | Apache-2.0 | Language and standard libraries |
| Compose Multiplatform, Skiko | JetBrains | Apache-2.0 | UI toolkit on Android, Linux desktop and Raspberry Pi |
| AndroidX (incl. Car App Library) | The Android Open Source Project | Apache-2.0 | Android shell, foreground service, Android Auto |
| Ktor | JetBrains | Apache-2.0 | HTTP client (uploads, tiles, gateway details) |
| SQLDelight | Cash App | Apache-2.0 | Local measurement database |
| HiveMQ MQTT Client | HiveMQ / dc-square | Apache-2.0 | MQTT 3.1.1/5 client on JVM/Android (candidate, see TDS) |
| Google Play services location (`play-services-location`) | Google | proprietary (Google terms) | `play` build variants only, under additional permission (a); absent from the `foss` variants |
| Google Play Billing Library | Google | proprietary (Google terms) | `play` build variants only (supporter subscription and tips), under additional permission (a); absent from the `foss` variants |
| Koin, Kermit, multiplatform-settings | respective authors | Apache-2.0 | Dependency injection, logging, settings |
| jSerialComm | Fazecast | LGPL-3.0 / Apache-2.0 (dual) | Serial NMEA GNSS input on Linux (optional) |
| OpenStreetMap | OpenStreetMap contributors | ODbL (data); tile servers under the [OSMF tile usage policy](https://operations.osmfoundation.org/policies/tiles/) | Base map tiles; attribution is shown on the map |
| ttnmapper.org coverage tiles and gateway data | JP Meijers / TTN Mapper | ttnmapper.org terms | Optional coverage overlay and gateway details |

The exact set and versions of runtime dependencies are pinned in the build
files and enumerated in a generated licence report once the implementation
exists (see the implementation plan, work package "release engineering").

## Trademarks

"TTN Mapper" is the name of JP Meijers' project; "The Things Network" and
"The Things Stack" are trademarks of The Things Industries B.V.; "ChirpStack"
is a trademark of Orne Brocaar; "Android", "Android Auto" and "Google Play"
are trademarks of Google LLC; "Raspberry Pi" is a trademark of Raspberry Pi
Ltd. Things Mapper does not ship third-party logos and does not claim any
affiliation with or endorsement by these parties. Icons in the application
are original or taken from libraries with a GPL-compatible licence.
