# Aircraft Power Settings

An installable web app (PWA) that calculates cruise power settings — RPM, manifold
pressure (where applicable), fuel flow, and indicated/true airspeed — at a desired
% power, for any altitude and temperature. Covers three aircraft, switchable with a
tab at the top of the app:

- **N7154N** — a 1968 Beechcraft E33 Bonanza (Naperville Flying Club) with a
  Continental IO-470-K (225 BHP @ 2600 RPM), constant-speed prop.
- **N3008U** — a 1977 Piper PA-28-181 Cherokee Archer II with a Lycoming O-360
  (180 BHP), fixed-pitch prop. Since a fixed-pitch prop has no independent
  manifold-pressure control, N3008U's page has a single input (% power) and
  computes RPM, fuel flow, and airspeed — there's no manifold pressure dial.
- **N114EL** — a Piper PA-32-301 Saratoga with a Lycoming IO-540-K (300 BHP @ 2700
  RPM), constant-speed prop. Same layout as N7154N: RPM and % power in, manifold
  pressure out.
- **N114EL T/O** — a second N114EL tab for **takeoff distance over a 50 ft barrier**
  (maximum effort, flaps 25°), from field elevation, OAT, altimeter setting, gross
  weight, wind component, and propeller (3- or 2-blade).

## Install on iPad

1. Turn on GitHub Pages for this repo: **Settings → Pages → Deploy from branch**,
   pick the branch this file is on and `/ (root)`, save. GitHub gives you a URL like
   `https://<user>.github.io/N7154N-engine-settings/`.
2. Open that URL in Safari on the iPad.
3. Tap the **Share** icon → **Add to Home Screen**.

That installs a full-screen app icon that works offline (a service worker caches the
app after the first load) — no App Store submission needed.

**For offline to stick, open the app once with a normal connection after installing
or updating it**, then leave it alone — the service worker precaches everything on
that first load, and from then on launches from the Home Screen icon are served from
that cache first, network or not.

**Launch it from the Home Screen icon, not a Safari tab or bookmark.** This is the
part that actually makes offline support durable long-term on iOS: Apple exempts
standalone web apps (ones added to the Home Screen, which is what `display:
standalone` in `manifest.json` triggers) from Safari's general policy of clearing a
site's cached data after about a week of disuse. That policy only applies to regular
Safari browsing. An icon you only ever launch from the Home Screen should stay
offline-capable indefinitely without needing to "refresh" it.

The app also calls `navigator.storage.persist()` as a best-effort hint not to evict
its data, though the Home Screen exemption above is what actually does the work on
iOS. Nothing in the app's own code can override the few things that *do* still clear
it: manually clearing Safari/website data, deleting and re-adding the Home Screen
icon, or iOS offloading app data device-wide under severe storage pressure (the same
fallback that can affect native apps). If any of those happen, one normal-connection
launch re-seeds the cache from scratch.

## How it works

### N7154N

You dial in **altitude, OAT, altimeter setting, RPM, and % power**; the app calculates
manifold pressure, fuel flow, TAS, and IAS.

The certified anchor points come from the Beechcraft **Cruise Power Settings** tables —
Debonair C33 / Bonanza E33 & F33 Pilot's Operating Handbook, Section V Performance, pages
5-21 through 5-23 (August 1982 revision) — 55% power at 2300 RPM, and 65%/75% power at
2450 RPM, each a grid of pressure altitude (SL–16,000 ft) × ISA deviation (−20°C /
standard / +20°C). RPM is limited to 1800–2450, and manifold pressure is checked against
two limits:

- the recommended-limits boundary in the **Manifold Pressure vs RPM** chart, pg 5-26 (a
  diagonal ceiling from ~21.4 inHg at 1800 RPM up to 24.0 inHg at 2450 RPM);
- **full throttle**, read off the cruise tables themselves. From 6,000 ft up, the cells
  whose fuel flow drops below normal for that power are already at full throttle, and they
  share one MP per altitude whatever the temperature: 23.1 / 21.7 / 20.1 / 18.6 / 17.3 /
  16.2 inHg at 6,000–16,000 ft. Below 6,000 ft that value is scaled by standard pressure.
  Every printed MP is at or under this line.

