# Hadronic Top-Quark Mass Reconstruction

Reconstruction of the **top-quark mass** from its hadronic decay
`t → b W → b q q̄′` (three jets) in semileptonic `tt̄` events, using **CMS Open
Data** and the Python [Scikit-HEP](https://scikit-hep.org/) stack (uproot,
awkward, vector, scipy) — no CMSSW/ROOT-C++ required.

*Author: Geraldine Lomeli Ponce*

---

## Results

| Quantity | Measured | PDG (2024) | Tension |
|---|---|---|---|
| Hadronic `W` (unbiased) | **81.8 ± 2.1 GeV** (stat.) | 80.369 ± 0.013 GeV | 0.7 σ |
| Top, uncorrected (W-window) | **163.3 ± 1.5 GeV** (stat.) | 172.57 ± 0.29 GeV | 6.1 σ |
| Top, after **b-regression** | **170.7 ± 1.4 GeV** (stat.) | 172.57 ± 0.29 GeV | 1.3 σ |

- Every fit is **error-weighted** (`curve_fit(..., sigma=√N, absolute_sigma=True)`)
  and reports **χ²/ndf and a p-value**.
- The fitted Gaussian width is **σ ≈ 29 GeV**, dominated by the **jet energy
  resolution** (~11 % per b-jet) — the top's natural width is only 1.4 GeV.
- The tensions are **not** a disagreement with the Standard Model: the quoted ±
  is statistical only, and the **systematic** error (the **jet energy scale**)
  is missing from it. See [Physics notes](#physics-notes).

(All plots are produced by running the notebook.)

---

## The data sample

| Property | Value |
|---|---|
| Process | `pp → tt̄`, semileptonic (μ+jets) channel |
| Primary dataset | SingleMuon, Run2016H-UL2016 NanoAODv9 |
| Collisions | proton–proton, √s = 13 TeV |
| Format | NanoAOD (~1–2 kB/event) |
| File | `0107961B-4308-F845-8F96-E14622BBA484.root` (1.8 GB) |
| Events in file | 2,383,660 |

> **This is real collision data, not simulation.** The generator-level
> `GenPart` branches are **absent**, so there is no Monte-Carlo truth: no
> jet-to-parton truth-matching and no MC calibration are possible. Every result
> is obtained from the reconstructed objects alone.

### Getting the data

The ROOT file is **not** committed (it is 1.8 GB). Download the corresponding
NanoAOD file from the CMS Open Data portal into this directory and set
`file_path` in the load cell of the notebook accordingly.

---

## Setup

```bash
python3 -m venv higgs_env
source higgs_env/bin/activate
pip install uproot awkward vector numpy scipy matplotlib jupyterlab
```

## How to run

```bash
# headless: re-execute and save outputs in place
jupyter nbconvert --to notebook --execute --inplace \
    --ExecutePreprocessor.timeout=900 mass_hadronictop.ipynb
```

### Analysis flow (`mass_hadronictop.ipynb`)

1. **Load** the muon, jet and MET branches (including `Muon_ptErr`,
   `Jet_bRegRes`, `Jet_bRegCorr`, `MET_significance`/`covXX/XY/YY`).
2. **Select** the μ+jets topology with an explicit **cut flow**:
   `2,383,660 → 1,552,928 (1 tight μ) → 78,652 (≥4 jets) → 43,726 (b-tag) →
   10,131 (W-window)`.
3. **Per-object uncertainty budget** — muon ~1.4 %, b-jet resolution ~11 %, MET
   significance; tie it to why the mass comes out low.
4. **Backgrounds** (QCD, W/Z+jets, single-top, dileptonic tt̄) and what each cut
   suppresses.
5. **Combinatorics** — all jet pairings vs the one chosen (qualitative; no
   invented signal fraction).
6. **Genuine, unbiased W** (highest-pT light-jet pair, no 80.4 GeV input) →
   fit & compare to PDG.
7. **Hadronic top** — verify the invariant-mass formula, then fit (initial and
   W-window) with χ²/ndf + p-value.
8. **b-jet energy scale** — apply `Jet_bRegCorr`, refit, recover ~7 GeV.
9. **Leptonic side** — transverse mass `m_T` (Jacobian edge) and the neutrino
   `p_z` quadratic.
10. **PDG comparison** with errors and tensions, then the **systematics**
    (JES headline).

---

## Physics notes

**The mass formula is correct.** The hadronic three-jet invariant mass
`M_bjj = √[(E_b+E_j1+E_j2)² − |p⃗_b+p⃗_j1+p⃗_j2|²]` is the proper invariant mass
of the summed four-vector (verified in the notebook against an explicit
`E²−|p|²`). It is correct because the hadronic side is fully measured; only the
*leptonic* side misses a four-vector (the neutrino), which is handled separately
via `m_T` and the `p_z` quadratic — not by changing the formula.

**A genuine vs a circular W.** Selecting the dijet by `argmin|m_jj−80.4|`
(used only to build the top) *assumes* the W mass and cannot measure it. An
unbiased estimator — the highest combined-pT light-jet pair — gives
`m_W = 81.8 ± 2.1 GeV`, consistent with the PDG within the large
jet-resolution width. So the **light-jet** scale is roughly right.

**The deficit is the b-jet energy scale (demonstrated).** With the W ~right but
the top ~9 GeV low, the missing energy must be in the b-jet. b-jets are
under-measured (they lose energy to semileptonic neutrinos): the per-jet
regression `Jet_bRegCorr` has median 1.08. Applying it shifts the top
**163.3 → 170.7 GeV**, within ~2 GeV of the PDG — a *demonstrated* jet-energy-scale
effect, not a hand-wave and not a bug in the formula.

**Goodness of fit & resolution.** Fits are error-weighted least squares with
χ²/ndf and a p-value. The σ ≈ 29 GeV peak width is set by the jet energy
resolution, far larger than the top's 1.4 GeV natural width.

**Systematics (not in the ±).** Only the statistical fit error is quoted. The
dominant systematics are the **JES** (headline; b-jet scale especially), jet
resolution (JER), combinatorial background, the W-window selection bias
(170 → 163 GeV), b-tagging efficiency, and residual physics backgrounds.
A proper measurement adds these in quadrature; they dwarf the statistical error.

**Not attempted** (impossible with this dataset): MC truth-matching, MC
cross-validation, and a quantified combinatorial-background fraction — there is
no `GenPart` truth in real collision data.

---

## Repository layout

```
mass_hadronictop.ipynb       analysis notebook (self-contained; plots embedded)
mass_hadronictop.pptx        presentation
README2.md                   this file
*.root                       data file — NOT committed (1.8 GB)
```

## Acknowledgements

Uses CMS Open Data made available under the CC0 waiver via the
[CERN Open Data Portal](https://opendata.cern.ch/). PDG values from the
[Particle Data Group](https://pdg.lbl.gov/).
