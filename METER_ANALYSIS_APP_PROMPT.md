# Smart Meter Forensic Analysis Application — GHCP Build Prompt

> **Instructions for GitHub Copilot:** Read this entire file before writing a single line of code.
> This document defines a complete web application. Build it exactly as specified.
> Do not add features not listed. Do not omit features that are listed.

---

## 1. Project Overview

Build a **single-page web application** (pure HTML + CSS + JavaScript, no build tools, no backend)
that allows a user to upload an electricity smart meter JSON data file, automatically runs a full
suite of **deterministic forensic analyses**, and renders a comprehensive **Meter Health & Theft
Risk Report** — all in the browser.

### Core Purpose
The primary focus is **energy theft and meter tampering detection**. The application must flag,
score, and explain every suspicious finding clearly — as if an engineer is handing this report
to a DISCOM (electricity distribution company) fraud investigator.

### Key Constraints
- **Zero dependencies except CDN-hosted libraries** (Chart.js, Lucide icons)
- **No LLM, no AI, no API calls** — every analysis is pure deterministic JavaScript
- **No server, no backend** — everything runs in the browser
- **Single file delivery**: `meter-forensic-report.html`
- Must work with the JSON schema described in Section 4

---

## 2. Tech Stack

| Layer | Technology |
|---|---|
| Markup | Semantic HTML5 |
| Styling | CSS (custom properties, no framework) |
| Logic | Vanilla JavaScript ES2022 (modules via `<script type="module">`) |
| Charts | Chart.js 4.x from CDN (`https://cdn.jsdelivr.net/npm/chart.js`) |
| Icons | Lucide Icons from CDN (`https://unpkg.com/lucide@latest`) |
| Fonts | Google Fonts — `Inter` (body) + `JetBrains Mono` (data/numbers) |

---

## 3. Design System

### Color Palette (CSS Custom Properties)
```css
:root {
  --color-bg:             #0f1117;
  --color-surface:        #161b22;
  --color-surface-2:      #1c2128;
  --color-surface-offset: #21262d;
  --color-border:         #30363d;
  --color-divider:        #21262d;

  --color-text:           #e6edf3;
  --color-text-muted:     #8b949e;
  --color-text-faint:     #484f58;

  /* Semantic Risk Colors */
  --color-critical:       #f85149;
  --color-critical-bg:    rgba(248, 81, 73, 0.1);
  --color-critical-border: rgba(248, 81, 73, 0.3);

  --color-high:           #e3b341;
  --color-high-bg:        rgba(227, 179, 65, 0.1);
  --color-high-border:    rgba(227, 179, 65, 0.3);

  --color-medium:         #d29922;
  --color-medium-bg:      rgba(210, 153, 34, 0.08);
  --color-medium-border:  rgba(210, 153, 34, 0.3);

  --color-low:            #3fb950;
  --color-low-bg:         rgba(63, 185, 80, 0.1);
  --color-low-border:     rgba(63, 185, 80, 0.3);

  --color-info:           #58a6ff;
  --color-info-bg:        rgba(88, 166, 255, 0.08);
  --color-info-border:    rgba(88, 166, 255, 0.25);

  --color-primary:        #238636;
  --color-primary-hover:  #2ea043;

  /* Spacing */
  --space-1: 0.25rem; --space-2: 0.5rem;  --space-3: 0.75rem;
  --space-4: 1rem;    --space-6: 1.5rem;  --space-8: 2rem;
  --space-12: 3rem;   --space-16: 4rem;

  /* Typography */
  --font-body: 'Inter', system-ui, sans-serif;
  --font-mono: 'JetBrains Mono', 'Fira Code', monospace;

  --text-xs:   clamp(0.75rem, 0.7rem + 0.25vw, 0.875rem);
  --text-sm:   clamp(0.875rem, 0.8rem + 0.35vw, 1rem);
  --text-base: clamp(1rem, 0.95rem + 0.25vw, 1.125rem);
  --text-lg:   clamp(1.125rem, 1rem + 0.75vw, 1.5rem);
  --text-xl:   clamp(1.5rem, 1.2rem + 1.25vw, 2.25rem);

  --radius-sm: 4px;  --radius-md: 8px;
  --radius-lg: 12px; --radius-xl: 16px;

  --shadow-sm: 0 1px 3px rgba(0,0,0,0.4);
  --shadow-md: 0 4px 12px rgba(0,0,0,0.5);
  --shadow-lg: 0 12px 32px rgba(0,0,0,0.6);

  --transition: 180ms cubic-bezier(0.16, 1, 0.3, 1);
}
```

### Typography Rules
- Page title: `--text-xl`, `--font-body`, weight 600
- Section headings: `--text-lg`, weight 600
- Body: `--text-base`, weight 400
- Labels/badges: `--text-xs`, weight 500
- All numbers, meter readings, timestamps: `--font-mono`

