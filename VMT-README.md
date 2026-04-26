# VMT and GHG Emissions Forecast

**Rutgers–New Brunswick, Edward J. Bloustein School of Planning and Public Policy
Alan M. Voorhees Transportation Center**

A self-contained, client-side HTML tool for forecasting future vehicle-miles of travel (VMT) and greenhouse gas (GHG) emissions on major roads within a metropolitan statistical area (MSA) based on projected changes in lane-miles, population, and fuel prices. All calculations run in the browser with no server, backend, or installation required.

---

## Overview

The tool applies a log-linear elasticity model to estimate how changes in three factors — road capacity (lane-miles), population, and fuel price — affect future VMT on major roads (freeways, expressways, and principal arterials). GHG emissions are then derived from the VMT forecast using a fleet-average emission factor that declines over time as the vehicle fleet electrifies.

Outputs include projected future VMT, change in VMT, a factor-level contribution breakdown, a year-by-year VMT trajectory chart, GHG metrics, a GHG trajectory chart, and a printable summary report.

---

## Getting Started

No installation or build step is required.

1. Save or download `vmt-estimator.html`.
2. Open the file in any modern web browser (Chrome, Firefox, Safari, Edge).
3. All inputs, calculations, and outputs are handled locally in the browser.

---

## Inputs

### Base year inputs

| Field | Default | Notes |
|---|---|---|
| Base VMT on major roads | 12,500 | Million miles/year; freeways, expressways, and principal arterials |
| Base lane-miles of major roads | 8,000 | Freeways, expressways, and principal arterials |
| Base population | 3,200 | Thousands |
| Base fuel price | $3.50 | Dollars per gallon |

### Future year projections

| Field | Default | Notes |
|---|---|---|
| Future lane-miles of major roads | 8,500 | Can be entered as absolute values or % change from base |
| Future population | 3,500 | Can be entered as absolute values or % change from base; thousands |
| Future fuel price | $4.00 | Can be entered as absolute values or % change from base; dollars per gallon |
| Forecast horizon | 10 | Years; accepted range 1–50 |

Future year values can be entered in two modes, toggled via the **Absolute values / % change from base** selector. Switching modes converts existing entries automatically.

### Elasticity inputs

Three elasticities control the sensitivity of VMT to each factor. Values are set via sliders.

| Parameter | Default | Range | Description |
|---|---|---|---|
| Lane-miles elasticity (ε₁) | 0.90 | 0.00–1.50 | % change in VMT per 1% change in lane-miles |
| Population elasticity (ε₂) | 1.00 | 0.00–2.00 | % change in VMT per 1% change in population |
| Fuel price elasticity (ε₃) | −0.30 | −1.50–0.00 | % change in VMT per 1% change in fuel price (negative: higher prices reduce VMT) |

#### Literature-based presets

Four preset elasticity combinations are provided, grounded in the induced-travel literature:

| Preset | ε₁ | ε₂ | ε₃ |
|---|---|---|---|
| Short-run typical | 0.90 | 1.00 | −0.30 |
| Long-run typical | 1.10 | 1.05 | −0.60 |
| Low responsiveness | 0.70 | 0.90 | −0.15 |
| High responsiveness | 1.30 | 1.20 | −0.80 |

### GHG emissions parameters

| Field | Default | Notes |
|---|---|---|
| Fleet type | Average US fleet (2024) — 404 g CO₂e/mi | Dropdown; sets the base emission factor |
| Custom emission factor | 404 | g CO₂e/mi; available when "Custom" is selected |
| Fleet electrification rate | 1.0%/yr | Annual reduction in emission factor; range 0–5%/yr; floored at 5% of starting EF |

**Fleet type options:**

| Option | Emission factor |
|---|---|
| Average US fleet (2024) | 404 g CO₂e/mi |
| Light-duty vehicles | 350 g CO₂e/mi |
| Fuel-efficient fleet | 270 g CO₂e/mi |
| 30% EV penetration | 180 g CO₂e/mi |
| 70% EV penetration | 80 g CO₂e/mi |
| Custom | User-defined |

Emission factors are sourced from US EPA and FHWA fleet-average estimates.

---

## Methodology

### VMT forecast model

The tool applies a log-linear elasticity model to project future VMT on major roads:

```
VMT_future = VMT_base × (ΔLane-miles)^ε₁ × (ΔPopulation)^ε₂ × (ΔFuel price)^ε₃
```

Where each Δ term is the ratio of future to base-year value:

```
futureVMT = baseVMT
          × (futLane  / baseLane )^ε₁
          × (futPop   / basePop  )^ε₂
          × (futPrice / basePrice)^ε₃
```

Intermediate years assume smooth (linear) annual change in each driver from the base to the future year value. The model applies only to VMT on major roads — the same facility class as the lane-miles input (freeways, expressways, and principal arterials).

### Factor contribution breakdown

The isolated contribution of each factor is computed by holding the other two factors at their base-year values:

```
Lane-miles effect  = baseVMT × (laneT / baseLane)^ε₁
Population effect  = baseVMT × (popT  / basePop )^ε₂
Fuel price effect  = baseVMT × (priceT/ basePrice)^ε₃
```

Each contribution is expressed as a percentage change from base-year VMT and displayed as a proportional bar.

### GHG emissions model

Annual GHG emissions are calculated as:

```
Annual GHG (MT CO₂e/yr) = VMT (million mi/yr) × EF (g CO₂e/mi) ÷ 10⁶
```

