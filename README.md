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

RPM, manifold pressure, fuel flow, and TAS are interpolated directly from the certified
Beechcraft **Cruise Power Settings** tables — Debonair C33 / Bonanza E33 & F33 Pilot's
Operating Handbook, Section V Performance, pages 5-21 through 5-23 (August 1982 revision),
at 55%, 65%, and 75% maximum continuous power, 2950 lb average cruise weight. Each table
is a grid of pressure altitude (SL–16,000 ft) × ISA deviation (−20°C / standard / +20°C);
the app bilinearly interpolates within that grid for whatever field elevation, altimeter
setting, and OAT you enter, and clamps (with a warning) outside that range.

Indicated airspeed is **not** tabulated in the POH chart — it's derived from the tabulated
TAS using the standard density-altitude correction (KIAS ≈ KTAS × √σ), which ignores
position error and is only an estimate.

The tables were transcribed from a scanned copy of the POH. Spot-check the numbers against
your own copy, and always cross-check any setting against N7154N's current POH and actual
weight before using it in flight.
