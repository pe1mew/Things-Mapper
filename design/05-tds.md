# 05 — Technical design specification (TDS)

Status: **Draft**

| Rev | Date | Author | Change |
|---|---|---|---|
| 0.1 | 2026-09-11 | R. Welling (with Claude) | First complete version |

Related: [02 Requirements](02-requirements.md), [03 FDS](03-fds.md),
[04 GUI design](04-gui-design.md), [06 Test plan](06-test-plan.md),
[07 Implementation plan](07-implementation-plan.md), decisions in
[adr/](adr/).

Identifiers: **TDS-n.m** = design element (module, component, mechanism).

---

## 1. Introduction

### 1.1 Purpose and scope

This document describes *how* Chirp Mapper is built: the module structure,
the interfaces between layers, the data model, the protocol adapters, the
map renderer, the platform shells, the cross-cutting mechanisms (security,
logging, configuration) and the build and release pipeline. It is the
blueprint for phase 2 and 3 of the implementation plan.

### 1.2 Design drivers

| Driver | Source | Architectural answer |
|---|---|---|
| One core for phone, car, desktop, kiosk — and iOS later | REQ-NF-040, REQ-P-005 | Kotlin Multiplatform; all logic in common code behind ports ([ADR-0002](adr/ADR-0002-technology-stack.md)) |
| Nothing lost, ever | REQ-NF-010, REQ-F-082 | Store-first pipeline, transactional outbox ([ADR-0005](adr/ADR-0005-storage-and-outbox.md)) |
| Ordered, bounded processing | REQ-F-045 | Single pipeline actor with a bounded channel |
| Same map on Compose and on a car `Surface` | REQ-F-120, REQ-F-064 | Own raster-tile renderer drawing through a `DrawTarget` abstraction ([ADR-0003](adr/ADR-0003-map-rendering.md)) |
| TLS by default, secrets never leak | REQ-NF-020, REQ-NF-021 | Secure-storage port, redaction at the logging sink, TLS-first clients |
| Runs on a Raspberry Pi 4 without a GPU driver stack | REQ-P-004, REQ-C-008 | Compose for Desktop with Skia software rendering fallback; no OpenGL-only libraries |
| Testable without hardware | REQ-F-140, REQ-NF-041 | Ports have in-memory/replay implementations; replay tool |
| No proprietary SDK in the core | REQ-C-003 | GMS only in an optional Android flavour |

### 1.3 Decisions

| ADR | Decision |
|---|---|
| [ADR-0001](adr/ADR-0001-license.md) | GPL-3.0-only for the repository |
| [ADR-0002](adr/ADR-0002-technology-stack.md) | Kotlin Multiplatform + Compose Multiplatform; Android Auto via Car App Library |
| [ADR-0003](adr/ADR-0003-map-rendering.md) | Own lightweight raster map renderer with `DrawTarget` abstraction; MapComposeMP as fallback for Compose shells |
| [ADR-0004](adr/ADR-0004-car-platform.md) | Android Auto first, POI category with app-drawn map, IoT fallback |
| [ADR-0005](adr/ADR-0005-storage-and-outbox.md) | SQLDelight database, transactional outbox for uploads |
| [ADR-0006](adr/ADR-0006-mqtt-client.md) | MQTT port with HiveMQ MQTT Client as JVM/Android implementation |

## 2. Architecture overview

### 2.1 Layers

```mermaid
flowchart TB
    subgraph Shells["Shells (platform code)"]
        AND[android: phone app,<br/>foreground service,<br/>Car App service]
        DESK[desktop: Compose for Desktop,<br/>kiosk, systemd, gpsd/NMEA]
        REPLAY[tools/replay CLI]
    end
    subgraph UI["ui (Compose Multiplatform, common)"]
        SCREENS[screens · view models · theme · adaptive layouts]
    end
    subgraph Core["core (common Kotlin)"]
        DOM[domain model]
        UC[use cases: SessionController,<br/>MeasurementPipeline, UploadWorker,<br/>GatewayRegistry, Statistics]
        PORTS[ports: LinkPort, PositionSource,<br/>MeasurementStore, OutboxStore,<br/>SecretStore, SettingsStore, Clock, Log]
        ADAPT[protocol adapters:<br/>TtsV3, ChirpStackV4, (Generic)]
        RULES[rules: level, gateway, normalisation]
    end
    subgraph Infra["infrastructure modules (common + expect/actual)"]
        DATA[data: SQLDelight, settings, secrets]
        NET[net: MQTT (HiveMQ), HTTP (Ktor),<br/>connectivity, TLS]
        MAP[map: model, tiles, cache, renderer]
    end
    AND --> UI
    DESK --> UI
    AND --> Core
    DESK --> Core
    REPLAY --> Core
    UI --> Core
    UI --> MAP
    Core --> PORTS
    DATA -. implements .-> PORTS
    NET -. implements .-> PORTS
    MAP --> NET
```

Dependency rule: arrows point inwards. `core` depends on nothing but the
Kotlin standard library, `kotlinx.coroutines`, `kotlinx.serialization`,
`kotlinx.datetime` and `kotlinx.io`. Infrastructure modules implement the
core's ports. Shells wire everything together (dependency injection) and own
the platform lifecycle.

### 2.2 Runtime views

**Android phone.** One process. `MainActivity` hosts the Compose UI.
`MappingService` (foreground, type `location`) owns the `SessionController`
while a session runs; the UI observes the same singleton state through the
DI graph, so the activity can be destroyed and recreated freely. The Car App
`Session`/`Screen`s run in the same process and observe the same state.

**Linux desktop / Raspberry Pi.** One JVM process started by the user or by
a systemd user unit; the Compose window is the only UI; the session lives as
long as the process. Optional embedded status HTTP server.

**Replay tool.** Separate JVM CLI process that uses `core` and `net` without
`ui`.

### 2.3 Threading and concurrency model (TDS-2.3)

- Everything is `kotlinx.coroutines`. Blocking I/O (SQLite, sockets) runs on
  `Dispatchers.IO`; UI state is exposed as `StateFlow` and collected on the
  main dispatcher.
