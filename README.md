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

The calculator is a physics-based model, not a transcription of the POH:

- `%BHP ≈ 100 × (RPM / RPM_rated) × (MP / MP_rated) × (T_std / T_actual)` — the standard
  linear RPM×MP approximation for a normally-aspirated engine, with a density/temperature
  correction.
- Full-throttle MP available at altitude is derived from the standard atmosphere pressure
  lapse rate.
- Fuel flow = `%BHP × rated BHP × BSFC ÷ fuel weight`, for both rich (best power) and
  leaned (best economy) mixtures.
- Indicated airspeed scales with the cube root of % power (power ∝ V³ in the cruise
  regime); true airspeed is IAS corrected for density altitude.

All the underlying constants (rated BHP/RPM/MP, BSFC, reference airspeed) are editable
in the **Aircraft profile** panel in the app and saved to the device. The shipped
defaults are reasonable estimates for a stock IO-470-K — **not** transcribed from
N7154N's current POH, since the actual certified charts weren't available while building
this. Open the profile panel and enter your POH's real numbers to calibrate it exactly,
and always cross-check any setting against the current POH before flight.
