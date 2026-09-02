---
description: WDM, robust S(t,f), soft weighting, track families, and the Gamma FAP
---

# Method

**TDI → A/E → WDM → robust `S(t,f)` → whiten → soft weight → sum along a track family → Gamma FAP.**

Everything is incoherent: power only, no phase, no waveform generation. That is what keeps it
cheap, and it is the line between this and a matched filter.

## Soft rather than hard voting

The classical Frequency-Hough thresholds the normalised power into a peak-gram and counts
0/1 peaks along each candidate track. The soft variant computes the local mean power and
weights each pixel smoothly by how far above it sits — no threshold.
([gist](https://gist.github.com/avivajpeyi/ae6507b881d19641fa41cf99779c0fd5))

In a controlled factorial (STFT vs WDM × hard vs soft, matched grids, held-out thresholds) the
soft weighting was the single largest sensitivity gain, **−0.007 to −0.010 in A50**, while the
transform choice contributed nothing. The gain comes from keeping the amplitude information a
0/1 vote discards.

The weight is bounded: `w = min(ρ − dof, clip)`. Clipping is **regime-dependent, not a
default**, and the crossover is measurable (recovery within one bin, fixed signal):

| max ρ in the map | unclipped | clip 20 | clip 5 |
|---:|---:|---:|---:|
| 7×10² | **0.37** | 0.18 | 0.23 |
| 1.4×10⁴ | 0.05 | 0.22 | **0.27** |
| 1.2×10⁵ | 0.00 | 0.00 | 0.02 |

Below ~10³ the unbounded weight wins; above ~10⁴ clipping wins decisively; beyond ~10⁵ nothing
works. Spritz glitches reach ~10¹⁰, so clipping is essential there and harmful on clean data.
Measure the outlier severity, set the clip from it.

## Why WDM, and why not for the obvious reason

The measured detection benefit of WDM over STFT is **zero** (slightly negative). It earns its
place on **normalisation correctness**: the WDM is critically sampled and orthonormal, so its
pixels are uncorrelated for white noise and the `√support` denominator is right *by
construction*. An STFT at 75% overlap shares three quarters of each segment with its neighbour,
so the effective independent count is ~¼ of the pixel count — and by a template-dependent
amount, which makes scores incomparable across a family.

Grid sizing follows the same drift criterion as `TFFT ≈ 1/√ḟ_max`:
`nt = 2·duration·√ḟ_max`, rounded to a power of two. Inheriting `nt` from elsewhere cost a
factor of 4 in resolution in an early study, which alone explained an apparent deficit against
the STFT baseline.

## Robust `S(t,f)` — the piece that is ours

`MedianBlockPSD`: block quantile of the power in coarse (time × frequency) tiles, divided by
`chi2.ppf(q, dof)` to undo the quantile bias, then log-bilinear interpolation back to the grid.
No optimizer, no warm start, ~40 lines.

Contamination from signals is **one-sided** — sources only add power — so the median is not
special; a lower quantile is strictly more robust, and `q ≈ 0.4` cancels the contamination bias
at ~6% tile occupancy for ~20% more scatter.

The property that matters: a smooth spline **absorbs signal without bound** (in-track log-PSD
bias grows to +1.35), while a block quantile **saturates** (+0.14, independent of amplitude)
because the median has a 50% breakdown point. On real Sangria data the spline's median ρ is
0.976 against the χ²₂ value of 1.386 — a 42% PSD over-estimate.

**Resolution matters as much as robustness.** `n_frequency_blocks` must not exceed the number of
WDM channels, or most blocks are empty and get interpolated across, and the PSD can no longer
track *resolved galactic binaries*. An unabsorbed GB line is horizontal in the time-frequency
plane, and a high-mass chirp is nearly flat through its early inspiral — so unwhitened lines
masquerade as flat, high-mass tracks and drive the accumulator into the high-Mc corner. This is
the same problem as an inspiral search contaminated by instrumental lines in LIGO, and the same
answer: remove the lines first.

## Track families and the GFH connection

A `TrackFamily` supplies `f(t; θ)` for a parameter grid; adding a source class means adding a
family, not touching the search. `NewtonianChirp` uses `(Mc, tc)`; `LinearTrack` uses
`(f0, ḟ)`.

The Generalized Frequency-Hough does something better. Transforming `x = f^{-(n-1)}` makes a
power-law chirp a **straight line** in `(t, x)`, and the search becomes a standard *linear*
Hough in `(x₀, k)`. For a Newtonian inspiral `f ∝ (tc − t)^{-3/8}`, so `n = 11/3` and

```
x(t) = A·(tc − t),   A = π^(8/3)·(256/5)·(G·Mc/c³)^(5/3)
k  = −A      → Mc = (−k / (π^(8/3)·256/5))^(3/5)
x₀ = A·tc    → tc = −x₀ / k
```

**Verified against our family to 1×10⁻¹⁵**: fitted slope / analytic = 1.00000000, `tc` to six
decimals, `Mc` exact. The linearisation buys three things we currently lack — shift-invariance
in `x₀` (so one `bincount` scores every intercept, the classical speed trick), a uniform grid,
and a straight degeneracy ridge instead of a curved one.

## Statistic and significance

Score is a **chirp-slice** sum — the band between two neighbouring tracks, not a line sampled at
the nearest bin — computed as a difference of prefix sums along frequency, so O(1) per
template-time.

Significance is a per-template **Gamma** false-alarm probability, fitted by method of moments:
shape and scale follow in closed form from the null mean and variance, so no optimizer runs.
Calibrated to 0.499 / 0.102 / 0.0091 at the 50/90/99th null percentiles. A Gamma rather than a
z-score because the statistic is right-skewed — the same z sits at very different tail
probabilities depending on how many pixels the slice holds.

Two requirements on the null, both learned the hard way:

- **Realizations must be independent.** Shifting or reversing a single segment gives the spread
  *across shifts of one realization*, which badly under-estimates the variance. On a year of
  data the honest null is the other chunks, each time-reversed so any chirp they contain becomes
  a falling track no template matches.
- **Moments must be smoothed against support.** Fitting two parameters per template for ~10⁵
  templates from a few dozen draws, then taking an extremum, is a winner's curse rather than a
  detection.

## Speed

JAX, with the track model, binning and gather all traced together — keeping the model outside
`jit` makes the "fast" path slower than NumPy. 2.5–3.8× over NumPy on CPU, unchanged on GPU.
Full-year Sangria: 23 chunks × 32k templates plus nulls in **~4 s**.