- **Pipeline actor:** `MeasurementPipeline` owns a
  `Channel<RawMessage>(capacity = 1000, onBufferOverflow = DROP_OLDEST)` with
  an `onUndeliveredElement` hook that increments the *dropped* counter; a
  single coroutine consumes it, so processing is strictly ordered and there is
  exactly one writer to the database per session.
- **Fix selection** runs in the position source's coroutine and publishes an
  immutable `PositionState` via `StateFlow`; the pipeline reads a snapshot.
- **Upload workers:** one per target, each a coroutine with a semaphore of 2
  in-flight requests, woken by a `Channel<Unit>` (conflated) on new outbox
  rows, backoff timers and connectivity events.
- **Map rendering:** the renderer is pure and synchronous; Compose calls it
  from the draw phase; the car surface renderer calls it from a dedicated
  render thread at ≤ 30 fps.
- No global mutable singletons: state lives in objects created by the DI
  graph with explicit scopes (application, session).

## 3. Module structure (TDS-3)

| Module (TDS id) | Type | Targets | Contents |
|---|---|---|---|
| `core` (TDS-3.1) | KMP library | android, jvm, iosArm64 (compile) | Domain, ports, use cases, adapters, rules, statistics, exporters (CSV/GeoJSON), replay format |
| `data` (TDS-3.2) | KMP library | android, jvm, iosArm64 | SQLDelight schema and repositories, settings, secret store expect/actual |
| `net` (TDS-3.3) | KMP library | android, jvm, iosArm64 (HTTP only) | MQTT port implementation (HiveMQ on JVM/Android), Ktor HTTP client, TLS helpers, connectivity monitor |
| `map` (TDS-3.4) | KMP library | android, jvm, iosArm64 | Map model, projection, tile source/cache, renderer, `DrawTarget` implementations (Compose, Android Canvas) |
| `ui` (TDS-3.5) | Compose Multiplatform library | android, jvm, iosArm64 | Screens, view models, theme, navigation, resources (strings), on-screen keyboard |
| `android` (TDS-3.6) | Android application | android | `Application`, `MainActivity`, `MappingService`, notifications, permissions, position sources, Car App service and screens, flavours `foss`/`play` |
| `desktop` (TDS-3.7) | JVM application | jvm | `main()`, window/kiosk, config file, gpsd/NMEA sources, systemd unit, packaging, status server |
| `tools/replay` (TDS-3.8) | JVM application | jvm | Record, publish, track modes |
| `testing` (TDS-3.9) | KMP library (test fixtures) | all | Fixture messages, fake ports, golden files |

Package root: `nl.pe1mew.chirpmapper` (open point TDS-OP-1 to confirm the
reverse-domain). Android `applicationId` = package root; desktop package
name `chirpmapper`.

Gradle: Kotlin DSL, version catalogue (`gradle/libs.versions.toml`),
convention plugins in `build-logic/` for the KMP library, Compose library
and application modules; `kotlin { compilerOptions { allWarningsAsErrors } }`
for `core`; explicit API mode for `core`, `data`, `net`, `map`.

## 4. Core (TDS-4)

### 4.1 Domain model (TDS-4.1)

Immutable Kotlin data classes mirroring FDS §3.1; value classes for
identifiers and units to prevent unit mix-ups:

```kotlin
@JvmInline value class Hz(val value: Long)
@JvmInline value class EpochNanos(val value: Long)
@JvmInline value class Dbm(val value: Double)
@JvmInline value class Db(val value: Double)

data class Measurement(
    val id: MeasurementId, val sessionId: SessionId, val receivedAt: EpochNanos,
    val networkId: String?, val device: DeviceIds, val time: EpochNanos,
    val fPort: Int?, val fCnt: Long?, val radio: RadioParams,
    val position: Position?, val positionAgeMs: Long?,
    val receptions: List<Reception>, val bestLevel: Dbm?, val pbOnly: Boolean,
    val experiment: String?, val flags: Set<MeasurementFlag>, // DUPLICATE, IMPORTED, TIME_ESTIMATED
)
```

`Level.of(rssi, snr)` and `LevelClass.of(level)` implement REQ-F-040
(`LevelClass` = RED, ORANGE, YELLOW, GREEN, CYAN, BLUE, UNKNOWN with the
legend colours as constants).

### 4.2 Ports (TDS-4.2)

```kotlin
interface LinkPort {                       // implemented by net (MQTT)
    val state: StateFlow<LinkState>
    val messages: Flow<RawMessage>          // topic, payload bytes, receivedAt
    suspend fun connect(config: LinkConfig, secret: Secret)
    suspend fun disconnect()
}
interface PositionSource {                 // gps, gpsd, nmea, manual, replay, car
    val fixes: Flow<Fix>; val status: StateFlow<SourceStatus>
    suspend fun start(); suspend fun stop()
}
interface MeasurementStore {               // data (SQLDelight)
    suspend fun storeUplink(tx: StoredUplink)      // measurement + receptions + gateway upserts + outbox rows, one transaction
    fun observeSession(id: SessionId): Flow<SessionSnapshot>
    fun measurements(id: SessionId): Flow<Measurement>   // streaming, for export/map
    // sessions, gateways, statistics queries …
}
interface OutboxStore { suspend fun nextBatch(target: UploadTarget, n: Int): List<OutboxItem>; suspend fun mark(item, result) … }
interface SecretStore { suspend fun put(ref: SecretRef, value: Secret); suspend fun get(ref): Secret?; suspend fun delete(ref) }
interface SettingsStore { fun <T> observe(key: SettingKey<T>): StateFlow<T>; suspend fun <T> set(key, value) }
interface Clock { fun now(): Instant; fun monotonicNanos(): Long }
interface Log { fun log(level, tag, message: () -> String, throwable: Throwable? = null) }
interface Connectivity { val online: StateFlow<Boolean> }
```

Test doubles for every port live in `testing` (in-memory store, scripted
link, replay position source).

