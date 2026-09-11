# Contributing Guidelines

Thank you for considering contributing to Chirp Mapper. The project is in
**phase 1 (design)**: the deliverables are documents, and the most valuable
contributions right now are reviews, corrections and additions to those
documents. Code contributions follow in phase 2.

## Getting started

1. **Fork the repository** and clone your fork:
   ```sh
   git clone https://github.com/<your-user>/Chirp-Mapper.git
   cd Chirp-Mapper
   ```
2. The original TTN Mapper app is included as a submodule for reference. It
   is *not* required to work on the design documents; initialise it only when
   you want to check a statement in the evaluation against the original code:
   ```sh
   git submodule update --init
   ```
3. **Create a branch** with a descriptive name:
   ```sh
   git checkout -b docs/requirements-gnss-sources
   ```

## Working on the design documents

- Documents live in `design/`; the index and conventions are in
  [design/README.md](design/README.md).
- **Requirement IDs are stable.** `REQ-…` identifiers are referenced from the
  FDS, TDS and test plan. Never renumber; deprecate an ID (status
  *withdrawn*) and add a new one instead.
- A change to a requirement needs a matching change in the documents that
  trace to it (FDS function, TDS component, test cases). If you cannot do all
  of them, say so in the pull request so the maintainer can follow up.
- Decisions with lasting architectural impact are recorded as an ADR in
  `design/adr/` (copy the structure of an existing one: context, options,
  decision, consequences).
- Keep the document header (status, revision table) up to date.
- Diagrams are Mermaid blocks or SVG files in `design/gui/`; both render on
  GitHub. Please do not add binary formats that cannot be diffed.
- Write in plain English, present tense, one requirement per row, and use
  the RFC 2119 words (*shall*, *should*, *may*) consistently.

## Working on code (phase 2 onwards)

- Kotlin Multiplatform with Gradle; the build and test commands are
  documented in the TDS and will be mirrored here once the modules exist.
- Every source file carries `SPDX-License-Identifier: GPL-3.0-only`.
- New behaviour comes with tests in the core module; protocol changes come
  with a recorded fixture message.
- Keep commits focused. Commit messages follow the
  [Conventional Commits](https://www.conventionalcommits.org/) format
  (`feat:`, `fix:`, `docs:`, `test:`, `build:` …).
- Update [changelog.md](changelog.md) under *Unreleased*.

## Submitting a pull request

1. Push your branch to your fork.
2. Open a pull request against `main` with a clear description of *what*
   changed and *why*. Reference issues and requirement IDs where applicable.
3. Pull requests are reviewed by the maintainer; be open to feedback.

## Licence of contributions

By contributing you agree that your contribution is licensed under the
project licence, the GNU GPL v3.0 (`GPL-3.0-only`) — "inbound = outbound".
Do not submit code or text you are not entitled to license under these
terms (for example, copies of proprietary SDK documentation or code from
projects with an incompatible licence). See [license.md](license.md).

## Reporting issues

- Check whether the issue has already been reported.
- For design issues, quote the document and section (or requirement ID).
- For (future) software issues, include platform, version, steps to
  reproduce and — if relevant — a redacted log. Never paste API keys or MQTT
  passwords; the application's bug-report function redacts them, please do
  the same when reporting by hand.

## Community standards

Follow the [Code of Conduct](code_of_conduct.md). Be respectful and
collaborative.
