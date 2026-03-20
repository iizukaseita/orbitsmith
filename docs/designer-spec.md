# OrbitSmith Mission Designer — Implementation Specification

**Version:** 1.0
**Date:** 2026-03-19
**Author:** Seita Iizuka + Claude
**Status:** Ready for implementation
**File:** `designer.html` (single-file, consistent with existing OrbitSmith pages)

---

## 1. Overview

### 1.1 Purpose
Mission Designer is a new page for OrbitSmith that allows users to define a satellite mission, configure the spacecraft, run feasibility analysis, find a suitable launch vehicle, and review a complete design summary — all interactively in the browser.

### 1.2 Positioning
- **Portfolio piece**: Demonstrates Seita's engineering depth beyond data visualization
- **Education tool**: Accessible to students and space enthusiasts, not just engineers
- **Free / open-source**: No backend required; all calculations run client-side in JS
- **Future AI-ready**: Template + rule engine structure can later accept AI-driven mission input

### 1.3 Scope
- Satellite class: SmallSat up to ~500 kg (CubeSat presets included)
- Phase A level analysis (initial feasibility, not detailed design)
- 7 mission type templates with adjustable parameters
- No user accounts, no data persistence (future: localStorage or export)

---

## 2. Site Integration

### 2.1 Navigation
- Add `DESIGNER` link to nav bar on all pages (index, tracker, imagery, about, contact)
- Add Designer tool card to `index.html` tools grid
- Nav link class: `nav-link`, active state when on designer.html

### 2.2 Design System (inherit from existing)
- Background: `#000`
- Fonts: `Orbitron` (headings, values), `IBM Plex Mono` + `Noto Sans JP` (body)
- Accent: `#00e5ff` (cyan)
- Cards: `rgba(4,8,16,0.7)` bg, `rgba(255,255,255,0.04)` border, 12px radius
- Status colors: Go `#00e676`, Warn `#ffc107`, No-Go `#ff5252`

### 2.3 i18n
- Same pattern as index.html: `data-i18n` attributes + language JSON
- Initial release: English only
- Structure ready for JA/ZH/ES expansion

### 2.4 Local Development Workflow
**IMPORTANT: Do NOT git push immediately. Local review first.**
```
1. Claude Code creates/edits ~/Desktop/Coding/Orbitsmith/designer.html
2. User opens file in browser: file:///Users/.../Orbitsmith/designer.html
3. User reviews, requests changes
4. Iterate until approved
5. THEN: git add, git commit, git push → Cloudflare Pages auto-deploy
```

---

## 3. Page Structure — 5-Step Wizard

### 3.1 Step Navigation
- Fixed step indicator bar: 5 numbered dots with labels
- States: active (cyan), done (green), future (dim)
- Click any dot to navigate (even backward)
- Smooth scroll to top on step change
- URL hash for step state: `#step1` through `#step5`

### 3.2 Step Flow

```
Step 1: Mission Type Selection
    ↓ (auto-populate defaults)
Step 2: Satellite Configuration
    ↓ (real-time calculation)
Step 3: Budget Analysis Dashboard
    ↓ (mass + orbit params)
Step 4: Launch Vehicle Recommendation
    ↓ (all data)
Step 5: Summary (Terminal)
```

---

## 4. Step 1 — Mission Type Selection

### 4.1 UI
- 7 mission cards in responsive grid (auto-fit, min 240px)
- Each card: icon (simple SVG), mission name, 1-line description, meta tags (mass range, orbit type)
- Click to select (cyan border highlight)
- "CONFIGURE SATELLITE →" button to proceed

### 4.2 Mission Templates

| # | Type | Mass Range | Default Mass | Default Orbit | Default Alt | Default Inc |
|---|------|-----------|-------------|--------------|------------|------------|
| 0 | Optical EO | 50–500 kg | 120 kg | SSO | 550 km | 97.4° |
| 1 | SAR | 100–500 kg | 350 kg | SSO | 620 km | 97.8° |
| 2 | IoT / M2M | 5–50 kg | 12 kg | LEO | 520 km | 52° |
| 3 | AIS / ADS-B | 5–30 kg | 8 kg | LEO | 580 km | 97.6° |
| 4 | Tech Demo | 1–100 kg | 50 kg | LEO | 400 km | 51.6° |
| 5 | Science | 10–300 kg | 180 kg | HEO/LEO | 600 km | 65° |
| 6 | Comms Relay | 50–500 kg | 250 kg | MEO/LEO | 1200 km | 55° |

