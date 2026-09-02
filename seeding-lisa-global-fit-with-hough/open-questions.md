---
description: Ranked next steps, and what to avoid
---

# Open questions

## 0. Two gaps that outrank everything below

**Search-level false-alarm control.** We compute a *per-template* FAP, `P₀(H(θ) > h)`. What
matters is `P₀(max over the whole search grid > h)`, and naive trial counting is unreliable
because neighbouring templates are correlated. The fix is to calibrate the distribution of the
**loudest cluster** on noise-only realizations and take an empirical quantile. Until that
exists, our FAPs are not comparable to published ones, and the catalogue-level metric should be
a false-discovery rate rather than a per-template number.

**The metric that decides whether any of this matters.** Not ROC. Run the *same* refinement from
a broad prior and from the candidate proposal, and record

```
G_L = N_likelihood(broad) / N_likelihood(candidate)
G_T = T(broad) / (T(candidate) + T_search)
```

`G_T` includes the search cost, so an expensive candidate finder cannot hide outside the
reported inference budget. This is the number that answers why the global-fit community should
care, and nothing here measures it yet.

## 1. The always-on demonstration

The scientific claim — a noise model that works where no signal-free epoch exists — rests on
this, and it has not been run. Sangria's MBHBs are transient, so an offset-chunk PSD is
perfectly legitimate there. Inject an EMRI, or use the verification binaries, so the offset
trick has nothing to offset to, and re-run the ablation. If the contrast is not large there, the
claim needs rethinking.

## 2. GFH linearisation

Implement `x = f^{-8/3}` as a coordinate transform on the WDM map and run the *existing* linear
machinery in `(x₀, k)`, comparing against the direct `(Mc, tc)` scan on the same chunks. The
mapping is verified exact (1×10⁻¹⁵); what is untested is whether the transformed search
reproduces the same candidates. Equal results confirm both and hand us the shift-invariant fast
path; different results localise a bug in one.

Worth asking the pyhough author: the non-uniform `gridx` in `LongT_GENERALIZED_fasthough_nonuni`
comes from `1/f^(n-1)` on a *uniform frequency* grid. On a WDM map the natural grid is uniform
in `f` too, so the same construction should carry over — but the interaction between that
non-uniform x-binning and a soft (non-0/1) weight has not been thought through.

## 3. Cross-window coherence

Nobili's largest single advantage: accumulate evidence across overlapping chunks rather than
deciding per chunk, with connected-region labelling on the FAP map. Several of our six misses
are marginal in one window and would likely survive across three.

## 4. The systematic Δtc

All nine matches are late by 1.3–10.9 h. Almost certainly the Newtonian track approximating
IMRPhenomD, but it is unexplained and it biases every seed handed downstream. A higher-order PN
or an IMRPhenomD frequency evolution as a `TrackFamily` would test it directly.

## 5. Glitch excision

Glitches are broadband **vertical** stripes — sparse in time, where GB lines are sparse in
frequency. Flag outlier time columns and excise them as gaps, reusing the existing gap
machinery. Cheap, independent of everything else.

## Sequencing

Each layer should justify itself before the next is built:

1. Does *calibrated* soft weighting help?
2. Does WDM help once the data are nonstationary and gapped?
3. Does phase add anything **after** search-level false-alarm calibration?
4. Do the candidates materially accelerate real waveform refinement?
5. Can their WDM footprints factor the global source problem?

This guards against building an elegant architecture before showing the candidate layer saves
the expensive computation.

## The phase experiment, if it happens

WDM coefficients are real, but the transform builds a complex pre-Wilson field and discards one
quadrature at the final projection. Exposing that field as an auxiliary and scoring pairwise
phase consistency along a track is a small change. Two constraints: treat it as a **candidate
reranker, not a likelihood** — the complex field is redundant against the real Wilson basis and
has non-trivial covariance — and note the prior art. Loosely coherent searches and the
phase-relaxed F-statistic already occupy partial-coherence territory, so this is a cheap WDM
implementation of partial coherence, not phase-aware Hough.

## Things not to spend effort on

- **Chasing 15/15 on Sangria MBHBs.** It is someone else's contribution and we would arrive
  behind; the noise model is the part that is ours.
- **A full `(f, ḟ, λ, β)` GB Hough.** Use WDM morphology to shrink the prior, then hand off to
  the F-statistic — the sky dimensions are not worth brute-force search.
- **A high-dimensional physical EMRI template bank.** A phenomenological track search is the
  better route.
- **Quoting detection counts from the ablation** — 18 of 23 chunks contain a merger, so the
  metric does not discriminate.
- **Swept-band integration in an STFT.** Integrating the band a track sweeps within a segment
  sounds right and fails, because it sums a variable band of *correlated* neighbouring bins and
  `√support` is then wrong by a template-dependent factor. Independently reported as making
  recovery worse in a WDM tiling too. The fix is to choose the tiling so a track stays within
  ~1 pixel per column, not to integrate over the sweep.

## Reported elsewhere, not reproduced here

From a parallel run; worth confirming before relying on them.

- A stationary-line forest alone produces no MBHB seed.
- An injected chirp is recovered in that same line forest.
- Two simultaneous synthetic chirps give separate usable seeds.
- Strict cross-window coherence retained 3/15 catalogue MBHBs.

That last number sits oddly against 9/15 here **without** any coherence requirement, since
coherence should only help. The likely difference is PSD configuration — worth diffing first.
