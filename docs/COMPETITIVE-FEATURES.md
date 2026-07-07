# Competitive landscape & features worth adopting

> **Please read this before planning new features.** It maps what the *other*
> Luxtronik Home Assistant integrations already do, links each one, and lists —
> honestly — the features they have that **this** integration does not yet have.
> The point is not to copy everything; it is to make a deliberate choice about
> what to adopt and what to deliberately leave to them. **Go look at these repos
> directly before deciding.**

_Last researched: 2026-07-07 (against the state of the linked repos on that date)._

---

## The other integrations

### 1. BenPru/luxtronik — the reference implementation ⭐ look here first
- Repo: https://github.com/BenPru/luxtronik
- ~161 stars, **actively community-maintained** (handed over by BenPru, current
  dev `rhammen`), last push 2026-07-05. Built on top of Bouni (below) and adds
  predefined entities + a full UI setup. Bouni and BenPru can run side by side.
- This is the integration most HA users are pointed to, and the one you are
  effectively compared against. **If you look at only one repo, look at this one.**

**What it does (and this integration does *not* yet):**

| Area | BenPru | This integration (luxtronik2-hass) |
|------|--------|-------------------------------------|
| **Setup** | Auto-discovery (HA finds the pump) + manual fallback | Manual IP entry only (no `zeroconf`/`dhcp` in `manifest.json`) |
| **Heating** | Native **`climate`** entity (`climate.heating`) — thermostat card, HVAC modes, target temp, "target temperature correction" | `select` (mode) + `number` (setpoint) — no native climate entity |
| **Hot water** | Native **`water_heater`** entity with Automatic / Party / Holiday / Off modes, target temp, **hysteresis**, **thermal disinfection** (target + scheduled day) | `select` + `number`; no `water_heater` entity, no thermal-disinfection scheduling, no hysteresis control |
| **Cooling** | Full **Cooling** device: `climate` + Approval binary sensor, Minimal Outdoor Temp, Start/Stop delay | **None** — no cooling device/control at all |
| **Entity structure** | 4 logical devices (Heatpump / Heating / Cooling / DHW) | Flat entity set (~31 entities) |
| **Entity types** | Uses `climate`, `water_heater`, `binary_sensor`, `number`, `select`, `switch`, `sensor` | `button`, `number`, `select`, `switch`, `sensor` (no `climate`, `water_heater`, `binary_sensor`) |
| **PV / solar boost** | Shown as **DIY automation examples** in their docs (e.g. `water_heater.set_temperature` on high solar power) — not a packaged feature | Shipped as a **packaged Solar Boost feature** (see "our niche" below) |

### 2. Bouni/luxtronik — the YAML groundwork
- Repo: https://github.com/Bouni/luxtronik
- ~98 stars, last meaningful update 2023. This is the low-level base BenPru
  builds on. YAML configuration; you define sensors by parameter ID yourself.
  Can **write** parameters via the `luxtronik.write` service (gated by a `safe`
  flag). No config flow, no predefined entities.
- Also maintains the shared Python backbone: **`Bouni/python-luxtronik`**
  (https://github.com/Bouni/python-luxtronik) — the library that talks the
  binary protocol. Both BenPru and this project ultimately rely on this data
  layer. Worth tracking for protocol/firmware updates.

### 3. doppel0/Luxtronik-2.0 — historical, ignore
- Repo: https://github.com/doppel0/Luxtronik-2.0
- A 2018 standalone Python reader script, 5 stars, not an HA integration. Listed
  only so nobody re-discovers it as "prior art" — no features to adopt.

---

## Our niche — what to protect and keep emphasizing

These are the things **BenPru does not ship as ready-made features** (their docs
show them as hand-built automations, if at all). They are the honest reason for
this integration to exist — do **not** regress them while chasing parity:

- **Bath Boost (Badebooster)** — one button → heat DHW to a higher target with a
  live **progress % + ETA** sensor, then **auto-revert** to Automatic. BenPru has
  a "Party" DHW mode, but the boost-with-progress-and-auto-revert helper is ours.
- **Solar Boost** — packaged automation that raises the DHW setpoint on PV
  surplus (bring-your-own grid/export sensor). BenPru leaves this to the user.
- **Night Heating Pause** — packaged, time-based floor-heating pause.
- **Smaller, opinionated surface** — ~31 entities, config-flow, EN+DE built in.
  Some users specifically want "just these features," not the full control panel.

---

## Suggested priorities (highest value ÷ risk first)

Nothing here is committed work — it's a shortlist to evaluate against BenPru:

1. **Native `climate` entity for heating** and **`water_heater` entity for DHW.**
   Biggest UX win: unlocks the standard HA thermostat / water-heater cards, HVAC
   modes, voice assistants, and generic thermostat automations. Today's raw
   `number`+`select` controls work but feel non-native. Highest priority.
2. **Auto-discovery** (`zeroconf`/`dhcp` in `manifest.json`) so users don't have
   to find and type the controller IP. Low risk, clear win, matches BenPru.
3. **Thermal disinfection** scheduling and **hysteresis** control for DHW — small,
   self-contained additions that close obvious gaps.
4. **Cooling support** — only if there's demand; it's a whole device area BenPru
   already does well, so weigh build cost vs. just recommending BenPru for cooling.

**Guiding rule for outreach & docs:** always credit BenPru/Bouni as the fuller,
excellent options and never claim superiority — this integration's honest pitch is
the *packaged comfort/energy features* and a smaller surface, not "better than."