Full default parameter tables for each template (solar panel area, battery, TX power, payload mass, duty cycle, data rate, etc.) to be defined during implementation based on SMAD reference values.

---

## 5. Step 2 — Satellite Configuration

### 5.1 UI
- 4 config panels in 2×2 grid: Orbit, Bus, Communications, Payload
- Each parameter: label + slider + numeric value + unit
- Recommended range hint text below relevant sliders (from mission template)
- Dropdown selects for: orbit type, downlink band, ground station

### 5.2 Parameters

#### Orbit Panel
| Parameter | Range | Unit | Default (Optical EO) | Notes |
|-----------|-------|------|---------------------|-------|
| Altitude | 200–2000 | km | 550 | |
| Inclination | 0–180 | deg | 97.4 | Auto-set for SSO |
| Eccentricity | 0–0.1 | — | 0.001 | |
| Orbit type | dropdown | — | SSO | SSO/LEO/MEO/HEO |

#### Bus Panel
| Parameter | Range | Unit | Default | Notes |
|-----------|-------|------|---------|-------|
| Total mass | 1–500 | kg | 120 | |
| Solar panel area | 0.1–20 | m² | 4.0 | |
| Battery capacity | 10–2000 | Wh | 240 | |
| Design life | 1–15 | yr | 5 | |

#### Communications Panel
| Parameter | Range | Unit | Default | Notes |
|-----------|-------|------|---------|-------|
| Downlink band | dropdown | — | X-band | UHF/S/X/Ka |
| TX power | 1–50 | W | 10 | |
| Antenna gain | 0–40 | dBi | 18 | |
| Ground station | dropdown | — | Svalbard | See §5.3 |

#### Payload Panel
| Parameter | Range | Unit | Default | Notes |
|-----------|-------|------|---------|-------|
| Payload mass | 1–200 | kg | 35 | |
| Peak power | 1–200 | W | 60 | |
| Duty cycle | 1–100 | % | 30 | |
| Data rate | 1–1000 | Mbps | 150 | |

### 5.3 Ground Station Presets
| Name | Latitude | Longitude | Antenna | G/T |
|------|----------|-----------|---------|-----|
| Svalbard (SvalSat) | 78.2°N | 15.4°E | 10m | 28 dB/K |
| Troll (KSAT) | 72.0°S | 2.5°E | 7.5m | 25 dB/K |
| Fairbanks | 64.9°N | 147.5°W | 11m | 30 dB/K |
| Hawaii (KSAT) | 19.5°N | 155.5°W | 7.5m | 25 dB/K |
| Tokyo (Univ.) | 35.7°N | 139.7°E | 3m | 15 dB/K |

---

## 6. Step 3 — Budget Analysis Dashboard

### 6.1 UI Layout
- 2-column grid of analysis cards (5 cards, last row may have 1)
- Each card: header (title + Go/Warn/No-Go badge), gauge ring (margin%), key stats, detail visualization
- All values update in real-time when Step 2 parameters change

### 6.2 Module: Power Budget

#### Calculation Model
```
Orbital mechanics:
  a = Re + h                          (orbit radius, Re = 6371 km)
  T = 2π √(a³/μ)                      (orbital period, μ = 3.986e14 m³/s²)
  ρ = arcsin(Re/a)                     (Earth angular radius)
  t_eclipse = T × ρ/π                  (worst-case eclipse duration, β=0)
  t_sunlit = T − t_eclipse

Power generation:
  P_gen = A_panel × S × η_cell × cos(θ_avg) × (1 − degradation)^years
  Where:
    S = 1361 W/m² (solar constant)
    η_cell = 0.30 (GaAs triple-junction)
    θ_avg = 23° (average sun incidence for SSO, template-dependent)
    degradation = 0.025/year

Power consumption:
  P_avg = P_bus_standby + P_payload_peak × duty_cycle/100
  P_bus_standby: template-dependent (20W for small, 40W for medium, 60W for large)

Battery DoD:
  E_eclipse = P_avg × t_eclipse / 3600    (Wh consumed in eclipse)
  DoD = E_eclipse / battery_capacity × 100 (%)

Margin:
  margin = (P_gen − P_avg) / P_avg × 100  (%)
```

