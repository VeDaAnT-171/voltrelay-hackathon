# VoltRelay — 3-Minute Presentation Script (word-for-word, timestamped)
*Audience: VoltRelay Operations & Customer Experience leadership. ~460 words ≈ 155 wpm.*
*On-screen cues in [brackets] reference the notebook's figures.*

---

**[0:00 – 0:25] · 1. The Business Problem**

[Slide: growth chart, fig 03 top panel]

"In eighteen months, VoltRelay nearly tripled — from 103 thousand swaps a month to 303 thousand, and from 62 lakh to nearly 2 crore in monthly revenue. So why are we here? Because after energy, battery wear, and station costs, **we still lose money on every single swap** — about ₹3 per swap even in our best month. You've asked which of four budget options fixes that. The data gives a clear, sequenced answer."

**[0:25 – 0:50] · 2. The Analytical Approach**

[Slide: pipeline diagram + hygiene table]

"We rebuilt the P&L swap-by-swap from 3.88 million events in DuckDB. Before trusting anything, we ran a nine-rule hygiene audit — corrected 139 thousand timestamps from a firmware bug, removed test stations and retry bursts, and re-read six thousand Hinglish support tickets that were mislabelled. And we costed battery wear from each pack's lifetime degradation — not noisy per-trip sensor readings, which overstate wear six-fold. Then we tested the pricing pilot with a difference-in-differences design and churn with an exposure-adjusted model."

**[0:50 – 2:00] · 3. The Most Important Insights**

[Slide: Lorenz curve, fig 04]

"Three findings matter most. **First: our failures are concentrated.** Twenty percent of stations produce forty-two percent of all 228 thousand failed swaps. Three-wheeler riders fail at eleven percent in meal peaks — because 83 of our 150 stations carry zero 3W slots. And on heat-alert days charging slows from 75 to 102 minutes and stockouts triple."

[Slide: Kyron decay, fig 09]

"**Second: the Kyron battery failure cost us twice.** Lots 2407 to 2409 hit end-of-life in under ten months — we wrote off 1,438 packs, 4.8 crore, a fifth of our fleet investment. Before dying, those packs cut rider range from 68 to 38 kilometres — and 6,000 'scooter weak lag raha hai' tickets were sitting mislabelled as 'other'."

[Slide: partner margin bars, fig 11]

"**Third: our biggest partner is our worst contract.** The day ZipDrop's discount went from twelve to twenty-eight percent, its margin flipped from plus ₹5.7 to minus ₹7.7 per swap. And ZipDrop is exempt from the peak surcharge — exempt partners ran sixty-four percent of their swaps in peak hours and free-rode 17 lakh of surcharges. The pilot also proved peak pricing doesn't shift demand — peak share moved just 2.5 points. It's revenue, not load-shifting."

**[2:00 – 3:00] · 4. Actionable Recommendations**

[Slide: 4-option decision matrix]

"So, the four options. **Batteries first** — 4.8 crore, dual-sourced, with 3W buffers, because stockouts are also our number-one churn driver: a new rider who hits an empty cabinet in week one is one and a half times more likely to quit. Pair it with upgrading the 51 Gen-1 chargers — 92-minute charge times are why hot hubs run empty. **Second, roll out peak pricing as what it is — revenue** — about 1.6 crore a year — but only after every partner pays it. **Third, do not sign the ZipDrop exclusive.** Renegotiate: off-peak tiers, surcharge billable, thirty-day terms — that alone recovers over 65 lakh a year. **And skip blanket Wave-3** — densify only the congested commercial hubs. Same rupees; margin, reliability, and retention all point at the same fix. Thank you."

---
*Timing check: §1 ≈ 65 words, §2 ≈ 75, §3 ≈ 185, §4 ≈ 135 — total ≈ 460 words.*
