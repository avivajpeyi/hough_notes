---
description: Can we speed up the
---

# Seeding LISA global fit with Hough

Find inspiral tracks in LISA TF maps well enough to **seed** a global fit. Output is a candidate
track + a parameter *region* + a calibrated significance — not a point estimate.

## Status

| | |
|---|---|
| Works | MBHB on LDC2 Spritz + Sangria training |
| Sangria | **9/15 MBHBs, 0 false alarms**, full year ~4 s |
| Spritz mbhb1 | Mc ×1.005, tc +0.4 h, through 3 glitches + 14 h gaps |
| Diagnosed not built | GBs — need Doppler demodulation |
| Not attempted | EMRI, stellar-origin BH |
| Baseline | Nobili et al. **15/15** on the same Sangria data |

## What a candidate is

```
track       f(t) on the data's time grid
parameters  (Mc, tc) + which family produced them
region      bounds at a stated score fraction — not a point
rank        calibrated FAP
support     time span + band actually occupied
```

`region` isn't optional — the `(Mc,tc)` surface is degenerate (Mc trades against tc), so a point
estimate misleads as an initialisation. See [Results](seeding-lisa-global-fit-with-hough/results.md#mctc-degeneracy).

## Scope

| class | role | status |
|---|---|---|
| MBHB | fastest validation; strongest prior art | working |
| GB | **the test of whether this helps the global-fit bottleneck** | deferred, not dropped |
| EMRI | exciting, but recent work makes single-harmonic hard to claim | not attempted |
| Stellar-origin BH | same annual Doppler as GB | not attempted |

**On GBs:** a full `(f, ḟ, λ, β)` Hough is the wrong first move (2 extra sky dims, ~1e4 sources).
But don't drop them — they're the case that decides whether a candidate layer actually relieves
the global fit. Staged version:

```
WDM morphology → (f̂, ḟ̂) → F-statistic / FastGB refinement → q(θ_GB)
```

WDM only shrinks the prior volume; the F-stat decides which apparent tracks are consistent with
real LISA response modulation. Question to answer: *can a WDM candidate finder produce proposals
as useful as a dedicated residual PTMCMC, at lower wall time?*

Expect the overlap graph to go giant-connected-component in the confusion regime. When it does,
leave sub-threshold GBs in the stochastic component rather than pretending sparsity holds.

## Novelty boundary

⚠️ **"Soft Hough for LISA" is not enough on its own.** StackSlide already sums normalised SFT
power, PowerFlux uses noise/antenna-weighted power, weighted Hough already carries noise weights.
Continuous accumulation is old GW territory.

The defensible claim is narrower: **a locally noise-calibrated continuous track statistic in an
orthogonal WDM representation, built to generate proposals for nonstationary LISA global
inference** — judged on downstream compute saved.

## Still a Hough transform?

Yes, a generalised soft one. Scatter ("each pixel votes for every template through it") and
gather ("each template sums pixels along its track") are transposes.

* **Kept:** accumulator over a parameter grid, strictly incoherent, one pass, no waveforms.
* **Replaced:** binary peak-gram → soft weights; straight lines → arbitrary curves; peak counting
  → Gamma FAP.
* **Lost, worth recovering:** shift-invariance. GFH gives it back — see
  [Method](seeding-lisa-global-fit-with-hough/method.md#track-families--the-gfh-connection).

## Code

```bash
cd ../wdm_hough
PYTHONPATH=src python -m pytest -q
PYTHONPATH=src python studies/sangria_search.py   ../data/LDC2_sangria_training_v1.h5
PYTHONPATH=src python studies/ablation_psd.py     ../data/LDC2_sangria_training_v1.h5
PYTHONPATH=src python studies/diagnose_sangria.py ../data/LDC2_sangria_training_v1.h5
```