#### Visualization
- Gauge ring: margin %
- Orbital cycle graph: 1 orbit period on x-axis, showing P_gen (cyan line, drops to 0 in eclipse) vs P_avg (amber dashed, constant). Shaded charge/discharge zones.

#### Go/No-Go
- Go: margin ≥ 15% AND DoD ≤ 30%
- Warn: margin 5–15% OR DoD 30–40%
- No-Go: margin < 5% OR DoD > 40%

---

### 6.3 Module: Link Budget

#### Calculation Model
```
Geometry:
  slant_range = √((a × cos(elev_min))² + 2×a×h + h²) − a × sin(elev_min)
  Where elev_min = 5° (minimum elevation angle)

Link equation (in dB):
  EIRP = P_tx_dBW + G_tx                (dBW)
  L_path = 20 × log10(4π × d × f / c)   (free space loss)
  L_atm = atmospheric_loss(elev_min)      (lookup table)
  L_misc = 2 dB                           (pointing, polarization, etc.)

  C/N0_received = EIRP + G/T_gs − L_path − L_atm − L_misc − k
  Where k = −228.6 dBW/K/Hz (Boltzmann)

  C/N0_required = Eb/N0_req + 10 × log10(data_rate)
  Where Eb/N0_req = 4.5 dB (QPSK, FEC 3/4)

  margin = C/N0_received − C/N0_required  (dB)

Passes/day:
  Estimated from GS latitude vs inclination (lookup table)
  SSO + polar GS (Svalbard): ~6–8
  SSO + mid-lat GS: ~3–5
  LEO + mid-lat GS (inc ~50°): ~4–6

Daily data volume:
  vol = data_rate × passes × avg_pass_duration (seconds) / 8  (bytes)
  avg_pass_duration ≈ 8–12 min depending on altitude

Frequency bands (center frequency for L_path calc):
  UHF: 0.4 GHz
  S-band: 2.2 GHz
  X-band: 8.2 GHz
  Ka-band: 26.5 GHz
```

#### Visualization
- Gauge ring: margin in dB
- Ground station map: Leaflet.js mini-map showing GS location + visibility circle (based on elev_min = 5° at orbit altitude). SSO ground track overlay.

#### Go/No-Go
- Go: margin ≥ 3 dB
- Warn: 1–3 dB
- No-Go: < 1 dB

---

### 6.4 Module: Orbit Decay

#### Calculation Model
```
Atmospheric density (NRLMSISE-00 simplified exponential):
  ρ(h) = ρ0 × exp(−(h − h0) / H)

  Scale height table (km):
    h=200: H=30.3,  ρ0=2.5e-10 kg/m³
    h=300: H=40.7,  ρ0=7.2e-12
    h=400: H=52.0,  ρ0=4.1e-13
    h=500: H=62.2,  ρ0=3.7e-14
    h=600: H=70.5,  ρ0=4.8e-15
    h=800: H=84.4,  ρ0=1.4e-16
  Interpolate between entries. Multiply by solar activity factor:
    Low (F10.7=70): ×0.5
    Medium (F10.7=150): ×1.0 (default)
    High (F10.7=250): ×2.5

Drag deceleration (per orbit):
  Δa_per_orbit = −2π × Cd × (A_drag/m) × ρ(h) × a²
  Where:
    Cd = 2.2 (drag coefficient)
    A_drag = estimated from mass and density (~cross-section)
    a = Re + h (current orbit radius)

Time integration:
  Step: 1 day
  Each step: compute ρ at current h → compute Δa → update h
  Continue until h < 120 km → that's re-entry time

FCC 5-year rule compliance:
  If natural_decay_time > design_life + 5 years:
    Need active deorbit
    ΔV = V_circ × (1 − √(2 × r_peri / (r + r_peri)))
    Where r_peri = Re + 120 km, r = Re + h, V_circ = √(μ/r)
```

#### Visualization
- Gauge ring: estimated lifetime (years)
- Decay chart: altitude vs time curve. FCC 5-year compliance line. If active deorbit needed, show both natural decay curve (dashed) and active deorbit curve (solid cyan).

#### Display Notes
- Show atmospheric model name: "NRLMSISE-00 (F10.7=150, Ap=15)"
- Show FCC citation: "47 CFR §25.283, eff. Sept 2024"

