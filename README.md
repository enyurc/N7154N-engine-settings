# N7154N Power Settings

An installable web app (PWA) that calculates cruise RPM, manifold pressure, fuel flow,
and indicated/true airspeed at a desired % power, for any altitude and temperature —
built for N7154N, a 1968 Beechcraft E33 Bonanza (Naperville Flying Club) with a
Continental IO-470-K (225 BHP @ 2600 RPM).

## Install on iPad

1. Turn on GitHub Pages for this repo: **Settings → Pages → Deploy from branch**,
   pick the branch this file is on and `/ (root)`, save. GitHub gives you a URL like
   `https://<user>.github.io/N7154N-engine-settings/`.
2. Open that URL in Safari on the iPad.
3. Tap the **Share** icon → **Add to Home Screen**.

That installs a full-screen app icon that works offline (a service worker caches the
app after the first load) — no App Store submission needed.

## How it works

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
