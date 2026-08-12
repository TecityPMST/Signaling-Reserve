# The Signaling Reserve

A weekly forensic working paper on the 2026 U.S. Strategic Petroleum Reserve exchange during the
Iran / Strait of Hormuz disruption. The paper's core claim is that the SPR exchange has functioned
as **active price suppression** rather than supply remedy, and that the signal and the supply are
spent in the same act.

**Current version: v2.10** · weekly cadence, published as a self-contained HTML file plus a PDF.

---

## Repository layout

```
index.html                      # the current paper — always this name, overwritten each week
signaling-reserve-vX.Y.pdf      # versioned PDF, one per week, kept as the archive
US Iran BBG Data*.xlsx          # the Bloomberg pull — the single source of oil data
INSTRUCTIONS.md                 # the weekly update runbook
README.md                       # this file
```

Prior versions are kept as files rather than overwritten. The change-log inside each paper carries
**only that version's delta**; full history lives in the earlier files.

---

## Data sources

| Series | Source | Notes |
|---|---|---|
| Brent front / spot | BBG `CO1 Comdty` / `COA Comdty` | ICE |
| WTI front / spot | BBG `CL1 Comdty` / `CLA Comdty` | NYMEX |
| Dubai front / spot | BBG `DBL1 Comdty` / `DBLA Comdty` | CME Mini-Dubai (Platts) |
| Oman front / spot | BBG `OQA1 Comdty` / `OQAA Comdty` | GME |
| SPR inventory | BBG `DOESSPR index` | **thousand barrels** — divide by 1000 for mb |
| US average gasoline | BBG `AUTMUSAG Index` | |
| News / context | Web, ≥2 reputable outlets per claim | never prices |

**`US Iran BBG Data` is the only source of oil data.** Prices, SPR, spreads and gasoline come from
that workbook and nowhere else. News and context come only from the web. The two are never mixed.

There is no separate price CSV. The series as plotted lives in the `text/plain` chart block of the
latest published HTML; the underlying data lives in the workbook. Anything else is a duplicate that
will eventually disagree with both.

### Sheet layout (`Data` sheet, 0-indexed columns)

Row 1 ticker · row 2 security name · row 3 O/H/L/C · data from row 4. Close = group start + 3.

```
date=0  CO1=4  COA=9  CL1=14  CLA=19  DBL1=24  DBLA=29  OQA1=34  OQAA=39  DOESSPR=44  AUTMUSAG=46
```

Coerce `#N/A` and blanks to `None`.

---

## Output contract

**Every run produces exactly two files:**

- **`index.html`** — the current paper, always under that exact name, overwritten each week. Named
  for direct upload to GitHub Pages, which serves `index.html` as a site's front page.
- **`signaling-reserve-vX.Y.pdf`** — carries the version number, and is never overwritten. The PDFs
  are the version archive.

Nothing else.

No run notes, no review memos, no extra CSV variants, no intermediate or working files in this
folder. If a run cannot produce a version — stale data, an unreadable prior version, an unresolved
data question — the outcome is reported back in the conversation, not written to disk.

## Standing conventions

Do not change these without an explicit decision, and record any change in the change-log.

- **The spread convention is the one used in v2.7–v2.10, and v2.10 is authoritative.**
  "Front" is the `COA` column; the spot basis is `COA` minus its paired `CO1`/`CO2` column, and
  Dubai likewise (`DBLA − DBL1`). Apply it consistently across the whole series — the failure mode
  is applying one sign to part of the chart and the opposite to the rest, which silently reverses
  the paper's claim about curve structure.
  The ticker in the paired column has differed between pulls (`CO1` in some, `CO2` in others), so
  read row 1 of the sheet each week rather than assuming a fixed column.
- **SPR operational floor = ~200 mb** (Tecity in-house estimate, analyst Zilin), stated as above the
  ~150 mb conventionally cited. Cushion = current SPR − 200.
- **Self-contained HTML.** Charts are static base64 PNGs in `<img id="cimg-…">`. No CDN, no Chart.js
  at runtime — they render blank in preview. The chart *data* lives in a non-executing
  `<script type="text/plain" data-chartsrc="1">` block near the end; that block is the source of
  truth for appends and never runs.
- **Versioning.** Increment the minor digit (v2.9 → v2.10 → v2.11). Major bumps only on request.

---

## Known data hazards

Every one of these has actually occurred. Check them before trusting a week's numbers.

1. **Front/spot sign inversion.** The single most damaging failure mode: it silently reverses the
   paper's central claim about curve structure. Guard by recomputing two or three already-published
   points from the new file and confirming they reproduce the plotted values exactly. If they don't,
   stop and reconcile before building — do not publish a series that changes convention mid-chart.
2. **Feed collapse — duplicated columns.** In June 2026 pulls, `CO1` equalled `COA` to the cent on
   21 of 22 days, and `DBL1` equalled `DBLA` likewise. A real market never does this. If front == spot
   for consecutive days, the feed is broken, not flat — do not build on it, re-pull.
3. **Mini-Dubai rolls monthly.** The Dubai basis steps discontinuously at each month turn. Levels are
   comparable *within* months, not across them. A month-boundary "flip" may be roll, not market.
4. **US exchange holidays.** On Juneteenth (Fri 19 Jun 2026) the CME/NYMEX-listed pairs (Dubai, WTI,
   Oman) had no print while ICE Brent traded normally. Leave the gap open; do not interpolate.
5. **Same-day prints revise.** A live session pulled intraday will settle differently — the 30 Jul 2026
   point moved on settlement, and the 6 Aug row moved ~$0.60 between two pulls hours apart. Prefer the
   last complete session, or label the live one and expect to revise it.
6. **EIA release timing.** The weekly SPR print lands after some pulls. If `DOESSPR` is `#N/A` for the
   latest week, the pull simply predates the release — re-pull rather than inventing a value.

---

## Version history

| Version | Built | Data through (SPR) |
|---|---|---|
| v1 – v2.4 | 27 May – 25 Jun 2026 | — |
| v2.5 | 25 Jun 2026 | 19 Jun |
| v2.6 | 2 Jul 2026 | 26 Jun |
| v2.7 | 16 Jul 2026 | 10 Jul |
| v2.8 | 23 Jul 2026 | 17 Jul |
| v2.9 | 30 Jul 2026 | 24 Jul |
| v2.10 | Aug 2026 | 31 Jul |

---

## Confidentiality and licensing

**This repository must be private.**

- The price data is Bloomberg-derived. Bloomberg's terminal licence generally prohibits
  redistribution; pushing BBG-derived series to a third-party host may breach it. Confirm with
  whoever owns the Tecity terminal agreement before the first push, and consider excluding the raw
  `*.xlsx` pulls via `.gitignore`.
- The paper contains in-house estimates and named internal analysts.
- The paper is an analytical reconstruction for informational purposes only — not investment, legal
  or financial advice.
