# ADR-0007 — Editions and monetisation: Free with ads, Paid without ads

Status: **Accepted** — maintainer decisions of 2026-10-24: editions with
Paid-only Android Auto; ad source = sponsor catalogue (OP-7); licence route =
L-A (OP-8) · Date: 2026-10-24

## Context

Publishing on Google Play and the Apple App Store costs money: the Google
Play developer registration is a one-time 25 USD, the Apple Developer
Program 99 USD per year. The maintainer has decided that these costs are
covered by the apps themselves:

- **Android:** a *Free* app with advertisements and a *Paid* app without
  advertisements. Both are functionally identical **except for the car**:
  Android Auto is offered **only in the Paid edition** (maintainer decision
  2026-10-24, after reviewing the car ad restrictions below). This keeps ads
  away from the car screen by construction and gives the Paid edition a
  clear added value.
- **iOS** (phase 4): the same model — Free with advertisements, Paid without,
  functionally identical (there is no car feature on iOS in scope; should
  CarPlay ever be added, it follows the same rule and is Paid-only).

Desktop and Raspberry Pi builds are not distributed through an app store and
are not part of this model (assumption: they stay free and ad-free).

The project is licensed GPL-3.0-only ([ADR-0001](ADR-0001-license.md)),
promises not to share location data with third parties (REQ-NF-022) and is
used while driving (REQ-F-072, REQ-F-120). These collide with common ad
practice in three places — licence, privacy and safety — and the decision
below resolves each of them.

## Decision drivers

- Functional parity between editions for everything a surveyor needs on the
  phone, tablet, desktop or kiosk: no "crippleware"; the single deliberate
  difference is the car experience.
- Driver safety: nothing that draws attention while a session runs.
- Store rules: Google Play ads policy (no disruptive ads, no ads in
  notifications or on the lock screen), the Android for Cars app quality
  guidelines (AD-1: no text ads beyond the advertiser's or product name;
  NA-1: no ads in notifications), AdMob/AdSense placement rules (no
  accidental clicks), App Store review guidelines.
- Privacy: location traces are personal data (REQ-NF-062); the ad channel
  must not leak them.
- Licence: GPL-3.0 compatibility of everything that is linked into the app
  and of the distribution channels.
- Low maintenance for a single maintainer.

## Decision

### D1 — Two editions per store, one code base

Each store gets two apps built from the same source; the edition is a
build-time constant.

| Aspect | Free edition | Paid edition |
|---|---|---|
| Price | free | **€6 once** (decided 2026-10-24, [ADR-0008](ADR-0008-cost-awareness-and-support.md) D5) |
| Features | all phone/tablet functions | all phone/tablet functions — identical |
| Android Auto (car) | **not included** — no car service in the app | **included** (REQ-F-120…124) |
| Advertisements | banner slots per D2 | none; no ad code, no ad SDK, no ad permission |
| Ad consent step | none — the sponsor catalogue stores nothing on the device for advertising | none |
| Start screen | at every app start (D2a) | none |
| Application id (Android) | `nl.pe1mew.thingsmapper` | `nl.pe1mew.thingsmapper.paid` |
| Bundle id (iOS, phase 4) | `nl.pe1mew.thingsmapper` | `nl.pe1mew.thingsmapper.paid` |
| Name / icon | *Things Mapper* | *Things Mapper* + edition badge (name OP-10, GUI-OP-5) |

Rejected alternative: one app with an in-app purchase "remove ads" (single
listing, no data migration) — it needs the proprietary Play Billing library
and StoreKit integration and does not match the maintainer's two-app model.
It can be revisited without architectural change because the edition logic
sits behind one flag.

### D2 — Where ads may and may not appear

Ads are **banners in reserved slots** only:

| Context | Ads |
|---|---|
| Secondary screens: sessions, session detail, statistics, profiles, settings, diagnostics, about | yes — one banner slot per screen |
| Map screen while **no** session runs | yes — one banner below the indicator chips, away from the Start control |
| Map screen and drive view while a session runs | **no** |
| Android Auto / car screen | **not applicable** — the car experience exists only in the Paid edition, which has no ads (car guidelines AD-1/NA-1, driver distraction) |
| Notifications, lock screen, widgets | **no** (Play ads policy, car guideline NA-1) |
| Onboarding, permission and consent dialogs, error dialogs | **no** |
| Interstitial, full-screen, rewarded, video or audio ads | **never** |
| Desktop, Raspberry Pi kiosk | **never** (not store-distributed) |
| Free start screen (SCR-14) | **no ad slot** — the start screen is a notice for the Paid edition (D2a), not an ad |

A slot never overlaps or directly adjoins the Start/Stop control or map
controls (minimum gap 48 dp), and a failed or empty ad collapses its slot
silently.

