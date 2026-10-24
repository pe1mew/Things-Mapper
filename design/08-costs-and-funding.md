# 08 — Costs and funding

Status: **Draft**

| Rev | Date | Author | Change |
|---|---|---|---|
| 0.1 | 2026-10-24 | R. Welling (with Claude) | First version: investments, recurring costs, income per sale, break-even, funding data |

Related: [ADR-0007 editions](adr/ADR-0007-editions-and-monetisation.md),
[ADR-0008 cost awareness and support](adr/ADR-0008-cost-awareness-and-support.md),
[02 Requirements §6.16–6.17](02-requirements.md),
[07 Implementation plan §9](07-implementation-plan.md).

---

## 1. Principles

1. **Hobby project.** Things Mapper is not a business and does not earn
   anyone a living. Income only covers the costs below; no paid support,
   consultancy or custom builds are offered (ADR-0008 D6).
2. **Money only.** The maintainer's time is volunteered and not counted.
   It is mentioned so that users see that the money figures are the small
   part.
3. **Transparent.** The figures in this document are published as signed
   *funding data* (§6) and shown in the apps' cost page (FDS-18.2) and in the
   README. Estimates are replaced by actual amounts as purchases are made.
4. **Surplus.** Income above the year's costs is kept as a reserve for the
   next hardware renewal and is shown in the funding data (assumption; to be
   confirmed by the maintainer).

All amounts: euro, prices in the Netherlands including VAT, indicative as of
October 2026, rounded. USD fees converted at roughly 1 USD ≈ €0.92.

## 2. One-time investments

