# DESIGN — gta6-twitch-ccu-forecast-01 v28

## What this is

A forecast program for make.fun competition
`gta6-twitch-ccu-forecast-01`: for each of hours 2–24 of GTA VI's
release day it outputs five quantiles (5/25/50/75/95%) of the
concurrent Twitch viewers in the "Grand Theft Auto VI" category.
Input is one case file (the target launch's hour-1 print, its
pre-release history, and 38 comparable past launches); output is
one JSON document per case. Python 3.13, standard library only,
no network access. Score is pinball loss on the log scale (lower
is better); practice answers are SullyGnome "Average viewers".

## The packaged cell

Median source per band: early = Y1, mid = 0.5·Y5 + 0.5·clone,
late = median(Y5, Z0, Y1); early ramp (0.00 → +0.02), mid ramp
(0.00 → −0.095), late ramp (0.00 → +0.05); size shift
−0.015 · max(0, lnV1 − 11.5), all bands; **disagreement shift
mu += 0.25 · sd(member medians)**; spread 0.975 · (0.45·sig(Y5) +
0.35·sig(Z0) + 0.20·sig(clone)), disagreement factor
1 + 0.80·sd(member medians), per-band asymmetric scales early
(0.90, 1.10), mid (1.10, 0.80), late (0.95, 0.85), late spread
ramp 0.975 → 0.90 across hours 15–24, spread cap 0.315 on the
final lower/upper spreads.

Direct-graded (exact RULES scorer, twice, deterministic):
practice **0.119641** (p00 0.109181 / p01 0.134558 / p02 0.079478 /
p03 0.155345), LOO-large **0.209943**, LOO-mega **0.211660**.
Median-only: practice **0.187966**, LOO-large **0.318543**.

## Why this cell — the round in seven rounds (107 cells)

Bar: v27, verified (practice 0.120587, LOO-large 0.210078,
LOO-mega 0.212041, median-only 0.189177 / 0.319158).

**Round 1 — fine-tuning around v27 (15 cells, 2 pass).**
Cap 0.30 + late ramp →0.92 has the best practice (0.120453) but
LOO fails; cap 0.31 + late →0.92 misses practice by +0.000008.
Early 0.02 + k=0.7 → 0.120482 / 0.210059 / 0.212017 and mid
−0.085 + k=0.7 → 0.120509 / 0.210014 / 0.211999 pass. The
neighbourhood is tight: every knob trades practice for LOO at
roughly the same rate, except the ones that follow.

**Round 2 — median levers (12 cells, 2 pass).** The breakthrough:
**disagreement-shifted median** mu += a·sd(member medians).
a=+0.1 → 0.120336 / 0.210024 / 0.211861; a=+0.2 → **0.120090 /
0.210032 / 0.211747**. Negative shifts hurt both gates
(−0.1: 0.120846; −0.2: 0.121138), so the direction is real: when
the members disagree, the median sits systematically LOW. (A
plausible mechanism: the member medians are themselves medians
of skewed per-launch distributions, and the disagreement is
one-sided — the high members are further from the consensus than
the low ones.) Per-band lnv1 slope deltas: all fail LOO.
Late-band half-blend weight 0.4 buys practice (0.120427) but
fails LOO; 0.6 fails both. Median-only of the round-1 best rule:
0.188960 practice — the median moved.

**Round 3 — wild ideas (15 cells, 4 pass).** Size-indexed cap
(cap loosens for big launches, the v27 warning made concrete):
all fail LOO — loosening the cap for big launches costs LOO more
than it buys practice, so the cap's LOO cost at 0.30 is real and
the v27 warning cuts the other way too: the cap is a LOO
liability at the low end, not just a practice asset. Asymmetric
cap, per-band cap, disagreement-indexed cap: all fail LOO.
Deepening the shift: a=0.15 → 0.120210; a=0.25 → **0.119984 /
0.210062 / 0.211719**; a=0.30 → practice 0.119884 but LOO-large
fails (+0.000034). a=0.25 + early 0.02 + k=0.7 → **0.119888 /
0.210055 / 0.211708** — the first sub-0.120 cell.

**Round 4 — refine the shift cell (13 cells, 10 pass).** Shift
grid: practice improves monotonically with a (0.119949 at 0.22
→ 0.119850 at 0.27) while LOO-large degrades (margin −0.000002
at 0.27 — essentially zero). LOO-buyers that work with the
shift: late ramp →0.92 (0.120017 / 0.209844 / 0.211520), k=0.75
(0.119897 / 0.209989 / 0.211642), k=0.8 (0.119906 / 0.209924 /
0.211577), cap 0.32 (0.119996 / 0.209978 / 0.211632), size cap
(0.119907 / 0.210021 / 0.211672). Asymmetric disagreement
(0.6, 0.75) and (0.55, 0.7) FAIL LOO now — v27's asymmetric
finding does not survive the shift. Early 0.025: practice
0.119796 but LOO fails.