### 4.3 Use cases (TDS-4.3)

| Component | Responsibility | FDS |
|---|---|---|
| `SessionController` | State machine FDS §5.1; owns the session scope; starts/stops link, position source, pipeline and upload workers; publishes `SessionState` | FDS-2 |
| `MeasurementPipeline` | Actor of TDS-2.3: parse (adapter registry) → dedupe → attach fix → rules → `storeUplink` → emit `MeasurementEvent` | FDS-3.2–3.5, FDS-5 |
| `FixSelector` | Implements FDS-4.2/4.3 from one or more `PositionSource`s; publishes `PositionState` (current fix, quality, age) | FDS-4 |
| `GatewayRegistry` | In-memory cache of the gateway table for the session; validation; *moved* detection | FDS-5.3 |
| `UploadWorker` | Per target: batch, build payload (`IngressPayloadBuilder`), send through `HttpPort`, apply FDS §5.3 state machine with backoff schedule | FDS-10 |
| `Statistics` | Session statistics and per-gateway aggregates from the store | FDS-9.2 |
| `Exporters` | Streaming CSV (RFC 4180) and GeoJSON writers over `kotlinx.io` sinks | FDS-7 |
| `Importer` | Legacy TTN Mapper CSV and Chirp Mapper CSV → imported session | FDS-7.3 |
| `AlertEngine` | Evaluates alert conditions from `SessionState`, `PositionState`, `LinkState`, last packet | FDS-11 |
| `Diagnostics` | Ring buffer of events, counters, redacted samples, bundle builder | FDS-13 |
| `ProfileService` | CRUD, validation, topic derivation, import/export/QR payload, connection test orchestration | FDS-1 |

### 4.4 Protocol adapters (TDS-4.4)

```kotlin
interface UplinkAdapter {
    val type: NetworkServerType
    fun topics(profile: Profile): List<String>
    fun accepts(profile: Profile, topic: String): Boolean        // device filter for wildcard subscriptions
    fun parse(raw: RawMessage, profile: Profile): ParseResult     // Parsed | Ignored(reason) | Error(reason, redactedSample)
}
```

- `TtsV3Adapter` and `ChirpStackV4Adapter` implement FDS §3.2 with
  `kotlinx.serialization` (`ignoreUnknownKeys = true`, `isLenient = true`,
  `coerceInputValues = true`) against small hand-written wire models; 64-bit
  integers accept numbers and strings via a custom serializer;
  RFC 3339 timestamps with nanoseconds are parsed by a dedicated parser (no
  platform date APIs).
- `PacketBrokerRules` (FDS §3.3), `Normalisation` (FDS-5.5) are shared.
- `AdapterRegistry` maps `NetworkServerType` → adapter; adding an adapter is
  a new class plus one registration line (REQ-F-008); the profile form is
  described by a small schema (`ProfileFieldSpec`) that the UI renders
  generically, so a new adapter does not need new UI code.
- `GenericJsonAdapter` (priority C) evaluates user-supplied JSON pointers.
- The legacy ChirpStack v3 format is *detected* (top-level `devEUI`) and
  reported, and parsed only when the optional `ChirpStackV3Adapter` is
  registered.

### 4.5 Rules (TDS-4.5)

Pure functions with exhaustive tests: `Level`, `LevelClass`,
`GatewayLocation.isValid`, `Eui.fromGatewayId`, `NetId.format`,
`CodingRate.normalise`, `Freshness.isFresh(fix, at, window)`,
`FixSelector.choose(candidates)`.

### 4.6 Replay and recording format (TDS-4.6)

JSON Lines: `{"t": <epoch ms>, "topic": "...", "payload": "<base64>"}` for
messages; GPX 1.1 or raw NMEA for tracks. `ReplayLink : LinkPort` and
`ReplayPositionSource : PositionSource` read them with a speed factor and
are used by tests, the replay tool and the in-app demo mode.

## 5. Data (TDS-5)

### 5.1 Database (TDS-5.1)

SQLDelight 2.x with the Android driver, the JVM SQLite driver (bundled
`sqlite-jdbc`, works on aarch64) and the native driver for iOS later.
`PRAGMA journal_mode=WAL`, `synchronous=NORMAL`, foreign keys on.

