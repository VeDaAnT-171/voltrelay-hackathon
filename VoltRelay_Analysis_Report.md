# VoltRelay Energy — Network Performance, Unit Economics & Rider Retention
**Gradient Learnings 1-Day Data Analytics Hackathon · Executive Analysis Report**
*Vedaant Gupta · IIT Kharagpur · built on DuckDB + Python over 3.88M swap events, 18 months, 6 cities*

---

## 1 · Problem Understanding

VoltRelay runs 150 battery-swap stations for 20,000 2W/3W gig riders across six Indian cities. Leadership sees top-line growth and must pick between four competing uses of next year's operating budget: more stations, more batteries, a network-wide peak-pricing rollout, or an exclusive deal with its largest fleet partner.

The data says the real question is different: **monthly swaps nearly tripled (103k → 303k) and revenue tripled (₹0.62 Cr → ₹1.97 Cr/month), yet the network still loses money on every swap after fixed costs.** Growth is not the problem; the margin and reliability engine underneath it is. Four mechanisms drive this, each traced below: (a) service failures concentrated at a small set of congested stations, with acute 3W stockouts; (b) a defective battery cohort that destroyed ₹4.79 Cr of CapEx and 44% of delivered range; (c) a partner contract (ZipDrop) amended into negative margin; and (d) a peak-pricing pilot that raises revenue but doesn't shift load — while exempt B2B partners free-ride it.

## 2 · Analytical Approach

- **All heavy computation in DuckDB** (views + window functions over the 3.88M-row event log and 1.49M-row telemetry table); only sub-10k-row aggregates ever touch Pandas. The notebook runs end-to-end on Colab's 12 GB without approaching memory limits.
- **Every cleaning rule was validated against the data before being applied**, and two turned out different from the case brief (see §3 — we document both honestly).
- Costing: energy at each station's grid tariff; **battery wear from per-pack lifetime SoH consumption** (`(initial−current)/(initial−60) × purchase cost`, allocated over the pack's swaps). We explicitly rejected a per-journey wear estimator after showing it is biased in *both* directions by SoH sensor noise and asymmetric in/out calibration (+₹185 floored vs −₹45 unfloored, against ₹37 lifetime-based — the notebook demonstrates this).
- Causal questions use design, not anecdote: difference-in-differences on a **stable station panel** for the pricing pilot (with a symmetry check on station openings), exposure-adjusted logistic regression for churn, and bounded activity windows for cohort-fair retention.

## 3 · Data Hygiene Audit (exact counts)

| # | Rule | Rows affected | Action |
|---|------|--------------:|--------|
| 1 | Test stations `STN-TST-01/02` | 62,031 events · 26,256 telemetry · 535 tickets · 2 stations | removed |
| 2 | Firmware v3.2.0 logged −5h30m, 10 Mar–14 Apr 25 | 139,490 | timestamps +5h30m; before/after hour-profile proof in notebook |
| 3 | Retry bursts ≤180 s (same rider+station+type) | 5,720 (0.15%) | kept first of burst |
| 4 | 21 `home_city` spellings → 6 canonical cities (via `home_zone` prefix) | 585 riders | rewritten |
| 5 | SoC/SoH > 100% capped; `km_since_last_swap` ≤0.5 or >150 nulled | 9,749 capped · 15,509 nulled | sensor drift / odometer resets |
| 6 | Completed vs failed attempts split | 228,285 failed (5.99%) | failure analysis stream |
| 7 | Telemetry `missing` hours (all at `poor` connectivity) | 9,384 | excluded — never coalesced to 0 (would fabricate stockouts) |
| 8 | Duplicate tickets removed; CSAT proven MNAR | 937 removed | stratified CSAT computed |
| 9 | Hinglish/English NLP re-classification of tickets | 6,568 relabelled | **6,020 hidden battery/range complaints** surfaced |

Clean analytical base: **3,809,346 events · 3,581,061 completed swaps**.

