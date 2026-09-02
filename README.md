---
description: Can we speed up the
---

# Seeding LISA global fit with Hough

Finding inspiral tracks in LISA time-frequency maps well enough to **seed** a global fit —
candidate tracks with a parameter *region* and a calibrated significance, not a point estimate.

## Status

| | |
|---|---|
| Working end to end | MBHB on LDC2 Spritz and Sangria training |
| Sangria training | **9/15 catalogue MBHBs, 0 false alarms**, full year in ~4 s |
| Spritz mbhb1 | Mc ×1.005, tc +0.4 h, through 3 glitches and 14 h of gaps |
| Diagnosed, not built | Galactic binaries — need Doppler demodulation |
| Not attempted | EMRI, stellar-origin BH |
| Comparison point | Nobili et al. get 15/15 on this same Sangria data |

## What a candidate is

```
track       f(t) on the data's own time grid
parameters  (Mc, tc), with the family that produced them
region      bounds at a stated score fraction — not a point
rank        calibrated false-alarm probability
support     the time span and band actually occupied
```

The `region` is not optional: the `(Mc, tc)` surface is degenerate, Mc trading against tc along
a ridge, so a point estimate is misleading as an initialisation. See
[Results](seeding-lisa-global-fit-with-hough/results.md#the-mc-tc-degeneracy).

## Scope

| class | track family | status |
|---|---|---|
| MBHB | `(Mc, tc)` Newtonian chirp | working |
| EMRI | SVD basis or slow chirp | not attempted |
| Stellar-origin BH | Doppler + slow chirp | not attempted |
| Galactic binaries | Doppler-modulated monochromatic | out of scope |

GBs are excluded by **density** — ~10⁴ resolvable sources, a blind search needs the sky as two
extra dimensions, and the global fit already handles them iteratively. Note that stellar-origin
BHs carry the *same* annual Doppler structure, so the split is not "GBs need Doppler and nothing
else does".

## Where this sits

**Speri et al. 2026** — semi-coherent EMRI search, SVD frequency basis, JAX/GPU, explicitly
producing proposals for the global fit. They **assume the PSD is known**.

**Nobili et al. 2026** — MBHB pre-merger excess power in STFT spectrograms, 15/15 on Sangria,
chirp mass to 3%. Their PSD is a median over a *disjoint preceding* chunk, which works because
MBHBs are transient.

For EMRIs, stellar-origin BHs and CWs there is no signal-free epoch anywhere in the data, so
neither approach is available. **A robust in-situ `S(t,f)` is the gap**, and the part of this
work that is genuinely new. Our MBHB search is behind Nobili's on the same data; the search is
the demonstrator, not the claim, and the
[ablation](seeding-lisa-global-fit-with-hough/results.md#psd-ablation) is designed around that.

## Is it still a Hough transform?

Yes — a *generalised soft* Hough. Scatter ("each pixel votes for every template through it") and
gather ("each template sums the pixels along its track") are transposes of the same operation.
Kept: an accumulator over a parameter grid, strictly incoherent, one pass, no waveforms.
Replaced: the binary peak-gram by
[soft weights](seeding-lisa-global-fit-with-hough/method.md#soft-rather-than-hard-voting),
straight lines by arbitrary parametric curves, peak counting by a Gamma FAP.

Dropped and worth recovering: the shift-invariance that makes classical FH fast. The GFH
linearisation gives it back — see
[Method](seeding-lisa-global-fit-with-hough/method.md#track-families-and-the-gfh-connection).

## Code

```bash
cd ../wdm_hough
PYTHONPATH=src python -m pytest -q
PYTHONPATH=src python studies/sangria_search.py   ../data/LDC2_sangria_training_v1.h5
PYTHONPATH=src python studies/ablation_psd.py     ../data/LDC2_sangria_training_v1.h5
PYTHONPATH=src python studies/diagnose_sangria.py ../data/LDC2_sangria_training_v1.h5
```