```sql
CREATE TABLE session (
  id TEXT PRIMARY KEY, profile_id TEXT NOT NULL, platform TEXT NOT NULL, app_version TEXT NOT NULL,
  started_at INTEGER NOT NULL, ended_at INTEGER, experiment TEXT, flags INTEGER NOT NULL DEFAULT 0);
CREATE TABLE measurement (
  id TEXT PRIMARY KEY, session_id TEXT NOT NULL REFERENCES session(id) ON DELETE CASCADE,
  received_at INTEGER NOT NULL, network_id TEXT, app_id TEXT, dev_id TEXT, dev_eui TEXT,
  time INTEGER NOT NULL, f_port INTEGER, f_cnt INTEGER,
  frequency INTEGER, modulation TEXT, bandwidth INTEGER, spreading_factor INTEGER, bit_rate INTEGER, coding_rate TEXT,
  lat REAL, lon REAL, alt REAL, accuracy_m REAL, hdop REAL, satellites INTEGER, speed REAL, bearing REAL,
  fix_time INTEGER, position_source TEXT, position_age_ms INTEGER,
  best_level REAL, pb_only INTEGER NOT NULL DEFAULT 0, experiment TEXT, flags INTEGER NOT NULL DEFAULT 0);
CREATE INDEX measurement_session_time ON measurement(session_id, time);
CREATE TABLE reception (
  measurement_id TEXT NOT NULL REFERENCES measurement(id) ON DELETE CASCADE, idx INTEGER NOT NULL,
  network_id TEXT, gateway_id TEXT NOT NULL, gateway_eui TEXT, antenna INTEGER,
  time INTEGER, timestamp INTEGER, fine_timestamp INTEGER, enc_fine_ts TEXT, enc_fine_ts_key TEXT,
  channel INTEGER, rssi REAL, signal_rssi REAL, snr REAL, level REAL,
  gw_lat REAL, gw_lon REAL, gw_alt REAL, gw_accuracy REAL, gw_location_source TEXT, description TEXT,
  via_packet_broker INTEGER NOT NULL DEFAULT 0, PRIMARY KEY (measurement_id, idx));
CREATE INDEX reception_gateway ON reception(gateway_id);
CREATE TABLE gateway (
  network_id TEXT NOT NULL DEFAULT '', gateway_id TEXT NOT NULL, eui TEXT, description TEXT,
  lat REAL, lon REAL, alt REAL, location_source TEXT, moved INTEGER NOT NULL DEFAULT 0,
  first_seen INTEGER NOT NULL, last_seen INTEGER NOT NULL, receptions INTEGER NOT NULL DEFAULT 0, best_level REAL,
  PRIMARY KEY (network_id, gateway_id));
CREATE TABLE outbox (
  measurement_id TEXT NOT NULL REFERENCES measurement(id) ON DELETE CASCADE, target TEXT NOT NULL,
  state TEXT NOT NULL, attempts INTEGER NOT NULL DEFAULT 0, next_attempt_at INTEGER NOT NULL,
  last_error TEXT, last_response TEXT, created_at INTEGER NOT NULL, PRIMARY KEY (measurement_id, target));
CREATE INDEX outbox_due ON outbox(target, state, next_attempt_at);
CREATE TABLE profile (
  id TEXT PRIMARY KEY, name TEXT NOT NULL, type TEXT NOT NULL, host TEXT NOT NULL, port INTEGER NOT NULL,
  tls INTEGER NOT NULL, insecure_ack INTEGER NOT NULL DEFAULT 0, application_id TEXT, tenant_id TEXT,
  device_ids TEXT NOT NULL, topic_override TEXT, username TEXT, contribute INTEGER NOT NULL DEFAULT 1,
  secret_ref TEXT, source TEXT NOT NULL DEFAULT 'user', created_at INTEGER NOT NULL, updated_at INTEGER NOT NULL);
CREATE TABLE diagnostics_event (id INTEGER PRIMARY KEY AUTOINCREMENT, at INTEGER NOT NULL, kind TEXT NOT NULL, message TEXT NOT NULL);
```

Measurement ids are UUIDv7 (time-ordered). Deleting a session cascades.
Migrations are numbered `.sqm` files, verified by SQLDelight's schema
verification task and by tests that open a database of every released
version.

### 5.2 Transactional outbox (TDS-5.2)

`storeUplink` writes measurement, receptions, gateway upserts and the outbox
rows (ttnmapper.org and/or custom, according to FDS-10.1) in **one**
transaction and then signals the upload workers. Workers read
`outbox_due`-indexed batches, mark `InFlight` in a transaction, send, and
mark the result. A crash between send and mark leads to at most one
duplicate upload, which ttnmapper.org tolerates (at-least-once,
REQ-NF-010). See [ADR-0005](adr/ADR-0005-storage-and-outbox.md).

### 5.3 Settings (TDS-5.3)

`multiplatform-settings` (Android `SharedPreferences`, JVM `Preferences`
backed by a file under `$XDG_CONFIG_HOME/chirpmapper/`), wrapped by a typed
`SettingKey<T>` registry with defaults and migration of keys. The kiosk
config file (TOML via `tomlkt`) overlays settings as read-only values.

### 5.4 Secret store (TDS-5.4)

`expect class PlatformSecretStore : SecretStore`.

- **Android:** AES-256-GCM with a key generated in the Android Keystore
  (`setUserAuthenticationRequired(false)`, StrongBox when available),
  ciphertexts in a private file; the deprecated `security-crypto` library is
  not used.
- **JVM (Linux):** if the Secret Service D-Bus API is reachable
  (`org.freedesktop.secrets`, via `dbus-java`), secrets go there; otherwise a
  file `secrets.json` under the config directory with mode `0600`
  (`PosixFilePermissions`) and a one-time warning in the UI; kiosk
  provisioning secrets are read from `*_file`/`*_env` and cached in memory
  only.
- The `Secret` type never implements `toString()` with its content; logging
  a `Secret` prints `«secret»`.

## 6. Network (TDS-6)

### 6.1 MQTT (TDS-6.1)

`MqttLink : LinkPort`, JVM/Android implementation on **HiveMQ MQTT Client**
(Apache-2.0, MQTT 3.1.1 and 5, TLS via `SSLContext`, Netty based, works on
Android API 24+ and on JVM; [ADR-0006](adr/ADR-0006-mqtt-client.md)). Settings
per FDS-3.1: MQTT 3.1.1 for TTS (only version it supports), clean session,
keep-alive 30 s, QoS 0, client id `chirpmapper-` + 8 random base32 chars,
automatic reconnect disabled in the library and implemented by the core's
link state machine (so that backoff, jitter and terminal errors are under
test), `connectTimeout` 10 s. TLS: platform trust store; optional per-profile
CA (PEM) added to a dedicated `TrustManager`; plaintext only with
`insecure_ack`. Message payloads are handed to the pipeline as byte arrays
without copying into strings until parsing.

### 6.2 HTTP (TDS-6.2)

Ktor client 3.x with the OkHttp engine on Android/JVM (Darwin later),
`HttpTimeout` (connect 10 s, request 30 s), `UserAgent` plugin with
`ChirpMapper/<version> (<platform>; <os>)`, `ContentNegotiation` with
`kotlinx.serialization`. Used by `UploadWorker` (POST JSON), the tile
fetcher (GET with `If-None-Match`/`If-Modified-Since`, `Cache-Control`
honoured, ≤ 2 concurrent per host through a per-host semaphore) and the
optional gateway-details lookup.

### 6.3 Connectivity (TDS-6.3)

`expect` `ConnectivityMonitor`: Android `ConnectivityManager` network
callback with validated-internet capability; JVM: polling probe every 30 s
(`HEAD` on the ingress host) combined with link state. Emits `online` for
the upload workers' immediate retry (FDS-10.2).