| ID | Item | Purpose | Estimate | Needed at |
|---|---|---|---|---|
| I-1 | Google Play developer registration | publish both Android editions | ≈ €23 (25 USD) | M2.3 |
| I-2 | Android test phone, current mid-range (Android 16) | main development and field-test device | €300–450 | M2.3 |
| I-3 | Older Android phone (Android 8–10), second-hand | minimum-version coverage (REQ-P-001) | €50–100 | M2.3 |
| I-4 | LoRaWAN mapping node | test transmitter for all field tests | €40–100 | M2.1 |
| I-5 | Aftermarket head unit with wired Android Auto | Android Auto on a real head unit (TC-F-04) and development without driving | €150–350 | M3.2 |
| I-6 | 12 V bench power supply (≥ 10 A), wiring harness, USB cables | run the head unit on the desk | €50–100 | M3.2 |
| I-7 | Raspberry Pi 5 (4 GB) with power supply, case and microSD | kiosk reference platform (REQ-P-004) | €100–130 | M3.1 |
| I-8 | Raspberry Pi Touch Display 2 (7") | kiosk display | €60–75 | M3.1 |
| I-9 | Raspberry Pi 4 (2 GB) | minimum-spec performance tests | €45–60 | M3.1 |
| I-10 | USB GNSS receiver (u-blox class) | gpsd/NMEA position source | €30–60 | M3.1 |
| I-11 | Apple computer — Mac mini, Apple silicon, 16 GB RAM | Xcode, iOS builds, signing and debugging; there is no supported way to develop and publish iOS apps without a Mac | €500–1,000 (base model launched at €719; refurbished or second-hand cheaper) | phase 4 |
| I-12 | iPhone that runs the current iOS | iOS test device | €300–700 (refurbished … new entry model) | phase 4 |
| I-13 | Apple Developer Program, first year | signing and App Store publication | ≈ €99 (99 USD; afterwards R-1) | phase 4 |

| Subtotal | Range |
|---|---|
| Android, Android Auto and Raspberry Pi (I-1…I-10) | ≈ €850–1,450 |
| Starting on iOS (I-11…I-13) | ≈ €900–1,800 |
| **Total** | **≈ €1,750–3,250** |

Items the maintainer already owns are entered with amount 0 in the funding
data. A test phone that doubles as a personal phone counts only partly.

## 3. Recurring costs per year

| ID | Item | Per year | From |
|---|---|---|---|
| R-1 | Apple Developer Program | ≈ €99 | phase 4 |
| R-2 | Google Play | €0 (registration was one-time) | — |
| R-3 | Android phone renewal (mid-range, every ~4 years; Play requires a new target API every year and old phones stop getting updates) | €75–115 | M2.3 |
| R-4 | iPhone renewal (every ~5 years) | €60–140 | phase 4 |
| R-5 | Mac renewal (every ~6 years; new Xcode versions drop old macOS) | €85–170 | phase 4 |
| R-6 | Wear and replacement of the Pi, head unit, node and cables | €20–40 | M3.1 |
| R-7 | Hosting (GitHub Pages) and CI (GitHub Actions, including macOS runners, free for public repositories) | €0 | — |
| R-8 | Own domain name | €0 (not planned; €10–15 if ever) | — |

| Phase | Recurring per year |
|---|---|
| Android only (R-3, R-6) | ≈ €95–155 |
| Android and iOS (R-1, R-3…R-6) | ≈ €340–565 |

**Time (not charged):** the platforms force work every year even without
new features — a new Android target API by each August deadline, new Xcode
and iOS SDK versions, store policy and privacy forms. Estimate: one to two
weeks of spare time per platform per year.

## 4. Income per sale

Store prices include VAT, which the stores remit per buyer country; the
store fee is 15 % on both stores for a developer of this size (Google
Play's reduced rate, Apple's Small Business Program).

| Product | Price | Excl. 21 % VAT (NL buyer) | After 15 % store fee |
|---|---|---|---|
| Paid edition (decided, ADR-0008 D5) | €6.00 once | €4.96 | **≈ €4.21** |
| Yearly supporter (decided) | €5.00 / year | €4.13 | ≈ €3.51 per year |
| Tip small / medium / large (decided) | €2 / €5 / €10 | €1.65 / €4.13 / €8.26 | ≈ €1.40 / €3.51 / €7.02 |
| Sponsor banner (paid directly, outside the stores) | flat fee per period (on the *Become a sponsor* page) | — | 100 % |

For other EU countries the net per Paid sale is roughly €4.00–4.40.

A Paid buyer pays €6 once. Used for five years that is €1.20 a year, while
every one of those years needs the platform work of §3. This is the
imbalance the awareness measures of ADR-0008 address.

## 5. Break-even

| Cost block | Amount | Paid sales needed (≈ €4.21 each) |
|---|---|---|
| Recurring, Android only | €95–155 / year | 23–37 per year |
| Recurring, Android and iOS | €340–565 / year | 81–134 per year (both stores together) |
| One-time, Android + Android Auto + Pi | €850–1,450 | 202–345 |
| One-time, starting on iOS | €900–1,800 | 214–428 |

Each yearly supporter replaces roughly 0.8 Paid sales per year; sponsor
fees go in full towards the costs.

### Recommendation for the Apple investment (OP-11)

The core already compiles for iOS in CI on GitHub's free macOS runners
(REQ-P-005), so no Apple hardware is needed before phase 4. Buy the Mac and
iPhone (I-11, I-12) only when:

1. the Android editions have covered their own recurring costs for twelve
   months (TGT-09), and
2. there is visible demand for iOS (for example repeated requests in the
   issue tracker or from the TTN community).

Refurbished hardware covers the need; a cloud Mac rental is an alternative
for an occasional build but impractical for day-to-day debugging.

## 6. Funding data (published figures)

The maintainer keeps the figures in a signed JSON file next to the sponsor
catalogue. It is the single source for the in-app cost page (FDS-18.2), the
yearly thank-you card (FDS-18.4), the start-screen price line (FDS-17.5) and
the README summary.

```
https://pe1mew.github.io/Things-Mapper/funding/v1/funding.json
https://pe1mew.github.io/Things-Mapper/funding/v1/funding.json.sig   (Ed25519, same key as the sponsor catalogue)
```

```json
{
  "thingsmapper_funding": 1,
  "updated": "2027-01-15",
  "currency": "EUR",
  "year": 2027,
  "paid_price": { "EUR": "6,00" },
  "recurring": [
    { "id": "R-1", "label": { "en": "Apple Developer Program", "nl": "Apple Developer Program" }, "amount": 99 },
    { "id": "R-3", "label": { "en": "Android test phone renewal (share)", "nl": "Vervanging Android-testtelefoon (deel)" }, "amount": 95 }
  ],
  "investments": [
    { "id": "I-11", "label": { "en": "Mac mini for iOS development", "nl": "Mac mini voor iOS-ontwikkeling" }, "amount": 719, "spent": "2027-03-01" }
  ],
  "income": { "paid_sales": 112.4, "support": 31.6, "sponsors": 50.0 },
  "costs_this_year": 352.0,
  "covered_percent": 55,
  "investments_open": 610.0,
  "time_note": { "en": "Development time is volunteered and not counted.", "nl": "Ontwikkeltijd is vrijwillig en telt niet mee." },
  "details_url": "https://github.com/pe1mew/Things-Mapper/blob/main/design/08-costs-and-funding.md"
}
```

Definitions: `costs_this_year` = recurring costs of the year plus the
year's share of investments (written off over the renewal periods of §3);
`covered_percent` = income of the year ÷ `costs_this_year`;
`investments_open` = investments not yet recovered. A small script in
`tools/funding/` computes these from a hand-maintained ledger (store
reports, receipts) and signs the file; it is updated at least quarterly.
The apps fetch it at most once per 24 hours, without identifiers, and show
the `updated` date next to the figures.

## 7. Practical notes

- **Payments:** Paid-edition sales and supporter purchases go through the
  stores; sponsors pay directly (invoice or GitHub Sponsors). Store builds
  contain no links to outside payment (REQ-C-011); the `foss`, desktop and
  Raspberry Pi builds link to GitHub Sponsors.
- **Taxes (Netherlands):** the stores collect and remit VAT on sales. Hobby
  income may still be taxable as *resultaat uit overige werkzaamheden*, with
  related costs deductible; check with the Belastingdienst. This is not
  tax advice.
- **EU Digital Services Act:** selling apps makes the publisher a trader;
  both stores publish the trader's contact details (OP-10).
