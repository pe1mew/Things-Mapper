# Architecture decision records

One file per lasting decision. Format: context → drivers → options →
decision → consequences. Status values: *Proposed*, *Accepted*,
*Superseded by ADR-nnnn*.

| ADR | Title | Status |
|---|---|---|
| [ADR-0001](ADR-0001-license.md) | Licence: GPL-3.0-only | Accepted, amended by ADR-0007 |
| [ADR-0002](ADR-0002-technology-stack.md) | Technology stack: Kotlin Multiplatform + Compose Multiplatform | Accepted |
| [ADR-0003](ADR-0003-map-rendering.md) | Map rendering: own raster renderer with `DrawTarget`, MapComposeMP fallback | Accepted (spike-gated) |
| [ADR-0004](ADR-0004-car-platform.md) | Car platform: Android Auto via Car App Library, POI category, IoT fallback | Accepted, amended by ADR-0007 (Paid edition only) |
| [ADR-0005](ADR-0005-storage-and-outbox.md) | Storage: SQLDelight with transactional outbox | Accepted |
| [ADR-0006](ADR-0006-mqtt-client.md) | MQTT client: port with HiveMQ MQTT Client implementation | Accepted |
| [ADR-0007](ADR-0007-editions-and-monetisation.md) | Editions: Free with ads, Paid without ads and with Android Auto | Accepted (sponsor catalogue; licence route L-A with additional permission) |
| [ADR-0008](ADR-0008-cost-awareness-and-support.md) | Cost awareness and voluntary support; Paid price €6; no commercial services | Accepted |

To add one: copy the structure of ADR-0006, take the next number, link it
from the TDS where the decision applies, and add a row here.
