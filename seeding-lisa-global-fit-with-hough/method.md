---
description: How the pipeline works and why each piece is there
---

# Method

`TDI → A/E → WDM → robust S(t,f) → whiten → soft weight → sum along track family → Gamma FAP`

Incoherent throughout: power only, no phase, no waveform generation.

## Soft vs hard voting

* Classical FH: threshold power → peak-gram → count 0/1 along a track.
* Soft: weight each pixel by how far above the local mean it sits. No threshold.
  ([gist](https://gist.github.com/avivajpeyi/ae6507b881d19641fa41cf99779c0fd5))
* Factorial (STFT×WDM, hard×soft, matched grids): **soft = −0.007 to −0.010 in A50**, transform ≈ 0.
* ⚠️ **Not representation-neutral yet.** WDM real coeff = 1 dof, STFT complex bin = 2 dof, so the
  same function of raw power means different things. Fair version:
  `u_p = −log P₀(R ≥ R_p)`, `H = Σ a_p g(u_p)`, `P₀` = local empirical null CDF.
  Our `ρ − dof` is a crude stand-in. **Cheapest high-value fix outstanding.**

### Clipping is regime-dependent, not a default

| max ρ in map | unclipped | clip 20 | clip 5 |
|---:|---:|---:|---:|
| 7e2 | **0.37** | 0.18 | 0.23 |
| 1.4e4 | 0.05 | 0.22 | **0.27** |
| 1.2e5 | 0.00 | 0.00 | 0.02 |

(recovery within 1 bin.) Below ~1e3 unbounded wins, above ~1e4 clipping wins, past ~1e5 nothing
works. Spritz glitches hit ~1e10 → clipping essential there, harmful on clean data.

## Why WDM

* Detection benefit over STFT: **zero** (slightly negative). Don't claim it.
* Real reason: orthonormal + critically sampled → pixels uncorrelated → `√support` correct
  **by construction**. STFT at 75% overlap → effective independent count ≈ ¼ of pixel count,
  by a *template-dependent* amount → scores not comparable across a family.
* Tiling: `nt = 2·duration·√ḟ_max` (WDM version of `TFFT ≈ 1/√ḟ_max`). Inheriting `nt` once cost
  a factor 4 in resolution.
* ⚠️ One tiling can't serve a year-long GB *and* a near-vertical MBHB. Think small bank:
  `{fine-f, balanced, fine-t}`, cluster candidates across them.

## Robust S(t,f) — the part that's ours

`MedianBlockPSD`: block quantile in coarse (t,f) tiles ÷ `chi2.ppf(q,dof)`, log-bilinear back to
the grid. ~40 lines, no optimizer.

* Contamination is **one-sided** (signals only add power) → median isn't special; `q ≈ 0.4`
  cancels the bias at ~6% tile occupancy for ~20% more scatter.
* **Failure mode it fixes: the signal whitening itself away.** Strong track → Ŝ ↑ → its own
  significance ↓. Three admissible cures: mask around provisional tracks / robust quantile /
  cross-fitting. We use robust quantile.
* Spline absorbs signal **without bound** (in-track bias → +1.35). Block quantile **saturates**
  (+0.14, amplitude-independent) — 50% breakdown point.
* On real Sangria the spline's median ρ = **0.976** vs χ²₂ = 1.386 → 42% PSD over-estimate.
* ⚠️ `n_frequency_blocks` must be ≤ number of WDM channels, or blocks are empty, get
  interpolated across, and **resolved GBs never get whitened away**. An unabsorbed GB line is
  horizontal; a high-mass chirp is nearly flat → lines masquerade as flat high-Mc tracks and
  drag the accumulator into the high-Mc corner. Same problem as lines contaminating an inspiral
  search in LIGO; same fix.

## Track families + the GFH connection

`TrackFamily` gives `f(t; θ)` on a grid. New source class = new family, search untouched.

**GFH does it better.** `x = f^(-(n-1))` makes a power-law chirp a *straight line*, so the search
becomes a standard linear Hough in `(x₀, k)`. Newtonian inspiral `f ∝ (tc−t)^(−3/8)` → `n = 11/3`:

```
x(t) = A(tc − t),   A = π^(8/3)·(256/5)·(G·Mc/c³)^(5/3)
k  = −A     → Mc = (−k / (π^(8/3)·256/5))^(3/5)
x₀ = A·tc   → tc = −x₀/k
```

**Verified to 1e-15** against our family: slope ratio 1.00000000, tc to 6 dp, Mc exact.

Buys three things we lack: shift-invariance in `x₀` (one `bincount` scores every intercept — the
classical speed trick), a uniform grid, and a **straight** degeneracy ridge instead of a curved one.

## Statistic + significance

* Score = **chirp-slice** sum (band between neighbouring tracks), via prefix sums → O(1) per
  template-time.
* Per-template **Gamma FAP**, method of moments — shape/scale in closed form from null mean and
  variance, no optimizer. Calibrated 0.499 / 0.102 / 0.0091 at the 50/90/99th null percentiles.
* Gamma not z-score: statistic is right-skewed, so the same z sits at very different tail
  probabilities depending on slice size.

Two hard-won requirements on the null:

1. **Realizations must be independent.** Shifting/reversing one segment gives the spread across
   shifts of *one* realization → variance badly under-estimated. Use the other chunks,
   time-reversed (a rising chirp becomes falling, matches nothing).
2. **Smooth the moments against support.** Fitting 2 params per template for ~1e5 templates from
   a few dozen draws then taking an extremum = winner's curse, not detection.

## Speed

* JAX with model + binning + gather all inside one `jit`. Keeping the model outside makes the
  "fast" path *slower* than NumPy.
* 2.5–3.8× over NumPy on CPU, unchanged on GPU. Full-year Sangria: **~4 s**.
