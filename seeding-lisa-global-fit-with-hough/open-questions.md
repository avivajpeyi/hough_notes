---
description: Ranked next steps, and what to avoid
---

# Open questions

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

## Things not to spend effort on

- **Chasing 15/15 on Sangria MBHBs.** It is someone else's contribution and we would arrive
  behind; the noise model is the part that is ours.
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