---

## 4. Input Data Schema

The application parses a JSON file with this top-level structure:

```
{
  "Meter_Number": string,
  "Meter_Make": string,
  "RTC_Date": string (DD-Mon-YYYY HH:MM:SS),
  "Visited_Date": string,
  "General": { ... },        // static meter info
  "Instantenous": { ... },   // live snapshot at reading time
  "Billing_List": [ ... ],   // array of monthly billing records (newest first)
  "LoadSurvey_List": [ ... ] // array of 30-min interval records (newest first)
}
```

### 4.1 General Object
```
{
  "Meter_No": string,
  "Meter_Make": string,           // manufacturer name
  "Firmware": string,
  "Internal_CT_Ratio": string,    // numeric, e.g. "1"
  "Internal_PT_Ratio": string,    // numeric, e.g. "300"
  "Demand_Integration_Period": string, // minutes, e.g. "30"
  "Load_Interval_Period": string  // minutes, e.g. "30"
}
```

### 4.2 Instantaneous Object
```
{
  "Active_Power_Signed": string,       // Watts
  "Reactive_Power_Signed": string,     // VAR
  "Apparent_Power": string,            // VA
  "Power_Factor": string,              // 0 to 1
  "Frequency": string,                 // Hz
  "Voltage1": string,                  // Phase R raw (scale by PT if needed)
  "Voltage2": string,                  // Phase Y
  "Voltage3": string,                  // Phase B
  "Current1": string,                  // Phase R amperes
  "Current2": string,                  // Phase Y
  "Current3": string,                  // Phase B
  "Active_Cummulative_KWH": string,
  "Apparent_Cummulative_KVAH": string,
  "Cumulative_KVARH_Lag": string,
  "Cumulative_KVARH_Lead": string,
  "No_Of_Power_Failures": string,
  "Cumulative_Tamper_Count": string,
  "Cumulative_Programming_Count": string,
  "Cumulative_Billing_Count": string,
  "RTC_Date": string
}
```

### 4.3 Billing_List Entry
```
{
  "Billing_Date": string,
  "Billing_DateTime": string (ISO),
  "Cumulative_Energy_KWH": string,
  "Cumulative_Energy_KVAH": string,
  "Cumulative_Energy_KVARH_Lag": string,
  "Cumulative_Energy_KVARH_Lead": string,
  "Maximum_Demand_KVA": string,
  "Maximum_Demand_KW": string,
  "Maximum_Demand_KVA_Date": string,
  "Maximum_Demand_KW_Date": string,
  // TOD1-8 KWH and KVAH
  "TOD1_KWH": string, "TOD1_KVAH": string,
  "TOD2_KWH": string, "TOD2_KVAH": string,
  "TOD3_KWH": string, "TOD3_KVAH": string,
  "TOD4_KWH": string, "TOD4_KVAH": string,
  "TOD5_KWH": string, "TOD5_KVAH": string,
  "TOD6_KWH": string, "TOD6_KVAH": string,
  "TOD7_KWH": string, "TOD7_KVAH": string,
  "TOD8_KWH": string, "TOD8_KVAH": string,
  // TOD MD per slot
  "Maximum_Demand_TOD1_KVA": string, "Maximum_Demand_TOD1_KW": string,
  ... (TOD2 through TOD8)
  "Power_Factor": string
}
```

### 4.4 LoadSurvey_List Entry
```
{
  "Date_Time": string (DD-Mon-YYYY HH:MM:SS),
  "KWH": string,           // kWh consumed in this 30-min interval
  "KVAH": string,          // kVAh apparent energy in interval
  "Ir": string,            // Phase R current (A)
  "Iy": string,            // Phase Y current (A)
  "Ib": string,            // Phase B current (A)
  "Vr": string,            // Phase R voltage (raw)
  "Vy": string,            // Phase Y voltage (raw)
  "Vb": string,            // Phase B voltage (raw)
  "Maximum_Demand_KVA": string,
  "Maximum_Demand_KW": string
}
```

---

## 5. Application Layout

### 5.1 Page Structure (Single HTML File)

