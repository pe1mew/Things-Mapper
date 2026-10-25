# 06 — Test plan

Status: **Draft**

| Rev | Date | Author | Change |
|---|---|---|---|
| 0.1 | 2026-09-11 | R. Welling (with Claude) | First complete version |
| 0.2 | 2026-10-24 | R. Welling (with Claude) | Project renamed to Things Mapper; editions (ADR-0007): TC-ED-01…14, both editions in CI, car tests on the Paid edition |
| 0.3 | 2026-10-24 | R. Welling (with Claude) | Sponsor catalogue is the only ad source (OP-7); licence route L-A (OP-8); no ad consent |
| 0.4 | 2026-10-24 | R. Welling (with Claude) | Free start screen; cost awareness and voluntary support (ADR-0008); budget per milestone |
| 0.5 | 2026-10-25 | R. Welling (with Claude) | Linux fixes: §6.22 packaging install tests TC-PK-01/02; TC-KI-08 boot to kiosk, TC-KI-09 provisioning-file precedence, TC-KI-03 owner/mode cases; TC-SO-06 desktop settings survive a power cut; CI-Desktop builds in `debian:12` |
| 0.6 | 2026-10-25 | R. Welling (with Claude) | Libraries tested as libraries (ADR-0009): principle, CI-iOS environment, §6.23 TC-LB-01…04, unit level on every target, PR checklist, traceability for REQ-NF-045 |

Related: [02 Requirements](02-requirements.md) (acceptance basis),
[03 FDS](03-fds.md) (expected behaviour), [05 TDS](05-tds.md) (test hooks,
budgets), [07 Implementation plan](07-implementation-plan.md) (when which
tests exist).

Identifiers: **TC-<area>-nn** = test case. Types: **U** unit, **I**
integration (several modules, fakes or containers), **S** system (built
application), **F** field (real network, real movement), **A** acceptance
(against a success target), **X** exploratory/checklist.

---

## 1. Introduction

### 1.1 Purpose
Define how Things Mapper is verified against its requirements and validated
in the field, which environments and data are needed, and when a milestone
may be called done.

### 1.2 Scope
The software of phase 2 and 3 (core, data, net, map, ui, Android app incl.
Android Auto, desktop/kiosk, replay tool). The design documents themselves
are reviewed, not tested; review checklists are in §9.

### 1.3 Principles
- **Test the core without hardware.** Every port has a fake; protocol
  behaviour is fixed by recorded fixtures; sessions are replayable. CI must
  run the whole functional suite without a phone, a Pi or a LoRaWAN node.
- **One requirement, at least one test.** The traceability matrix (§8) is
  maintained with the code; a pull request that adds behaviour adds or
  updates the matrix.
- **Field tests are formal.** A written protocol (§7), a fixed route, and a
  side-by-side comparison with TTN Mapper v3 define the acceptance of the
  data path (TGT-02, TGT-03).
- **Security is tested, not assumed.** Redaction, TLS enforcement and
  secret handling have their own automated tests.
- **Libraries are tested as libraries.** The shared libraries in `lib/`
  are built and tested without any shell; their common tests run on every
  target (JVM, Android, iOS simulator), and every platform implementation
  of a port runs the same contract suite (ADR-0009, REQ-NF-045).

## 2. Test strategy

### 2.1 Levels

| Level | What | Where | Gate |
|---|---|---|---|
| Unit (U) | Rules, adapters, payload builder, exporters, state machines, renderer geometry, view-model reducers | Library tests on JVM, Android host and iOS simulator in CI on every PR (shell code: JVM) | 100 % pass, coverage of `core` ≥ 80 % lines (REQ-NF-041) |
| Integration (I) | Pipeline + SQLDelight + fake link + replay positions; MQTT link against a Mosquitto container; upload worker against a stub ingress server; migrations | CI on every PR (containers) | 100 % pass |
| System (S) | Built apps on emulator/desktop: onboarding, profiles, session, map, export, diagnostics; Compose UI tests; screenshot tests | CI nightly + before each milestone | No open High defects |
| Field (F) | Real node, real network server(s), real movement, side-by-side with TTN Mapper v3, Pi in a car, Android Auto head unit | Before beta and before 1.0 | Protocol §7 passed |
| Acceptance (A) | Success targets TGT-01…08 | Milestone reviews | Targets met or consciously deferred |