**Where the data contradicted the brief — findings we report as-is:**
- The brief's duplicate key (including battery IDs) matches **zero** rows; real retry bursts exist only on a looser key, and at the **same ~0.15% rate in realtime and offline_batch** — the claimed poor-connectivity concentration is not in this extract.
- CSAT missingness is real MNAR (response rate 43.7% for fastest-quartile resolutions vs 28.3% for slowest) but the bias **inflates** naive CSAT where service is worst — only satisfied riders answer after a slow ticket. Naive 3.557 vs stratified 3.561: direction matters more than magnitude.
- Tariff codes `PREPAID`/`PROMO_FREE` described in the brief do not occur in the extract.
- **Battery master dates are unreliable**: 983,481 cleaned completed swaps (27.5%) reference a pack *before* its recorded `commission_date` (in the raw log, 826k even precede `manufacture_date`), touching 5,144 of 6,500 packs. All age-based battery analysis uses post-commission readings only; the Kyron lots' own dates are internally consistent, so the <10-month end-of-life finding stands.

## 4 · Key Insights

### Q1 — Growth without profit
Contribution margin per swap (revenue − grid energy − battery wear): **₹8.1 (H1-24) → ₹5.6 (Q4-24) → ₹7.8 (Q2-25)**. The Q4-24 squeeze coincides with the ZipDrop discount amendment (Nov-24) and accelerating Kyron wear; H1-25 recovers on peak-surcharge revenue. After station fixed OpEx the network is **negative in all 18 months** (−₹12.7/swap Jan-24 → −₹3.3/swap Jun-25): cash-positive before battery depreciation, underwater after it. Average battery wear is ₹37/swap — the single largest cost line after discounts, and the lever the Kyron failure pulled.

### Q2 — Failures are concentrated, not endemic
- **20% of stations account for 41.8% of all 228k failures** (Lorenz curve). Baseline failure runs ~4.3%; the spikes are seasonal: 12.4% (May-24) and 10.0% (May-25), when heat-alert days slow charging from 75 → 102 minutes, quarantine 6.8× more packs, and push stockout rates from 3.1% → 10.3% of attempts.
- **3W riders fail at 10.9% during meal peaks vs 5.8% for 2W** — 83 of 150 stations have zero 3W slots; the remainder have only 4–6.
- Monsoon wet days multiply outage minutes 6× (3.8 → 22.8/day) but demand drops in rain, so stockout rates barely move: heat, not rain, is the stockout driver.

### Q3 — Equipment and expansion
- **Gen1 chargers are the thermal bottleneck**: 91.8-minute average charge vs 42.2 for Gen3, and 7.24% failure vs 4.85% — at identical fixed cost (~₹23–24k/month). Upgrading the 51 Gen1 sites beats building new ones.
- Expansion waves went to the wrong places: Wave-1/2 **commercial hubs** cover fixed cost 8.3–9.9× with revenue, while transit-hub sites cover only 2.7–3.0× and residential ~4.5×. Meanwhile failure rates at Launch-era stations kept climbing even in zones that received Wave-2 sites — new coverage didn't relieve existing congestion.
- The 10 stations with a **competitor within 1.5 km** (since 15 Jan 25) under-grew the rest of the network by ~4 index points in H1-25 — real but second-order next to the operational issues.

### Q4 — The Kyron battery failure
Lots KY-2407/08/09 (commissioned Jul–Sep 24) degraded to ~60% SoH in **under 10 months** — Cellora/Amptek packs of the same vintage sit near 80–86%. On 15 Jun 25 VoltRelay retired **1,438 packs = ₹4.79 Cr of CapEx (19.7% of total fleet investment)**. Before retirement these packs poisoned the rider experience: a pack issued at ≤72% SoH delivers **38 km vs 68 km at ≥95% (−43.8%)**, forcing extra paid swaps per shift — and the re-classified tickets show **6,020 battery/range complaints hidden** under "other"/"app_issue" ("scooter weak lag raha hai"), tracking the decay month-for-month.

### Q5 — Pricing and partners
- The **1 Jul 24 base hike** (₹60→65 2W, ₹100→110 3W) met essentially inelastic demand: swaps per active rider still grew 5–7% in the following quarter across all plan types.
- The **Bengaluru+Pune peak pilot** (from 1 Oct 24; +₹15 peak, −₹8 off-peak): on a stable pre-pilot station panel, DiD shows peak-hour share moved only **−2.5 pp** — riders did not shift hours. Volume DiD is −321 swaps/station/month vs control, so the surcharge trimmed some marginal demand, but billing-dispute share did *not* rise (DiD −3.2 pp). Net: **peak pricing is a revenue instrument, not a load-shifter.**
- **The exemption is the scandal**: surcharge-exempt partners (ZipDrop, Swiggle Go) ran **64.0% of their pilot-city swaps in peak hours vs 40.5% for billable partners** — 116,116 un-surcharged peak swaps ≈ **₹17.4 L** given away in the pilot alone.
- **Partner economics** (CM/swap after energy, wear, and working-capital cost): best — DabbaXpress ₹9.5, UrbanErrand ₹9.8, RideMitra ₹9.3, QuickCart ₹9.0, FeastFly ₹8.0. Worst — **ZipDrop −₹3.3**, CargoTuk −₹6.5, HaulKing −₹6.0 (the 3W-cargo segment is loss-making per swap on 4.8 kWh pack wear). ZipDrop is the volume leader (548k swaps) and the margin floor: its realized discount jumped **12.0% → 28.0% exactly at the 1 Nov 24 amendment**, flipping CM/swap from **+₹5.7 to −₹7.7**, compounded by 45-day payment terms and the peak exemption.

