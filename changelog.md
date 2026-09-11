# Changelog

All notable changes to this project are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/), and
this project adheres to [Semantic Versioning](https://semver.org/) for the
software. Design documents carry their own revision table in their header.

---

## [Unreleased] — Phase 1: design

**Added**
- Repository scaffold for the Chirp Mapper remake of TTN Mapper: README,
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

**Changed**
- Licence set to GNU GPL v3.0 (`GPL-3.0-only`), inherited from the original
  TTN Mapper Android app of which this project is a derived work (was MIT in
  the copied scaffold).