### 2.2 Entry and exit criteria (per milestone)
Entry: CI green; test environment available; fixtures for new protocol
fields recorded. Exit: all planned TCs for the milestone executed; no open
*Critical*/*High* defects; *Medium* defects have an owner and a target
milestone; the traceability matrix has no requirement of the milestone
without a passed test; release notes list known issues.

### 2.3 Defect severity

| Severity | Definition |
|---|---|
| Critical | Data loss, secret exposure, crash on start, cannot map at all |
| High | Requirement of priority M not met; wrong data uploaded; session ends unexpectedly |
| Medium | Requirement of priority S not met; degraded UX with workaround |
| Low | Cosmetic, priority C, documentation |

## 3. Test items

Tested: everything in TDS §3 (all modules and shells). Not tested by this
plan: ttnmapper.org's server-side processing (only its acceptance response is
checked), the Google Play review outcome, third-party tile servers beyond
contract behaviour.

## 4. Test environments

| Environment | Content | Used for |
|---|---|---|
| CI-JVM | GitHub-hosted Ubuntu, JDK 21, Testcontainers (Mosquitto 2.x, stub ingress) | U, I |
| CI-Android | Emulators API 30, 34, 36 (x86_64), `freeFoss` and `paidFoss` variants | I (service, secret store), S (UI tests) |
| CI-Desktop | Compose UI tests headless (x86_64), `packageDeb` in a `debian:12` container on x86_64 and arm64 runners, install tests in clean distribution containers (TC-PK) | S |
| CI-iOS | GitHub-hosted macOS runner with Xcode and the iOS simulator (free for public repositories): `iosSimulatorArm64Test` of every library | U, I |
| LAB-Phone-A | Current mid-range phone, Android 15/16, `freePlay`, `paidPlay` and `freeFoss` variants | S, F |
| LAB-Phone-B | Older phone, Android 8.1 or 10 (minSdk coverage) | S |
| LAB-Tablet | 10" tablet (expanded layout) | S |
| LAB-Pi4 | Raspberry Pi 4 (2 GB), Raspberry Pi OS 64-bit, official 7" display 800×480, u-blox USB GNSS via gpsd, LTE hotspot | S, F, performance |
| LAB-Pi5 | Raspberry Pi 5, Touch Display 2 (720×1280) | S |
| LAB-Desktop | Ubuntu 24.04 x86_64 (X11 and Wayland/XWayland) | S |
| LAB-Car | Desktop Head Unit (DHU) on the developer machine; one real Android Auto head unit (wired) on a 12 V bench supply (doc 08 I-5, I-6); Paid edition | S, F |
| LAB-Billing | Play Console license testers and test products for both listings (StoreKit sandbox in phase 4) | S |
| NS-TTS | The Things Stack Sandbox application `thingsmapper-test` with two devices (`mapper-01`, `mapper-02`) and an MQTT API key; one real node transmitting every 10 s at SF7 (GPS-less payload) | I (recording), F |
| NS-CS4 | ChirpStack v4 in docker (with MQTT broker), one simulated device (ChirpStack simulator) and — where available — the same real node through a private gateway | I (recording), F |
| TOOLS | replay tool, `adb`, Perfetto, `dumpsys batterystats`, Battery Historian, `mitmproxy` (negative TLS tests), `tcpdump` | performance, security |

## 5. Test data

| Data set | Content | Source |
|---|---|---|
| FIX-TTS | ≥ 12 recorded TTS v3 `ApplicationUp` messages: LoRa SF7…SF12, FSK, LR-FHSS, with/without `dev_eui`, with Packet Broker receptions (forwarder ids present/absent, `ttnv2`), gateway location present/absent/null-island, `simulated:true`, missing `f_port`, `coding_rate` in both places, timestamps with 9 fractional digits | upstream `TestJsonParsing.kt` (GPL, attributed; kept in the test-only directory `testing/upstream-fixtures/`, REQ-C-009) + new recordings from NS-TTS |
| FIX-CS4 | ≥ 10 ChirpStack v4 events: LoRa, FSK, LR-FHSS, multiple `rxInfo`, `crcStatus` ≠ OK, location sources, 64-bit fields as strings, missing `time` | recordings from NS-CS4 + ChirpStack documentation examples |
| FIX-CS3 | 2 ChirpStack v3 events (legacy detection) | upstream fixture |
| FIX-BAD | Malformed JSON, empty payload, 100 kB payload, wrong topic, binary protobuf payload | synthetic |
| GOLD-INGRESS | Expected `/android/v4` JSON for each FIX-TTS/FIX-CS4 message with a fixed position (golden files) | derived; reviewed against TTN Mapper v3 output for the upstream fixtures |
| TRACK-URBAN | 1 h GPX, 30 km/h average, tunnels (fix gaps) | recorded drive |
| TRACK-HIGHWAY | 20 min GPX at 100–130 km/h | recorded drive |
| TRACK-WALK | 30 min GPX walking | recorded |
| REC-SESSION | JSON-lines recording of a real 1 h session synchronised with TRACK-URBAN | recorded with the replay tool |
| LEGACY-CSV | CSV exported by TTN Mapper v3 (Room v2 schema), 41 columns | from the original app |
| DB-VERSIONS | Database files of every released schema version | archived per release |

## 6. Test case catalogue

Columns: id, title, type, requirements, procedure (summary), expected result.

### 6.1 Profiles and linking (TC-PR)

| ID | Title | Type | Req | Procedure | Expected |
|---|---|---|---|---|---|
| TC-PR-01 | Create TTS profile, derived username/topics | U | REQ-F-002 | Build profile with app `a`, tenant `ttn`, devices `d1,d2` | username `a@ttn`; topics `v3/a@ttn/devices/d1/up`, `…/d2/up` |
| TC-PR-02 | Single-tenant TTS (empty tenant) | U | REQ-F-002 | tenant empty | username `a`, topics without `@` |
| TC-PR-03 | ChirpStack profile topics with prefix and wildcard | U | REQ-F-003, F-005 | app UUID, EUIs, prefix `eu/`, "all devices" | `eu/application/<uuid>/device/+/event/up`, filter accepts only listed EUIs |
| TC-PR-04 | Validation of ids, EUIs, host | U | REQ-F-002, F-003 | invalid inputs | field errors, Save disabled |
| TC-PR-05 | TLS default and insecure acknowledgement | S | REQ-F-002, NF-020 | new profile; toggle TLS off | port switches 8883→1883; Save blocked until acknowledgement; open-lock icon in list |
| TC-PR-06 | Test connection outcomes | I | REQ-F-004 | run against Mosquitto with: wrong host, TLS cert mismatch, wrong password, ACL-denied topic, ok without traffic, ok with traffic | each step reports the documented outcome; nothing stored |
| TC-PR-07 | Export/import profile JSON without secret | S | REQ-F-006 | export, inspect file, import on second device | file has no key; import asks for key; profile identical |
| TC-PR-08 | QR import refuses key-like content | S | REQ-F-006 | QR with `"api_key":"NNSXS.…"` | refused with explanation |
| TC-PR-09 | Activate another profile while mapping | S | REQ-F-007 | running session; activate B | prompt to stop; after stop B active |
| TC-PR-10 | Secrets stored in secure store, masked | I | REQ-NF-021 | save profile; inspect preferences/db files; open editor | no secret in db/prefs; field masked; reveal toggle works |

### 6.2 Session control (TC-SE)

| ID | Title | Type | Req | Procedure | Expected |
|---|---|---|---|---|---|
| TC-SE-01 | Session state machine | U | REQ-F-010, F-012 | drive `SessionController` with fake link/position | transitions per FDS §5.1; session rows created/closed |
| TC-SE-02 | Start without profile / permission | S | REQ-F-010, F-100 | fresh install, press Start | empty-state / permission flow, no crash |
| TC-SE-03 | Foreground service start and notification | I (Android) | REQ-F-011, F-092, NF-060 | start; check service type, notification, `startForeground` timing | service `location` type, notification within 1 s, Stop action works |
| TC-SE-04 | Screen off, background 30 min | S | REQ-F-011 | replay session, screen off | all replayed messages stored |
| TC-SE-05 | Activity destroyed and recreated | S | REQ-F-011 | rotate, kill activity (`am kill` of the UI only) | session continues; UI reattaches with correct state |
| TC-SE-06 | Kiosk auto-start and resume after kill | S (Pi) | REQ-F-013, NF-012 | `auto_start_mapping`; `kill -9`; reboot | new session starts within 20 s; previous closed |
| TC-SE-07 | Keep screen on setting | S | REQ-F-015 | toggle | flag applied/removed |

### 6.3 Ingest and protocols (TC-IN)

| ID | Title | Type | Req | Procedure | Expected |
|---|---|---|---|---|---|
| TC-IN-01 | TTS v3 field extraction | U | REQ-F-021 | parse every FIX-TTS | fields per FDS §3.2.1 (golden) |
| TC-IN-02 | ChirpStack v4 field extraction | U | REQ-F-021 | parse every FIX-CS4 | fields per FDS §3.2.2 |
| TC-IN-03 | Packet Broker rules | U | REQ-F-023 | PB fixtures | pseudo-gateway skipped; forwarder ids/network id; pb_only flag |
| TC-IN-04 | Network id formats | U | REQ-F-024 | TTS `net_id` variants | `NS_TTS_V3://ttn@000013` etc. |
| TC-IN-05 | Lenient parsing | U | REQ-F-022 | FIX-TTS with extra members, string numbers, missing optionals | parsed; nulls where absent |
| TC-IN-06 | Malformed messages | U/I | REQ-F-022, F-112 | FIX-BAD via pipeline | counted; redacted sample; session continues |
| TC-IN-07 | ChirpStack v3 detection | U | REQ-F-003 | FIX-CS3 | "v3 format not supported" diagnostic |
| TC-IN-08 | Simulated uplink ignored | U | FDS §3.2.1 | `simulated:true` | counted, not stored |
| TC-IN-09 | Link connect/subscribe/receive over TLS | I | REQ-F-020, NF-020 | Mosquitto with TLS cert; publish FIX-TTS | messages delivered to pipeline; QoS 0; MQTT 3.1.1 |
| TC-IN-10 | Reconnect with backoff | I | REQ-F-025, NF-011 | stop/start broker repeatedly | backoff 1,2,4…60 s ±20 %; resubscribe; state visible; no give-up in 30 min |
| TC-IN-11 | Terminal auth failure | I | REQ-F-025 | wrong password | session → Error with explanation; no retry loop |
| TC-IN-12 | Plaintext refused without acknowledgement | I | REQ-NF-020 | profile TLS off, no ack | connect refused by core |
| TC-IN-13 | Custom CA | I | REQ-NF-020 | broker with private CA; profile with CA PEM | connects; without PEM fails with certificate error |
| TC-IN-14 | Duplicate merge | U | REQ-F-026 | same fCnt within 30 s | merged when enabled; separate + flagged when disabled |
| TC-IN-15 | Burst 10 msg/s for 60 s | I | REQ-F-045 | replay at speed | 600 stored, order preserved, dropped counter 0 |
| TC-IN-16 | Queue overflow counted | U | REQ-F-045 | block store, push 1 500 | oldest dropped, counter 500, no exception |

### 6.4 Positioning (TC-PO)

| ID | Title | Type | Req | Procedure | Expected |
|---|---|---|---|---|---|
| TC-PO-01 | Freshness window | U | REQ-F-031 | fixes at t−5, t−15 with window 10 | first attached, second → no position |
| TC-PO-02 | Coarse fix never replaces fresh precise fix | U | REQ-F-033 | gps 4 m then network 800 m 2 s later | current fix stays gps until expiry |
| TC-PO-03 | Source priority | U | REQ-F-033 | equal accuracy gpsd vs manual | gpsd wins |
| TC-PO-04 | Fields recorded (accuracy, hdop, sats, speed, bearing, source) | U/I | REQ-F-032, F-035 | gpsd TPV/SKY fixtures; Android `Location` with extras | all present in measurement |
| TC-PO-05 | gpsd source | I | REQ-F-030 | replay tool `track` mode | fixes flow; reconnect after gpsd restart |
| TC-PO-06 | NMEA serial source | I | REQ-F-030 | pseudo-tty with NMEA | GGA/RMC/GSA parsed; accuracy from HDOP |
| TC-PO-07 | Android GPS provider + GnssStatus | S | REQ-F-030 | LAB-Phone-A outdoors | satellites count shown; 1 Hz fixes |
| TC-PO-08 | GNSS indicator states and warnings | S | REQ-F-034 | simulate 23 m accuracy, stale 14 s | `poor`, `stale` states, warning banner after 30 s |
| TC-PO-09 | Tunnel gap (TRACK-URBAN) | I | REQ-F-031 | replay through gap | measurements in gap stored without position; none uploaded |

### 6.5 Rules (TC-RU)

| ID | Title | Type | Req | Procedure | Expected |
|---|---|---|---|---|---|
| TC-RU-01 | Level formula and classes | U | REQ-F-040 | table of rssi/snr incl. boundaries −100, −105, … | exact class per legend; snr ≥ 0 ignored |
| TC-RU-02 | Best level and PB ring | U | REQ-F-041 | multi-reception fixtures | best level; pb_only |
| TC-RU-03 | Gateway location validation | U | REQ-F-042 | null island, 91°, 181°, valid | invalid → no marker/line, data kept |
| TC-RU-04 | Gateway registry update and *moved* | U | REQ-F-043 | same gateway with new location > 100 m | moved flag; counts and best level |
| TC-RU-05 | Stored without position | U | REQ-F-044 | no fix | row with null position, flag |
| TC-RU-06 | Experiment applied | U | REQ-F-046 | session with experiment | field on measurement and payload |
| TC-RU-07 | Normalisation | U | REQ-F-047 | `CR_4_5`, Hz strings, EUI case | `4/5`, longs, upper-case EUI |

### 6.6 Storage and export (TC-ST)

| ID | Title | Type | Req | Procedure | Expected |
|---|---|---|---|---|---|
| TC-ST-01 | Transactional store | I | REQ-NF-010 | inject failure after measurement insert | nothing committed; retry succeeds |
| TC-ST-02 | Migration from every released version | I | REQ-NF-013 | open DB-VERSIONS | schema current, data intact |
| TC-ST-03 | Sessions list and delete | S | REQ-F-051 | create 3 sessions; delete one; delete all | counts, confirmations, cascade |
| TC-ST-04 | CSV export correctness | U/S | REQ-F-052 | export REC-SESSION; parse with a CSV library; compare with DB | all rows/columns, RFC 4180 quoting, legacy column order, no trailing comma |
| TC-ST-05 | CSV export 100 000 rows streamed | I | REQ-F-053 | large synthetic session | completes; heap flat; cancel works |
| TC-ST-06 | GeoJSON validity | U | REQ-F-052 | export; validate against GeoJSON schema | valid; feature kinds/properties |
| TC-ST-07 | Android SAF export and share | S | REQ-F-053 | export to Downloads; share | file created with MIME `text/csv` |
| TC-ST-08 | Kiosk export to USB | S (Pi) | REQ-F-053 | insert stick; export | file on stick |
| TC-ST-09 | Legacy CSV import | S | REQ-F-054 | import LEGACY-CSV | imported session on map, flagged, never uploaded |
| TC-ST-10 | Size warning | S | REQ-F-055 | lower threshold; fill | warning with shortcut |

### 6.7 Map (TC-MA)

| ID | Title | Type | Req | Procedure | Expected |
|---|---|---|---|---|---|
| TC-MA-01 | Projection round trip | U | REQ-F-060 | lat/lon ↔ world ↔ screen at zooms 2–19 | error < 1e-9 world units |
| TC-MA-02 | Visible tile set | U | REQ-F-061 | camera cases incl. rotation, date line | correct z/x/y set |
| TC-MA-03 | Tile cache and validators | I | REQ-F-061, NF-006 | stub tile server; repeated loads | second load from cache; `If-None-Match` sent; ≤ 2 concurrent per host |
| TC-MA-04 | User-Agent and attribution | I/S | REQ-F-061, NF-061 | capture requests; view map | UA `ThingsMapper/…`; attribution visible for every preset and custom source |
| TC-MA-05 | Coverage overlay URL follows network id | U | REQ-F-062 | TTS profile | `…/network/NS_TTS_V3%3A%2F%2Fttn%40000013/…`; ChirpStack → unavailable |
| TC-MA-06 | Layers toggles and persistence | S | REQ-F-063, F-067 | toggle; restart | state restored |
| TC-MA-07 | Follow mode and re-centre | S | REQ-F-060 | pan → follow off, control visible; tap → follow on | per FDS-8.4 |
| TC-MA-08 | Gateway tap → sheet | S | REQ-F-073 | tap marker | sheet with distance/bearing |
| TC-MA-09 | Earlier sessions on map | S | REQ-F-065 | select two sessions | both drawn, distinguishable |
| TC-MA-10 | Legend | S | REQ-F-066 | toggle | six classes with dBm |
| TC-MA-11 | Renderer golden images | U | REQ-F-063 | render fixed scene to bitmap | pixel match within tolerance in light and night mode |

### 6.8 Indicators and statistics (TC-IS)

| ID | Title | Type | Req | Procedure | Expected |
|---|---|---|---|---|---|
| TC-IS-01 | Chip states | U | REQ-F-070 | reducer with link/position/upload states | texts and colours per GUI §2.4 |
| TC-IS-02 | Last-packet age counts up | S | REQ-F-070 | wait | updates each second |
| TC-IS-03 | Session statistics | U | REQ-F-071 | REC-SESSION | packets, gateways, distance (± 1 %), histogram, per-gateway table |
| TC-IS-04 | Distance ignores jumps | U | FDS-9.2 | inject 5 km jump in 1 s | excluded |
| TC-IS-05 | Drive view readability | X | REQ-F-072, NF-030 | view at 1 m in daylight and at night | numerals ≥ 56 sp, contrast ok |

### 6.9 Upload (TC-UP)

| ID | Title | Type | Req | Procedure | Expected |
|---|---|---|---|---|---|
| TC-UP-01 | Payload golden files | U | REQ-F-080, F-047 | build payload for GOLD-INGRESS inputs | byte-equal JSON (ignoring member order), nulls omitted |
| TC-UP-02 | Payload equals TTN Mapper v3 for upstream fixtures | U | TGT-03 | compare with recorded v3 uploads | only `useragent` differs |
| TC-UP-03 | Consent gating | U | REQ-F-081, F-086 | consent off / profile contribute off / no position | no outbox row |
| TC-UP-04 | Outbox with pipeline transaction | I | REQ-F-082, NF-010 | store 100 with stub ingress down | 100 pending; restart app; all sent when up |
| TC-UP-05 | Retry schedule and expiry | U | REQ-F-082 | fake clock | 5 s, 30 s, 2 m, 10 m, 30 m, 1 h, hourly; expired at 24 h |
| TC-UP-06 | Response handling | I | REQ-F-083 | stub returns 200 `success:false "rate limited"`, 429, 500, 400 | rejected/retry per FDS §5.3; message visible in diagnostics |
| TC-UP-07 | Concurrency ≤ 2 | I | REQ-F-088 | 50 pending, slow stub | never > 2 in flight |
| TC-UP-08 | Custom endpoint with auth header | I | REQ-F-085 | stub requiring bearer | header sent; separate queue state |
| TC-UP-09 | Installation id show/reset | S | REQ-F-087 | reset | new UUID; used in next payload |
| TC-UP-10 | Network change triggers immediate retry | S | REQ-NF-011 | airplane mode on/off | send within 5 s of connectivity |
| TC-UP-11 | Real ingress acceptance | F | REQ-F-080, TGT-02 | field session | `success:true` for ≥ 99 % geo-tagged; visible on ttnmapper.org within 24 h |

### 6.10 Alerts and sounds (TC-AL)

| ID | Title | Type | Req | Procedure | Expected |
|---|---|---|---|---|---|
| TC-AL-01 | Packet sound | S | REQ-F-090 | enable; receive | sound each packet; silent mode respected |
| TC-AL-02 | Alert conditions | U | REQ-F-091 | fake clock: GNSS lost 31 s, link down 31 s, no packet 5 min | alerts raised and cleared |

### 6.11 Settings and onboarding (TC-SO)

| ID | Title | Type | Req | Procedure | Expected |
|---|---|---|---|---|---|
| TC-SO-01 | Onboarding flow | S | REQ-F-100 | fresh install | steps per FDS-12.1; consent default off; skippable profile |
| TC-SO-02 | Settings catalogue present and persisted | S | REQ-F-101 | change each setting; restart | persisted |
| TC-SO-03 | Localisation nl | S | REQ-F-102 | device locale nl | translated; layouts intact at +30 % |
| TC-SO-04 | No hard-coded strings | U (lint) | REQ-F-102 | Android Lint `HardcodedText` | zero |
| TC-SO-05 | Theme auto/dark/light | S | REQ-NF-031 | switch | tokens applied; level colours unchanged |
| TC-SO-06 | Desktop settings survive a power cut | I | REQ-F-101, F-013 | JVM: a child process changes settings in a loop and is killed with `SIGKILL` at random moments (100 runs); load the settings after each kill | file always parses and holds either the previous or the new value; stored in `$XDG_CONFIG_HOME/thingsmapper/settings.properties`; nothing under `~/.java` |

### 6.12 Diagnostics and security (TC-DS)

| ID | Title | Type | Req | Procedure | Expected |
|---|---|---|---|---|---|
| TC-DS-01 | Redaction in logs | U | REQ-F-112, NF-021 | log strings with API key, password members | masked |
| TC-DS-02 | Bug-report bundle has no secrets | S | REQ-F-111 | configure profile with key `NNSXS.TEST…`; export bundle; grep | not found anywhere |
| TC-DS-03 | Diagnostics counters | S | REQ-F-110, NF-050 | provoke parse errors, reconnects | counters and events shown |
| TC-DS-04 | No cleartext without acknowledgement (network capture) | S | REQ-NF-020 | `tcpdump` during TLS session | no plaintext credentials |
| TC-DS-05 | Permissions manifest | U (lint) | REQ-NF-060 | inspect merged manifests of all variants | only listed permissions; no `AD_ID` in any variant |
| TC-DS-06 | Dependency licences | U (CI) | REQ-C-001 | licence report | all GPL-3.0-compatible except Google Play services in `play` variants (additional permission (a)); no advertising, analytics or tracking library |
| TC-DS-07 | Payload size cap | U | TDS-10.4 | 1 MB message | rejected, counted |

### 6.13 Car (TC-CA)

All car tests run against the **Paid** edition (REQ-F-160); TC-ED-13
verifies that the Free edition has no car service.

| ID | Title | Type | Req | Procedure | Expected |
|---|---|---|---|---|---|
| TC-CA-01 | Map template renders and updates | S (DHU) | REQ-F-120 | replay session; DHU | map drawn, rows update, night mode switch |
| TC-CA-02 | Start/Stop coupling | S (DHU) | REQ-F-120 | start in car; stop on phone and vice versa | both reflect state |
| TC-CA-03 | Profiles list selection only | S | REQ-F-122 | open profiles | list; no text input anywhere |
| TC-CA-04 | Fallback grid/list | S | REQ-F-121 | build with `carCategory=iot` | tiles and recent list work |
| TC-CA-05 | Car app quality checklist | X | REQ-NF-032, TGT-05 | walk the checklist on DHU and real head unit | all items pass |
| TC-CA-06 | Car location source | S | REQ-F-123 | grant car location | used when better |

### 6.14 Kiosk (TC-KI)

| ID | Title | Type | Req | Procedure | Expected |
|---|---|---|---|---|---|
| TC-KI-01 | Full-screen kiosk on 800×480 and 720×1280 | S (Pi) | REQ-F-130, P-004 | start with `kiosk=true` on LAB-Pi4/Pi5 | layouts SCR-30/31; targets ≥ 64 px |
| TC-KI-02 | On-screen keyboard | S (Pi) | REQ-F-130 | edit profile | keyboard appears, input works |
| TC-KI-03 | Provisioning file | S | REQ-F-131 | valid file; file with error; secret file 0644; secret file 0600 owned by another user (e.g. root); secret file 0400 owned by the kiosk user; relative `api_key_file` | applied; error screen with line; refused with diagnostic naming path, owner, mode and the expected ones (both cases); accepted; resolved against the provisioning file's directory |
| TC-KI-04 | systemd restart | S (Pi) | REQ-NF-012 | kill process | restarted in ≤ 10 s; session resumes |
| TC-KI-05 | Intermittent connectivity | S (Pi) | REQ-F-133 | toggle hotspot during replay | session continues; queue drains; cached tiles shown |
| TC-KI-06 | Unsynced clock flag | I | REQ-F-132 | offset system clock 1 h vs GNSS | `TIME_UNSYNCED` flag |
| TC-KI-07 | Status page | S | REQ-F-135 | enable; curl | JSON with state; read-only |
| TC-KI-08 | Boot to kiosk | S (Pi) | REQ-F-130, P-004, NF-012 | fresh Raspberry Pi OS (labwc), autologin, package installed, autostart lines of the Pi guide added; power on; then log out of the session | app full screen without any input; `systemctl --user show thingsmapper-kiosk` lists `WAYLAND_DISPLAY`/`DISPLAY`; unit not started before the session; unit inactive after logout, no restart loop |
| TC-KI-09 | Provisioning-file precedence | U | REQ-F-131 | combinations of `--config` (existing / missing), user file and system file | exactly one file read: `--config`, else user, else system; missing `--config` path → start-screen error without fall-through; values never merged; diagnostics names the file used |

### 6.15 Replay tool (TC-RT)

| ID | Title | Type | Req | Procedure | Expected |
|---|---|---|---|---|---|
| TC-RT-01 | Record → publish round trip | I | REQ-F-140 | record from Mosquitto; publish back; compare | identical payloads and relative timing (± 50 ms) |
| TC-RT-02 | Track server gpsd-compatible | I | REQ-F-140 | `gpspipe`-style client | valid TPV/SKY JSON |
| TC-RT-03 | In-app demo mode | S | REQ-F-141 | start demo on desktop | session with replayed data |

### 6.16 Network/tiles (TC-N)

| ID | Title | Type | Req | Procedure | Expected |
|---|---|---|---|---|---|
| TC-N-06 | Tile traffic discipline | I | REQ-NF-006 | pan across cached area; count requests | zero re-downloads for cached tiles; ≤ 2 concurrent |
| TC-N-07 | HTTP timeouts and UA | I | TDS-6.2 | slow stub | connect 10 s / request 30 s; UA header |

### 6.17 Performance (TC-P)

| ID | Title | Type | Req | Procedure | Expected |
|---|---|---|---|---|---|
| TC-P-01 | Cold start | S | REQ-NF-001 | 5 cold starts, measure to first map frame | median ≤ 3 s (phone), ≤ 8 s (Pi 4) |
| TC-P-02 | Message-to-point latency | I/S | REQ-NF-002 | replay with timestamps; instrument | p95 ≤ 500 ms phone, ≤ 1 s Pi |
| TC-P-03 | Frame rate with 20 k points/lines | S | REQ-NF-003 | load synthetic session; pan/zoom 30 s; Perfetto / frame counter | ≥ 30 fps phone, ≥ 15 fps Pi 4 |
| TC-P-04 | Memory with 50 k measurements | S | REQ-NF-004 | load; export; map | heap ≤ 256 MB, no OOM |
| TC-P-05 | Pipeline throughput | I | REQ-F-045 | 10 msg/s × 60 s | queue depth stays < 100 |

### 6.18 Field (TC-F)

| ID | Title | Type | Req | Procedure | Expected |
|---|---|---|---|---|---|
| TC-F-01 | Urban drive, phone, TTS | F | TGT-02 | protocol §7, 1 h | ≥ 99 % stored, ≥ 99 % uploaded ≤ 5 min after connectivity |
| TC-F-02 | Side-by-side with TTN Mapper v3 | F | TGT-03 | protocol §7 with second phone | same measurements ± duplicates; payload diff = `useragent` |
| TC-F-03 | Pi in car, gpsd, LTE hotspot | F | TGT-04 | 4 h drive | no restart; queue drained; fps target |
| TC-F-04 | Android Auto on real head unit | F | TGT-05 | 30 min drive | usable, no distraction violations, night mode |
| TC-F-05 | Battery | F | REQ-NF-005, TGT-07 | 1 h screen-off session, `batterystats` before/after | ≤ 8 % |
| TC-F-06 | ChirpStack v4 private network | F | REQ-F-003 | private gateway + CS4 | measurements stored; upload disabled per profile works |
| TC-F-07 | Walk test | F | REQ-F-030 | 30 min | plausible positions; accuracy recorded |

### 6.19 Compliance and accessibility (TC-CX)

| ID | Title | Type | Req | Procedure | Expected |
|---|---|---|---|---|---|
| TC-CX-01 | TalkBack pass | X | REQ-NF-033 | navigate all screens | labels, order |
| TC-CX-02 | Contrast check | X | REQ-NF-030 | tokens against surfaces | ≥ 4.5:1 text, ≥ 3:1 chips |
| TC-CX-03 | Play pre-launch and Data Safety consistency | X | REQ-NF-060, NF-023 | compare declarations of both editions with TC-DS-02/04 and TC-ED-02/06 findings | consistent |
| TC-CX-04 | OSM policy checklist | X | REQ-NF-061 | UA, attribution, caching, concurrency | all satisfied |
| TC-CX-05 | Font scale 200 % | S | REQ-NF-030 | phone | no clipped controls |

### 6.20 Editions and advertising (TC-ED)

| ID | Title | Type | Req | Procedure | Expected |
|---|---|---|---|---|---|
| TC-ED-01 | Functional parity of editions | S (CI) | REQ-F-151, F-152 | run the complete system/UI suite (all TC-S cases except TC-CA) against `freeFoss` and `paidFoss` in the CI matrix | identical results; any edition-specific failure is a defect |
| TC-ED-02 | Paid edition contains no ad code | U (CI) | REQ-F-152, NF-060 | `apkanalyzer`/dex scan and merged-manifest check of `paidPlay`/`paidFoss` | no ad SDK or sponsor-provider classes reachable, no consent SDK, no `AD_ID`, no ad network hosts in resources |
| TC-ED-03 | `AdPolicy` decision table | U | REQ-F-153 | all combinations of edition × screen × session state × modal × consent × online | visible exactly per FDS-17.2 |
| TC-ED-04 | No ads while mapping and in notifications | S | REQ-F-153 | Free edition: start a session on the map, open drive view, inspect notification and lock screen; stop | map/drive slot collapsed while Starting/Running/Stopping; never in notification or lock screen; reappears when idle |
| TC-ED-05 | Slot placement and spacing | S (screenshot) | REQ-F-153 | screenshots at 360×640 dp, 411×891 dp and tablet; measure distance to Start/Stop, map controls, attribution | ≥ 48 dp; attribution never covered; label *Sponsor*/*Ad* visible |
| TC-ED-06 | No personal data reaches the ad source | S | REQ-F-154, NF-022 | mitmproxy during a replayed session in the Free edition; search all ad-related requests for coordinates, device/gateway ids, profile data, installation id | none found; sponsor requests are plain GETs without identifiers |
| TC-ED-07 | *Withdrawn 2026-10-24* — no ad consent (REQ-F-155 withdrawn, OP-7) | — | — | — | — |
| TC-ED-08 | Ad failure and traffic budget | S | REQ-F-156 | offline, no fill, invalid catalogue signature; 1 h foreground use with network counter | slot collapses silently; ≤ 1 refresh per 60 s; ≤ 1 MB/h; no wake locks from ad code |
| TC-ED-09 | Import from Free into Paid | S | REQ-F-158 | Free edition with 3 sessions, 20 pending uploads and 2 profiles; install Paid; import; repeat import | data complete; profiles flagged for keys; pending uploads sent once (stub ingress counts); repeated import adds nothing |
| TC-ED-10 | Sponsor catalogue handling | U/I | REQ-F-159, NF-022 | stub host serving valid, tampered and expired catalogues; count fetches | tampered/expired rejected (no ads); ≤ 1 fetch per 24 h; images cached; tap opens external browser with `utm_source=thingsmapper` |
| TC-ED-11 | Desktop and kiosk builds without ads | S | REQ-F-161 | inspect desktop/Pi packages and UI | no slots, no ad requests |
| TC-ED-12 | Store declarations | X | REQ-NF-060, NF-064, REQ-C-010 | compare Data Safety (per edition), "contains ads", trader declaration and listings with TC-ED-02/06 findings | consistent |
| TC-ED-13 | Free edition has no car service | U (CI) | REQ-F-160 | merged manifest of `free*` variants; connect Free edition to the DHU | no `CarAppService`, no `automotive_app_desc`; app not listed on the DHU |
| TC-ED-14 | Upgrade entry | S | REQ-F-157 | Free edition: check settings, about, map, drive view, notifications | entry only in settings, about and the start screen; opens the Paid listing; absent in Paid |
| TC-ED-15 | Free start screen rules | U + S | REQ-F-162 | `StartScreenPolicy` table; then launcher start, resume from background, open from the mapping notification, session running, rotation and process restore, first start, Paid edition, desktop | shown only on a launcher start without a running session; first start shows the notice in onboarding; both buttons active at once; back = continue; no sponsor content; never in Paid or desktop |

