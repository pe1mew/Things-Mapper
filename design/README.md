# Chirp Mapper — design documents

This folder holds the phase 1 deliverables of the Chirp Mapper project: the
evaluation of the original TTN Mapper Android app and the requirements and
designs of its multi-platform remake.

## Documents

| # | Document | Purpose | Status |
|---|---|---|---|
| 01 | [Evaluation of TTN Mapper Android v3](01-evaluation.md) | What the original does, how it is built, what is good, what is not; keep / change / drop per feature | Draft v0.1 |
| 02 | [Requirements and targets](02-requirements.md) | Stakeholders, scope, platform targets, numbered requirements (functional, non-functional, constraints), success targets | Draft v0.1 |
| 03 | [Functional design specification (FDS)](03-fds.md) | Functions of the system, their behaviour, rules, data contracts, state machines, error handling — independent of technology | Draft v0.1 |
| 04 | [GUI design](04-gui-design.md) + [`gui/`](gui/) | Design principles, information architecture, screen inventory and wireframes per form factor, navigation, theming, accessibility | Draft v0.1 |
| 05 | [Technical design specification (TDS)](05-tds.md) | Architecture, modules, interfaces, data model, protocol adapters, map renderer, platform shells, security, build and release | Draft v0.1 |
| 06 | [Test plan](06-test-plan.md) | Strategy, environments, test data, test case catalogue traced to requirements, field test protocol, exit criteria | Draft v0.1 |
| 07 | [Implementation plan](07-implementation-plan.md) | Phases, milestones, work packages, dependencies, risks, release strategy | Draft v0.1 |
| ADR | [Architecture decision records](adr/) | One file per lasting decision: context, options, decision, consequences | — |

Reading order for a newcomer: 01 → 02 → 03 → 04 → 05 → 06 → 07. The ADRs are
referenced from the TDS where a decision matters.

## Conventions

### Identifiers and traceability

| Prefix | Meaning | Defined in | Referenced from |
|---|---|---|---|
| `EVAL-nn` | Evaluation finding | 01 | 02 (rationale of requirements) |
| `REQ-F-nnn` | Functional requirement | 02 | 03, 05, 06 |
| `REQ-NF-nnn` | Non-functional requirement (performance, reliability, security, usability, maintainability) | 02 | 03, 05, 06 |
| `REQ-P-nnn` | Platform requirement | 02 | 04, 05, 06 |
| `REQ-C-nnn` | Constraint (legal, licence, external interface) | 02 | 05 |
| `TGT-nn` | Measurable success target | 02 | 06, 07 |
| `FDS-n.m` | Function in the functional design | 03 | 04, 05, 06 |
| `SCR-nn` | Screen in the GUI design | 04 | 05, 06 |
| `TDS-n.m` | Component / design element | 05 | 06, 07 |
| `TC-nnn` | Test case | 06 | — |
| `WP-n.m` | Work package | 07 | — |
| `ADR-nnnn` | Decision record | adr/ | 05 |

IDs are never reused. A withdrawn requirement keeps its row with status
*withdrawn* and a pointer to its replacement.

### Requirement wording

- *shall* — mandatory; *should* — expected unless there is a documented
  reason; *may* — optional.
- Priority uses MoSCoW: **M**ust, **S**hould, **C**ould, **W**on't (this
  release).
- Each requirement has a rationale (often an `EVAL-nn` finding) and an
  acceptance hint used by the test plan.

### Document header

Every document starts with a status line and a revision table:

```
Status: Draft | Review | Approved
| Rev | Date | Author | Change |
```

### Diagrams

Mermaid for flows, sequences and state machines (renders on GitHub); SVG in
`gui/` for wireframes. No binary diagram formats.

## Glossary (short)

| Term | Meaning |
|---|---|
| Mapping device / node | The LoRaWAN end device carried by the surveyor whose uplinks are used for mapping |
| Network server (NS) | The LoRaWAN network server the mapping device is registered on: The Things Stack v3 (TTS) or ChirpStack v4 |
| Reception | The metadata of one uplink as received by one gateway (RSSI, SNR, gateway id/location, timing) |
| Measurement | One uplink paired with the surveyor's position and all its receptions — the unit that is stored and uploaded |
| Session | All measurements between one start and stop of mapping |
| Profile | The stored connection settings for one mapping device on one network server (broker, credentials, topic, filters) |
| Packet Broker (PB) | The peering hub between LoRaWAN networks; receptions forwarded through it carry the forwarding network's id |
| Ingress API | ttnmapper.org's HTTP endpoint that accepts measurements (`/android/v4`) |
| Coverage tiles | Raster tiles served by tms.ttnmapper.org that show existing coverage per network |
| LoRdrive mode | TTN Mapper's name for drawing gateway markers and reception lines on the map while driving |
| Level | TTN Mapper's signal quality figure used for colouring: RSSI plus SNR when SNR is negative |
