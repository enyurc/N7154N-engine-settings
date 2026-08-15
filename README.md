# Aircraft Power Settings

An installable web app (PWA) that calculates cruise power settings — RPM, manifold
pressure (where applicable), fuel flow, and indicated/true airspeed — at a desired
% power, for any altitude and temperature. Covers two aircraft, switchable with a
tab at the top of the app:

- **N7154N** — a 1968 Beechcraft E33 Bonanza (Naperville Flying Club) with a
  Continental IO-470-K (225 BHP @ 2600 RPM), constant-speed prop.
- **N3008U** — a 1977 Piper PA-28-181 Cherokee Archer II with a Lycoming O-360
  (180 BHP), fixed-pitch prop. Since a fixed-pitch prop has no independent
  manifold-pressure control, N3008U's page has a single input (% power) and
  computes RPM, fuel flow, and airspeed — there's no manifold pressure dial.

## Install on iPad

1. Turn on GitHub Pages for this repo: **Settings → Pages → Deploy from branch**,
   pick the branch this file is on and `/ (root)`, save. GitHub gives you a URL like
   `https://<user>.github.io/N7154N-engine-settings/`.
2. Open that URL in Safari on the iPad.
3. Tap the **Share** icon → **Add to Home Screen**.

That installs a full-screen app icon that works offline (a service worker caches the
app after the first load) — no App Store submission needed.

## How it works

### N7154N

You dial in **altitude, OAT, altimeter setting, RPM, and % power**; the app calculates
manifold pressure, fuel flow, TAS, and IAS.

The certified anchor points come from the Beechcraft **Cruise Power Settings** tables —
Debonair C33 / Bonanza E33 & F33 Pilot's Operating Handbook, Section V Performance, pages
5-21 through 5-23 (August 1982 revision) — 55% power at 2300 RPM, and 65%/75% power at
2450 RPM, each a grid of pressure altitude (SL–16,000 ft) × ISA deviation (−20°C /
standard / +20°C). RPM is limited to 1800–2450, and manifold pressure is checked against
the recommended-limits boundary in the **Manifold Pressure vs RPM** chart, pg 5-26 (a
diagonal ceiling from ~21.4 inHg at 1800 RPM up to 24.0 inHg at 2450 RPM) — the app warns
when a requested RPM/% power combination would exceed it.

RPM/% power combinations that exactly match a certified table entry (2300 RPM @ 55%, or
2450 RPM @ 65%/75%) reproduce the tabulated manifold pressure, fuel flow, and TAS exactly.
Anything else is **estimated**: a manifold-pressure-vs-power line is fit through the
certified points at each RPM (using the real 65%/75% gap at 2450 RPM for the slope, scaled
proportionally for other RPMs) and solved for the requested % power. The app labels every
result "certified" or "estimated" accordingly.

Indicated airspeed is **not** tabulated in the POH chart at all — it's derived from TAS
using the standard density-altitude correction (KIAS ≈ KTAS × √σ), which ignores position
error and is only an estimate.

The tables were transcribed from scanned copies of the POH. Spot-check the numbers against
your own copy, and always cross-check any setting against N7154N's current POH and actual
weight before using it in flight.

### N3008U

You dial in **altitude, OAT, altimeter setting, and % power**; the app calculates the
resulting RPM, fuel flow, TAS, and IAS.

Because N3008U has a fixed-pitch prop, there's no manifold-pressure dial and RPM is an
*output*, not an input — you set power with the throttle and the RPM is whatever results
at that altitude and temperature.

The data comes from two nomograph-style charts in the PA-28-181 POH (Section 5
Performance, Report VB-790): **Figure 5-18 "Engine Performance"** (RPM vs. density
altitude, with fan lines at 55/60/65/70/75% power) and **Figure 5-20 "Speed Power —
Performance Cruise"** (true airspeed vs. density altitude, fan lines at 55/65/75%).
Both charts plot against a shared, unlabeled vertical axis that works out to be density
altitude — each pressure-altitude line is drawn to cross the chart's "STD TEMP" reference
exactly where OAT equals that altitude's standard temperature.

Density altitude itself is computed **exactly** from pressure altitude and OAT (standard
ISA: 15°C sea level, 1.98°C/1000 ft lapse), not read off a chart. RPM and TAS per %power
line were fit as functions of density altitude by digitizing the charts and calibrating
against their own worked examples, both of which the app's model reproduces exactly:

- 5,500 ft pressure altitude, 40°F, 65% power → 2,440 RPM
- 5,500 ft pressure altitude, 30°F, 55% power → 101 KTAS

Reading values off a hand-drawn nomograph is inherently less precise than transcribing a
printed table, so **every value on the N3008U page is labeled "estimated."** The chart is
built wheel fairings installed and prints "subtract 8 kts if removed" — a **Wheel
Fairings** switch next to the % power dial lets you match N3008U's actual configuration,
and the true/indicated airspeed figures apply that flat 8 kt offset accordingly. Fuel flow
(best power, leaned per Lycoming instructions) comes directly from the
POH's fuel-flow table — 7.8/9.0/10.5 GPH at 55/65/75% power — which isn't
altitude/temperature dependent in the source data. The model covers roughly sea
level–10,000 ft pressure altitude; the real airspeed curves fold back over past their
best-power altitude, so the app doesn't attempt to model beyond that range.

Always cross-check any setting against N3008U's current POH and actual weight before
using it in flight.