### 6.21 Cost awareness and support (TC-SU)

| ID | Title | Type | Req | Procedure | Expected |
|---|---|---|---|---|---|
| TC-SU-01 | Funding data handling | U/I | REQ-F-167 | stub host with valid, tampered and expired files; count fetches; capture Paid-edition traffic | tampered rejected; ≤ 1 fetch per 24 h and only when needed; no identifiers; Paid fetches the funding file but never the sponsor catalogue |
| TC-SU-02 | Cost page | S | REQ-F-163 | open with fresh data, with cached data offline, without data offline, with data older than 120 days | figures and date; cached figures with date; explanation without amounts; outdated note |
| TC-SU-03 | Thank-you card decision table | U | REQ-F-165 | `YearlyCardPolicy` with a fake clock: install age, session count, last shown, supporter, speed, car screen, open dialog | shown exactly per FDS-18.4 |
| TC-SU-04 | Thank-you card in use | S | REQ-F-165 | replay a session; stop while moving (replayed 30 km/h) and while standing; advance the clock 12 months | not shown while moving or running; shown once when standing; not again within 12 months; the user's figures match the statistics |
| TC-SU-05 | Store purchases | S | REQ-F-166 | Play license tester: buy, cancel and renew the subscription, buy each tip, restore after reinstall | purchases acknowledged/consumed; supporter badge; card suppressed; no function changes; Free keeps banners and start screen |
| TC-SU-06 | Support outside the stores | S | REQ-F-166 | `freeFoss`, desktop, kiosk: tap *Support*; dependency/dex scan | GitHub Sponsors opens in the browser; card suppressed for 12 months; no Play Billing classes in `foss` |
| TC-SU-07 | No outside payment links in store builds | U (CI) | REQ-C-011 | scan strings, resources and URLs of `freePlay` and `paidPlay` | no GitHub Sponsors, PayPal or similar payment URLs |
| TC-SU-08 | Why this update | U (CI) / S | REQ-F-164 | release build without the line; *About → What's new* after an update | CI refuses the build; the installed version's notes with the line; no dialog after the update |
| TC-SU-09 | Data Safety purchase history | X | REQ-NF-060 | compare the Play declarations with the billing use | purchase history declared for the `play` variants |
| TC-SU-10 | Prices shown | X | REQ-F-162, REQ-C-010 | start screen in EUR and non-EUR locales; compare with the store listings | €6 in EUR locales, generic text elsewhere; consistent with the store price |