```
┌─────────────────────────────────────────────────────────────────┐
│  HEADER: Logo | App Title | Theme Toggle                        │
├─────────────────────────────────────────────────────────────────┤
│  UPLOAD ZONE (shown when no file loaded)                        │
│    Large drag-drop area + "Click to upload" + JSON format note  │
├─────────────────────────────────────────────────────────────────┤
│  REPORT (shown after file is loaded)                            │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  METER IDENTITY CARD (top banner)                        │   │
│  │  Meter No | Manufacturer | PT/CT Ratio | Visited Date    │   │
│  ├──────────────────────────────────────────────────────────┤   │
│  │  RISK SCORE BANNER                                       │   │
│  │  [████████░░] 78/100 HIGH RISK  (animated progress bar)  │   │
│  │  [ CRITICAL ×2 ] [ HIGH ×3 ] [ MEDIUM ×4 ] [ LOW ×1 ]   │   │
│  ├──────────────────────────────────────────────────────────┤   │
│  │  KPI ROW (6 cards)                                       │   │
│  │  Total kWh | Latest Month kWh | Max Demand | PF | Tamper │   │
│  ├──────────────────────────────────────────────────────────┤   │
│  │  SECTION TABS                                            │   │
│  │  [Theft Risk] [Load Profile] [Billing] [Power Quality]   │   │
│  │  [TOD Analysis] [Raw Findings]                           │   │
│  ├──────────────────────────────────────────────────────────┤   │
│  │  TAB CONTENT AREA                                        │   │
│  │  (changes based on active tab)                           │   │
│  ├──────────────────────────────────────────────────────────┤   │
│  │  DOWNLOAD REPORT BUTTON                                  │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

### 5.2 Upload Zone Behavior
- Full viewport centered layout with dashed border drop zone
- Supports: drag-and-drop file, click-to-browse
- Only accepts `.json` files
- Shows animated spinner while parsing
- Shows error card if JSON is invalid or schema doesn't match
- Animated entrance on file load success

### 5.3 Tab: Theft Risk (Default Active Tab)
This is the PRIMARY tab. It must be the most detailed and alarming when issues are found.

Layout:
```
┌── EXECUTIVE SUMMARY ─────────────────────────────────────────┐
│  2-3 sentence plain-English summary of overall risk           │
└───────────────────────────────────────────────────────────────┘
┌── FINDINGS TABLE (all theft-related findings) ───────────────┐
│  Severity Badge | Finding Name | Value | Threshold | Action   │
└───────────────────────────────────────────────────────────────┘
┌── TAMPER TIMELINE ───────────────────────────────────────────┐
│  Scatter chart: Tamper events mapped to billing cycles        │
└───────────────────────────────────────────────────────────────┘
┌── LOAD-BILLING RECONCILIATION ───────────────────────────────┐
│  Bar chart: Per-month sum(LoadSurvey kWh) vs Billing kWh      │
└───────────────────────────────────────────────────────────────┘
┌── ANOMALY INTERVALS ─────────────────────────────────────────┐
│  Table of top 20 load survey intervals flagged as anomalous   │
└───────────────────────────────────────────────────────────────┘
```

### 5.4 Tab: Load Profile
- Line chart: 30-min interval kWh across all available days (x=time, y=kWh)
- Daily average load curve: average kWh per 30-min slot across all days
- Heatmap: Weekday vs Hour consumption matrix (7×48 grid, colored by intensity)
- Key stats: Avg daily consumption, Peak interval, Off-peak average

### 5.5 Tab: Billing Analysis
- Line chart: Monthly kWh consumption trend (derived from cumulative diff)
- Bar chart: Monthly Max Demand (kVA) trend
- Table: Per-month billing summary with month-on-month delta
- Carbon estimate: Monthly kWh × 0.72 kgCO₂/kWh = tonnes CO₂

### 5.6 Tab: Power Quality
- Phase voltage trend (3 lines: Vr, Vy, Vb) from LoadSurvey
- Phase current trend (3 lines: Ir, Iy, Ib) from LoadSurvey
- Phase imbalance % chart over time
- Power Factor trend over billing months
- Key stats: Avg phase voltage, Avg current imbalance %, Min/Max frequency

### 5.7 Tab: TOD Analysis
- Stacked bar chart: TOD1–8 kWh for each billing month
- TOD distribution donut chart for latest month
- Table: Each TOD's total kWh, KVAH, and % of total consumption
- TOD sum vs cumulative total validation check result

### 5.8 Tab: Raw Findings
- Full JSON-formatted list of all computed analysis outputs
- Collapsible sections per analysis module
- Copy-to-clipboard button

---

## 6. Analysis Engine — All Modules

Implement as a JavaScript module: `class MeterAnalysisEngine`. Call `engine.analyze(jsonData)`
which returns an `AnalysisReport` object consumed by the UI renderer.

### 6.0 Data Parsing & Normalization

```javascript
// All values in the JSON are strings. Parse them safely:
function safeFloat(val, fallback = 0) {
  const n = parseFloat(val);
  return isNaN(n) ? fallback : n;
}

// Parse date strings in format "DD-Mon-YYYY HH:MM:SS"
function parseDate(str) { /* handle null/empty */ }

