# The Quark Wake Effect — Complete Physics Reference

> **2026 Discovery**: First direct physical proof that the universe's earliest matter (Quark-Gluon Plasma) behaved as a near-perfect liquid, published by MIT + CERN CMS Collaboration.

---

## Table of Contents
1. [Core Physics: Recreating the Big Bang](#1-core-physics-recreating-the-big-bang)
2. [The QCD Phase Diagram](#2-the-qcd-phase-diagram)
3. [Quark-Gluon Plasma Thermodynamics](#3-quark-gluon-plasma-thermodynamics)
4. [Bjorken Hydrodynamics](#4-bjorken-hydrodynamics)
5. [The Quark Wake Effect — Mach Cone Formation](#5-the-quark-wake-effect--mach-cone-formation)
6. [Jet Quenching: BDMPS Radiative Energy Loss](#6-jet-quenching-bdmps-radiative-energy-loss)
7. [Collisional Energy Loss](#7-collisional-energy-loss)
8. [The Perfect Liquid: KSS Viscosity Bound](#8-the-perfect-liquid-kss-viscosity-bound)
9. [The Z Boson Tagging Technique](#9-the-z-boson-tagging-technique)
10. [Two-Particle Correlations and Azimuthal Harmonics](#10-two-particle-correlations-and-azimuthal-harmonics)
11. [Cooper-Frye Freeze-out](#11-cooper-frye-freeze-out)
12. [Color Glass Condensate Initial State](#12-color-glass-condensate-initial-state)
13. [What Was Measured: Observables](#13-what-was-measured-observables)
14. [How Everything Connects](#14-how-everything-connects)
15. [Python Simulation Programs](#15-python-simulation-programs)
16. [Scientific References](#16-scientific-references)

---

## 1. Core Physics: Recreating the Big Bang

In the first **~10 microseconds** after the Big Bang, the universe was too hot (T > 155 MeV ≈ 1.8 × 10¹² K) for protons or neutrons to form. Quarks and gluons — the fundamental building blocks of nuclear matter — moved freely in a state called **Quark-Gluon Plasma (QGP)**.

To study this, scientists at [CERN](https://home.cern/science/accelerators/large-hadron-collider) accelerate lead nuclei (Pb, Z=82, A=208) to 99.9999% the speed of light and smash them together, momentarily recreating microscopic droplets of this primordial plasma at temperatures **~100,000× hotter than the sun's core**.

```
[Lead Ion]──(~2.76 TeV/nucleon)──> HEAD-ON COLLISION <──(~2.76 TeV/nucleon)──[Lead Ion]
                                          │
                                   T ~ 4×10¹² K
                                   τ_thermalize ~ 0.1–0.5 fm/c
                                          │
                                   ┌──────▼──────┐
                                   │     QGP     │  ← Primordial soup
                                   │   "Ocean"   │     recreated
                                   └─────────────┘
                                          │
                                   Expands + cools
                                          │
                                   T ~ 155 MeV → Hadronization
                                          │
                                   Thousands of hadrons
                                   detected by CMS
```

---

## 2. The QCD Phase Diagram

Quantum Chromodynamics (QCD) — the theory of the strong nuclear force — predicts a **phase transition** between ordinary hadronic matter and QGP.

```
Temperature T
     │
4×10¹²K│                    ┌─────────────────────────────┐
       │                    │   QUARK-GLUON PLASMA (QGP)  │
       │                    │   (LHC, Early Universe)      │
Tc~1.8×│ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┼─── crossover at μ_B ≈ 0    │
10¹² K │                    │                              │
       │  HADRON GAS        │                              │
       │  (protons,neutrons)│  1st order phase boundary →  │
       └────────────────────┴──────────────────────────────►
                            μ_B (baryon chemical potential)
                         (0 for LHC;  high for neutron stars)
```

### Critical Temperature (Lattice QCD result):

```
T_c = 155 ± 1.5 MeV  ≈  1.8 × 10¹² K
```

At μ_B = 0 (LHC conditions), the transition is a smooth **crossover**, not a sharp phase transition.

**Order parameters:**
- **Polyakov loop** `⟨L⟩`: measures colour deconfinement
- **Chiral condensate** `⟨ψ̄ψ⟩`: measures chiral symmetry restoration

---

## 3. Quark-Gluon Plasma Thermodynamics

### 3.1 Stefan-Boltzmann Equation of State

For an ideal relativistic gas of quarks and gluons (T ≫ T_c):

```
ε = g* · (π²/30) · T⁴

where:
  ε  = energy density  [GeV/fm³]
  g* = effective degrees of freedom
     = g_gluons + (7/8)·g_quarks
     = 16 + (7/8)·(2·2·3·Nf) = 16 + (7/8)·36 = 47.5  (for Nf=3 flavours)
  T  = temperature  [GeV]
```

Pressure and entropy density:

```
P = ε/3                   (ultra-relativistic EoS)
s = (4/3)·(ε/T) = dP/dT  (entropy density)
```

### 3.2 Speed of Sound

Crucially sets the Mach cone angle:

```
c_s² = ∂P/∂ε

Ideal QGP:     c_s² = 1/3  →  c_s = 1/√3 ≈ 0.577c
Near Tc:       c_s² ≈ 0.10–0.15  (softest point of EoS, speed dip)
```

The **softening near T_c** creates a characteristic dip in the speed of sound — measured by the shape of collective flow patterns.

### 3.3 Debye Screening Mass

Colour charge is screened in QGP over length scale λ_D:

```
m_D² = g²T²·(Nc/3 + Nf/6)

where:
  g    = QCD coupling constant (g² = 4π·α_s)
  Nc   = 3  (number of colours)
  Nf   = 3  (number of active flavours at LHC T)
  m_D  ≈ gT ≈ 0.4–1.0 GeV at LHC temperatures
```

---

## 4. Bjorken Hydrodynamics

### 4.1 Boost-Invariant Longitudinal Expansion

J.D. Bjorken's 1983 model describes the 1+1D longitudinal expansion after a heavy-ion collision, assuming boost invariance in rapidity y.

**Proper time and space-time rapidity:**

```
τ = √(t² - z²)           (proper time)
η_s = ½·ln[(t+z)/(t-z)]  (space-time rapidity)
```

**Conservation equation for energy density:**

```
dε/dτ + (ε + P)/τ = 0
```

**Solution for ideal QGP (c_s² = 1/3):**

```
ε(τ) = ε(τ₀)·(τ₀/τ)^(4/3)

T(τ) = T₀·(τ₀/τ)^(1/3)

where:
  τ₀ ≈ 0.1–0.5 fm/c   (thermalization time)
  T₀ ≈ 300–600 MeV     (initial temperature at LHC)
  1 fm/c ≈ 3.3 × 10⁻²⁴ s
```

### 4.2 Initial Energy Density (Bjorken estimate)

```
ε(τ₀) = (1/(π·R_A²·τ₀)) · dE_T/dy

where:
  R_A ≈ 1.2·A^(1/3) fm  (nuclear radius, A=208 for Pb)
  R_Pb ≈ 6.6 fm
  dE_T/dy = transverse energy per rapidity unit
           ≈ 2000 GeV for central Pb-Pb at 5.02 TeV

→ ε₀ ≈ 15–50 GeV/fm³  (≈100–300× normal nuclear density)
  (value depends on τ₀: τ₀=0.5 fm/c → ε₀≈29 GeV/fm³; τ₀=1.0 fm/c → ε₀≈15 GeV/fm³)
```

---

## 5. The Quark Wake Effect — Mach Cone Formation

### 5.1 Physical Picture

When a **supersonic** high-energy quark (v ≈ c) punches through the QGP medium (c_s ≈ 0.577c), it deposits energy and momentum via:
- Collisional scattering with thermal partons
- Bremsstrahlung (BDMPS gluon radiation)

This creates a **hydrodynamic Mach shock cone** — exactly like a supersonic aircraft or boat wake.

```
          Quark path ──────────────────────►
                    \  ╲                 /
                θ_M  \  ╲             /   ← Mach cone
                      \  ╲         /       shock front
                       \  ╲     /
                        \  ╲ /
                         wake ripple deposited
                         in QGP medium
```

### 5.2 Mach Cone Angle

```
cos(θ_M) = c_s / v_parton

For v_parton ≈ c (ultra-relativistic quark):
  cos(θ_M) = c_s/c = 1/√3

  θ_M = arccos(1/√3) ≈ 54.7°   (ideal QGP)

Near Tc where c_s is reduced:
  c_s ≈ 0.33c  →  θ_M ≈ arccos(0.33) ≈ 70.7°
```

### 5.3 Hydrodynamic Source Term

The energy-momentum deposited by the quark sources the fluid equations:

```
∂_μ T^μν = J^ν(x)

where J^ν is the source 4-current from the passing parton.

The linearized perturbation δε, δu in the fluid creates:
  - Mach shock waves at angle θ_M
  - A diffusion wake on the Cherenkov-like cone
```

### 5.4 Observable: Azimuthal Correlation Double Hump

The Mach cone manifests as a **double-hump** in particle azimuthal angle correlations:

```
Peak 1: Δφ = π + θ_M  (left horn of cone)
Peak 2: Δφ = π - θ_M  (right horn of cone)

For ideal QGP: peaks at π ± 54.7° = 124.7° and 235.3°
```

The 2026 CMS experiment observed this signature by subtracting background flow with Z-boson tagging.

---

## 6. Jet Quenching: BDMPS Radiative Energy Loss

### 6.1 The BDMPS-Z Formalism

Baier-Dokshitzer-Mueller-Peigné-Schiff (BDMPS) describes induced gluon bremsstrahlung when a quark traverses a QCD medium. Multiple soft scatterings cause quantum interference (LPM effect), suppressing radiation at low ω.

### 6.2 Transport Coefficient q̂

The key medium property — transverse momentum squared acquired per unit path length:

```
q̂ = ⟨p_T²⟩ / L        [GeV²/fm]

Estimated values:
  q̂ ≈ 1–3 GeV²/fm  at RHIC (√s_NN = 200 GeV)
  q̂ ≈ 3–10 GeV²/fm  at LHC (√s_NN = 2.76–5.02 TeV)
```

### 6.3 Characteristic Gluon Energy (LPM cutoff)

```
ω_c = ½·q̂·L²

where:
  L ≈ 5–8 fm   (typical medium path length in central Pb-Pb)
  → ω_c ≈ 10–50 GeV
```

### 6.4 Average Radiative Energy Loss

```
⟨ΔE⟩_rad = (α_s·C_R/2)·q̂·L²  =  α_s·C_R·ω_c

Full differential spectrum:
  dI/dω ∝ α_s·C_R·(1/ω)·√(q̂/ω)   for ω < ω_c

Differential energy loss:
  dE/dx|_rad ≈ (α_s·C_R·q̂/2)·ln(E/ω_c)

where:
  α_s ≈ 0.3        (strong coupling at ~10 GeV scale)
  C_R = C_F = 4/3  (quark Casimir)
  C_R = C_A = 3    (gluon Casimir)
  L               = medium path length [fm]
  E               = parton energy [GeV]
```

**Numerical estimate for a 100 GeV quark, L = 5 fm:**
```
ω_c ≈ ½ × 5 × 25 = 62.5 GeV  (using q̂ = 5 GeV²/fm)
⟨ΔE⟩ ≈ 0.3 × (4/3) × 62.5 ≈ 25 GeV  (25% energy loss!)
```

---

## 7. Collisional Energy Loss

Elastic scattering of the hard quark off thermal partons also contributes:

```
dE/dx|_coll = (4π·α_s²·T²/3v²)·C_R·[ln(E·T/m_D²) + const]

where:
  T   = local medium temperature [GeV]
  v   ≈ c  (ultra-relativistic parton)
  m_D ≈ gT  (Debye screening mass, colour-electric screening)

Numerical estimate (T = 300 MeV, α_s = 0.3, C_R = 4/3):
  dE/dx|_coll ≈ 0.3–1.0 GeV/fm

Compare to radiative:
  dE/dx|_rad ≈ 2–5 GeV/fm  (dominant at high pT)
```

**Total energy loss:**
```
dE/dx|_total = dE/dx|_rad + dE/dx|_coll
```

The ratio radiative/collisional ≈ 2:1 to 5:1 for quark pT > 10 GeV.

---

## 8. The Perfect Liquid: KSS Viscosity Bound

### 8.1 Kovtun-Son-Starinets (KSS) Bound

From AdS/CFT duality (string theory / holography), the minimum shear viscosity to entropy ratio is:

```
η/s ≥ ℏ/(4π·k_B) ≈ 6.08 × 10⁻²⁵ J·s/m³ / (J/K/m³)
                   ≈ 0.08  (in natural units where ℏ = k_B = 1)
```

### 8.2 Why QGP Is a "Near-Perfect" Fluid

QGP measurements at RHIC and LHC give:

```
η/s|_QGP ≈ 0.08–0.24  (just 1–3× the KSS bound)

Compare:
  Water at 20°C:    η/s ≈ 380 × ℏ/4πk_B  (not perfect at all)
  Superfluid ⁴He:   η/s ≈ 8 × ℏ/4πk_B
  QGP:              η/s ≈ 1–3 × ℏ/4πk_B  ← closest to perfect known
```

### 8.3 Why This Matters for the Wake

A fluid with small η/s:
- Transmits pressure waves with **minimal damping**
- Preserves Mach cone structure over fm-scale distances
- Creates sharp, observable double-hump correlation signal

High viscosity would **smear out** the wake and make it undetectable.

### 8.4 Viscous Correction to Pressure Waves

The attenuation length for Mach waves:

```
λ_att = 2·τ_s·(1/k)

where τ_s = η/(ε+P) = (η/s)/T  is the viscous relaxation time

For η/s = 1/(4π):
  τ_s = (η/s)/T = (1/4π)/0.300 GeV⁻¹ × ℏc = 0.052 fm/c  at T = 300 MeV
```

---

## 9. The Z Boson Tagging Technique

### 9.1 Why the Z Boson Is Perfect

The Z boson (mass M_Z = 91.2 GeV) is a **colour-neutral** gauge boson that couples only via the electroweak force. It is therefore completely **transparent to QGP** — the strong force has no effect on it.

```
Z boson properties:
  Mass:      M_Z = 91.187 ± 0.002 GeV
  Width:     Γ_Z = 2.495 GeV  (τ ≈ 3×10⁻²⁵ s)
  Decay:     Z → e⁺e⁻ or Z → μ⁺μ⁻  (cleanly detected!)
  Lifetime:  τ_Z = ℏ/Γ_Z = 6.58×10⁻²⁵/2.495 ≈ 2.64×10⁻²⁵ s
  Couples:   via electroweak force ONLY
  Strong:    does NOT feel QCD → escapes QGP untouched ✓
```

### 9.2 The Tagging Method

```
Pb-Pb collision:
  ├─ Z boson  ──────────────────────► escapes untouched
  │  (pT_Z, φ_Z = known precisely)
  │
  └─ recoil quark ──► enters QGP ──► loses energy via BDMPS
                                    └─► leaves as jet (pT_jet < pT_quark)

Conservation of momentum (before QGP):
  pT_quark_initial = pT_Z  (balanced in pp collision)
  φ_quark_initial  = φ_Z + π  (back-to-back)
```

### 9.3 Energy Imbalance Observable

```
x_jZ = pT_jet / pT_Z

In pp (no QGP):     ⟨x_jZ⟩ ≈ 1.0  (balanced)
In Pb-Pb (with QGP): ⟨x_jZ⟩ < 1.0  (jet loses energy to QGP wake)

The shift: Δx_jZ = 1 - ⟨x_jZ⟩_PbPb ∝ ⟨ΔE⟩/pT_Z
```

### 9.4 Missing Energy Distribution

The "lost" energy reappears as soft particles in the **wake region**:

```
ΔpT = pT_Z - pT_jet  ≈ ⟨ΔE⟩_BDMPS

CMS 2026 observation: The missing pT is found in soft particles
at Δφ ≈ π ± θ_M (the Mach cone double hump)  ← direct wake signature
```

---

## 10. Two-Particle Correlations and Azimuthal Harmonics

### 10.1 Two-Particle Correlation Function

```
C(Δη, Δφ) = (1/N_trig) · dN_pairs / (dΔη · dΔφ)

where:
  Δη = η₁ - η₂  (pseudorapidity difference)
  Δφ = φ₁ - φ₂  (azimuthal angle difference)
```

### 10.2 Fourier Decomposition (Flow Harmonics)

The azimuthal particle distribution relative to the event plane Ψ_n:

```
dN/dφ ∝ 1 + 2·Σ_n v_n · cos[n·(φ - Ψ_n)]

Coefficients:
  v₁ = directed flow     (sidewards push from pressure gradient)
  v₂ = elliptic flow     (almond-shaped initial geometry → oval flow)
  v₃ = triangular flow   (initial geometry fluctuations)
  v₄, v₅, ...           (higher harmonics)

v₂ at LHC:  v₂ ≈ 0.05–0.15  (depends on centrality)
```

### 10.3 Wake Signature in Azimuthal Correlations

After subtracting collective background flow:

```
Remaining signal at Δφ ≈ π ± θ_M:

  Near side (Δφ ≈ 0):   jet fragmentation peak
  Away side (Δφ ≈ π):   modified by QGP absorption + Mach cone split

Double-hump structure:
  Peak at Δφ = π - θ_M  ≈ 125°
  Dip    at Δφ = π       ≈ 180°  (absorbed by medium)
  Peak at Δφ = π + θ_M  ≈ 235°
```

---

## 11. Cooper-Frye Freeze-out

When the QGP cools to T_fo ≈ 150–160 MeV, quarks and gluons recombine into hadrons (**hadronization**). The observable particle spectrum on the freeze-out hypersurface Σ is:

```
E · dN/d³p = g/(2π)³ · ∫_Σ p^μ · dσ_μ · f(p·u/T_fo)

where:
  g       = degeneracy factor
  dσ_μ   = hypersurface normal vector element
  u^μ    = local fluid 4-velocity (includes wake contribution!)
  T_fo   = freeze-out temperature ≈ 150–160 MeV
  f(·)   = Bose-Einstein or Fermi-Dirac distribution

For a Mach wake: u^μ has an angular structure → 
  cos(θ_M) imprinted in dN/dφ  ← the measurement!
```

The **Cooper-Frye formula converts the hydrodynamic Mach wave into the observable particle angular distribution**.

---

## 12. Color Glass Condensate Initial State

Before the QGP forms, the colliding nuclei are described by the **Color Glass Condensate (CGC)** — a saturated state of gluons in ultra-relativistic nuclei.

### 12.1 Saturation Scale

```
Q_s²(x, A) = (4π²·α_s·N_c)/(N_c²-1) · xG(x, Q_s²) / (π·R_A²)

Numerically for Pb at LHC (x ~ 10⁻³):
  Q_s ≈ 1.5–2.5 GeV

where:
  x      = momentum fraction of struck gluon
  xG(x,Q²) = gluon distribution function
  R_A    = nuclear radius
  N_c    = 3 (number of colours)
```

### 12.2 Role in Wake Measurement

The CGC sets:
- **Initial gluon density** → determines energy available for QGP
- **Initial state fluctuations** → seed v₂, v₃, v₄ background flow
- **Hard scattering rate** → determines Z+jet pair production rate

The CMS analysis subtracts the initial-state CGC contribution to isolate the final-state QGP wake signal.

---

## 13. What Was Measured: Observables

### 13.1 The 2026 CMS Measurement

The key observable was the **azimuthal angle distribution of soft particles recoiling against the Z-boson-tagged jet**, in central Pb-Pb vs. pp collisions:

```
Signal = [dN/dΔφ]_PbPb - [dN/dΔφ]_pp

Expected in ideal QGP:  Double hump at Δφ = π ± 54.7°
Observed:               Double hump consistent with c_s ≈ 0.57c
                        → confirms near-perfect fluid behaviour
```

### 13.2 Nuclear Modification Factor

Jet suppression relative to pp:

```
R_AA = (dN_AA/dp_T) / (T_AA · dσ_pp/dp_T)

where T_AA = ∫ρ_A(r)·ρ_B(r) d²r  (nuclear overlap function)

In absence of QGP: R_AA = 1
Observed at LHC:   R_AA ≈ 0.2–0.5  for jets with pT > 100 GeV
→ 50–80% of jet energy absorbed by QGP
```

### 13.3 Summary of Key Numbers

| Quantity | Value | Meaning |
|----------|-------|---------|
| T_c | 155 MeV | QGP formation temperature |
| T_initial (LHC) | 300–600 MeV | Initial QGP temperature |
| c_s (ideal QGP) | 1/√3 ≈ 0.577c | Speed of sound |
| θ_M (ideal QGP) | 54.7° | Mach cone angle |
| η/s (QGP) | ~0.08–0.24 | Near-KSS bound |
| KSS bound | ℏ/(4πk_B) | Minimum possible η/s |
| q̂ (LHC) | 3–10 GeV²/fm | Jet transport coefficient |
| M_Z | 91.2 GeV | Z boson mass |
| ⟨ΔE⟩ | ~25 GeV | Jet energy loss in QGP |
| R_AA (jets) | 0.2–0.5 | Jet nuclear suppression |

---

## 14. How Everything Connects

```
CGC Initial State
    │ Q_s sets initial gluon density
    ▼
Pb+Pb Collision  [√s_NN = 5.02 TeV]
    │ τ₀ ~ 0.1 fm/c
    ▼
QGP Formation  [T₀ ~ 400 MeV, ε₀ ~ 15 GeV/fm³]
    │ Bjorken expansion: T(τ) = T₀·(τ₀/τ)^(1/3)
    │
    ├─── Z boson produced + recoil quark
    │     │                   │
    │     │ Escapes (no QCD)  │ Enters QGP
    │     ▼                   ▼
    │   [Clean reference]  BDMPS energy loss ΔE
    │   pT_Z known         dE/dx = (α_s·C_R·q̂)·ln(E/ω_c)
    │                      + collisional loss
    │                           │
    │                      Mach wake deposited
    │                      at angle θ_M = arccos(c_s/c)
    │                           │
    ▼                           ▼
Hydrodynamic evolution [η/s ~ 1/(4π)]  ← perfect fluid
    │ Navier-Stokes + relativistic viscous hydro
    ▼
Cooper-Frye Freeze-out [T_fo ~ 155 MeV]
    │ p^μ·dσ_μ → observable hadrons
    ▼
CMS Detector Measurement
    │
    ├─ Mach double-hump in Δφ  ← 2026 BREAKTHROUGH
    ├─ x_jZ = pT_jet/pT_Z < 1  (energy imbalance)
    └─ R_AA < 1                 (jet suppression)
```

---

## 15. Python Simulation Programs

Run the Python programs in this directory to visualize the mathematics:

| Program | What It Shows |
|---------|--------------|
| `qgp_thermodynamics.py` | QGP EoS, T(τ) Bjorken cooling, speed of sound |
| `mach_cone_simulation.py` | Mach cone geometry, wake pattern, double-hump |
| `jet_quenching.py` | BDMPS energy loss vs pT, L, q̂ |
| `viscosity_kss.py` | η/s comparison: QGP vs other fluids |
| `z_boson_tagging.py` | x_jZ imbalance distributions pp vs Pb-Pb |
| `azimuthal_correlations.py` | Two-particle Δφ correlations with Mach cone signal |

Run all: `python3 <program>.py` — each saves PNG graphs to `./plots/`

---

## 16. Scientific References

| # | Source |
|---|--------|
| [1] | CERN CMS Experiment — [Wake of Partons](https://cms.cern/news/wake-partons) |
| [2] | Gizmodo — [Baby Universe Primordial Soup](https://gizmodo.com/the-baby-universe-really-was-a-goopy-soup-research-suggests-2000716678) |
| [3] | MIT Physics News — [Primordial Soup Study](https://physics.mit.edu/news/study-the-infant-universes-primordial-soup-was-actually-soupy/) |
| [4] | Tech Explorist — [Early Universe Hot Soupy](https://www.techexplorist.com/early-universe-just-hot-soupy/101962/) |
| [5] | Space.com — [LHC Primordial Soup](https://www.space.com/science/particle-physics/large-hadron-collider-reveals-primordial-soup-of-the-early-universe-was-surprisingly-soupy) |
| [6] | Physics Letters B — [Peer-reviewed journal](https://www.sciencedirect.com/journal/physics-letters-b) |
| [7] | Discover Magazine — [First Direct Evidence](https://www.discovermagazine.com/physicists-find-the-first-direct-evidence-that-the-universe-s-primordial-soup-behaved-like-a-liquid-48609) |
| [8] | Bjorken, J.D. (1983) — Phys. Rev. D **27**, 140 — Original Bjorken flow paper |
| [9] | Baier et al. (1997) — Nucl. Phys. B **484**, 265 — BDMPS formalism |
| [10] | Kovtun, Son, Starinets (2005) — Phys. Rev. Lett. **94**, 111601 — KSS bound |
| [11] | Casalderrey-Solana & Teaney (2006) — Phys. Rev. D **74**, 085012 — Mach cone in QGP |
| [12] | McLerran & Venugopalan (1994) — Phys. Rev. D **49**, 2233 — Color Glass Condensate |

---

*Analysis compiled from: CERN CMS (2026), Grok-4, DeepSeek-chat, and cross-referenced with primary QCD literature.*
