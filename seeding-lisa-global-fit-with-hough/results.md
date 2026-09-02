---
description: Spritz, Sangria, and the PSD ablation
---

# Results

## Sangria training — MBHB, full year

23 chunks of 30.3 d stepping by half a chunk, threshold FAP < 10⁻⁶.

| | |
|---|---:|
| chunks flagged | 12 |
| **false alarms** | **0** |
| catalogue sources matched | **9/15** |
| mass ratios | ×0.71 – ×1.12 |
| merger-free chunk FAP | 2.9×10⁻² … 2.0×10⁻¹ |
| runtime, full year | ~4 s |

| truth Mc | tc [d] | ratio | Δtc [h] | FAP |
|---:|---:|---:|---:|---:|
| 7.82e5 | 55.6 | 0.800 | +8.8 | 1.0e-28 |
| 7.73e5 | 133.4 | **1.005** | +1.3 | 7.2e-43 |
| 2.73e6 | 157.6 | 0.891 | +8.9 | 5.0e-09 |
| 2.50e6 | 215.3 | 0.934 | +4.6 | 2.9e-09 |
| 1.11e6 | 236.4 | 0.806 | +7.2 | 3.7e-25 |
| 8.57e5 | 257.3 | 0.924 | +5.7 | 1.0e-21 |
| 1.97e6 | 271.3 | 1.117 | +3.2 | 7.6e-07 |
| 1.03e6 | 282.5 | 0.990 | +3.2 | 3.5e-08 |
| 3.87e6 | 341.6 | 0.707 | +10.9 | 5.9e-09 |

Missed: 101.2, 129.3, 130.3, 138.6, 191.3, 199.6 d.

Two things to note. **Δtc is systematically positive** (+1.3 to +10.9 h) — most likely the
Newtonian track approximating IMRPhenomD, but unexplained, and it biases every seed. And
**Nobili et al. report 15/15 on this same data** with ~10 false detections per year at the same
threshold; we are at 9/15 with 0. Different operating points, but they are ahead.

## Whitening quality drives everything

![before](../.gitbook/assets/sangria-before.png)
![after](../.gitbook/assets/sangria-after.png)

Top row: a chunk holding a merger. Bottom row: a merger-free chunk. Columns: whitened ρ → soft
weight → `(Mc, tc)` accumulator → log FAP.

The only difference between the figures is the PSD's frequency resolution. In the first, bright
horizontal stripes — **resolved galactic binaries** — survive whitening; because a high-mass
chirp is nearly flat, high-Mc templates lie along those lines and integrate their power, so the
accumulator peaks in the high-Mc corner and a merger-free chunk returns FAP 10⁻²³. In the
second the lines are absorbed, the chirp is the brightest feature in the weight map, the
accumulator peaks on the truth, and the same empty chunk returns 10⁻².

## Spritz — MBHB through glitches and gaps

31 d, merger 30.4 d in, 14 h of gaps, 3 injected glitches (one 0.4 d before merger).

| | mbhb1 | mbhb2 |
|---|---|---|
| recovered Mc | **×1.005** | best FAP 4×10⁻² |
| Δtc | +0.43 h | — |
| FAP | 9×10⁻¹⁹ | **correctly insignificant** |

mbhb2 has ~45× lower signal rms; reporting "nothing here" is the right answer.

![spritz track](../.gitbook/assets/spritz-mbhb1-track.png)

## The Mc–tc degeneracy

![degeneracy](../.gitbook/assets/mbhb1-degeneracy.png)

The score surface is a tilted ridge — Mc trades against tc. Within 80% of the peak,
Mc/Mc_true ∈ [1.00, 1.37] and tc ∈ [−10, +12] h; truth sits inside the 80% contour but outside
95%. Letting the track exponent float shows the score is flat from p = 0.34 to 0.45 with
amplitude compensating; that flat direction *is* the degeneracy.

Two consequences. **Report a region, never the arg-max** — a point estimate is misleading as a
global-fit initialisation. And **separate candidates by track overlap, not parameter distance**:
two distant `(Mc, tc)` pairs can trace nearly the same curve and are one source.

The GFH linearisation should straighten this ridge, which is one reason it is worth doing.

## PSD ablation

Search held fixed — same transform, family, null, threshold, statistic — varying only `S(t,f)`.
The FAP is common-mode, so residual mis-calibration cancels in the comparison.

| arm | sources | median ρ | Mc ratio | **Mc scatter** |
|---|---:|---:|---:|---:|
| `median_block` (robust, in situ) | 9/15 | 1.362 | 0.882 | **0.104** |
| `offset_chunk` (Nobili Eq. 6) | 8/15 | 1.403 | 0.881 | 0.192 |
| `global_median` | 10/15 | 1.451 | 0.878 | 0.259 |
| `spline` (non-robust) | 10/15 | **0.976** | 0.959 | 0.136 |

χ²₂ median is 1.386; below it means the PSD absorbed signal.

**Detection count does not discriminate here and should not be quoted as if it does** — 18 of 23
chunks contain a merger, so a trigger-happy arm scores well by chance, and per-source recovery
is 8–10/15 in every arm including the naive global median.

**Parameter accuracy discriminates**, and it is the seeding deliverable: the robust in-situ
estimator gives the tightest chirp mass, by 1.3× (vs spline) to 2.5× (vs global median).

The spline arm is a clean demonstration of signal absorption on real data — median ρ 0.976 vs
1.386. It still detects, because uniform suppression scales signal and null together, but its
whitening is wrong and its masses are worse.

**Caveat that bounds the claim.** Sangria's MBHBs are transient, so the offset-chunk PSD is
legitimate here — this is the case *least* favourable to the argument, and it still wins on
parameter accuracy. The always-on demonstration, where an offset chunk has no signal-free epoch
to use, has not been run.

## Galactic binaries — diagnosis only

A coherent year-long periodogram finds **0/36** verification binaries. Not a search failure: the
annual Doppler spreads each source over tens of `1/T` bins (HMCnc 48 peak-to-peak, predicted 39).
In-segment ridge SNR is ~2600, so the signal is there.

`pm.remove_doppler`, `f/(1 + n·v/c)`, transfers directly. `get_detector_velocities` does not —
it uses a ground-IFO ephemeris, i.e. Earth orbit **plus sidereal rotation**; LISA has the
heliocentric orbit (v/c = 9.94×10⁻⁵) and no sidereal term. With a LISA ephemeris, barycentric
resampling then a coherent year-long FFT:

| source | raw width | demodulated | peak gain |
|---|---:|---:|---:|
| HMCnc | 9 bins | 2 | ×5.7 |
| ZTFJ1539 | 4 | 2 | ×2.5 |
| ZTFJ2243 | 6 | 2 | ×4.5 |

The fitted orbital phase agrees across four independent sources (2.76–3.27), which confirms the
ephemeris rather than a per-source fudge.