// Sort Billing_List ascending by Billing_DateTime
// Sort LoadSurvey_List ascending by Date_Time
// The raw file has newest-first ordering
```

### 6.1 Module: Meter Identity

Extract and expose:
- `meterNumber` (from `Meter_Number`)
- `manufacturer` (from `General.Meter_Make`)
- `firmware` (from `General.Firmware`)
- `ptRatio` (from `General.Internal_PT_Ratio`, default 1)
- `ctRatio` (from `General.Internal_CT_Ratio`, default 1)
- `demandIntegrationPeriod` (minutes, from `General.Demand_Integration_Period`)
- `lastVisited` (from `Visited_Date`)
- `rtcDate` (from `RTC_Date`)

### 6.2 Module: Cumulative Event Counters

Extract from `Instantenous`:
- `tamperCount` = `Cumulative_Tamper_Count`
- `powerFailures` = `No_Of_Power_Failures`
- `programmingCount` = `Cumulative_Programming_Count`
- `billingCount` = `Cumulative_Billing_Count`
- `tamperRatePerCycle` = `tamperCount / billingCount`

### 6.3 Module: Monthly Billing Derivation

For each consecutive pair of billing records (sorted ascending):
```
monthlyKWH[i] = Billing_List[i].Cumulative_Energy_KWH - Billing_List[i-1].Cumulative_Energy_KWH
monthlyKVAH[i] = similar
monthlyKVARH_Lag[i] = similar
monthlyMaxDemandKVA[i] = Billing_List[i].Maximum_Demand_KVA
monthlyPowerFactor[i] = Billing_List[i].Power_Factor
loadFactor[i] = monthlyKWH[i] / (monthlyMaxDemandKVA[i] × hoursInMonth) × 100
```

### 6.4 Module: Load Survey Processing

Parse all LoadSurvey_List entries. For each entry:
- `timestamp`: parse Date_Time
- `kwh`: safeFloat(KWH)
- `kvah`: safeFloat(KVAH)
- `ir`, `iy`, `ib`: phase currents
- `vr`, `vy`, `vb`: phase voltages
- `totalCurrent`: ir + iy + ib
- `currentImbalancePct`: (max(ir,iy,ib) - min(ir,iy,ib)) / avg(ir,iy,ib) × 100
  (skip if any current is 0 to avoid divide-by-zero)
- `voltageImbalancePct`: (max(vr,vy,vb) - min(vr,vy,vb)) / avg(vr,vy,vb) × 100

Group intervals by billing month (match each interval's month to the billing cycle it falls in).

### 6.5 Module: Load-Billing Reconciliation

For each billing month:
1. Sum all LoadSurvey kWh values whose timestamps fall within [prev_billing_date, this_billing_date]
2. Compare with derived `monthlyKWH` from billing records
3. Calculate mismatch:
   `mismatchPct = abs(sumIntervalKWH - billingKWH) / billingKWH × 100`
4. If `sumIntervalKWH < billingKWH * 0.85` → interval data significantly LESS than billed (possible data suppression / interval manipulation)
5. If `sumIntervalKWH > billingKWH * 1.15` → interval data significantly MORE than billed (possible billing register manipulation)

### 6.6 Module: Load Survey Gap Detection

After sorting LoadSurvey ascending by timestamp:
1. For each consecutive pair of intervals, calculate `gapMinutes`
2. Expected gap = `Load_Interval_Period` minutes (usually 30)
3. Flag gaps:
   - `30 < gap < 120 minutes`: Minor gap (1-3 missing intervals)
   - `120 <= gap < 480 minutes`: Significant gap (4-16 missing intervals)
   - `gap >= 480 minutes`: Major gap (>8 hours missing)
4. Count each category
5. Total missing intervals = sum of (gap / 30 - 1) across all gaps

### 6.7 Module: Night Load Anomaly Detection

For each day in LoadSurvey:
- `dayLoad` = sum kWh from 06:00–22:00
- `nightLoad` = sum kWh from 22:00–06:00
- `nightRatio` = nightLoad / (dayLoad + nightLoad) × 100

Compute rolling 7-day average night ratio.
Flag intervals where:
- Night kWh = 0 while current day shows non-zero daytime load (possible bypass)
- Night kWh suddenly drops by > 70% from rolling average (discontinuity)
- Night kWh spikes by > 300% from rolling average (unauthorized usage)

### 6.8 Module: Phase Imbalance Analysis

For each LoadSurvey interval where all three currents > 0.1A:
- Calculate `currentImbalancePct` (defined in 6.4)
- Flag intervals where `currentImbalancePct > 15%` as suspicious
- Severe: `currentImbalancePct > 30%` → possible single-phase bypass
- Calculate percentage of total valid intervals that are imbalanced

For voltage:
- Flag intervals where any single phase voltage deviates > 10% from the average of all three phases

### 6.9 Module: Tamper Risk Scoring

Deterministic scoring rubric:

```
tamperRatePerCycle:
  > 10/cycle  → +35 points, CRITICAL
  5–10/cycle  → +25 points, HIGH
  2–5/cycle   → +15 points, MEDIUM
  > 0         → +5  points, LOW
  0           → 0   points