## 7. Map (TDS-7)

### 7.1 Model and projection (TDS-7.1)

`Camera(centreLat, centreLon, zoom: Double, rotationDeg, viewportPx)`;
Web Mercator helpers convert lat/lon ↔ world coordinates at zoom 0 in
`[0,1)²` (doubles). Overlay geometry is stored in world coordinates once;
per frame only an affine transform (scale by `2^zoom·tileSize`, translate,
rotate) is applied — cheap enough for 20 k points (REQ-NF-003).

### 7.2 Tiles (TDS-7.2)

`TileSource(urlTemplate, subdomains, minZoom, maxZoom, attribution,
tileSize=256, headers)`; presets OSM standard/greyscale/night (greyscale and
night are colour filters applied at draw time) and custom XYZ. `TileCache`:
memory LRU (≈ 64 tiles per source) + disk LRU (configurable, default 200 MB
phone / 1 GB desktop) keyed by source id + z/x/y with the HTTP validators;
`TileLoader` deduplicates in-flight requests, retries once, and produces
placeholder parent/child tiles while loading (scaled from cached
neighbours) so the map never flashes white.

### 7.3 Renderer (TDS-7.3)

```kotlin
interface DrawTarget {                       // minimal 2D API
    fun drawImage(img: RasterImage, dst: RectF, alpha: Float, filter: ColorFilter?)
    fun drawCircles(points: FloatArray, radiusPx: Float, fill: Int?, stroke: Int?, strokeWidthPx: Float)
    fun drawLines(segments: FloatArray, color: Int, widthPx: Float)
    fun drawMarker(img: RasterImage, xPx: Float, yPx: Float, anchor: Anchor, rotationDeg: Float)
    fun drawText(text: String, xPx: Float, yPx: Float, style: TextStyle)
    fun withTransform(matrix: Matrix3, block: () -> Unit)
}
```

Implementations: `ComposeDrawTarget` (common, over `DrawScope`),
`AndroidCanvasDrawTarget` (over `android.graphics.Canvas`, used by the car
`SurfaceCallback`), later `CoreGraphicsDrawTarget`.

`MapRenderer.render(camera, layers, target)` draws the layer stack of
FDS-8.1:

| Layer | Data structure | Technique |
|---|---|---|
| `TileLayer`, `OverlayTileLayer` | visible tile set from camera | draw cached raster, colour filter for greyscale/night, opacity for coverage |
| `LineLayer` | segments per `LevelClass` in world coords (`FloatArray` chunks of 4 096) | one `drawLines` call per class per chunk; optional "last N minutes" limit |
| `PointLayer` | points per `LevelClass`; a per-zoom grid bin (cell = 1 px at current zoom) built incrementally | one `drawCircles` per class; at zoom < 12 draw bins instead of points (LOD) so 100 k points stay cheap |
| `MarkerLayer` | gateways with valid location, spatial hash for hit-testing | `drawMarker` + label at zoom ≥ 11 |
| `PositionLayer` | current fix, heading, accuracy | circle + arrow |
| attribution, legend | Compose overlays (not drawn by the renderer) | |

Incremental updates: `MeasurementEvent`s append to the layer buffers (O(1));
no per-frame allocation beyond the transform. Gestures (`MapView`
composable): drag, pinch, wheel, double-tap, fling with decay; follow mode
per FDS-8.4. Performance is validated by the spike in WP-2.4; if the
targets are missed on the Pi, `MapComposeMP` replaces `ComposeDrawTarget`
rendering for the Compose shells while the car keeps the own renderer
([ADR-0003](adr/ADR-0003-map-rendering.md)).

## 8. UI (TDS-8)

- **Compose Multiplatform** (Material 3), `androidx.lifecycle` `ViewModel`
  and `androidx.navigation` in their multiplatform artefacts; one
  `NavHost` with the destinations of GUI design §3.
- **State:** each screen has a `ViewModel` exposing a single immutable
  `UiState` `StateFlow` built from core flows (`SessionState`,
  `PositionState`, `LinkState`, `OutboxState`, `SessionSnapshot`); events go
  back as sealed `UiEvent`s. The car screens reuse `DriveViewModel`.
- **Adaptive layout:** `WindowSizeClass` selects phone (bottom navigation),
  expanded (rail + side panel) or kiosk (`AppMode.KIOSK` from config) scaffolds;
  the kiosk scaffold picks landscape/portrait by window aspect.
- **Theme:** tokens of GUI design §2.1 as `ColorScheme`s; `LevelColors`
  constant object; typography scale; `LocalDensity` overrides for the kiosk
  (touch targets ≥ 64 px).
- **Resources:** Compose Multiplatform resources for strings (`values/`,
  `values-nl/`), ICU plurals, no hard-coded user-facing strings (enforced by
  a lint rule).
- **Accessibility:** semantics on every control; content descriptions from
  resources; focus order set explicitly on the map screen.
- **On-screen keyboard:** an in-app `SoftKeyboard` composable (QWERTY,
  numeric, symbols) attached to text fields when `AppMode.KIOSK` and no
  system IME is present.
- **Map screen:** `MapView` (module `map`) with overlays for chips, controls,
  legend and attribution.

## 9. Platform shells (TDS-9)

### 9.1 Android application (TDS-9.1)