**Full-throttle (shaded) POH settings.** Above roughly 6,000 ft some printed settings are
already at full throttle and make less power than their table's title. The POH lists them
with a lower fuel flow than the normal 9.8 / 11.5 / 13.4 GPH for 55 / 65 / 75%. Those normal
flows are almost exactly proportional to power (constant BSFC), so each printed fuel flow
converts straight to the power it really makes. For example, the 75% table's 11.1 GPH at
12,000 ft is about 62.6%, and 9.6 GPH at 16,000 ft is about 54%. The 30 shaded entries
range from 47% to 74%.

- **The % power dial always shows actual power**, to 0.1% when it isn't a whole number.
  The 75% preset at 12,000 ft sets 2450 RPM and 62.6%, shows the printed MP, fuel flow, and
  airspeed, and says you're in the full-throttle area of the POH tables. The reference
  table marks those rows the same way.
- **Estimates between settings** interpolate at the power each printed setting really
  makes. With nothing shaded this is identical to before. The 2450 RPM line runs through
  the midpoint of the 65% and 75% settings. Where those two merge into one full-throttle
  setting, the slope between them is rounding noise. Below a 5% gap it blends into the
  table's typical 0.21 inHg per %, reaching it at a 1% gap.
- **Full throttle at other RPMs** is estimated from the shaded 2300 and 2450 RPM entries.
  At a fixed full-throttle MP, power is roughly proportional to RPM; scaling one entry to
  the other's RPM this way lands within 1.5 percentage points. Asking for more power than
  that limits % power to the full-throttle figure, with a message.
- **Where the POH's 65% and 75% entries are the same full-throttle setting** but print
  slightly different MP or airspeed (a few cases, e.g. 135 vs 139 KTAS at 16,000 ft hot),
  the 75% figures are shown and the note gives the 65% ones.

**Turning the RPM dial never changes % power.** If a lower RPM would put the current power
out of limits, the RPM stops at the last setting on the way down that still works. The
cruise power pane shows a message saying the RPM is at a limit and which one. Pushing past
either end of the dial (1800 or 2450 RPM) shows the same kind of message. Raising RPM is
always allowed.