loadBillingMismatch (worst month):
  > 15%       → +25 points, CRITICAL
  10–15%      → +18 points, HIGH
  5–10%       → +10 points, MEDIUM
  2–5%        → +5  points, LOW

majorLoadGaps (gaps > 8 hours):
  > 10        → +15 points, HIGH
  5–10        → +10 points, MEDIUM
  1–5         → +5  points, LOW

currentImbalancedIntervalsPct:
  > 20%       → +10 points, HIGH
  10–20%      → +6  points, MEDIUM
  5–10%       → +3  points, LOW

programmingCount:
  > 10        → +10 points, HIGH
  5–10        → +6  points, MEDIUM
  > 2         → +3  points, LOW

powerFailureRate (failures/billingCount):
  > 10/cycle  → +5  points, MEDIUM (possible physical interference)
  5–10/cycle  → +3  points, LOW

TODSumMismatch (> 2%):
  +5 points, MEDIUM

Total Score → Risk Level:
  0–20   → GREEN  (LOW RISK)
  21–40  → YELLOW (MEDIUM RISK)
  41–60  → ORANGE (HIGH RISK)
  61–100 → RED    (CRITICAL RISK)
```

Each finding must include:
- `id`: unique string
- `name`: human-readable name
- `severity`: 'CRITICAL' | 'HIGH' | 'MEDIUM' | 'LOW' | 'INFO'
- `value`: the actual measured value
- `threshold`: the threshold that was breached
- `description`: 1–2 sentence plain English explanation
- `recommendation`: actionable next step for investigator
- `points`: score contribution

### 6.10 Module: TOD Sum Validation

For the latest billing record:
```
sumTOD_KWH = TOD1_KWH + TOD2_KWH + ... + TOD8_KWH
cumulativeTotalKWH = Billing_List[latest].Cumulative_Energy_KWH
                   - Billing_List[earliest].Cumulative_Energy_KWH
mismatchPct = abs(sumTOD_KWH - cumulativeTotalKWH) / cumulativeTotalKWH × 100
```
Flag if mismatch > 2%.

### 6.11 Module: Power Factor Trend

For each billing month:
- Extract Power Factor from `Billing_List[i].Power_Factor`
- Flag sudden improvement by > 0.03 in a single month without prior trend
  (unexpected PF jump may indicate capacitor bank bypass or CT ratio manipulation)
- Flag PF < 0.85 as inefficiency finding (not theft, but important)

### 6.12 Module: Maximum Demand Anomaly

For consecutive billing months:
- Calculate MD delta: `mdDelta[i] = maxDemandKVA[i] - maxDemandKVA[i-1]`
- Flag sudden drops > 30% in MD when monthly kWh consumption is similar or higher
  (load continues but MD suddenly drops = possible MD tampering)

### 6.13 Module: Carbon Footprint Estimation

India grid average emission factor: **0.716 kgCO₂/kWh** (CEA 2023 baseline)
```
For each derived monthly kWh:
  carbonKg[i] = monthlyKWH[i] × 0.716
  carbonTonnes[i] = carbonKg[i] / 1000