#### Go/No-Go
- Go: natural decay ≤ design_life + 5 years
- Warn: needs ΔV but < 50 m/s
- No-Go: needs ΔV ≥ 50 m/s or no propulsion

---

### 6.5 Module: Mass Budget

#### Calculation Model
```
Subsystem mass estimation (empirical ratios):
  m_structure = 0.25 × m_total (adjusted per template: 0.20 for CubeSat, 0.28 for large)
  m_power = (A_panel × 4.0 kg/m²) + (battery_Wh / 150 Wh/kg)
  m_adcs = template value (1 kg CubeSat, 5 kg small, 12 kg medium, 20 kg large)
  m_obc = template value (0.5 kg CubeSat, 2 kg small, 4 kg medium)
  m_comms = template value (0.5 kg UHF, 3 kg S-band, 5 kg X-band, 8 kg Ka-band)
  m_thermal = 0.04 × m_total
  m_harness = 0.05 × m_total
  m_propulsion = 0 or template value if propulsion included
  m_payload = user input

  m_used = sum of above
  m_margin = m_total − m_used
  margin_pct = m_margin / m_total × 100

Satellite envelope estimation:
  volume = m_total / avg_density
  avg_density: 300–600 kg/m³ per template (CubeSat ~1300, small sat ~400, large ~350)
  Dimensions from template aspect ratio:
    Optical EO: L×W×H = 1.3 : 1.0 : 0.8
    SAR: 1.0 : 1.0 : 0.9
    CubeSat: standard form factors (1U=10×10×10, 3U=10×10×30, 6U=20×10×30)
```

#### Visualization
- Gauge ring: margin %
- Stats table: each subsystem mass
- Satellite envelope: isometric box with L×W×H dimensions, volume, avg density

#### Go/No-Go
- Go: margin ≥ 20%
- Warn: 10–20%
- No-Go: < 10%

---

### 6.6 Module: Thermal

#### Calculation Model
```
Single-node equilibrium temperature:

Heat inputs:
  Q_sun = α × S × A_sun_proj                         (direct solar)
  Q_albedo = α × albedo × S × A_nadir × F_view       (Earth reflected)
  Q_IR = ε × E_IR × A_nadir × F_view                 (Earth infrared)
  Q_internal = P_avg × 0.8                             (80% of electrical → heat)

  Where:
    α = solar absorptance (default 0.3, adjustable)
    ε = IR emissivity (default 0.8, adjustable)
    S = 1361 W/m²
    albedo = 0.3
    E_IR = 237 W/m²
    F_view = 0.3–0.5 (depends on altitude, ~Re²/(Re+h)²)
    A_sun_proj = projected area to sun (template-dependent)
    A_nadir = nadir-facing area

Heat output:
  Q_rad = ε × σ × A_rad × T⁴
  Where σ = 5.67e-8 W/m²/K⁴

Equilibrium:
  Hot case (sunlit): all inputs active
    T_hot = ((Q_sun + Q_albedo + Q_IR + Q_internal) / (ε × σ × A_rad))^(1/4)
  Cold case (eclipse): Q_sun = 0, Q_albedo = 0
    T_cold = ((Q_IR + Q_internal × 0.5) / (ε × σ × A_rad))^(1/4)

Transient (orbital cycle graph):
  Simplified thermal mass: C_th = m_total × c_p (c_p ≈ 900 J/kg/K for aluminum)
  Time step: 10 seconds
  dT/dt = (Q_in − Q_rad) / C_th
  Show 1 orbit of temperature variation
```

#### Visualization
- Gauge ring: "OK" or temperature exceedance
- Orbital temperature cycle graph: T vs time over 1 orbit. Operating range band (−20°C to +50°C default) shown as shaded OK zone. Hot peak and cold valley annotated.

#### Display Notes
- Show model assumptions: "Single-node model. α/ε = 0.3/0.8. Solar 1361 W/m², albedo 0.3, Earth IR 237 W/m²."

#### Go/No-Go
- Go: hot AND cold within operating range
- Warn: within 5°C of limit
- No-Go: exceeds operating range

---

## 7. Step 4 — Launch Vehicle Recommendation

### 7.1 UI
- Filter bar: All / Rideshare / Dedicated / SSO-capable
- Ranked card list (sorted by payload margin descending)
- Each card: rank, name, provider, tags, margin %, cost/kg, LEO/SSO capacity
- "#1 BEST MATCH" badge on top result