### Q6 — Retention: experience, not vintage
With cohort-fair bounded windows (active in days 31–60), **D30 retention is stable at ~84–86% across cohorts** — there is no vintage collapse; the brief's assumed "slide" disappears once left-censored pre-2024 signups (3,600 riders) and the saturating "ever-active" definition are corrected. The churn risk lives **within** cohorts: exposure-adjusted logistic regression on each rider's first 14 days (n≈16k, base inactivity rate 15.2%) ranks, per 1 SD:
1. **Early stockout/abandon rate — OR 1.46** (riders with ≥1 early failure churn at 17.3% vs 13.4%)
2. **Partner-billed plan — OR 1.35** (retention outsourced to partner dispatchers)
3. Degraded-pack exposure (<72% SoH) — OR 1.09
4. Price-shock cohort (PAYG post-hike) — OR 1.04; queue wait 1.01; competitor/PEAK ≈ 1.0 (noise)

The product fix for retention is the same as the ops fix: **don't let a new rider hit an empty cabinet in week one.**

## 5 · The Budget Decision Roadmap

| Option | Evidence | Verdict |
|---|---|---|
| **A. Blanket Wave-3 stations** | New-site revenue covers 8.3× fixed cost at commercial hubs but 3.0× at transit hubs; 42% of failures sit at 20% of *existing* stations — capacity, not coverage, binds | **No as blanket.** Densify only top-decile congested commercial/market zones |
| **B. More batteries** | ₹4.8 Cr replaces the retired Kyron fleet (dual-sourced Cellora/Amptek, 3W buffers at the 83 zero-slot sites). Direct margin from converting ~60% of 11.6k monthly stockouts is small (~₹0.07 Cr/yr at ₹7.8 CM), **but** stockouts carry the top churn odds (1.46) and the Apr–Jun 25 failure spike (6.4→10.0%) is the shortage bleeding already | **Yes — first priority**, paired with Gen1 charger upgrades at hot hubs (92→42 min) so new packs aren't throttled |
| **C. Network-wide peak pricing** | Pilot proves no load-shifting (−2.5 pp) → treat as revenue: ~**₹1.6 Cr/yr** at ₹15 on billable peak swaps (10% attrition haircut). Exemption leak already ₹17.4 L in-pilot | **Yes, after** making the surcharge billable for all B2B and funding a genuine off-peak rebate |
| **D. ZipDrop exclusive** | CM −₹3.3/swap vs +₹5.2 median partner → ₹12.9/swap gap ≈ **₹0.65 Cr/yr** foregone at current volume, before the exemption leak and 45-day working capital | **No at current terms.** Renegotiate: off-peak volume tiers, surcharge billable, ≤30-day terms — *then* talk exclusivity |

**Sequenced recommendation:** B (batteries + Gen1 upgrades at the top-20% failure stations) → C (rollout with exemptions closed) funds itself → renegotiate D → A only as targeted densification. This attacks margin (₹37/swap wear line, ₹17.4L+ leakage), reliability (41.8% failure concentration), and retention (OR 1.46) with the same rupees.

## 6 · Honest Limitations
- Simulated-style extract; wear model assumes linear value consumption to a 60% SoH floor (matches the observed retirement threshold).
- DiD assumes parallel trends; station-opening symmetry was verified (33% base-growth in both arms), but city-level shocks can't be fully excluded.
- Churn model is observational and exposure-adjusted — associations, not randomized causes.
- CSAT corrections re-weight observed quartiles; truly unobservable non-response bias remains.

*Every figure in this report is produced by `VoltRelay_Hackathon_Analysis.ipynb` and stored in `key_numbers.json`; the notebook re-runs end-to-end on the raw CSVs.*