| Element | Design |
|---|---|
| `ChirpMapperApp : Application` | Starts Koin with modules `core`, `data`, `net`, `map`, `androidPlatform`; creates notification channels |
| `MainActivity` | Single activity, `setContent { App() }`, edge-to-edge, handles permission results and the QR scanner (`ML Kit`-free: uses `zxing-android-embedded`, Apache-2.0) |
| `MappingService : Service` | Foreground service, `foregroundServiceType="location"`; started with `startForegroundService()` from the UI or the car session; calls `startForeground()` in `onCreate` with the status notification; holds a `PARTIAL_WAKE_LOCK` while `SessionState.Running`; stops itself on session end; `onTaskRemoved` keeps running; notification actions *Stop* via `PendingIntent` to a `BroadcastReceiver` |
| Position sources | `LocationManagerSource` (GPS provider, 1 s, 0 m, `GnssStatus` callback for satellites, `Location.hasAccuracy/hasSpeed/hasBearing`), `NetworkProviderSource` (display fallback only), `FusedSource` in flavour `play` only |
| Permissions | `ACCESS_FINE_LOCATION` (+ `ACCESS_COARSE_LOCATION`), `POST_NOTIFICATIONS` (33+), `FOREGROUND_SERVICE`, `FOREGROUND_SERVICE_LOCATION`, `INTERNET`, `ACCESS_NETWORK_STATE`, `WAKE_LOCK`, `CAMERA` (QR, optional feature). No `AD_ID`, no background-location permission (the service is started while the app is visible). Battery optimisation: the app does **not** call `ACTION_REQUEST_IGNORE_BATTERY_OPTIMIZATIONS`; a diagnostics hint links to `ACTION_IGNORE_BATTERY_OPTIMIZATION_SETTINGS` when drop-outs are detected. Doze is rarely entered while moving (significant-motion), so no exemption is needed for the normal use case |
| Sounds | `SoundPool` for the packet cue, `RingtoneManager` picker for the user's choice, `AudioAttributes` usage `USAGE_ASSISTANCE_SONIFICATION` |
| Export | `ActivityResultContracts.CreateDocument("text/csv")` and share sheet via `FileProvider` |
| Flavours | `foss` (default, no Google dependencies, F-Droid friendly), `play` (adds `play-services-location` for `FusedSource`) |
| `minSdk` 26, `targetSdk` 36, `compileSdk` 36; R8 enabled with keep rules for `kotlinx.serialization` and HiveMQ/Netty | |

### 9.2 Android Auto (TDS-9.2)

| Element | Design |
|---|---|
| `ChirpMapperCarAppService : CarAppService` | `createHostValidator()` = `ALLOW_ALL_HOSTS_VALIDATOR` in debug, Google's host validator in release; `onCreateSession()` returns `MappingCarSession` |
| Manifest | `<category android:name="androidx.car.app.category.POI"/>`, `androidx.car.app.minCarApiLevel = 7`, `<uses-permission android:name="androidx.car.app.MAP_TEMPLATES"/>`, `automotive_app_desc.xml` with `<uses name="template"/>`; the IoT fallback swaps the category and drops `MAP_TEMPLATES` via a build flag (`carCategory=poi|iot`) |
| `MapCarScreen` | `MapWithContentTemplate` with a `Pane`/`ItemList` of the five indicator rows (from `DriveViewModel`), `ActionStrip` with Start/Stop and Profiles; `MapController` with `SurfaceCallback`: on `onSurfaceAvailable` a render thread draws with `MapRenderer` + `AndroidCanvasDrawTarget` at ≤ 30 fps only while visible (`onVisibleAreaChanged` sets the stable area for the camera centre), dark mode from `CarContext.isDarkMode`; `onScroll`/`onScale` disable follow temporarily and it re-engages after 10 s |
| `ProfilesCarScreen` | `ListTemplate` of saved profiles; selecting one activates it (stops a running session with a confirmation `MessageTemplate`) |
| Fallback screens | `GridCarScreen` (`GridTemplate`, six tiles) and `RecentPacketsCarScreen` (`ListTemplate`), same view model |
| Session coupling | Start from the car → `MappingService` starts; the car screen observes the same `SessionState`; the phone UI shows a "controlled from car" hint |
| Location | `CarHardwareLocation` via `CarSensors` when `CarAppPermission` granted, wrapped as `CarPositionSource` and fed to `FixSelector` with the same gating |
| Testing | Desktop Head Unit (DHU) with the `car-app-testing` library; distraction rules checked against the car app quality checklist |

### 9.3 Desktop / Raspberry Pi (TDS-9.3)