The emission factor declines each year as the vehicle fleet electrifies, with a floor at 5% of the starting value:

```
EF(t) = max(EF₀ × (1 − evRate)^t, EF₀ × 0.05)
```

Where `evRate` is the annual fleet electrification rate (as a decimal) and `t` is the number of years from the base year.

### Counterfactual GHG

A counterfactual GHG trajectory is computed assuming VMT remains at its base-year level while the emission factor still declines:

```
Counterfactual GHG(t) = VMT_base × EF(t) ÷ 10⁶
```

The gap between projected GHG and the counterfactual represents emissions attributable to the VMT change.

### Cumulative additional GHG

Cumulative additional GHG is the sum of the annual gap between projected and counterfactual GHG over the full forecast horizon:

```
Cumulative additional GHG = Σ(t=1 to T) [Projected GHG(t) − Counterfactual GHG(t)]
```

A positive value indicates net additional emissions due to VMT growth; a negative value indicates net avoided emissions due to VMT reduction.

### Summary metrics

| Metric | Formula |
|---|---|
| Future VMT | baseVMT × (rLane^ε₁) × (rPop^ε₂) × (rPrice^ε₃) |
| Change in VMT | futureVMT − baseVMT |
| Percent change | (multiplier − 1) × 100% |
| Base year annual GHG | baseVMT × EF₀ ÷ 10⁶ |
| Future year annual GHG | futureVMT × EF(T) ÷ 10⁶ |
| Change in annual GHG | Future GHG − Base GHG |
| Cumulative additional GHG | Σ annual gap over forecast horizon |

---

## Outputs

### VMT forecast metrics

Three real-time metric cards display:

- **Future VMT on major roads** — million miles/year
- **Change in VMT** — million miles/year (positive = increase; negative = decrease)
- **Percent change** — over the specified forecast horizon

### Factor contribution breakdown

A bar chart shows the isolated percentage-point contribution of each factor (lane-miles, population, fuel price) to the total VMT change, scaled relative to the largest contributor.

### VMT trajectory chart

A line chart plots projected total VMT and the three individual factor-effect lines over the forecast horizon. Each factor-effect line holds the other two variables at base-year values, illustrating each driver's standalone contribution to the trajectory.

### GHG metrics

Four real-time metric cards display:

- **Base year annual GHG** — MT CO₂e/yr
- **Future year annual GHG** — MT CO₂e/yr
- **Change in annual GHG** — MT CO₂e/yr
- **Cumulative additional GHG** — MT CO₂e over the forecast horizon

### Emission factor trajectory chart

A small chart shows how the fleet emission factor (g CO₂e/mi) declines from the base year to the end of the forecast horizon under the specified electrification rate.

### GHG trajectory chart

A line chart plots projected annual GHG and the counterfactual (base VMT, declining EF only). The gap between the two lines represents the GHG impact attributable to the VMT change.

### Print / PDF summary

The **Print / Save as PDF** button captures the current state of all inputs, elasticities, metrics, and charts and opens a print-ready page in a new tab. Using the browser's "Save as PDF" option produces a formatted report dated at the time of printing.

---

## Technical notes

- The tool is a **single HTML file** with no external dependencies except Chart.js, which is loaded from the Cloudflare CDN (`cdnjs.cloudflare.com`). An internet connection is required for chart rendering.
- All calculations are triggered automatically on any input change.
- The input mode toggle (absolute / % change) converts values in place without requiring re-entry.
- Charts render using [Chart.js 4.4.1](https://www.chartjs.org/).
- Light and dark mode are supported automatically via CSS `prefers-color-scheme`.
- The tool is mobile-responsive and touch-accessible, including larger tap targets and prevention of unwanted zoom on iOS.
- Content Security Policy (CSP) meta headers restrict script sources to `self` and `cdnjs.cloudflare.com`; all other outbound connections are blocked.
- The pop-up blocker must allow pop-ups from the file's origin for the Print/PDF feature to work.

---

## Data sources and references

| Parameter | Source |
|---|---|
| Elasticity values | Millard-Ball & Rosen (2025). *Road Capacity as a Fundamental Determinant of Vehicle Travel*. UC Institute of Transportation Studies. [https://escholarship.org/uc/item/180452jz](https://escholarship.org/uc/item/180452jz) |
| Emission factors | US EPA and FHWA fleet-average estimates |
| Fleet type presets | US EPA fleet emission factor guidance |

---

## Limitations

- The model applies only to VMT on major roads (freeways, expressways, and principal arterials) — the same facility class as the lane-miles input. Local and collector roads are not modeled.
- The log-linear elasticity model assumes constant elasticities over the forecast horizon. Non-linear behavioral responses (e.g., saturation effects, mode shift) are not captured.
- Intermediate-year projections assume smooth linear change in all three drivers. Sudden discontinuities (e.g., policy shocks, economic crises) are not modeled.
- The GHG model does not account for changes in fleet composition beyond aggregate electrification rate, nor does it model upstream (lifecycle) emissions.
- The tool does not model induced demand feedback loops, land use change, or transit substitution effects.

---

## License and attribution

Developed by the **Alan M. Voorhees Transportation Center**, Edward J. Bloustein School of Planning and Public Policy, Rutgers–New Brunswick.

When citing or adapting this tool, please reference the Voorhees Transportation Center and the elasticity literature cited above.