### 7.2 Rocket Database (JSON)

```javascript
const ROCKETS = [
  {
    name: "Falcon 9",
    provider: "SpaceX",
    country: "USA",
    capacity_leo_kg: 22800,
    capacity_sso_kg: 15800,
    cost_per_kg_usd: 5500,
    rideshare: true,
    sso_capable: true,
    success_rate: 0.99,
    status: "active",
    launch_sites: ["Cape Canaveral", "Vandenberg"]
  },
  // ... more entries
];
```

#### Rockets to include (minimum set):
| Rocket | Provider | LEO (kg) | SSO (kg) | $/kg | Rideshare |
|--------|----------|----------|----------|------|-----------|
| Falcon 9 | SpaceX | 22,800 | 15,800 | 5,500 | Yes |
| Falcon Heavy | SpaceX | 63,800 | — | 2,500 | Yes |
| PSLV-XL | ISRO | 3,800 | 1,750 | 17,000 | Yes |
| Vega-C | Arianespace | 2,350 | 2,200 | 22,000 | Yes |
| Electron | Rocket Lab | 300 | 200 | 25,000 | No |
| Epsilon S | JAXA | 1,500 | 600 | 35,000 | No |
| Long March 2D | CASC | 3,500 | 1,300 | 8,000 | Yes |
| Soyuz-2.1b | Roscosmos | 8,200 | 4,900 | 12,000 | Yes |
| H3 | JAXA/MHI | 6,500 | 4,000 | 28,000 | Yes |
| LauncherOne | Virgin Orbit | 500 | 300 | 24,000 | No |

#### Matching Logic
```
For each rocket:
  if orbit requires SSO:
    capacity = capacity_sso_kg
  else:
    capacity = capacity_leo_kg

  if capacity > satellite_mass:
    margin = (capacity − satellite_mass) / capacity × 100
    include in results

Sort by: cost_per_kg ascending (rideshare favored), then margin
Filter by: rideshare, dedicated, sso_capable toggles
```

---

## 8. Step 5 — Summary (Terminal)

### 8.1 UI
- Full-width terminal component
- Header: window dots (r/y/g) + title "ORBITSMITH — MISSION SUMMARY"
- Body: typewriter animation, line-by-line reveal (100ms per kv line, 200ms per section header)
- Status bar at bottom: system status, mission type, orbit info

### 8.2 Terminal Content Structure
```
─── MISSION ────────────────────────
Mission type         [value]
Orbit                [value]
Design life          [value]
Total mass           [value]
Envelope (L×W×H)     [value]

─── BUDGET ANALYSIS ────────────────
Power budget         GO/WARN/NOGO (+margin%)
  Generation         [value] W
  Consumption        [value] W
  Battery DoD        [value]%
Link budget          GO/WARN/NOGO (+margin dB)
  Achievable rate    [value] Mbps
  Passes/day         [value]
  Daily downlink     [value] GB
Orbit decay          GO/WARN/NOGO (lifetime)
  FCC 5-yr rule      [status]
  Atmosphere model   NRLMSISE-00
Mass budget          GO/WARN/NOGO (margin%)
  Margin remaining   [value] kg
Thermal              GO/WARN/NOGO (range)

─── LAUNCH VEHICLE ─────────────────
Recommended          [rocket name]
Payload margin       [value]%
Est. cost            [value]

▸ Mission design complete. Export ready.█
```

### 8.3 Export (Future)
- Export PDF button (planned, not MVP)
- Share link button (planned, not MVP)
- Save PNG button (planned, not MVP)

---

## 9. Help System

### 9.1 Inline Tooltips
- Small `?` icon next to every technical parameter and result value
- Click/hover reveals tooltip popup with explanation
- Tooltip: dark bg, cyan border, max 200px wide

### 9.2 Help Data Structure
```javascript
const HELP = {
  "inclination": {
    "en": {
      "short": "Angle between orbit plane and equator",
      "long": "The orbital inclination is the angle between the satellite's orbit plane and Earth's equatorial plane. ~97° is Sun-synchronous orbit (SSO), ideal for Earth observation as it maintains consistent solar illumination."
    },
    "ja": {
      "short": "軌道面と赤道面の角度",
      "long": "軌道傾斜角は、衛星の軌道面と地球の赤道面が成す角度です。約97°は太陽同期軌道（SSO）と呼ばれ、常に同じ太陽角度で地表を観測できるため地球観測衛星に最も多く使われます。"
    }
  },
  "dod": {
    "en": {
      "short": "Battery discharge depth per eclipse",
      "long": "Depth of Discharge (DoD): the percentage of battery capacity used during each eclipse period. Li-ion batteries degrade faster above 30% DoD. Lower is better for battery lifetime."
    },
    "ja": { ... }
  },
  // ... all parameters and results
};
```