### 6.22 Packaging (TC-PK)

| ID | Title | Type | Req | Procedure | Expected |
|---|---|---|---|---|---|
| TC-PK-01 | x86_64 `.deb` installs on every supported release | S (CI) | REQ-P-003, C-008 | in clean `debian:12`, `debian:13`, `ubuntu:22.04` and newest-Ubuntu-LTS containers: `apt-get update && apt-get install -y ./thingsmapper_*_amd64.deb`, then `thingsmapper --version` | installs without unmet dependencies on every release; prints the version and git SHA; exit code 0 |
| TC-PK-02 | arm64 `.deb` installs on Raspberry Pi OS bases | S (CI) | REQ-P-004, C-008 | as TC-PK-01 in clean arm64 `debian:12` and `debian:13` containers (bases of Raspberry Pi OS Bookworm and Trixie); once per milestone also on LAB-Pi4 with Raspberry Pi OS Bookworm | as TC-PK-01; the kiosk unit and `/usr/share/thingsmapper/kiosk/autostart` are installed, the unit is not enabled |

### 6.23 Libraries (TC-LB)

| ID | Title | Type | Req | Procedure | Expected |
|---|---|---|---|---|---|
| TC-LB-01 | Library build stands alone | U (CI) | REQ-NF-045 | clean checkout with `app/` deleted; build and `check` the `lib/` build | builds and all library tests pass; nothing in `lib/` refers to `app/` |
| TC-LB-02 | Common tests pass on every target | U (CI) | REQ-NF-045, P-005 | run every library's `commonTest` as JVM test, Android host test and `iosSimulatorArm64Test` | same tests pass on all three; no target skipped in CI |
| TC-LB-03 | Port contract suites | U/I (CI) | REQ-NF-045, NF-040 | run each contract suite of `testing` against every implementation of its port — e.g. `SecretStoreContract` against the in-memory fake, Secret Service, the `0600` file store, the Android Keystore store (instrumented, CI-Android) and later the iOS Keychain store | every implementation passes the same suite |
| TC-LB-04 | Public API changes are explicit | U (CI) | REQ-NF-045 | change a public declaration of a library without updating its ABI dump | CI fails until the dump is updated in the same pull request |

