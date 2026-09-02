---
description: What's next, in order, and what to skip
---

# Open questions

## Two gaps that outrank everything

**Search-level FAP.** We compute per-template `P₀(H(θ) > h)`. What matters is
`P₀(max over the grid > h)` — and trial counting is unreliable because neighbouring templates
correlate. Fix: calibrate the **loudest-cluster** distribution on noise-only runs, take an
empirical quantile. Until then our FAPs aren't comparable to published ones, and the
catalogue-level number should be an FDR.

**The metric that decides if any of this matters.** Not ROC. Same refinement from a broad prior
vs from the candidate:

```
G_L = N_likelihood(broad) / N_likelihood(candidate)
G_T = T(broad) / (T(candidate) + T_search)
```

`G_T` includes search cost so an expensive finder can't hide outside the budget. **Nothing here
measures either yet.**

## Ranked

1. **Always-on demo.** The whole claim rests on it and it's not run. Sangria MBHBs are
   transient → offset-chunk PSD is fine there. Inject an EMRI (or use VGBs) so there's nothing
   to offset to, rerun the ablation. If the contrast isn't large, rethink the claim.
2. **GFH linearisation.** Mapping verified exact (1e-15); untested is whether the transformed
   search returns the same candidates. Run `x = f^(-8/3)` through the existing *linear*
   machinery, compare to the direct `(Mc,tc)` scan. Same → confirms both + gives the fast path.
   Different → localises a bug in one.
   * ❓ for Andrew: `gridx` in `LongT_GENERALIZED_fasthough_nonuni` is `1/f^(n-1)` on a uniform
     f-grid. Carries over to WDM fine — but how does that non-uniform x-binning interact with a
     **soft** (non-0/1) weight? Does the shift-invariant `bincount` survive?
3. **Cross-window coherence.** Nobili's biggest edge — accumulate across overlapping chunks
   instead of deciding per chunk, connected-region labelling on the FAP map. Several of our 6
   misses are marginal in one window, probably fine across three.
4. **The systematic Δtc** (+1.3 to +10.9 h). Likely Newtonian vs IMRPhenomD. Test with a
   higher-order PN or IMRPhenomD frequency evolution as a `TrackFamily`.
5. **Glitch excision.** Glitches = broadband **vertical** stripes (sparse in time); GB lines =
   horizontal (sparse in frequency). Flag outlier time columns, excise as gaps, reuse existing
   gap machinery. Cheap, independent.

## Sequencing

Make each layer justify itself before building the next:

1. Does *calibrated* soft weighting help?
2. Does WDM help once data are nonstationary + gapped?
3. Does phase add anything **after** search-level FAP calibration?
4. Do candidates materially speed up real waveform refinement?
5. Can their WDM footprints factor the global source problem?

Guards against building an elegant architecture before showing the candidate layer saves the
expensive compute.

## Phase experiment, if it happens

WDM coeffs are real, but the transform builds a **complex pre-Wilson field** and throws away one
quadrature at the final projection. Expose it, score pairwise phase consistency along a track.

* Treat as a **candidate reranker, not a likelihood** — the complex field is redundant against
  the real Wilson basis, non-trivial covariance.
* Prior art: loosely coherent (Dergachev), phase-relaxed F-stat (Cutler). Sell as a cheap WDM
  implementation of partial coherence, not "phase-aware Hough".

## Don't bother with

* **Chasing 15/15 on Sangria MBHBs** — someone else's contribution, we'd arrive behind.
* **Quoting detection counts from the ablation** — 18/23 chunks have a merger, doesn't
  discriminate.
* **Full `(f, ḟ, λ, β)` GB Hough** — use WDM morphology to shrink the prior, hand off to the
  F-statistic. Sky dims aren't worth brute force.
* **High-dimensional physical EMRI template bank** — phenomenological track search is better.
* **Swept-band integration in an STFT** — sums a variable band of *correlated* bins, so
  `√support` is wrong by a template-dependent factor. Independently reported as making things
  worse in a WDM tiling too. Fix the tiling instead, so a track stays within ~1 pixel/column.

## Reported elsewhere, not reproduced here

From a parallel run — worth confirming.

* Stationary-line forest alone → no MBHB seed.
* Injected chirp recovered in that same line forest.
* Two simultaneous synthetic chirps → separate usable seeds.
* Strict cross-window coherence retained **3/15**.

⚠️ That 3/15 sits oddly against **9/15 here with no coherence requirement** — coherence should
only help. Likely a PSD-config difference. Diff that first.