Total lifetime carbon = sum(carbonKg) tonnes
```

---

## 7. Findings — Severity Definitions

| Severity | Color | Meaning |
|---|---|---|
| CRITICAL | `--color-critical` (red) | Strongly indicates active tampering or theft; immediate physical inspection required |
| HIGH | `--color-high` (amber) | Significant anomaly; strong theft indicator; should be investigated within 48 hours |
| MEDIUM | `--color-medium` (yellow) | Suspicious pattern; requires cross-checking with field records |
| LOW | `--color-low` (green) | Minor anomaly; informational; monitor over next billing cycle |
| INFO | `--color-info` (blue) | Neutral observation; context for investigator |

---

## 8. Findings to Detect and Report

Implement EVERY finding below. Each produces a Finding object (defined in 6.9).

### F-01: Tamper Count Per Billing Cycle
- Metric: `tamperCount / billingCount`
- Thresholds: CRITICAL > 10/cycle, HIGH 5–10, MEDIUM 2–5, LOW > 0
- Description template: "Meter has recorded {tamperCount} tamper events across {billingCount}
  billing cycles, averaging {rate:.1f} events per cycle. This is {risk_level} for energy theft
  investigation."
- Recommendation: "Conduct immediate physical inspection of meter enclosure. Check for signs of
  magnet placement, cover damage, or CT bypass wiring."

### F-02: Tamper Count Absolute Value
- Metric: `tamperCount`
- CRITICAL > 500, HIGH 100–500, MEDIUM 20–100, LOW > 0
- Note: Cumulative tamper count of 614 is CRITICAL

### F-03: Load-Billing Reconciliation Mismatch (Worst Month)
- Metric: worst `mismatchPct` across all months
- CRITICAL > 15%, HIGH 10–15%, MEDIUM 5–10%, LOW 2–5%
- Description: "The sum of 30-minute interval readings for {month} was {sumKWH:.1f} kWh, but
  the billing register recorded {billingKWH:.1f} kWh — a {mismatchPct:.1f}% discrepancy."
- Recommendation: "Request meter internal memory audit. Compare meter display readings against
  AMR-downloaded data. Check CT/PT wiring integrity."

### F-04: Load Survey Missing Intervals
- Metric: `totalMissingIntervals` and `majorGapCount`
- HIGH > 10 major gaps, MEDIUM 5–10, LOW 1–5
- Description: "Found {totalMissingIntervals} missing 30-minute recording intervals including
  {majorGapCount} gaps exceeding 8 hours. Missing intervals cannot be accounted for by
  logged power failures."

### F-05: Unexplained Major Load Gaps
- Metric: gaps > 8 hours NOT preceded by a corresponding power failure entry
- CRITICAL if > 5 unexplained major gaps
- Recommendation: "Obtain load survey data from secondary measurement if available. Cross-check
  with substation fault logs for the same dates."

### F-06: Phase Current Imbalance
- Metric: % of intervals with current imbalance > 15%
- HIGH if > 20% of intervals are imbalanced
- MEDIUM if > 10%
- Description: "Phase current imbalance exceeded 15% in {pct:.0f}% of recorded intervals.
  Severe single-phase current imbalance can indicate a single-phase CT bypass or
  jumper wire on one phase."
- Recommendation: "Inspect CT secondary connections. Verify all three CTs are intact and
  properly connected."

### F-07: Meter Programming Events
- Metric: `programmingCount`
- HIGH > 10, MEDIUM 5–10, LOW > 2
- Description: "The meter firmware/parameters have been modified {programmingCount} times since
  installation. Unauthorized programming changes can alter CT/PT ratios, TOD schedules, or
  demand integration periods."
- Recommendation: "Retrieve meter programming log with timestamps. Verify each change was
  authorized by the DISCOM engineering team."

### F-08: TOD Sum vs Cumulative Total Mismatch
- Metric: `todSumMismatchPct`
- MEDIUM > 2%, LOW > 0.5%
- Description: "Sum of all 8 TOD register values ({sumTOD:.1f} kWh) differs from the cumulative
  register ({cumulativeKWH:.1f} kWh) by {pct:.2f}%. These must match exactly."

### F-09: Night Load Anomaly
- Metric: nights with zero load while daytime load is non-zero
- CRITICAL > 10 such nights
- HIGH 5–10, MEDIUM 2–5
- Description: "Detected {count} nights with zero recorded consumption despite active daytime
  load the same day. Zero night load with active day load may indicate meter bypass active
  only during night hours."

### F-10: Power Factor Sudden Jump
- Metric: single-month PF improvement > 0.03
- MEDIUM severity
- Description: "Power Factor improved from {pf_old:.2f} to {pf_new:.2f} in {month} without
  a corresponding change in load profile. Unexplained PF improvements may indicate
  measurement path alteration."

### F-11: Maximum Demand Sudden Drop
- Metric: MD drop > 30% in a single month with stable or rising consumption
- HIGH severity
- Description: "Maximum Demand dropped from {md_old:.1f} kVA to {md_new:.1f} kVA ({pct:.0f}%)
  while monthly consumption {consumption_change}. MD register manipulation can reduce
  demand-related charges."

### F-12: Power Failure Rate
- Metric: `powerFailures / billingCount`
- MEDIUM > 10/cycle (possible physical interference with supply)
- LOW 5–10/cycle

### F-13: Load Factor Assessment
- Metric: Average monthly load factor across all months
- INFO: Report actual value
- Flag: Load factor < 30% with high tamper count (INFO → MEDIUM escalation)
- Description: "Average load factor is {lf:.1f}%. This represents the ratio of actual consumption
  to the theoretical maximum if consuming at peak demand continuously."

### F-14: Phase Voltage Deviation
- Metric: % of intervals with any phase voltage deviating > 10% from 3-phase average
- MEDIUM > 15% of intervals
- Description: "Phase voltage imbalance detected in {pct:.0f}% of recorded intervals.
  Consistent voltage anomalies may indicate PT wiring interference."

---

## 9. Risk Score Display

Show a full-width banner at the top of the report:

```html
<!-- Risk Score Banner -->
<div class="risk-banner risk-level-{critical|high|medium|low}">
  <div class="risk-score-circle">
    <span class="score-number">{score}</span>
    <span class="score-max">/100</span>
  </div>
  <div class="risk-label">{CRITICAL RISK | HIGH RISK | MEDIUM RISK | LOW RISK}</div>
  <div class="risk-badges">
    <span class="badge badge-critical">{n} Critical</span>
    <span class="badge badge-high">{n} High</span>
    <span class="badge badge-medium">{n} Medium</span>
    <span class="badge badge-low">{n} Low</span>
  </div>
  <div class="risk-recommendation">{top recommendation text}</div>