### D2a — Free start screen (maintainer decision 2026-10-24)

The Free edition always starts with a start screen (SCR-14). It says that
this is the Free edition and that the Paid edition (€6 once) removes the
banners and this screen and adds Android Auto, and offers *Buy the Paid
edition* and *Continue with Free*. Rules that keep it fair and safe:

- shown at every start of the app from the launcher; not when the app
  returns from the background, not when it is opened from the mapping
  notification, never while a session runs; at the very first start the
  onboarding's *Welcome* step carries the same notice;
- *Continue with Free* is available immediately — no countdown;
- no sponsor or third-party content on it, so it is a notice, not an
  interstitial ad (D2 still holds);
- one line links to the cost page ([ADR-0008](ADR-0008-cost-awareness-and-support.md) D1).

### D3 — Ad privacy

The ad source receives no location, measurement, gateway, device, profile or
installation-id data. Only non-personalised (contextual or untargeted) ads
are requested; on iOS no tracking that requires App Tracking Transparency.
The Paid edition makes no ad-related network request at all.

### D4 — Ad source: sponsor catalogue (OP-7 decided)

`AdProvider` is a port; the screens only know an `AdSlot`. Two
implementations are designed:

| | **A. Sponsor catalogue** (no SDK) — **chosen** | **B. Ad network SDK** (Google Mobile Ads + UMP consent) — not chosen |
|---|---|---|
| How | Small signed JSON catalogue + banner images on the project's static site (GitHub Pages); the app rotates banners locally; a tap opens the sponsor's site in the browser | Google's SDK serves non-personalised banners; Google's User Messaging Platform collects consent in the EEA/UK/CH |
| Who sells the ads | the maintainer (e.g. LoRaWAN hardware vendors, community sponsors) | the network (automatic fill) |
| Tracking / identifiers | none; no impression reporting; static `utm_source` on links | SDK-level device signals; `AD_ID` permission unless removed; Data Safety "advertising" declarations |
| Consent dialog | not needed (nothing read from or stored on the device for ads) | required in the EEA/UK/CH before the first request |
| GPL-3.0 | compatible (all code is ours) | **not compatible** without licence route L-A or L-B (D5) |
| F-Droid | allowed (flagged "Ads" anti-feature) | not allowed (proprietary dependency) |
| Revenue | depends on sponsors found; predictable flat fees | automatic but small for a niche app with a few thousand users |
| iOS | same code (common Kotlin) | Google Mobile Ads iOS SDK |

**Decided (OP-7, 2026-10-24): A only**, in both stores. It keeps the
privacy promise, the licence and F-Droid intact and needs no consent dialog.
Option B is not pursued: the additional permission (D5) deliberately does
not cover advertising, analytics or tracking libraries, so adopting an ad
network later would need a new ADR and the agreement of all contributors.
The `AdProvider` port stays, so the slot code does not depend on the
catalogue format. The project site offers a *Become a sponsor* page (prices,
formats, content rules) that the apps link to from *About*.

### D5 — Licence route L-A (OP-8 decided)

Two parts of the plan conflict with GPL-3.0-only as set in ADR-0001:

1. **Linking a proprietary ad SDK** (option B) into a GPL program is not
   permitted by the GPL without an additional permission from all copyright
   holders.
2. **Distributing GPL-3.0 software through the Apple App Store** is widely
   regarded as incompatible with GPLv3 because of the store's usage rules
   (FSF position; VLC was removed from the App Store in 2011 for this
   reason). This affects the iOS editions **regardless of the ad source**.

ADR-0001 treats Things Mapper as a derived work of ttnmapper-android-v3, so
an exception granted by the maintainer alone would not cover upstream-derived
parts. Two routes:

| Route | What | Effect |
|---|---|---|
| **L-A — clean implementation + additional permission** *(chosen 2026-10-24)* | Application code under `app/` is written from the design documents (FDS, TDS); no source code from ttnmapper-android-v3 is copied into it; upstream test fixtures stay in a marked test-only directory that is never part of a distributed binary and is replaced by own recordings where possible. Protocol formats, the upload contract and the mapping rules are used as functional specifications. All code is then copyright of the maintainer and contributors, who add a GPLv3 section 7 additional permission (draft below); contributors agree to it in the contributing guide. | Enables B and the App Store; project stays GPL-3.0 for everyone else |
| **L-B — upstream permission** | Ask JP Meijers (together with OP-1) for written permission to distribute derived parts with the additional permission below | Keeps the freedom to reuse upstream code; depends on upstream agreement |

