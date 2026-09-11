# ADR-0001 — Licence: GPL-3.0-only

Status: **Accepted** · Date: 2026-09-11

## Context

Chirp Mapper is a remake of the TTN Mapper Android app
(`ttnmapper-android-v3`, JP Meijers). The upstream repository's `LICENSE`
file is the unmodified GNU GPL v3.0 text; the source files carry no
per-file notices and there is no "or any later version" statement. The
remake reuses the upstream's behaviour, data contracts, mapping rules,
Packet Broker conventions, recorded test fixtures and — where convenient —
code, which makes it a derived work in the sense of the GPL. The repository
scaffold copied from another project of the maintainer was MIT-licensed and
had to be corrected.

## Decision drivers

- Legal correctness for a public repository containing a derived work.
- Simplicity for contributors (one licence for everything).
- Compatibility with the planned dependency set and distribution channels.

## Options

| Option | Assessment |
|---|---|
| **GPL-3.0-only** | Matches the upstream grant exactly; no consent needed; Apache-2.0/MIT/BSD/LGPL/MPL dependencies are compatible one-way; F-Droid and GitHub distribution are straightforward; Google Play accepts GPL apps in practice. |
| GPL-3.0-or-later | Preferable for long-term flexibility, but the upstream grant is v3 without "or later"; relicensing the derived parts to "or later" would need the upstream copyright holders' agreement. Could be applied to *new* files only, at the cost of a mixed repository. |
| Permissive licence via clean-room rewrite | Would require not reusing any upstream expression (code, fixtures) and documenting the clean-room process; contradicts the project's intent to build on the original and adds no practical benefit for a community tool. |

## Decision

The whole repository is licensed **GPL-3.0-only**. Source files carry
`SPDX-License-Identifier: GPL-3.0-only`. Contributions are accepted under
the same terms (inbound = outbound). The upstream authors are credited in
`license.md` and in the About screen.

## Consequences

- Every runtime dependency must be GPL-3.0-compatible; the TDS dependency
  table is checked by a licence report in CI (TC-DS-06). Proprietary SDKs
  (including Google Play Services libraries) cannot be part of the `foss`
  build; the optional `play` flavour links Play Services location as a
  system-provided library, which is the common practice for GPL Android apps
  but should be reviewed before the first Play release.
- Binary distributions must offer the corresponding source; the public
  repository and tagged releases satisfy this.
- A future **iOS shell** faces the known tension between GPLv3 and the App
  Store terms (the FSF's position is that they are incompatible). Options
  then: distribution outside the App Store (TestFlight/enterprise/alternative
  marketplaces where available) or an additional permission, which would
  require the agreement of all copyright holders including upstream. This
  is recorded as risk R-9 in the implementation plan and decided in
  phase 4.
- If the project ever becomes a fully clean-room implementation, this ADR
  must be revisited before any licence change.

## Links

[license.md](../../license.md), [01 Evaluation EVAL-31](../01-evaluation.md),
[02 Requirements REQ-C-001](../02-requirements.md).