</div>
```

The score bar should animate from 0 to final score on load using CSS animation.

---

## 10. Charts Specification

Use Chart.js 4. All charts must:
- Use dark background: `backgroundColor: 'transparent'`
- Use `--color-text-muted` for axes
- Use `--color-border` for grid lines
- Have a legend only when > 1 dataset
- Be responsive with `maintainAspectRatio: false`

### Chart 1 — Monthly kWh Trend (Billing tab)
- Type: Line
- X: Month label (e.g., "Mar 2026")
- Y: Derived monthly kWh
- Color: `#58a6ff` with area fill at 20% opacity
- Point radius: 4, hover radius: 6

### Chart 2 — Monthly Max Demand Trend (Billing tab)
- Type: Bar
- X: Month label
- Y: Max Demand kVA
- Color: `#3fb950`

### Chart 3 — Load-Billing Reconciliation (Theft tab)
- Type: Grouped Bar (2 datasets)
- Dataset 1: "Billing Register kWh" → `#58a6ff`
- Dataset 2: "Load Survey Sum kWh" → `#f85149`
- If mismatch > 5%, show red asterisk annotation

### Chart 4 — 30-min Load Profile Heatmap (Load Profile tab)
- Render as an HTML/CSS grid (48 columns × N days rows)
- Color intensity scaled from `--color-surface-offset` to `#58a6ff`
- X: 00:00 to 23:30 (48 half-hour slots)
- Y: Each day (show last 30 days if data is large)
- Hover tooltip shows: date, time, kWh value

### Chart 5 — Daily Average Load Curve (Load Profile tab)
- Type: Line
- X: 00:00 to 23:30
- Y: Average kWh per slot across all days
- Mark peak slot and off-peak slot with annotation

### Chart 6 — TOD Distribution Donut (TOD tab)
- Type: Doughnut
- 8 segments, one per TOD
- Colors: use distinct palette from Chart.js defaults
- Center label: Total kWh

### Chart 7 — TOD Monthly Stacked Bar (TOD tab)
- Type: Stacked Bar
- X: Month
- Y: kWh
- 8 stacks (TOD1–8)

### Chart 8 — Phase Voltage Trends (Power Quality tab)
- Type: Line
- 3 lines: Vr, Vy, Vb
- Colors: R=`#f85149`, Y=`#e3b341`, B=`#58a6ff`
- Show last 7 days of LoadSurvey data (to keep chart readable)

### Chart 9 — Phase Current Trends (Power Quality tab)
- Type: Line
- 3 lines: Ir, Iy, Ib
- Same colors as voltage chart

### Chart 10 — Current Imbalance Over Time (Power Quality tab)
- Type: Line
- X: Time
- Y: Imbalance %
- Reference line at y=15% (warning threshold) and y=30% (critical threshold)
- Color areas above threshold in red

---

## 11. KPI Cards

Show 6 KPI cards in a grid below the risk banner:

| Card | Value | Sub-label | Color Logic |
|---|---|---|---|
| Cumulative kWh | `Active_Cummulative_KWH` | "Lifetime consumption" | info |
| Last Month kWh | `monthlyKWH[latest]` | month name | info |
| Peak Demand | `Maximum_Demand_KVA` (instantaneous) | "kVA" | info |
| Power Factor | `Power_Factor` | "Phase avg" | green if >0.95, yellow if 0.85–0.95, red if <0.85 |
| Tamper Events | `tamperCount` | "Lifetime" | red if critical |
| Missing Intervals | `totalMissingIntervals` | "Load survey gaps" | red if >50 |

---

## 12. Report Generation (Download)

When user clicks "Download Report":
1. Capture the full report DOM using `document.getElementById('report-container').outerHTML`
2. Prepend a proper `<html><head>` with all inline CSS (already embedded styles)
3. Include all chart images by calling `chart.toBase64Image()` on each Chart.js instance
   and replacing `<canvas>` elements with `<img>` tags
4. Create a Blob and trigger download as `meter_{meterNumber}_report_{date}.html`
5. The downloaded file must be a fully self-contained HTML that renders correctly
   without internet connection

