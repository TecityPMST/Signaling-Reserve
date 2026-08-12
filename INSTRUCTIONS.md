# Weekly update runbook

How to take the paper from vX.Y to vX.Y+1. Read `README.md` first — particularly
**Standing conventions** and **Known data hazards**.

Working folder (inputs *and* outputs) is this folder:
`OneDrive - Straits Developments Pte Ltd\Bloomberg [BB]'s files - PMST Research Dashboard\The Signaling Reserve`

Environment: `pip install openpyxl matplotlib weasyprint pymupdf`

> **Pin this folder locally before running.** In File Explorer, right-click *The Signaling Reserve*
> → **Always keep on this device**. OneDrive files-on-demand placeholders cannot be opened by any
> read path (shell, file tools, ripgrep all fail), and this blocked the 7 Aug 2026 run outright —
> the prior version could not be read, so no new version could be built on it.

---

## Step 0 — Get the data

Drop the new `US Iran BBG Data_<date>.xlsx` into this folder. Use the **newest by modified time**;
a `_updated` or distinct-named file supersedes an earlier same-week one.

> OneDrive is files-on-demand. If a script errors reading the xlsx, the file is cloud-only —
> open it once to hydrate it, then process.

## Step 1 — Freshness guard

Read `index.html` — it is always the current published version — and find the last label in the
`brentChart` `labels` array. Confirm the version it declares matches the newest
`signaling-reserve-vX.Y.pdf` in the folder; if they disagree, stop and reconcile before building. **If the newest xlsx date is not after it, stop** and report that you are waiting
for this week's file. Do not build.

Separately: if `DOESSPR` has no new print, the SPR series cannot advance. That is a legitimate
reason to wait a day rather than publish a prices-only update.

## Step 2 — Parse and validate

Parse the `Data` sheet per the column map in `README.md`, then run these checks **before** building:

- [ ] `CO1 != COA` on most days — if they match to the cent for consecutive days the feed is broken
- [ ] `CL1 != CLA` likewise
- [ ] Latest `DOESSPR` is newer than the last plotted SPR point
- [ ] Recompute two or three already-published weekly points from the new file and confirm they
      reproduce the plotted values exactly — the fastest test that the sign convention still holds
- [ ] Read row 1 of the sheet and confirm which ticker sits in the paired column (`CO1` vs `CO2`);
      it has varied between pulls
- [ ] Confirm the convention matches v2.10: front = `COA`, basis = `COA` − paired column
- [ ] Scan the Dubai basis for month-turn steps (contract roll) and for any single spurious ±$13 print

**Prior-week revision.** Compare the previous version's latest points against the new file. A
preliminary print often settles differently — update those points and note it in the change-log.

## Step 3 — Verify the week's news

Search for the week's Iran / Hormuz / Houthi / ceasefire / SPR developments. **Every claim needs
≥2 reputable outlets** (EIA, Reuters, AP, CNBC, NPR, CNN, Al Jazeera, Bloomberg, Axios, CBS, ABC,
France 24, Kpler, TradingEconomics). Record exact dates and outlet names.

Mark each claim honestly in the verification ledger:

- `✓` verified — two or more independent outlets
- `≈` inference, or reported but not executed — e.g. a deal described as "close" but unsigned
- Claims by a belligerent are **claims**, not facts. State the corroboration ratio where one exists.

Never take a price from the web.

## Step 4 — Build the next version

Work from a copy of `index.html` (the prior version) with targeted string replacements. Build into
a temporary file and only replace `index.html` once the run has completed and passed QA — otherwise
a mid-run failure leaves you with no readable prior version. Update:

- Version tags: `<title>`, `.v-badge`, masthead `.meta`, `.sub`, byline, Sources heading, disclaimer
- **Change-log**: replace the whole body with one new block, framed as the delta *against the last
  published version*. Lead with the headline moves — SPR, contracted remaining, cumulative drawn,
  cushion, Brent — shown as `old → new`
- Chart arrays in the `text/plain` block: append the new label and values; revise prior points the
  file has revised; widen a y-axis bound if a value exceeds it, and lower the SPR y-min if the new
  low sits within ~5 mb of it
- Drawdown ledger: add a `<tr class="recent">` row; ensure stock, Δ and mb/day reconcile
- Narrative: abstract, §01 bullets, KEY FINDING boxes, §05 outlook (recompute contracted-remaining
  and cushion), §06 risk, and **any figure title or caption the new week's trend contradicts**
- Verification ledger and Sources: append the week's rows and citations

Recompute every derived figure across the whole document. Leave no prior-week number standing as a
current-status claim.

## Step 5 — Render charts and PDF

Render four PNGs with matplotlib (palette: accent `#7c1d12`, green `#1f3a2e`, gold `#9a6a1e`,
blue `#274b6d`, grey `#9c8f7a`, grid `#d8cdb8`), axis bounds matching the HTML, gaps for nulls
(`spanGaps` off). X ticks must always include the final date and never let the last two labels
collide. Replace each `<img id="cimg-…">` src by id.

Then WeasyPrint with this CSS injected before `</head>`:

```css
.chart-box{height:auto!important;position:static!important}
.cimg{width:100%!important;height:auto!important;display:block}
.drop::first-letter{float:none!important;font-size:1em!important;margin:0!important;color:inherit!important}
h2,.sub{break-after:avoid;break-inside:avoid}
h3{break-after:avoid}
p{orphans:2;widows:2}
figure,table,.key,blockquote,.ledger .row{break-inside:avoid}
.quote-grid{display:block!important;break-inside:auto!important;margin-top:12px!important}
.quote-card{margin-bottom:18px;break-inside:avoid}
.keep{break-inside:auto!important}
```

Two WeasyPrint traps, both encountered in practice:

- the drop-cap float triggers an assertion — disable it (above)
- a CSS **grid** containing a tall child raises `AssertionError: assert not page_is_empty`.
  Overriding `.quote-grid{display:block}` fixes it. If a new assertion appears, look for a grid or
  a `break-inside:avoid` wrapper taller than one page.

## Step 6 — QA, every time

- [ ] Pages > 0; no thin pages; no section header stranded near the foot of a page
- [ ] For every chart: `labels.length == data.length`
- [ ] **Every plotted point cross-checked against the xlsx** — this should return zero mismatches
- [ ] Ledger arithmetic reconciles (stock differences equal the stated Δ)
- [ ] Stale-value scan: grep the prior version's headline numbers and confirm none survives as a
      current-status claim. Distinguish legitimate historical progressions from stale claims
- [ ] Superlatives are computed, not assumed ("largest weekly fall", "widest since…")
- [ ] Render the figure pages to PNG and **look at them**

Fix, re-render, re-scan until clean.

## Step 7 — Sync and publish

**Output contract: exactly two files per run. Nothing else.**

- **`index.html`** — replaces the previous one, always that exact name (ready to upload to GitHub)
- **`signaling-reserve-vX.Y.pdf`** — new file each week, version in the name, never overwritten

No run notes, no review memos, no CSV exports or copies, no scratch files, no versioned HTML
alongside `index.html`. If the run cannot publish, say so in the conversation — do not leave a note
behind in the folder.

- Save both into this folder
- If mirroring to a git repo, commit as `vX.Y — <one-line theme>` and push
- Circulate only after a human has read the narrative and news framing

## Rollback

Reverting is: delete the new `signaling-reserve-vX.Y.pdf`, and restore `index.html` to the prior
version. Because `index.html` is overwritten rather than versioned, **keep the previous copy until
the new one is signed off** — the PDF archive alone will not rebuild it. Nothing else needs
unwinding, since the workbook is an input and is never written to.