## 7. Field test protocol

**Preparation.** Charge devices to 100 %; node transmitting every 10 s at
SF7 (EU868), registered in NS-TTS (and NS-CS4 for TC-F-06); phone A with
Things Mapper (`freePlay`, `paidPlay` or `freeFoss` as under test), phone B with TTN Mapper v3
linked to the same device; both subscribed (MQTT allows several
subscribers); Pi 4 box powered from the car with gpsd running and a hotspot
from phone C; note app versions, git SHA, weather, time.

**Route.** Fixed 1 h urban loop with at least one tunnel or underground car
park (fix loss), one stretch ≥ 80 km/h, and known gateways of two networks
(home network and one reachable through Packet Broker). Record the route
GPX independently (dedicated GNSS logger).

**Procedure.** Start all devices within 30 s of each other; drive the loop;
at the mid-point switch phone A to airplane mode for 5 min (queue test);
stop all within 30 s of each other; export CSV from all devices and the
diagnostics bundle from Things Mapper; read `batterystats` for phone A.

**Evaluation.**
1. Node uplinks sent (node counter or TTS console events) vs. measurements
   stored by A and B → ≥ 99 % stored (TGT-02).
2. Join A and B exports on `dev_id` + `f_cnt`: same set (± duplicates),
   same receptions, positions within accuracy of each other; compare A's
   payloads (from the diagnostics upload log) with B's recorded uploads →
   only `useragent` differs (TGT-03).
