# License Information

## This repository: GNU GPL v3.0

Everything in this repository — the design documents under `design/`, the
application source that will live under `app/`, the user documentation under
`documentation/` and the build and packaging files — is licensed under the
**GNU General Public License, version 3** (SPDX: `GPL-3.0-only`).
See [LICENSE](LICENSE) for the full text.

- New work: Copyright (c) 2026 Remko Welling (PE1MEW).
- Portions derived from TTN Mapper Android v3: Copyright (c) JP Meijers and
  the ttnmapper-android-v3 contributors.

### Why GPL-3.0, and why "only"

Chirp Mapper is a remake of [ttnmapper-android-v3](https://github.com/ttnmapper/ttnmapper-android-v3),
which is distributed under the GNU GPL v3.0 (its `LICENSE` file is the
unmodified GPLv3 text; the sources carry no per-file notices and no
"or any later version" grant). The remake is a **derived work**: its
behaviour, data contracts and parts of its code (protocol mappings, the
signal-level colour scale, the recorded test fixtures, the mapping rules) are
taken from, or closely follow, the original. Section 5 of the GPL requires a
derived work to be licensed as a whole under the same licence, so this
repository uses GPL-3.0. Because the upstream grant does not include "or any
later version", the combined work is licensed under **version 3 only**;
adopting a later GPL version would require the agreement of the upstream
copyright holders. The full reasoning is recorded in
[design/adr/ADR-0001-license.md](design/adr/ADR-0001-license.md).

Practical consequences for contributors and users:

- Source files carry an SPDX header: `SPDX-License-Identifier: GPL-3.0-only`.
- Contributions are accepted under the same licence (inbound = outbound).
- Runtime dependencies must be GPL-3.0-compatible (Apache-2.0, MIT, BSD,
  LGPL, MPL-2.0 are; proprietary SDKs are not). See the dependency table
  in the TDS.
- Binary distributions (APK, Android Auto, `.deb`, Raspberry Pi images) must
  offer the corresponding source, which this public repository does.

## Third-party components and derived work

| Component | Author / copyright | Licence | Role in Chirp Mapper |
|---|---|---|---|
| [ttnmapper-android-v3](https://github.com/ttnmapper/ttnmapper-android-v3) | JP Meijers and contributors | GPL-3.0 | Git submodule; study reference and **origin of this repository's licence**; protocol mappings, colour scale, mapping rules and test fixtures are derived from it |
| [TTN Mapper ingress-api](https://github.com/ttnmapper/ingress-api) | JP Meijers | MIT | Protocol reference for the measurement upload (`/android/v4`); no code included |
| [TTN Mapper documentation](https://docs.ttnmapper.org) | JP Meijers and contributors | MIT | Source of the mapping rules (accuracy, HDOP, experiments, gateway location) |
| [The Things Stack](https://github.com/TheThingsNetwork/lorawan-stack) API definitions | The Things Network Foundation / The Things Industries | Apache-2.0 | Reference for the TTS v3 MQTT uplink JSON; Chirp Mapper hand-writes a minimal model, no generated code is vendored |
| [ChirpStack](https://github.com/chirpstack/chirpstack) API definitions | Orne Brocaar | MIT | Reference for the ChirpStack v4 MQTT uplink event JSON |
| Kotlin, Kotlin Multiplatform, kotlinx libraries | JetBrains and contributors | Apache-2.0 | Language and standard libraries |
| Compose Multiplatform, Skiko | JetBrains | Apache-2.0 | UI toolkit on Android, Linux desktop and Raspberry Pi |
| AndroidX (incl. Car App Library) | The Android Open Source Project | Apache-2.0 | Android shell, foreground service, Android Auto |
| Ktor | JetBrains | Apache-2.0 | HTTP client (uploads, tiles, gateway details) |
| SQLDelight | Cash App | Apache-2.0 | Local measurement database |
| HiveMQ MQTT Client | HiveMQ / dc-square | Apache-2.0 | MQTT 3.1.1/5 client on JVM/Android (candidate, see TDS) |
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
Ltd. Chirp Mapper does not ship third-party logos and does not claim any
affiliation with or endorsement by these parties. Icons in the application
are original or taken from libraries with a GPL-compatible licence.
