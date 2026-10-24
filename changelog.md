# Changelog

All notable changes to this project are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/), and
this project adheres to [Semantic Versioning](https://semver.org/) for the
software. Design documents carry their own revision table in their header.

---

## [Unreleased] — Phase 1: design

**Added**
- Repository scaffold for the Things Mapper remake of TTN Mapper: README,
  contributing guide, code of conduct, licence information, `.gitignore`.
- `ttnmapper-android-v3` added as a git submodule (study reference).
- `design/01-evaluation.md` — evaluation of the original TTN Mapper Android
  app (architecture, protocol support, security, UX, build, tests) with a
  keep / change / drop disposition per feature.
- `design/02-requirements.md` — stakeholders, scope, platform targets,
  functional and non-functional requirements with stable IDs, success
  targets.
- `design/03-fds.md` — functional design specification (functions, data
  contracts, rules, state machines, error handling).
- `design/04-gui-design.md` and `design/gui/*.svg` — GUI design and
  wireframes for phone, desktop, Raspberry Pi touch and Android Auto.
- `design/05-tds.md` — technical design specification (Kotlin Multiplatform
  layered architecture, modules, data model, protocol adapters, map
  renderer, platform shells, build and release).
- `design/06-test-plan.md` — test strategy, environments, test case
  catalogue traced to requirements, field test protocol.
- `design/07-implementation-plan.md` — phases, milestones, work packages,
  risks.
- `design/adr/` — architecture decision records (licence, technology stack,
  map rendering, car platform, storage and upload).
- Editions and monetisation (ADR-0007, 2026-10-24): Free edition with
  advertisements and Paid edition without, otherwise identical except that
  Android Auto is Paid-only; requirements §6.16 (REQ-F-150…161, REQ-P-009,
  REQ-NF-063/064, REQ-C-010, TGT-09), FDS-17, GUI §5a with wireframe
  `design/gui/free-edition-ads.svg`, TDS-9.5 (build variants, `ads`
  module, edition transfer), test cases TC-ED-01…14, work packages
  WP-2.16 and WP-3.7; open points OP-7 (ad source) and OP-8 (licence
  route for ad SDKs and the App Store).
- `design/08-costs-and-funding.md` — one-time investments (incl. the Mac
  and iPhone needed to start on iOS, Android test phones, an Android Auto
  bench and a Raspberry Pi kit), recurring costs, income per sale,
  break-even, published funding data.
- ADR-0008 cost awareness and voluntary support: cost page, *Why this
  update* in release notes, yearly thank-you card, voluntary yearly
  support (store billing or GitHub Sponsors), no commercial services;
  requirements §6.17 (REQ-F-163…167), REQ-C-011, OP-11, FDS-18, GUI §5b
  with wireframe `design/gui/support-and-costs.svg`, TDS-9.6, test cases
  TC-ED-15 and TC-SU-01…10, WP-2.17, budget per milestone.

**Changed**
- Prices: Paid edition €6 once, yearly supporter €5, tips €2 / €5 / €10; the Free edition starts with a start
  screen offering the Paid edition (REQ-F-157 adjusted, REQ-F-162).
- Decisions of 2026-10-24: the sponsor catalogue is the only ad source
  (no ad SDK, no consent step — REQ-F-155 and TC-ED-07 withdrawn);
  licence route L-A — clean implementation of `app/` plus a GPLv3
  section 7 additional permission for store/platform vendor libraries
  (never advertising or analytics libraries) and app-store distribution,
  now in `license.md`.
- Android Auto is part of the Paid edition only (ADR-0004 amended);
  ADR-0001 amended: GPL-3.0-only stays, an additional permission for ad
  libraries and app stores is pending (OP-8); rule "no upstream code in
  `app/`" added (REQ-C-009).
- Project renamed from **Chirp Mapper** to **Things Mapper** (2026-10-24):
  repository moved to `pe1mew/Things-Mapper` (GitHub redirects the old
  name); all identifiers follow the new name (`thingsmapper` package and
  binary names, `ThingsMapper/<version>` user agent,
  `nl.pe1mew.thingsmapper` package root, `thingsmapper_profile` JSON key,
  `/etc/thingsmapper/` paths).
- Licence set to GNU GPL v3.0 (`GPL-3.0-only`), inherited from the original
  TTN Mapper Android app of which this project is a derived work (was MIT in
  the copied scaffold).