3. Outbox: all geo-tagged measurements `done` within 5 min after the
   airplane-mode phase; none expired/rejected unexpectedly.
4. After 24 h check the session's points on ttnmapper.org (advanced maps →
   device) for A's `userid`.
5. Battery delta ≤ 8 % (TC-F-05); Pi: no restart in `journalctl`, fps log
   ≥ 15 (TC-F-03).

## 8. Traceability matrix (requirement → test cases)

| Requirement | Test cases |
|---|---|
| REQ-P-001…009 | TC-SE-03, TC-CA-*, TC-KI-01, TC-KI-08, TC-PK-01, 02, TC-CX-05, TC-LB-02, CI matrix (TDS-11), TC-DS-05, TC-ED-01 |
| REQ-F-001…009 | TC-PR-01…10 |
| REQ-F-010…015 | TC-SE-01…07 |
| REQ-F-020…028 | TC-IN-01…16 |
| REQ-F-030…036 | TC-PO-01…09, TC-F-07 |
| REQ-F-040…047 | TC-RU-01…07, TC-UP-01 |
| REQ-F-050…056 | TC-ST-01…10 |
| REQ-F-060…069 | TC-MA-01…11, TC-N-06 |
| REQ-F-070…073 | TC-IS-01…05, TC-MA-08 |
| REQ-F-080…088 | TC-UP-01…11 |
| REQ-F-090…092 | TC-AL-01, 02, TC-SE-03 |
| REQ-F-100…103 | TC-SO-01…06 |
| REQ-F-110…112 | TC-DS-01…03, TC-IN-06 |
| REQ-F-120…124 | TC-CA-01…06, TC-F-04 (Paid edition) |
| REQ-F-130…135 | TC-KI-01…09, TC-F-03 |
| REQ-F-140, 141 | TC-RT-01…03 |
| REQ-F-150…162 | TC-ED-01…15 |
| REQ-F-163…167 | TC-SU-01…10 |
| REQ-NF-001…006 | TC-P-01…05, TC-N-06, TC-F-05 |
| REQ-NF-010…013 | TC-ST-01, 02, TC-UP-04, TC-IN-10, TC-KI-04, TC-KI-08 |
| REQ-NF-020…024 | TC-IN-09, 12, 13, TC-PR-10, TC-DS-01, 02, 04, 06 |
| REQ-NF-030…034 | TC-IS-05, TC-CX-01, 02, 05, TC-SO-03, TC-CA-05 |
| REQ-NF-040…045 | CI gates (coverage, lint), review checklist §9, TC-LB-01…04 |
| REQ-NF-050 | TC-DS-03 |
| REQ-NF-060…064 | TC-DS-05, TC-CX-03, 04, TC-SO-01, TC-ED-02, TC-ED-12 |
| REQ-C-001…011 | TC-SU-07, TC-SU-10, TC-DS-06, TC-UP-02, TC-DS-05, TC-CX-04, CI arm64 job, TC-PK-01, 02 (REQ-C-008), TC-ED-12, review of `app/` for upstream code (REQ-C-009) |
| TGT-01…09 | parity checklist (M2.4 review), TC-F-01…05, TC-P-*, CI gates, adapter-guide review, store revenue reports (TGT-09) |