| Element | Design |
|---|---|
| Entry | `main()` parses `--config`, `--kiosk`, `--replay`, `--headless-status`; loads TOML config (`tomlkt`); starts Koin; `application { Window(...) }` with `undecorated`/`fullscreen` in kiosk mode |
| Rendering | Skiko default (OpenGL) with automatic fallback to software (`skiko.renderApi=SOFTWARE`) when GL is unavailable; on the Pi the systemd unit sets `SKIKO_RENDER_API=SOFTWARE` by default (Pi 4's VideoCore VI GL works with the KMS driver but software rendering is the predictable baseline; the spike WP-2.4 decides the default) |
| Position sources | `GpsdSource` (TCP `?WATCH={"enable":true,"json":true}`, TPV/SKY parsing, reconnect), `NmeaSerialSource` (jSerialComm; GGA/RMC/GSA parsing shared with gpsd's NMEA fallback), `ManualSource`, `ReplaySource` |
| Files | Config `$XDG_CONFIG_HOME/chirpmapper/config.toml`, data `$XDG_DATA_HOME/chirpmapper/chirpmapper.db`, cache `$XDG_CACHE_HOME/chirpmapper/tiles/`, logs `$XDG_STATE_HOME/chirpmapper/log/` (rotating 10 MB × 5) |
| Kiosk | `chirpmapper-kiosk.service` (systemd user unit: `Restart=on-failure`, `RestartSec=5`, `Environment=SKIKO_RENDER_API=SOFTWARE`), autologin + Xorg/`labwc` session documented in `documentation/raspberry-pi-kiosk.md`; touch works through the toolkit's pointer events; screen rotation via the display stack, not the app |
| Status server | Optional Ktor CIO server on `127.0.0.1:8787` serving `/status` (JSON, HTML) read-only |
| Packaging | `compose.desktop { nativeDistributions { targetFormats(Deb) } }` with a bundled JRE (jlink); built natively per architecture (x86_64 on `ubuntu-22.04`, aarch64 on `ubuntu-24.04-arm` runners); AppImage may follow |
| Shutdown | Long-press dialog offers *Exit* and, when permitted by polkit, *Power off* |

### 9.4 Replay tool (TDS-9.4)

`chirpmapper-replay record|publish|track` built with `clikt`; `record`
uses `MqttLink` and writes JSON Lines; `publish` uses a publisher variant of
the MQTT client; `track` serves gpsd-style JSON on a TCP port from GPX/NMEA
with `--speed` and `--loop`; a `demo` sub-command runs a local Mosquitto
container (docker) or an embedded broker (`moquette`, Apache-2.0) and both
publish and track from a bundled recording.

## 10. Cross-cutting mechanisms (TDS-10)

### 10.1 Dependency injection
Koin (KMP). Scopes: *application* (stores, settings, secret store, HTTP,
tile cache, diagnostics), *session* (link, pipeline, fix selector, upload
workers, alert engine). Shells register platform implementations of the
ports; tests register fakes from `testing`.

### 10.2 Logging and redaction
Kermit with a `RedactingLogWriter` that masks values of keys matching
`(?i)(api[_-]?key|password|token|secret|authorization)` in structured
fields and strings matching the TTS API key shape (`NNSXS\.[A-Z0-9]{20,}`).
`Secret` never reaches a log. Levels: `verbose` off in release; `debug`
toggle in diagnostics; file sinks on desktop.

### 10.3 Error handling policy
Domain errors are sealed results (`ParseResult`, `LinkError`,
`UploadResult`); exceptions are reserved for programming errors and are
caught at coroutine scope boundaries with a diagnostics event. User-facing
messages come from the FDS §6 table via resource ids.

### 10.4 Security threat model (summary)

| Threat | Control |
|---|---|
| Credential interception on the wire | TLS default, certificate validation, plaintext only with acknowledgement and visible warning (REQ-NF-020) |
| Credential extraction from device/backups | Keystore-encrypted store (Android), 0600 files or Secret Service (Linux), `allowBackup=false`, no secrets in exports/bug reports (REQ-NF-021) |
| Secrets in logs | `RedactingLogWriter`, `Secret` type, review checklist |
| Malicious broker/server input | Lenient but bounded parsing (payload cap 64 kB, string caps), no code paths that interpret payload content, JSON pointers of the generic adapter cannot escape the document |
| Malicious tile/config URLs | URL scheme allow-list (`https`, `http` with acknowledgement), no redirects to other hosts for tiles |
| QR provisioning abuse | Profile JSON only, key-like strings refused, user confirms before saving |
| Supply chain | Pinned versions, Maven Central/Google only, licence report, Dependabot |

### 10.5 Time
`Clock` port; wall-clock for measurement receipt time (`receivedAt`) and
monotonic time for freshness and backoff (so that a clock jump — common on a
Pi without RTC — does not invalidate fixes or reset backoff). On the kiosk,
the GNSS time is compared with the system clock and a `TIME_UNSYNCED` flag
is set on measurements when they differ by > 5 s (REQ-F-132).

### 10.6 Versioning
SemVer for the software (`MAJOR.MINOR.PATCH`), Android `versionCode` =
`MAJOR*10000 + MINOR*100 + PATCH`, git tags `vX.Y.Z`, `useragent` and
`report.txt` carry the version and git SHA.

## 11. Build, CI and release (TDS-11)

| Item | Design |
|---|---|
| Toolchain | JDK 21 (Temurin), Kotlin and Compose Multiplatform latest stable at WP-2.1, AGP matching; Gradle wrapper with configuration cache |
| Static analysis | `ktlint` (Gradle plugin), `detekt` with the project rule set, Android Lint (`warningsAsErrors` for `HardcodedText`, `MissingPermission`, `UnusedResources`), a custom detekt rule forbidding `println`/`Log` outside the logging sink |
| Tests | `core`/`map`/`data` unit tests on JVM (`kotlin.test`, Turbine for flows, `kotest-assertions`), SQLDelight verification, Android instrumentation for the service and secret store, Compose UI tests (desktop runner), integration tests with a Mosquitto container (Testcontainers) driven by the replay tool, screenshot tests for the wireframe screens |
| CI (GitHub Actions) | `pr.yml`: lint + tests on `ubuntu-latest`; `android.yml`: assemble `foss` debug APK, run instrumentation on an emulator (API 30, 34); `desktop.yml`: `packageDeb` on `ubuntu-22.04` (x86_64) and `ubuntu-24.04-arm` (aarch64); `ios-compile.yml` (optional macOS job): `compileKotlinIosArm64` for `core`, `data`, `net`, `map` |
| Release | Tag → `release.yml` builds signed APK/AAB (`foss` for GitHub/F-Droid, `play` for Play) with signing material from repository secrets, `.deb` x86_64 and arm64, `SHA256SUMS`, licence report (`licensee`), CycloneDX SBOM; changelog section becomes the release notes |
| Distribution | GitHub Releases (all), Google Play (internal → closed → production tracks), F-Droid (reproducible `foss` build, later), `.deb` repository (later) |

## 12. Dependencies (planned)

| Dependency | Module(s) | Licence | Note |
|---|---|---|---|
| Kotlin stdlib, kotlinx.coroutines, serialization, datetime, io | all | Apache-2.0 | |
| Compose Multiplatform (+ Material 3, resources), androidx.lifecycle, androidx.navigation (multiplatform) | ui, shells | Apache-2.0 | |
| Skiko | desktop | Apache-2.0 | linux-arm64 artefacts available |
| SQLDelight + drivers, sqlite-jdbc | data | Apache-2.0 | aarch64 native lib bundled in sqlite-jdbc |
| multiplatform-settings | data | Apache-2.0 | |
| HiveMQ MQTT Client | net | Apache-2.0 | Netty transitively |
| Ktor client (OkHttp engine), Ktor server CIO (status page) | net, desktop | Apache-2.0 | |
| Koin | all | Apache-2.0 | |
| Kermit | all | Apache-2.0 | |
| tomlkt | desktop | Apache-2.0 | |
| jSerialComm | desktop | LGPL-3.0 / Apache-2.0 (dual) | dynamically linked, GPL-compatible |
| dbus-java (optional) | desktop | MIT | Secret Service |
| zxing-android-embedded | android | Apache-2.0 | QR import |
| AndroidX Car App Library (`app`, `app-projected`, `app-testing`) | android | Apache-2.0 | |
| clikt | tools | Apache-2.0 | |
| moquette (optional, demo broker) | tools | Apache-2.0 | |
| Test: kotlin.test, Turbine, kotest-assertions, Testcontainers, MockK | tests | Apache-2.0 / MIT | |
| Fallback: MapComposeMP | ui (only if ADR-0003 fallback is triggered) | Apache-2.0 | |

All GPL-3.0-compatible (REQ-C-001). A generated report per release lists the
exact versions.

## 13. Performance budgets and how they are met

| Budget | Mechanism | Verified by |
|---|---|---|
| Start ≤ 3 s phone / ≤ 8 s Pi (REQ-NF-001) | Lazy DI, no network on the critical path, database opened asynchronously, map shows cached tiles first | TC-P-01 |
| Message → point ≤ 500 ms phone / 1 s Pi (REQ-NF-002) | Single pipeline hop, one transaction per message, incremental layer append, no full redraw | TC-P-02 |
| ≥ 30 / 15 fps with 20 k points + 20 k lines (REQ-NF-003) | World-coordinate buffers, batched draws per class, LOD binning, no allocation in the frame | TC-P-03, spike WP-2.4 |
| 50 k measurements, heap ≤ 256 MB (REQ-NF-004) | Streaming queries for export, layer buffers as primitive arrays (≈ 24 B/point, 32 B/segment), tile memory cache bounded | TC-P-04 |
| Battery ≤ 8 %/h (REQ-NF-005) | One socket, keep-alive 30 s, GPS at 1 Hz only while running, no wakeful polling, batched DB writes | TC-F-05 (field) |
| Tile traffic (REQ-NF-006) | Disk cache with validators, per-host semaphore 2, no prefetch | TC-N-06 |

## 14. Risks, technical debt and open points

| ID | Item | Mitigation |
|---|---|---|
| TDS-R-1 | Own map renderer underperforms on Pi 4 | Spike WP-2.4 with go/no-go; fallback MapComposeMP (ADR-0003) or Pi 5 minimum |
| TDS-R-2 | HiveMQ/Netty size and Android battery behaviour | Measure in TC-F-05; alternative Paho Java (EPL-2.0) behind the same port |
| TDS-R-3 | Compose for Desktop on Wayland-only Pi sessions | Require X11/XWayland (REQ-C-008); document `labwc` XWayland setup |
| TDS-R-4 | Google Play POI category refusal | Build flag for IoT fallback (TDS-9.2), decision at WP-3.3 |
| TDS-R-5 | ttnmapper.org contract changes or rate limits | OP-1 coordination; payload builder isolated and versioned; server messages surfaced |
| TDS-R-6 | Secret Service absent on kiosk images | 0600 file fallback with warning; documented |
| TDS-OP-1 | Confirm package root / application id `nl.pe1mew.chirpmapper` | Maintainer, WP-2.1 |
| TDS-OP-2 | Exact Compose/Kotlin/AGP versions at WP-2.1 | Developer |
| TDS-OP-3 | Whether to ship `armeabi-v7a` (older phones) | Maintainer, WP-2.3 |

## 15. Traceability (requirements → design)

| Requirement group | Design elements |
|---|---|
| REQ-P-001…008 | TDS-3 (targets), TDS-9.1, 9.2, 9.3, TDS-8 (adaptive layouts), TDS-11 (arm64 build), flavour `foss` |
| REQ-F-001…009 | TDS-4.3 `ProfileService`, TDS-4.4 `ProfileFieldSpec`, TDS-5.4, TDS-8 |
| REQ-F-010…015 | TDS-4.3 `SessionController`, TDS-9.1 `MappingService`, TDS-9.3 kiosk |
| REQ-F-020…028 | TDS-4.4, TDS-6.1, TDS-2.3 |
| REQ-F-030…036 | TDS-4.3 `FixSelector`, TDS-9.1/9.2/9.3 position sources |
| REQ-F-040…047 | TDS-4.5, TDS-4.3 `MeasurementPipeline`, `GatewayRegistry` |
| REQ-F-050…056 | TDS-5.1, 5.2, TDS-4.3 `Exporters`, `Importer` |
| REQ-F-060…069 | TDS-7 |
| REQ-F-070…073 | TDS-8 view models, TDS-4.3 `Statistics` |
| REQ-F-080…088 | TDS-4.3 `UploadWorker`, TDS-5.2, TDS-6.2, TDS-6.3 |
| REQ-F-090…092 | TDS-4.3 `AlertEngine`, TDS-9.1 sounds/notification |
| REQ-F-100…103 | TDS-8 (onboarding, resources), TDS-5.3 |
| REQ-F-110…112 | TDS-4.3 `Diagnostics`, TDS-10.2 |
| REQ-F-120…124 | TDS-9.2 |
| REQ-F-130…135 | TDS-9.3 |
| REQ-F-140, 141 | TDS-4.6, TDS-9.4 |
| REQ-NF-001…006 | TDS-13 |
| REQ-NF-010…013 | TDS-5.2, TDS-2.3, TDS-9.3 supervision, TDS-5.1 migrations |
| REQ-NF-020…024 | TDS-5.4, TDS-6.1, 6.2, TDS-10.2, 10.4, TDS-11 |
| REQ-NF-030…034 | TDS-8 |
| REQ-NF-040…044 | TDS-2.1, TDS-3, TDS-11 |
| REQ-NF-050 | TDS-4.3 `Diagnostics`, TDS-10.2 |
| REQ-NF-060…062 | TDS-9.1 permissions, TDS-7.2, TDS-8 privacy page |
| REQ-C-001…009 | TDS-12, TDS-9.1 flavours, TDS-11, TDS-7.2 |