---

## 13. Error Handling

### Invalid File
- Show a red error card: "Invalid file format. Expected a meter JSON file with
  Meter_Number, Billing_List, and LoadSurvey_List fields."

### Missing Sections
- If `LoadSurvey_List` is empty or missing: show yellow info banner
  "Load survey data unavailable. Interval-based analyses will be skipped.
  Only billing-based findings will be reported."
- If `Billing_List` has fewer than 2 entries: "Insufficient billing history for trend analysis."

### All Null Values
- For each analysis, check for valid data before running
- Show "Insufficient data" placeholders in charts/tables rather than broken renders

---

## 14. Accessibility

- Every `<canvas>` must have an `aria-label` describing the chart
- All interactive elements must be keyboard-navigable
- Tab key must cycle through section tabs
- All severity badges must have `role="status"` and `aria-label`
- Minimum touch target: 44×44px
- All text must meet WCAG AA contrast on the dark theme

---

## 15. Code Organization

Organize JavaScript as follows inside the single `<script type="module">` block:

```javascript
// ─── SECTION 1: Data Parser ────────────────────────────────────
// Functions: parseJsonFile(), normalizeBillingList(), normalizeLoadSurvey()

// ─── SECTION 2: Analysis Engine ────────────────────────────────
// Class: MeterAnalysisEngine
//   Methods: analyze(), computeMonthlyBilling(), computeLoadSurveyStats(),
//            reconcileBillingVsIntervals(), detectLoadGaps(),
//            detectNightLoadAnomaly(), detectPhaseImbalance(),
//            computeTamperRisk(), computeTODValidation(),
//            computePowerFactorTrend(), computeMDAnomaly(),
//            computeCarbonFootprint(), generateFindings(),
//            computeRiskScore()

// ─── SECTION 3: Risk Scorer ─────────────────────────────────────
// Function: scoreFindings(findingsArray) → { score, level, summary }

// ─── SECTION 4: Chart Renderer ──────────────────────────────────
// Class: ChartRenderer
//   Methods: renderAll(), renderMonthlyKWH(), renderMaxDemand(),
//            renderReconciliation(), renderLoadHeatmap(),
//            renderLoadCurve(), renderTODDonut(), renderTODMonthly(),
//            renderPhaseVoltage(), renderPhaseCurrent(),
//            renderImbalance()

// ─── SECTION 5: UI Renderer ─────────────────────────────────────
// Class: UIRenderer
//   Methods: renderReport(), renderRiskBanner(), renderKPIs(),
//            renderTheftTab(), renderFindingsTable(),
//            renderAnomalyIntervals(), renderBillingTab(),
//            renderLoadProfileTab(), renderPowerQualityTab(),
//            renderTODTab(), renderRawFindingsTab()

// ─── SECTION 6: Report Exporter ─────────────────────────────────
// Function: exportReport(meterNumber, charts)

// ─── SECTION 7: App Bootstrap ───────────────────────────────────
// Event listeners: drag-drop, file input, tab clicks
// init()
```

---

## 16. Sample Output Expectations

When loaded with the reference meter file (meter 252909):

Expected Risk Score: **~75–85 points** → CRITICAL RISK

Expected Critical Findings:
- F-01: Tamper rate 5.4/cycle → HIGH (25 pts)
- F-02: Absolute tamper count 614 → CRITICAL (35 pts)

Expected High Findings:
- F-07: Programming count 6 → LOW (3 pts)
- F-12: Power failures 536 → MEDIUM
- F-03: Load-billing reconciliation (if intervals available)

Expected Info Findings:
- F-13: Load factor per month
- Carbon footprint per month

---

## 17. Final Deliverable Checklist

Before considering the application complete, verify:

- [ ] File upload works with drag-drop and click-to-browse
- [ ] JSON parsing handles all null/empty string values without crashing
- [ ] All 14 findings (F-01 through F-14) are computed and displayed
- [ ] Risk score is computed correctly and animates on load
- [ ] All 10 charts render without errors
- [ ] Theft Risk tab is shown by default
- [ ] Tab switching works without chart re-render bugs (destroy + recreate)
- [ ] KPI cards show correct values
- [ ] Download Report produces a working standalone HTML file
- [ ] Phase imbalance chart shows threshold lines at 15% and 30%
- [ ] Load heatmap renders last 30 days of interval data
- [ ] Executive summary text reflects actual risk level and top findings
- [ ] Dark theme is consistent across all components
- [ ] Application works fully offline (CDN only, no other network calls)
- [ ] No console errors on load with valid file
- [ ] All numbers display with `font-family: var(--font-mono)` and proper decimal formatting
- [ ] Responsive layout works at 375px and 1280px viewport widths

---

*End of GHCP Build Prompt. Begin implementation.*
