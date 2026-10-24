# Things Mapper

**A multi-platform LoRaWAN coverage mapper — a remake of TTN Mapper.**

Things Mapper pairs the reception metadata of a LoRaWAN mapping device
(RSSI, SNR, receiving gateways) with the position of the surveyor and turns
it into a live coverage map. Measurements are stored locally and — with the
user's consent — contributed to the community coverage map at
[ttnmapper.org](https://ttnmapper.org).

It is a ground-up remake of JP Meijers' **TTN Mapper** Android app
([ttnmapper-android-v3](https://github.com/ttnmapper/ttnmapper-android-v3),
GPL-3.0), re-designed for today's network servers (The Things Stack v3,
ChirpStack v4), today's platform rules, and more than one kind of device:

| Target | Form factor | Status |
|---|---|---|
| Android phone / tablet | touch, on the move | designed |
| Android Auto | car head unit (projection from the phone) — Paid edition | designed |
| Linux desktop (x86-64) | mouse/keyboard, development and desk use | designed |
| Raspberry Pi 4/5 + touch display | dedicated in-vehicle mapping box | designed |
| iOS | out of scope for now — the layered architecture keeps the door open | — |

> **Project status: Phase 1 — design.** There is no application code yet.
> Phase 1 delivers the evaluation of the original app, the requirements,
> and the functional, GUI and technical designs plus the test and
> implementation plans. See [design/README.md](design/README.md).
>
> The project started under the name *Chirp Mapper* and was renamed to

## Editions

| Edition | Where | Price | Advertisements | Android Auto |
|---|---|---|---|---|
| Things Mapper — Free | Google Play (iOS later) | free | sponsor banners on secondary screens and on the idle map, never while mapping; a start screen offering the Paid edition | — |
| Things Mapper — Paid | Google Play (iOS later) | €6 once | none, no start screen | ✔ |
| Desktop / Raspberry Pi | GitHub Releases (`.deb`) | free | none | — |

Apart from advertisements, the start screen and Android Auto the editions
are identical. Sponsor banners come from the project's own sponsor list and
never receive your location, measurements or device data. Background:
[ADR-0007](design/adr/ADR-0007-editions-and-monetisation.md).

## Costs and support

Things Mapper is a hobby project: the income only covers its money costs,
and the development time is volunteered. Indicative figures (details in
[design/08-costs-and-funding.md](design/08-costs-and-funding.md)):

| | One-time | Per year |
|---|---|---|
| Android, Android Auto test bench, Raspberry Pi kit | ≈ €850–1,450 | ≈ €95–155 |
| Starting on iOS (Mac, iPhone, Apple Developer Program) | ≈ €900–1,800 | ≈ €245–410 extra |

A Paid sale brings in about €4.20 after VAT and store fees — once, for the
lifetime of the app — while Google and Apple demand updates every year.
*Voor niets gaat de zon op*: if you use Things Mapper year after year,
consider the yearly supporter option in the app, or
[GitHub Sponsors](https://github.com/sponsors/pe1mew) for the desktop and
Raspberry Pi builds. The app shows the current figures on its *What Things
Mapper costs* page. No paid support or custom work is offered
([ADR-0008](design/adr/ADR-0008-cost-awareness-and-support.md)).

## Why a remake?

The original app still works, but it was written for a world that has moved
on: TTN v2 (shut down in 2021) is still its default network, ChirpStack is
supported in its v3 JSON format only, MQTT runs unencrypted, and the whole
application lives in a couple of Android singletons that make it impossible
to run anywhere but on a phone. The evaluation in
[design/01-evaluation.md](design/01-evaluation.md) lists what is kept, what
is changed and what is dropped.

Design goals of the remake:

- **One core, many shells.** All mapping logic (protocol adapters, positioning
  rules, storage, upload queue, map model) lives in a platform-independent
  Kotlin Multiplatform core; phone, car, desktop and Raspberry Pi are thin
  shells on top of it.
- **Current network servers.** The Things Stack v3 and ChirpStack v4 out of
  the box, TLS by default, extension point for others.
- **Nothing lost.** Every received packet is stored locally first; uploads are
  queued and retried, exports are complete and correct.
- **Glanceable.** A map and a handful of big numbers that can be read from the
  driver's seat, on a phone, a 7-inch touch screen or a car display.
- **Respectful.** Explicit consent for uploads, secrets in the platform key
  store, no secrets in logs or bug reports; advertising only in the Free
  edition, never while mapping and never with your location or data.

## Repository layout

| Path | Contents |
|---|---|
| `design/` | All design documents of phase 1 (evaluation, requirements, FDS, GUI design, TDS, test plan, implementation plan, ADRs) |
| `design/gui/` | Wireframes (SVG) referenced by the GUI design |
| `app/` | Application source — lands in phase 2 (see the implementation plan) |
| `documentation/` | End-user documentation — written alongside the implementation |
| `ttnmapper-android-v3/` | Git submodule: the original TTN Mapper Android app, the study reference and the origin of the GPL-3.0 licence of this repository |

The submodule is a reference only; it is not needed to read the design
documents. Initialise it with `git submodule update --init` when you want to
compare against the original code.

## Design documents

Reading order:

1. [Evaluation of TTN Mapper Android v3](design/01-evaluation.md)
2. [Requirements and targets](design/02-requirements.md)
3. [Functional design specification (FDS)](design/03-fds.md)
4. [GUI design](design/04-gui-design.md)
5. [Technical design specification (TDS)](design/05-tds.md)
6. [Test plan](design/06-test-plan.md)
7. [Implementation plan](design/07-implementation-plan.md)
8. [Costs and funding](design/08-costs-and-funding.md)
9. [Architecture decision records](design/adr/)

## Roadmap

| Phase | Content |
|---|---|
| 1 | Evaluation, requirements, FDS, GUI design, TDS, test plan, implementation plan *(this repository state)* |
| 2 | Core library + replay tool, desktop application, Android application (feature parity with TTN Mapper for TTS v3 / ChirpStack v4) |
| 3 | Raspberry Pi kiosk build with GNSS, Android Auto, hardening and first public release |
| later | iOS shell (Free and Paid editions), additional network servers, multi-device sessions |

## Contributing

See [contributing.md](contributing.md) and the
[code of conduct](code_of_conduct.md). Notable changes are tracked in
[changelog.md](changelog.md). During phase 1, contributions are review
comments and pull requests on the design documents; requirement IDs are
stable and referenced from the FDS, TDS and test plan, so please follow the
change process described in the contributing guide.

## Acknowledgements

- **JP Meijers** and the TTN Mapper contributors for the original app, the
  ttnmapper.org platform and its public ingress API and coverage tiles, on
  which this remake builds.
- **The Things Network / The Things Industries** for The Things Stack and its
  MQTT integration.
- **Orne Brocaar** for ChirpStack.
- **OpenStreetMap contributors** for the base map data (ODbL).

Things Mapper is an independent project. It is not affiliated with or endorsed
by TTN Mapper, The Things Network, The Things Industries, ChirpStack, Google
or Raspberry Pi Ltd.

## About this project: AI-assisted development

This project is made with the help of **Claude** (Anthropic), under my
supervision. The goal is to practice and experiment with programming,
leveraging AI tools such as Claude, and to learn to work with these tools.
While AI assistance accelerates the work, I cannot guarantee the originality
or accuracy of all content, as the sources used by large language models are
not always transparent or verifiable. The results and information presented
here have not been exhaustively validated. **Do not rely on this project or
its output for critical applications without independent verification.** The
disclaimer below applies in full.

## Disclaimer

This program is distributed in the hope that it will be useful, but WITHOUT
ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS
FOR A PARTICULAR PURPOSE. See the GNU General Public License for more details.

## License

Things Mapper is free software, licensed under the **GNU General Public
License v3.0** (`GPL-3.0-only`) — see [LICENSE](LICENSE). The licence was
adopted from the original TTN Mapper Android app. The application code is a
clean implementation and carries an additional permission for app-store
distribution and store/platform vendor libraries (never advertising or
analytics libraries). Third-party components, data sources and trademarks are
listed in [license.md](license.md).
