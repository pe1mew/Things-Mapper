# ADR-0008 — Cost awareness and voluntary support

Status: **Accepted** (maintainer decisions, 2026-10-24) · Date: 2026-10-24

## Context

Things Mapper is a hobby project. Its money costs are small but real and
partly recurring: test hardware for Android, an Android Auto bench setup, a
Raspberry Pi kit, and — to start on iOS — a Mac and an iPhone plus Apple's
yearly fee ([08 Costs and funding](../08-costs-and-funding.md)). The Paid
edition costs €6 once, for the lifetime of the app, while the platforms
demand maintenance every year. Users of the original TTN Mapper have long
been used to paying nothing — in Dutch: *voor niets gaat de zon op* (only
the sun rises for free). The maintainer wants users to be aware of the
costs, how small they are, and that they recur.

## Decision drivers

- Make the costs visible without nagging; keep the app's *Respectful*
  principle (no guilt, no dark patterns, nothing while driving).
- Give people who use the app for years a way to contribute in proportion.
- Comply with store payment rules: Google Play requires its billing system
  for in-app purchases of digital goods, and anything given in return (even
  a badge) makes a payment a digital purchase; both stores restrict links
  to outside payment inside apps.
- No feature split beyond ADR-0007 (ads and Android Auto).
- Hobby scale: no commercial obligations.

## Decision

### D1 — Cost transparency page

A *What Things Mapper costs* page in *About* (SCR-15) shows this year's
costs, how much is covered, the open investments and the note that time is
volunteered. The figures come from signed funding data published next to
the sponsor catalogue (doc 08 §6), so all builds show the same numbers
without any tracking. The README carries the same summary.

### D2 — "Why this update" in every release

Every release note (store *What's new*, GitHub release, changelog, the
in-app *What's new* in About) has a short *Why this update* line naming the
mandatory platform work it contains, e.g. "required because Google Play
demands target API 36 from 31 August 2026". Invisible maintenance becomes
visible. No extra pop-up is used for it.

### D3 — Yearly thank-you card

At most once per twelve months, after the user stops a session while not
moving, a card thanks the user with their own year in numbers (computed on
the device) and one line on the yearly costs, with *Support* and *Not now*.
Never while a session runs, never in the car, never at app start. Shown in
all editions and builds.

### D4 — Voluntary yearly support

| Build | How | What the supporter gets |
|---|---|---|
| Store builds (`freePlay`, `paidPlay`; iOS later) | store in-app purchase: a yearly supporter subscription at **€5 per year** and one-off tips of **€2, €5 and €10** (decided 2026-10-24) | a thank-you badge in About; the yearly card stays away while supporting — **no features, no removal of ads or start screen** (the Paid edition does that) |
| `foss` variants, desktop, Raspberry Pi | link to the maintainer's GitHub Sponsors page in the browser | nothing tracked; the yearly card stays away for twelve months after tapping *Support* |

Store billing uses the Google Play Billing Library (covered by additional
permission (a), license.md) and StoreKit on iOS (system framework).

### D5 — Realistic price for the Paid edition

The Paid edition costs **€6 once** (nearest store price point; other
currencies as converted by the stores). The Free start screen and the cost
page explain what the price covers.

### D6 — No commercial services

No paid support, consultancy, custom builds or service contracts are
offered; sponsorship is limited to banner slots in the sponsor catalogue.
Income only covers costs (doc 08 §1).

### D7 — Tone

All texts thank rather than blame: what the app costs, who pays, how to
help. No countdowns, no blocking dialogs, no repeated reminders; the Free
start screen (ADR-0007 D2a) is the only recurring notice and it never
blocks for longer than one tap.

## Consequences

- Requirements §6.17 (REQ-F-163…167), REQ-C-010 (price €6) and REQ-C-011
  (payment rules), REQ-NF-060 (purchase history in Data Safety), OP-11 (Apple investment go/no-go); FDS-18; GUI screens
  SCR-15…17; TDS module `support` with Play Billing in the `play` variants;
  test cases TC-SU-01…10; work package WP-2.17; budget section in the
  implementation plan.
- The funding data must be kept current (at least quarterly) — otherwise the
  cost page shows its `updated` date as stale.
- A GitHub Sponsors profile and `.github/FUNDING.yml` are set up in WP-3.7.

## Links

[08 Costs and funding](../08-costs-and-funding.md), [ADR-0007](ADR-0007-editions-and-monetisation.md),
[02 Requirements §6.17](../02-requirements.md), [03 FDS FDS-18](../03-fds.md),
[04 GUI §5b](../04-gui-design.md), [05 TDS §9.6](../05-tds.md).