**Round 5 — shift + LOO headroom (12 cells, 12 pass).** With the
late →0.92 / k=0.75 headroom the shift goes deeper: a=0.30 +
late →0.92 + k=0.75 → 0.119930 / 0.209830 / 0.211444. Best
practice of the round: **shift 0.25 + mid −0.085 + k=0.75 →
0.119806 / 0.209991 / 0.211665**. Median-only of the shift-0.25
rule: 0.188525 / 0.318774 — the median moved again.

**Round 6 — push the new best (13 cells, 12 pass).** Shift
0.30 + mid −0.085 → 0.119705 / 0.210043; shift 0.35 → practice
0.119617 but LOO fails. Mid ramp deeper: −0.09 → 0.119719 /
0.209998; **−0.095 → 0.119632 / 0.210008 / 0.211725** (round
best). The mid ramp buys practice through p02/p03 (0.0794 /
0.1553) while p01 worsens (0.1346) — p01 is the practice case
closest in size to a typical LOO-large launch, so the ramp is
still a within-band slope correction fitted to four practice
cases, and its LOO-large margin at the best cell is only
−0.000070.

**Round 7 — final check (8 cells, 8 pass).** Shift 0.25 +
mid −0.10 → 0.119545 (best practice, LOO-large margin
−0.000060); shift 0.25 + mid −0.095 + early 0.025 → 0.119539
but LOO-large margin −0.000006 — rejected, a margin that thin
is selection noise, the same trap as v26's −0.08 cell. Shift
0.27 + mid −0.095 + late →0.92 → 0.119720 / 0.209816 /
0.211531 (LOO margins −0.000262 / −0.000510) — the robust
alternate. **Packaged: shift 0.25 + mid −0.095 + k=0.8 →
0.119641 / 0.209943 / 0.211660** — the best practice among
cells with LOO-large margin at least −0.0001 (precedent: v26
packaged −0.000134/−0.000090, v27 packaged −0.000124/−0.000178);
it costs +0.000096 of practice vs the best cell for 2.2× the
LOO-large margin (−0.000135) and a mega margin of −0.000381.

## Honest caveat (read before trusting the number)

The band parameters (ramps, slope, pivot, base scale, k, cap,
shift) were chosen on LOO-large AND on practice across seven
rounds of selection, so NEITHER gate is a clean holdout for the
packaged cell — the LOO-large margin (−0.000135) and mega margin
(−0.000381) are thin and optimistically biased by selection,
and the practice margin (−0.000946 vs v27) is the cleaner but
still selected evidence. Three specific warnings: (1) the
disagreement shift is fitted on four practice cases and 851
LOO cells, but its mechanism (median sits low when members
disagree) was never derived from first principles — it is an
empirical regularity, and the final launch is a new regime
(lnV1 ≈ 14+, beyond the library max 13.68) where member
disagreement will be at its largest; the shift's effect there
is an extrapolation, not a measurement; (2) the mid ramp at
−0.095 is the steepest ever packaged, and it buys its practice
through the two biggest practice launches while the
LOO-typical case (p01) gets monotonically worse as the ramp
steepens; (3) the neighbourhood holds together (10 of 13
round-4 cells, 12 of 12 round-5 cells, 12 of 13 round-6 cells
and 8 of 8 round-7 cells pass the v27 gates), and the per-case
gains are broad rather than one-case luck — but practice is
only 4 launches. Treat this as the best verified fallback, not
a guarantee. The robust alternate (shift 0.27 + mid −0.095 +
late ramp →0.92: 0.119720 / 0.209816 / 0.211531) is the more
LOO-robust packaging of the same idea if the LOO margins
matter more than 0.000079 of practice.

## Why 0.100 still held (twenty-eighth round)

The median moved again — 0.1892 → 0.1880 practice, 0.3192 →
0.3185 LOO-large — but the gap to 0.100 is still ~0.088 of
median loss, and per-hour OLS residual sd is still 0.35–0.45
log. The disagreement shift is the biggest median move since
v24, yet the practice number is still 0.0196 above the target.
No median family remains unbuilt or unmeasured after 28 rounds
except ideas that need data the case file does not have
(final-only streams/categories, which cannot be validated).

## Dead ends this round (kept for the record)

- Size-indexed cap, asymmetric cap, per-band cap,
  disagreement-indexed cap: all fail LOO-large. The cap is a
  LOO liability at the low end, not a free practice asset.
- Asymmetric disagreement (v27's finding): fails LOO once the
  shift is in place.
- Per-band lnv1 slope deltas, late-band half-blend weight:
  fail LOO.
- Quantile tilt (v26): spread-shrink in disguise, not an
  independent lever. Not re-tried.
- Shift a ≥ 0.30 without LOO headroom, mid ramp −0.10 with
  early 0.025: practice bests with LOO margins at noise level.