**Decided (OP-8, 2026-10-24): route L-A.** The additional permission is
in force in [license.md](../../license.md#additional-permission-gplv3-section-7),
narrowed to what the decisions need:

- **(a)** store and platform vendor libraries (Google Play services for
  location in the `play` variants; the Google Play Billing Library should a
  store purchase ever be added) — explicitly **not** advertising, analytics,
  tracking or crash-reporting libraries;
- **(b)** distribution through application stores with restrictive terms
  (Apple App Store), with the Corresponding Source freely available.

It applies to the code of which the maintainer and contributors are the
copyright holders; the upstream excerpts in the evaluation and the
test-only fixtures stay under the plain GPL-3.0. Source headers carry
`SPDX-License-Identifier: GPL-3.0-only` plus a pointer to the permission.
This is a design-level assessment, not legal advice; the text should be
reviewed (e.g. against the FSF's guidance on GPL exceptions) before the
first App Store submission.

Selling a GPL app is permitted by the GPL. Anyone may build and distribute an
ad-free version from the public source; the Paid edition's value is
supporting the project, convenience and store updates. This is accepted.

### D6 — Moving from Free to Paid

- **Android:** both apps are signed with the same app-signing key (Play App
  Signing allows reusing the key of another app in the same developer
  account). The Free edition exposes a read-only `ContentProvider` protected
  by a *signature* permission; on first start the Paid edition offers to
  import profiles (without secrets), sessions and settings. Secrets are
  re-entered (keystore keys cannot leave their app).
- **iOS (phase 4):** both apps share an App Group container for the same
  import.
- **Fallback (all):** profile export/import (FDS-1.5) and CSV import
  (FDS-7.3).

### D7 — Distribution matrix

| Build | Ads | Channel |
|---|---|---|
| Android Free, Play variant | sponsor catalogue | Google Play |
| Android Paid, Play variant | no | Google Play (paid) |
| Android Free, FOSS variant | sponsor catalogue (A) only | F-Droid / GitHub Releases — publish or not: OP-9 |
| Android Paid, FOSS variant | no | not distributed (CI only) |
| iOS Free / Paid | yes / no | App Store (phase 4, under additional permission (b)) |
| Linux desktop, Raspberry Pi | no | GitHub Releases (`.deb`) |

### D8 — Publisher obligations

- Google Play: payments profile for the paid app; "contains ads" declaration
  for the Free app; Data Safety form per edition; target audience not
  children.
- Apple: Paid Applications agreement; privacy nutrition label per edition;
  privacy manifests of any SDK; no ATT prompt (no tracking).
- EU Digital Services Act: a monetised app makes the publisher a *trader*;
  both stores require the trader declaration and **display the publisher's
  address, phone number and e-mail publicly** in the EU. The maintainer
  decides which contact details to use (OP-10).
- Stores collect and remit VAT for EU sales; income reporting remains the
  maintainer's responsibility.

## Consequences

- The Free edition's manifest contains no `CarAppService`, so the app does
  not appear on Android Auto at all; Google's Android Auto review applies to
  the Paid app only. The Free edition mentions the car feature in its
  start screen and its "ad-free edition" entry (settings, about) — never
  during mapping.
- Requirements: new section *Editions and monetisation* (REQ-F-150…161),
  REQ-P-009 (iOS editions), REQ-NF-063/064, TGT-09; amended REQ-NF-022,
  REQ-NF-060, REQ-C-001, REQ-C-003, REQ-C-005, REQ-C-009, REQ-P-002, TGT-05;
  open points OP-7…OP-10. ADR-0004 notes that the car experience ships in
  the Paid edition only.
- FDS-17 (editions and advertising), onboarding and settings additions;
  GUI ad-slot specification and wireframe; TDS `ads` module, Android build
  variants `free`/`paid` × `foss`/`play`, `AdPolicy`, edition transfer;
  test cases TC-ED-01…12; work packages WP-2.16 and WP-3.7.
- ADR-0001 is amended (GPL-3.0-only stays; additional permission once OP-8
  is decided) and the development rule "no upstream code in `app/`" applies
  from WP-2.1 onwards so that route L-A stays open.
- OP-7 and OP-8 were decided on 2026-10-24 (sponsor catalogue, route
  L-A); REQ-F-155 (ad consent) and TC-ED-07 are withdrawn; the additional
  permission is in force in `license.md`; the clean-implementation rule
  applies from WP-2.1.

## Links

[02 Requirements §6.16](../02-requirements.md), [03 FDS FDS-17](../03-fds.md),
[04 GUI design §5a](../04-gui-design.md), [05 TDS §9.5](../05-tds.md),
[06 Test plan TC-ED](../06-test-plan.md), [07 Implementation plan WP-2.16, WP-3.7](../07-implementation-plan.md),
[ADR-0001](ADR-0001-license.md), [ADR-0004](ADR-0004-car-platform.md).