## 9. Review checklists (design and code)

- **Design review (phase 1 exit):** every REQ has an FDS function or a
  design element; every EVAL finding with Change/Drop/New has a REQ; open
  points have owners; ADRs cover the stack, map, car, storage, MQTT and
  licence decisions.
- **Adapter guide review (TGT-08):** a new adapter touched only its own
  package and one registration line.
- **Pull request checklist:** tests added/updated, traceability row updated,
  no new hard-coded strings, no secrets in code or fixtures, licence header
  present, changelog entry; every new implementation of a port runs its
  contract suite (TC-LB-03).

## 10. Reporting and defect management

GitHub Issues with labels `bug`, `severity:critical|high|medium|low`,
`area:*`, `platform:*`; test runs recorded per milestone in
`design/test-reports/<milestone>.md` (created in phase 2) with the executed
TC list, results, environment and known issues. CI publishes JUnit reports
and coverage.

## 11. Exit criteria per milestone (summary)

| Milestone | Must pass |
|---|---|
| M2.1 Foundation | all U/I tests of TC-IN, TC-RU, TC-PO (non-hardware), TC-ST-01/02/04/06, TC-UP-01…07, TC-RT-01/02; TC-LB-01…04; coverage gate |
| M2.2 Desktop MVP | TC-MA-*, TC-P-02/03 on desktop, TC-SE-01, TC-IS-*, TC-PO-05/06, TC-SO-06, TC-KI-09, TC-PK-01 |
| M2.3 Android MVP | TC-SE-02…05, TC-PR-*, TC-SO-01/02, TC-ST-07, TC-DS-01…05, TC-P-01/03 on phone, TC-ED-01/02/13 |
| M2.4 Parity | TC-F-01, TC-F-02, TC-F-05, TC-ST-09, TC-SO-03, TC-ED-15, TC-SU-01…04, 06…08, TGT-01/03 review |
| M3.1 Kiosk | TC-KI-*, TC-PK-02, TC-F-03, TC-P-01/03 on Pi |
| M3.2 Android Auto (Paid) | TC-CA-*, TC-F-04, TC-ED-13 |
| M3.3 Release 1.0 | TC-CX-*, TC-F-06, TC-ED-*, TC-SU-*, full regression on both editions, TGT-06 |

## 12. Risks to testing

| Risk | Mitigation |
|---|---|
| ttnmapper.org processing delay makes TC-UP-11 slow | Evaluate after 24 h; use `success:true` as the immediate criterion |
| No real Android Auto head unit available | DHU covers most; borrow a unit for TC-F-04 before 1.0 |
| Flaky broker/container tests | Testcontainers with health checks; retries only at the container level |
| Battery test variance | Same device, same route, three runs, median |
| Fixture privacy (device ids, positions) | Fixtures anonymised (ids replaced, positions shifted) before commit |