**Changing % power, altitude, temperature, or altimeter setting** can also put the setting
out of limits. The app then fixes it and shows a message saying what it changed. Over the
pg 5-26 ceiling, RPM is raised first. If the setting isn't obtainable at this altitude, %
power is limited at the current RPM to the highest step that is, and the message says if
a higher RPM would allow the requested power.

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
line were each traced directly off their own chart line at fine resolution across a wide
density-altitude span (RPM: ~0–8,200 ft; TAS: ~100–9,700 ft) and independently fit as a
straight line against density altitude — every line came back close to perfectly linear
over the traced range (RPM residuals under 2.5 RPM against the fit; TAS under 0.8 kt,
typically under 0.25 kt), so no line borrows another's rate. Validated against ~4,000
chart-traced points spanning all 8 lines (98%+ within ±5 RPM / 100% within ±1 kt of the
app's output), plus live spot-checks against the running app. The model is calibrated
against the charts' own worked examples, both of which it reproduces exactly:

- 5,500 ft pressure altitude, 40°F, 65% power → 2,440 RPM
- 5,500 ft pressure altitude, 30°F, 55% power → 101 KTAS

Reading values off a hand-drawn nomograph is inherently less precise than transcribing a
printed table, so **every value on the N3008U page is labeled "estimated."** The chart is
built wheel fairings installed and prints "subtract 8 kts if removed" — a **Wheel
Fairings** switch next to the % power dial (defaults to **removed**, N3008U's actual
configuration) applies that flat 8 kt offset to true/indicated airspeed accordingly. Fuel
flow (best power, leaned per Lycoming instructions) comes directly from the POH's
fuel-flow table — 7.8/9.0/10.5 GPH at 55/65/75% power — which isn't altitude/temperature
dependent in the source data. The digitized model covers roughly sea level–10,000 ft
pressure altitude; the real airspeed curves fold back over past their best-power altitude,
so the app doesn't attempt to model beyond that range.

**% power runs 10–75%, matching N7154N's dial** — but both POH fan-line charts stop at
55%. Below that (shaded red on the dial, like N7154N's own sub-40% zone) isn't digitized
chart data at all; it's extrapolated from the certified 55% chart line using basic
propulsion physics, anchored to be continuous at the 55% boundary: RPM follows the
fixed-pitch propeller affinity law (BHP ∝ RPM³ at a roughly constant advance ratio), true
airspeed follows the same power-to-speed cube relationship already used to anchor the
65%/75% airspeed lines to the chart's one solid data point, and fuel flow scales linearly
with % power (roughly constant BSFC). It's a materially rougher estimate than the
digitized 55–75% range above it.

Always cross-check any setting against N3008U's current POH and actual weight before
using it in flight.

### N114EL

You dial in **altitude, OAT, altimeter setting, RPM, and % power**; the app calculates
manifold pressure, fuel flow, TAS, and IAS. Same layout as N7154N.

**Manifold pressure** comes from the PA-32-301 POH's **Power Setting Table — Lycoming
IO-540K Engine** (Section 5, Figure 5-25, Report VB-1060, revised 10/7/1983): MP at
55/65/75% power for 2200–2600 RPM (100 RPM steps), sea level–14,000 ft pressure altitude,
at standard temperature. The app interpolates linearly between rows and columns and applies
the table's own temperature correction (0.15 inHg per 5°C from standard altitude
temperature: add when warmer, subtract when colder). Any 100 RPM column at exactly
55/65/75% reproduces the table value exactly and is labeled **certified**; this was
checked for all 148 printed cells through the live UI. Two misprints in the scanned table
are corrected:

- 75% / 1,000 ft / 2300 RPM prints **20.6**. That can't sit between its 27.6 and 25.9
  neighbors, so the app uses **26.6**, which fits every adjacent cell.
- The row labeled **17000** sits between 11,000 and 13,000 ft and has a −9°C standard
  temperature, so it's the **12,000 ft** row.

**Full throttle.** The table's "—" cells are settings full throttle can't reach. The app
models full-throttle MP as 0.97 × standard ambient pressure plus a small offset per power
level (0.265 / 0.652 / 0.925 inHg at 55/65/75%), fit to that boundary. It puts all 148
printed values at or under the line and all 77 "—" cells over it, with 0.009 inHg of
margin on each side, so no extra tolerance is applied. When a change to % power or the
conditions leaves a setting needing more MP than full throttle provides (which can include
a hot day, per the POH's note), the app raises RPM first. It lowers % power only if 2600
RPM still isn't enough, and the cruise power pane says what it changed. A cold day can
bring a "—" cell back within reach. For that case each altitude row is fit as
MP = A + B/RPM through its printed cells (within 0.18 inHg) and extended. Those values are
marked with an asterisk and labeled estimated.

**Turning the RPM dial never changes % power**, the same as N7154N. If a lower RPM would
need more MP than full throttle gives, the RPM stops at the last setting that still works
and the pane says the RPM is at a limit and why. **Below 2300 RPM**, the dial only goes as
far as the POH power table publishes the setting: 55–75% power, built only from printed
cells (not "—") at this altitude. Otherwise it stops at 2300 RPM with a notice. Pushing past
either end of the dial (2200 or 2600 RPM) also shows a notice. Raising RPM is always
allowed. Tested over ~78,000 RPM steps across altitude, temperature, altimeter, and % power:
% power never changed, no step ended on an unreachable setting, and no step went below
2300 RPM outside the published table. Every refusal showed a notice with the right reason.

**Other RPM / % power settings** are estimated. 50 RPM steps interpolate between columns,
and % power interpolates between the 55/65/75% tables. **Below 55%** (red on the dial,
10–40%), MP continues each RPM's 55–65% line. That matches basic engine physics: at fixed
RPM, brake power is close to linear in MP above a friction-loss intercept, which this table
puts at roughly 7–12 inHg at zero power.

**True airspeed** is digitized from Figure 5-29, **Speed — Cruise Power** (peak EGT,
wheel fairings installed, 3,600 lb). Its 55/65/75% lines are straight in density altitude;
each was traced and fit individually. The fits are within 0.35 kt of the 440 traced points,
and they reproduce the chart's worked example: 6,000 ft, +6°C, 65% → 136 kt. A **Wheel
Fairings** switch (default **installed**, the chart's baseline) applies the chart's
"subtract 8 kts if not installed." Below 55%, TAS follows the cube-law relationship from
the 55% line, like N3008U. Density altitude outside about 0–14,000 ft is clamped to the
chart edge.

**Fuel flow** is the POH's approximate 11.9/13.8/16.0 GPH at 55/65/75% (peak EGT),
interpolated between those points and scaled proportionally below 55%; the three points
give a nearly constant BSFC. The readout also shows the chart's 50°F-rich-of-peak figure:
about 16% more fuel and +2 kt.

Always cross-check any setting against N114EL's current POH and actual weight before
using it in flight.

### N114EL takeoff distance

You dial in **field elevation, OAT, altimeter setting, gross weight, and wind
component** (headwind positive, tailwind negative), and pick the **propeller**. The app
gives takeoff distance over a 50 ft barrier, liftoff and barrier speeds, and density
altitude, plus a table of distances for every 200 lb and 5 kt at the current conditions.

The data comes from the PA-32-301 POH's **Maximum Effort Takeoff Performance — Flaps
25°** chart (Section 5, Figure 5-15, Report VB-1060, revised 8/4/1982): 2700 RPM and full
throttle before brake release, paved, level, dry runway. The chart is a three-step
nomograph, and the app follows the same three steps:

1. **OAT × pressure altitude → distance at 3600 lb, no wind.** Each of the six altitude
   curves (sea level–10,000 ft) was traced off a 600 dpi scan, corrected for the scan's
   slight tilt, and fit as ln(distance) cubic in OAT. The fits are within about 5 ft RMS
   (13 ft max) of the traced line. Altitudes between curves are interpolated linearly.
   The curves are close to, but not exactly, a single function of density altitude (about
   20 ft apart at equal density altitude), so the app keeps them separate, as you would
   reading the chart.
2. **Weight.** The seven straight guide lines from the 3600 lb reference line to 2600 lb
   were traced (about 60% of the starting distance at 2600 lb). The app interpolates
   between neighbouring lines.
3. **Wind.** The same applies to the eight headwind lines (to 15 kt, about 77–84% at
   15 kt) and the seven tailwind lines (to 5 kt, about +17%).

The result is on the chart's 3-blade scale. The 2-blade scale is a straight conversion,
fit to its labeled ticks: 1.1105 × 3-blade + 11 ft. The **Propeller** switch chooses which
one is shown, defaulting to 3-blade, and the other appears underneath. Liftoff and barrier
speeds come from the chart's table, interpolated by weight.

**Checked against the chart's worked example:** 1,200 ft, 16°C, 3,480 lb, 10 kt
headwind.

| Step | Chart's drawn arrow | App |
|---|---|---|
| After temperature/altitude | ~1,780 ft | 1,784 ft |
| After weight | ~1,690 ft | 1,699 ft |
| Final, 3-blade | 1,450 ft | 1,463 ft (shown as 1,460) |
| Final, 2-blade | prints 1,600 ft; arrow meets the scale at ~1,620 | 1,636 ft |
| Liftoff / barrier speed | 65 / 70 KIAS | 65 / 70 KIAS |

Across about 82,000 combinations of altitude, temperature, weight, and wind, distance
always increases with altitude, temperature, weight, and tailwind, and decreases with
headwind.

**No extrapolation.** The chart covers sea level–10,000 ft pressure altitude, −40 to
+40°C, 2600–3600 lb, up to 15 kt headwind or 5 kt tailwind. Its curves stop at about
4,300 ft on the first step, so hot-and-high conditions run off the top. Outside any of
that, the app shows no distance and says why. A pressure altitude below sea level uses
the sea-level curve, which slightly overstates the distance. Every number is labeled
**estimated**: it's a digitized chart, with the same few-percent tolerance as reading it
by hand.

Always cross-check takeoff performance against N114EL's current POH, the actual runway
surface and slope, and the aircraft's actual weight, and add your own safety margin.
