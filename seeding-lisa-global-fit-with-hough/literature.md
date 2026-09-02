---
description: Current literature on hough + searching/seeding in LISA
---

# Literature

<div><figure><img src="../.gitbook/assets/Screenshot 2026-09-02 at 4.23.04 pm.png" alt=""><figcaption></figcaption></figure> <figure><img src="../.gitbook/assets/Screenshot 2026-09-02 at 4.22.09 pm.png" alt=""><figcaption></figcaption></figure> <figure><img src="../.gitbook/assets/Screenshot 2026-09-02 at 4.21.57 pm.png" alt=""><figcaption></figcaption></figure></div>

## The two direct competitors

**Nobili et al. 2026** — MBHB pre-merger, STFT excess power (`2602.16792`)

* Chirp *slices* — band between two neighbouring chirp tracks — not line integrals.
* PSD = median over a **disjoint preceding 5-day chunk**, offset ~10 d. Works only because MBHBs are transient.
* Gamma fit to the per-slice background, FAP from its survival function.
* Region tracking across chunks: connected-component labelling on the FAP map, regions matched by overlap between timestamps.
* **15/15 on Sangria**, 14 pre-merger, chirp mass <3%, ~10 false detections/yr at FAP 1e-6, <1 s per 10-day chunk.
* Runs A and E independently, raises a flag on coincidence.
* ⇒ direct baseline. We're at 9/15 with 0 FA. They're ahead.

**Speri et al. 2026** — EMRI, semi-coherent (`2510.20891`)

* SVD basis for the frequency evolution (~20 components), search coefficients not physical params.
* Semi-coherent statistic: max over phase per SFT segment; linear-chirp approx inside each segment.
* JAX + Adam + differential evolution, 512 walkers. 1 yr in ~1 GPU-hour.
* SBI (neural spline flow) maps recovered track → EMRI params.
* 94% det at SNR 30, FAP 1e-2. Frequency evolution to ~1%.
* **States they assume the PSD is known.** Explicitly framed as global-fit proposals.
* Their Fig 1: GBs and EMRIs separate cleanly in `(f, ḟ)`.

## Hough lineage

* **Astone et al. 2014** — Frequency-Hough. Binary peakmap, pixels vote into `(f0, ḟ)`.
* **Miller et al. 2018** — Generalized FH. `x = f^(-(n-1))` linearises a power law → standard linear Hough in `(x₀, k)`. **This is the thing we should adopt** (see Method).
* **Krishnan et al. 2004** — original Hough for CW.
* **Weighted Hough** — noise/antenna weights on peaks, incl. nonstationarity. Still thresholded.
* **StackSlide / PowerFlux** — accumulate *continuous* normalised power, not binary peaks.
  ⚠️ This is why "soft Hough" alone is not a novelty claim.

## Partial coherence (relevant if we add phase)

* **Dergachev, loosely coherent** — interpolates between power sums and full coherence.
* **Cutler, phase-relaxed F-statistic** — phase freedom between segments, ~10–15% gain.
* ⇒ a WDM pairwise-phase score is a *cheap implementation* of partial coherence, not a new idea.

## WDM

* **Cornish 2020** — TF framework, wavelet packets, locally stationary noise, tracks as compact objects.
* **Cornish, covariance** — Fourier coefficients lose diagonal covariance under nonstationarity; WDM stays ~diagonal *if* the dynamic spectrum varies slowly over a cell. **Not valid across gaps/glitches/rapid changes.**
* **Digman & Cornish** — dynamic WDM spectrum for the cyclostationary Galactic background; sparse wavelet likelihood, big speedups for stellar-origin binaries.
* **Vajpeyi et al. 2026** — `wdm_transform`, NumPy+JAX, exact round-trip. Notes WDM is better for bursts, STFT for long-lived signals where phase helps. Flags tiling optimisation as open.

## Global-fit pipelines (the customers)

| pipeline | how sources are found | relevance |
|---|---|---|
| **GLASS** | GB RJMCMC in narrow bands, progressive 1.5→3→6→12 months; MBHB max-likelihood preprocessing | search and sampler already separate |
| **Erebor** | fixed-dim PTMCMC on residuals → GMM birth proposals for RJMCMC | **closest competitor** for a cheap GB candidate engine |
| **Gee-Moo** | explicit frequentist search then Bayesian PE; F-stat + grid + Powell + PSO | precedent for "search first, infer second"; reports product-space convergence trouble |

⇒ All three spend real effort on candidate generation. That's the gap we're aiming at.
