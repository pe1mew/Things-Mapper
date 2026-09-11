# ADR-0006 — MQTT client: port with HiveMQ MQTT Client implementation

Status: **Accepted** · Date: 2026-09-11

## Context

Uplink metadata arrives over MQTT from The Things Stack (MQTT 3.1.1, QoS 0
only, TLS on 8883) and ChirpStack v4 (any MQTT 3.1.1/5 broker, usually
Mosquitto). The client must run inside an Android foreground service and in
a JVM process on Linux/Pi, support TLS with a custom CA, and expose enough
control for the core's own reconnect state machine. The original used a
community fork of the abandoned Paho Android service (EVAL-14) and could
not do TLS at all (EVAL-10).

## Decision drivers

- Maintained library, Apache-2.0 or compatible.
- Works on Android (no Android `Service` coupling) and plain JVM.
- MQTT 3.1.1 and 5, TLS with `SSLContext`, back-pressure friendly API.
- Reconnect policy owned by the core (testable), not hidden in the library.
- A path to iOS/native later.

## Options

| Option | Assessment |
|---|---|
| **HiveMQ MQTT Client** | Actively maintained, Apache-2.0, MQTT 3.1.1 and 5, TLS via `SSLContext`, reactive/async/blocking APIs, works on Android API 24+ and JVM; Netty-based (adds ≈ 2 MB, needs R8 keep rules). |
| Eclipse Paho Java client | EPL-2.0/EDL, widely used, MQTT 3.1.1 (v5 in a separate client); maintenance-mode releases; synchronous callbacks; fine as a fallback behind the same port. |
| Paho Android service / hannesa2 fork | Couples to an Android `Service`, beta from JitPack; rejected. |
| Pure-Kotlin multiplatform clients (e.g. kmqtt) | Attractive for iOS/native from one code base; smaller communities, less battle-tested TLS; candidate for the iOS shell later. |
| Own client on Ktor sockets | Not justified. |

## Decision

Define `LinkPort` in the core and implement it with the **HiveMQ MQTT
Client** for JVM/Android (`Mqtt3` API for TTS, `Mqtt5` optional for
ChirpStack). Library auto-reconnect is disabled; the core's link state
machine (FDS §5.2) drives connect/backoff/terminal errors so that the
behaviour is unit-tested with a scripted link. The iOS implementation is
decided in phase 4 (native library via expect/actual or a pure-Kotlin
client).

## Consequences

- R8/ProGuard keep rules for Netty; APK size increase accepted.
- TLS: platform trust store by default; per-profile CA via a dedicated
  `TrustManager`; plaintext only with acknowledgement (REQ-NF-020).
- Battery behaviour on Android is measured (TC-F-05); if unsatisfactory the
  port allows a Paho-based implementation without touching the core.
- Integration tests run against a Mosquitto container (TC-IN-09…13).

## Links

[03 FDS FDS-3.1, §5.2](../03-fds.md), [05 TDS §6.1](../05-tds.md).