### 9.3 Guide Mode (Future)
- "GUIDE" toggle button in page header
- When ON: side panel with contextual explanation per step
- Planned for post-MVP

---

## 10. Technical Implementation Notes

### 10.1 File Structure
Single file: `designer.html` (consistent with tracker.html, imagery.html pattern)
- All CSS inline in `<style>`
- All JS inline in `<script>`
- External dependencies via CDN only:
  - Leaflet.js (ground station map in Link Budget)
  - Chart.js or inline SVG (orbital cycle graphs)
  - Fonts via Google Fonts (already loaded by nav)

### 10.2 Performance
- All calculations client-side JavaScript
- Orbit decay time integration: pre-compute on "RUN ANALYSIS" click, cache result
- Debounce slider inputs (100ms) for real-time recalculation
- Lazy-render Step 3 graphs only when step is visible

### 10.3 Responsive
- 2-column → 1-column below 768px
- Mission cards: 1 column on mobile
- Config panels: stacked on mobile
- Analysis cards: full width on mobile

### 10.4 Constants
```javascript
const CONST = {
  Re: 6371e3,           // Earth radius (m)
  mu: 3.986004418e14,   // Earth gravitational parameter (m³/s²)
  S: 1361,              // Solar constant (W/m²)
  k_dB: -228.6,         // Boltzmann constant (dBW/K/Hz)
  sigma: 5.670374419e-8,// Stefan-Boltzmann constant (W/m²/K⁴)
  c: 299792458,         // Speed of light (m/s)
  albedo: 0.3,          // Earth average albedo
  E_IR: 237,            // Earth IR emission (W/m²)
  Cd: 2.2,              // Drag coefficient
};
```

### 10.5 Local Preview
Before any git operations, user will preview locally:
```bash
# Claude Code workflow:
cd ~/Desktop/Coding/Orbitsmith
# Edit designer.html
# User opens in browser manually to review
# After approval:
git add designer.html
git commit -m "Add Mission Designer (v9a)"
git push origin main
# → Cloudflare Pages auto-deploys
```

### 10.6 Version
- This feature starts a new version: v9a
- Worker (v8a) does NOT need changes — Designer is pure client-side

---

## 11. Implementation Priority

### Phase 1 (MVP)
1. Page scaffolding + nav integration
2. Step 1: Mission selection (7 templates)
3. Step 2: Config UI (all 4 panels with sliders)
4. Step 3: Power Budget + Link Budget (basic, no map yet)
5. Step 3: Orbit Decay + Mass Budget + Thermal (basic)
6. Step 4: Launch vehicle list (static ranking)
7. Step 5: Terminal summary

### Phase 2 (Polish)
1. Leaflet.js ground station map in Link Budget
2. Orbital cycle graphs (Power + Thermal)
3. Satellite envelope visualization (Mass Budget)
4. Decay chart with FCC 5-year line
5. Help tooltips on all parameters
6. Filter bar on Launch Vehicle step

### Phase 3 (Enhancement)
1. i18n (JA/ZH/ES)
2. Export (PDF/PNG/share link)
3. Guide mode
4. URL state persistence (share a design via URL params)
5. Hologram satellite model (optional, for About page or elsewhere)

---

## 12. Reference Materials

### Mockup Files (from this design session)
- `designer-mockup-v2.html` — Full UI mockup with all 5 steps
- `satellite-hologram.html` — Holographic wireframe concept (future use)
- `summary-concepts.html` — Summary page concepts (Terminal selected)

### Data Sources
- SMAD (Space Mission Analysis and Design) — template default values
- NASA DAS (Debris Assessment Software) — orbit decay validation
- FCC 47 CFR §25.283 — 5-year deorbit rule (eff. Sept 2024)
- NRLMSISE-00 — atmospheric density model
- SpaceX, ISRO, Arianespace, Rocket Lab, JAXA user guides — rocket data

---

*End of specification.*
