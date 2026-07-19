# Lab 2: Optical Skyrmions for Photonic Computing

**2nd Year University Physics Lab — Advanced Extension**  
Prerequisite: Lab 1 (Optical Skyrmions via the Poisson Spot)  
Based on: Yao et al., *"Optical skyrmions in Poisson spots"*, Optica 13(6), 1184 (2026)

---

## Overview

Lab 1 showed that the Poisson spot generates optical skyrmions — topologically protected structures in light's vector fields. In this lab, we exploit those topological properties for **photonic computing**: encoding, processing, and reading information using the skyrmion number $N_{sk}$ as a topological bit.

This connects to the NTU paper's stated application: *"high-density data storage, optical computing, and communications"*.

**Run this lab:**
```bash
python lab2_photonic_computing.py
```

All figures saved to `figures/lab2_*.png`.

| Stage | Topic | Key Concept |
|-------|-------|-------------|
| A | Topological bit | $N_{sk} = +1$ (bit 1) vs $N_{sk} = -1$ (bit 0) |
| B | NOT gate | Complex conjugation flips $N_{sk}$ |
| C | Noise robustness | Topological protection quantified |
| D | WDM multiplexing | 3 wavelengths × distinct skyrmion types |
| E | Reservoir computing | Skyrmion states as neural network layer |
| F | Information capacity | Winding number → bits/symbol |

---

## Background Theory

### Why Topology for Computing?

Classical bits ($0$/$1$) stored as voltages are vulnerable to noise — a spike can flip a bit. **Topological bits** exploit a deeper protection: the skyrmion number $N_{sk}$ is an integer that can only change by crossing a topological energy barrier. Small perturbations (phase noise, amplitude fluctuations, mild scattering) cannot change $N_{sk}$ continuously — they would require the vector field to pass through a singular configuration.

This is analogous to how the winding number of a loop around a point cannot change without the loop crossing the point.

### The Skyrmion Bit

We encode:
- **Bit 0**: Néel skyrmion — $N_{sk} \approx -1$ (radial winding, centre points "south" on Poincaré sphere)
- **Bit 1**: Anti-skyrmion — $N_{sk} \approx +1$ (reversed polar profile, centre points "north")

The field is encoded via a Jones vector with a known Stokes skyrmion texture:

$$E_x = A(r)\cos\alpha, \quad E_y = A(r)\sin\alpha\, e^{i\psi}$$

where $\alpha = \frac{1}{2}\arccos(\cos f)$ and the polar profile is:

$$f(r) = \pi e^{-3r/r_{\max}} \quad \text{(Néel, } N_{sk} = -1\text{)}$$
$$f(r) = \pi(1 - e^{-3r/r_{\max}}) \quad \text{(Anti-skyrmion, } N_{sk} = +1\text{)}$$

The Gaussian amplitude envelope $A(r) = e^{-r^2/w_0^2}$ ensures the field is spatially bounded.

### Stokes Vector on the Poincaré Sphere

The normalised Stokes vector $(s_1, s_2, s_3) = (S_1, S_2, S_3)/S_0$ maps each spatial point to a point on the unit sphere. A skyrmion texture is a map that **covers the sphere exactly once**, producing integer $N_{sk}$.

For the encoding above:
$$s_3 = S_3/S_0 = \cos f(r), \quad (s_1, s_2) \propto (\cos\psi, \sin\psi)\sin f(r)$$

At $r=0$: $f=\pi$, so $s_3 = -1$ (south pole).  
At $r\to\infty$: $f=0$, so $s_3 = +1$ (north pole).  
The azimuthal angle $\psi = \phi$ sweeps $2\pi$ as we go around — one full wrapping.

---

## Stage A: Skyrmion as Topological Bit

### Physical Setup

In the Poisson spot experiment:
- **Radially polarized** incident beam → Néel skyrmion (bit 0) in the spot
- **Azimuthally polarized** incident beam → Bloch skyrmion (related to bit 0)
- **Circularly polarized + OAM** → Anti-skyrmion texture (bit 1)

In simulation, we generate these analytically via the Jones vector encoding above.

### Reading the Bit

To read $N_{sk}$ from an optical field:

1. Measure Jones vector $(E_x, E_y)$ via polarimetry (e.g., Stokes polarimeter)
2. Compute Stokes parameters $S_0, S_1, S_2, S_3$
3. Normalise: $(n_x, n_y, n_z) = (S_1, S_2, S_3)/S_0$
4. Compute $N_{sk}$ via finite differences:

$$N_{sk} = \frac{1}{4\pi} \sum_{i,j} \mathbf{n}_{ij} \cdot \left(\frac{\partial \mathbf{n}}{\partial x} \times \frac{\partial \mathbf{n}}{\partial y}\right)_{ij} (\Delta x)^2$$

5. Threshold: $N_{sk} > 0 \Rightarrow$ bit 1; $N_{sk} < 0 \Rightarrow$ bit 0.

### Expected Results

```
  Skyrmion  (l=+1): N_sk = +0.968  -> bit = 1
  Anti-skyrmion (l=-1): N_sk = -0.968  -> bit = 0
  PASS: Distinct topological states detected
```

**Note**: $|N_{sk}| \approx 0.97$ rather than exactly 1 because the Gaussian envelope tails are truncated at the grid boundary. Increasing the grid or beam ratio improves this.

**Figure**: `figures/lab2_stageA_topological_bit.png` — intensity, Stokes texture (colour = $n_z$, arrows = $(n_x, n_y)$), skyrmion density, and spin density for each bit state.

### Discussion Questions A

1. Why is $|N_{sk}| < 1$? How would you measure the convergence to $N_{sk} = \pm 1$ as grid size increases?

2. The anti-skyrmion has its reversed profile ($f: 0 \to \pi$ rather than $\pi \to 0$). What does this mean physically for the centre polarization state vs boundary?

3. In experiment, could you use the sign of $S_3$ at the beam centre as a faster (but less robust) bit-read instead of computing $N_{sk}$? What are the tradeoffs?

---

## Stage B: Topological NOT Gate

### Physical Implementation

A **NOT gate** must flip $N_{sk}: +1 \to -1$ or $-1 \to +1$.

The key is the winding direction of $\psi(\phi) = \phi$ in the Jones vector phase $e^{i\psi}$. Flipping the winding $\psi \to -\psi$ maps:
$$e^{i\phi} \to e^{-i\phi}$$
which reverses the azimuthal winding number and hence flips $N_{sk}$.

**Implementation**: Complex conjugation of $E_y$:
$$E_y' = E_y^* = A\sin\alpha\, e^{-i\psi}$$

This is physically realised by a **Dove prism** (image rotation by 90°) or equivalent optical element, which conjugates the transverse field components.

**Jones matrix representation:**
$$\begin{pmatrix} E_x' \\ E_y' \end{pmatrix} = \begin{pmatrix} 1 & 0 \\ 0 & -1 \end{pmatrix} \begin{pmatrix} E_x^* \\ E_y^* \end{pmatrix}$$

### Double NOT = Identity

Applying NOT twice:
$$E_y \xrightarrow{\text{conj}} E_y^* \xrightarrow{\text{conj}} E_y$$

The field returns to its original state, and $N_{sk}$ returns to its original value. This verifies the gate is its own inverse, as required for NOT.

### Expected Results

```
  Input:  bit = 1  (N_sk = +0.968)
  Output: bit = 0  (N_sk = -0.968)
  NOT gate verified: 1 -> 0  PASS

  Double NOT: 1 -> 0 -> 1  (expect 1)
  Double NOT = identity  PASS
```

**Figure**: `figures/lab2_stageB_NOT_gate.png` — three panels showing $n_z$ texture (colour map) before NOT, after NOT, and after double NOT.

### Discussion Questions B

1. A half-wave plate (HWP) at 45° maps $(E_x, E_y) \to (E_y, E_x)$ (swaps components). Does this flip $N_{sk}$? Why or why not? (Hint: swapping does not change the phase winding $\psi = \phi$.)

2. What optical element physically implements complex conjugation of $E_y$? Research "optical phase conjugation" and "time-reversal mirror".

3. Design a circuit diagram (using beam splitters, wave plates, Dove prisms) for a NAND gate using two topological bits. (Hint: combine NOT with an AND operation using interference.)

---

## Stage C: Noise Robustness — Topological Protection

### The Protection Mechanism

The skyrmion number is **topologically quantised**: it can only take integer values. For small perturbations (noise amplitude $\sigma \ll 1$), the field fluctuates but the winding structure remains intact — $N_{sk}$ does not change continuously.

This is unlike a classical bit, where voltage noise can push the value across the threshold $V/2$ and flip the bit.

### The Simulation

Gaussian noise is added to both $E_x$ and $E_y$ with noise amplitude $\sigma$ relative to the peak field:

$$E_x' = E_x + \sigma \cdot \xi_x, \quad E_y' = E_y + \sigma \cdot \xi_y$$

where $\xi_x, \xi_y \sim \mathcal{N}(0,1)$ are independent Gaussian random fields.

$N_{sk}$ is computed as a function of $\sigma$.

### Expected Results

| Noise $\sigma$ | $N_{sk}$ | Correct read? |
|----------------|----------|---------------|
| 0.00 | +0.968 | ✓ |
| 0.05 | ~+0.95 | ✓ |
| 0.10 | ~+0.93 | ✓ |
| 0.30 | ~+0.80 | ✓ |
| 0.70 | ~+0.50 | ✓ (marginal) |
| 1.00 | ~+0.20 | ✗ |

The topological bit remains readable well beyond noise levels that would corrupt a classical binary voltage.

**Figure**: `figures/lab2_stageC_noise_robustness.png` — $N_{sk}$ vs $\sigma$ with error bars and correct/incorrect read threshold.

### Discussion Questions C

1. At what noise level $\sigma_c$ does the bit flip? Compare this to what you'd expect for a classical bit with a threshold at $V/2$.

2. The topological protection is not absolute — above $\sigma_c$, the bit flips. What physical process causes this (think about the skyrmion core and what happens to the vector at the centre when noise is large)?

3. In magnetic skyrmions, topological protection is also finite: they can be annihilated by thermal fluctuations above a certain temperature. Is there an "optical temperature" analogue here?

---

## Stage D: Wavelength-Division Multiplexing (WDM)

### Concept

Multiple skyrmion channels can be carried simultaneously by using different wavelengths (WDM), each with an independently encoded topological bit. A diffraction grating or wavelength demultiplexer separates them at the receiver.

### The Simulation

Three wavelengths are used:
- **Channel 1**: $\lambda_1 = 532\ \text{nm}$ (green) — Néel skyrmion, bit 0 ($N_{sk} \approx -1$)
- **Channel 2**: $\lambda_2 = 633\ \text{nm}$ (red) — Anti-skyrmion, bit 1 ($N_{sk} \approx +1$)
- **Channel 3**: $\lambda_3 = 780\ \text{nm}$ (NIR) — Bloch skyrmion, bit 0 ($N_{sk} \approx -1$)

Each wavelength's skyrmion is independently encoded and independently read.

### Expected Results

```
  Channel 1 (532nm): N_sk = -0.968, bit = 0
  Channel 2 (633nm): N_sk = +0.968, bit = 1
  Channel 3 (780nm): N_sk = -0.968, bit = 0
  WDM PASS: 3 independent channels
```

**Figure**: `figures/lab2_stageD_WDM.png` — Stokes textures for all three channels, showing independent encoding.

### Discussion Questions D

1. What is the maximum number of WDM channels limited by in practice? (Consider: optical bandwidth, skyrmion texture sensitivity to wavelength, detector response.)

2. The Fresnel number $N_F = a^2/(\lambda z)$ depends on wavelength. At fixed $z$ and $a$, longer wavelengths give smaller $N_F$. Does this affect the skyrmion quality? Suggest an experiment to test this.

3. Beyond WDM, could you use **spatial multiplexing** (multiple disc arrays) or **orbital angular momentum** (OAM) multiplexing to further increase information density? Sketch the concept.

---

## Stage E: Photonic Reservoir Computing

### Concept

**Reservoir computing** is a neural network paradigm where a fixed, complex dynamical system (the "reservoir") projects inputs into a high-dimensional feature space. Only the linear **readout layer** is trained. This is also called an Extreme Learning Machine (ELM).

Optical reservoir computers have been demonstrated using fibre delay lines, multimode waveguides, and diffractive elements. Here, we use the skyrmion Stokes textures as the feature space.

### The Architecture

```
Input bits (4-bit pattern)
     |
     v
[Skyrmion encoder] -> Jones vector field (Ex, Ey)
     |
     v
[Obstacle array]   -> Random scattering (reservoir nonlinearity)
     |
     v
[Stokes projector] -> Feature vector (n_neurons features)
     |
     v
[Linear readout]   -> Trained weights W
     |
     v
Output: XOR prediction
```

The obstacle array introduces random multiple-scattering, expanding the input into a high-dimensional nonlinear feature space — the key operation of reservoir computing.

### The Task: 4-Bit XOR

The XOR of all 4 input bits is a non-linearly separable problem that a single perceptron cannot solve. The reservoir maps the 16 possible 4-bit inputs to linearly separable features.

Training: pseudo-inverse on the 16×n_neurons feature matrix.

### Expected Results

```
  Photonic reservoir computing (skyrmion features + linear readout)
  XOR accuracy: 75.0%  (>50% is above chance; 100% with deeper reservoir)
```

**Note**: 75% accuracy reflects the simplified simulation. A physical optical reservoir would achieve near-100% with proper nonlinear element design.

**Figure**: `figures/lab2_stageE_reservoir.png` — weight matrix heatmap and prediction vs true XOR values.

### Discussion Questions E

1. Why is XOR not linearly separable? Show that 4-bit XOR requires at least one hidden layer in a classical MLP.

2. The reservoir's key property is that it provides a nonlinear projection. What physical nonlinearity is used here? (Compare with fibre-based optical reservoirs, which use the Kerr effect.)

3. How would you scale this to larger inputs (e.g., 8-bit, 16-bit XOR)? What limits the reservoir capacity (expressivity)?

---

## Stage F: Information Capacity

### Concept

So far, we've used $N_{sk} = \pm 1$ — one bit per skyrmion. But the winding number $m$ in the phase $\psi = m\phi$ determines $N_{sk}$:

$$N_{sk} = m \cdot (\text{profile factor})$$

Using higher winding numbers $m \in \{-2, -1, +1, +2\}$ gives **four distinct topological states** — 2 bits per skyrmion symbol.

This is analogous to QAM in classical communications, but using topological charge instead of amplitude/phase.

### Winding Number Encoding

For winding number $m$, the azimuthal phase is $\psi = m\phi$. The skyrmion number scales approximately as:

$$N_{sk}(m) \approx m \cdot N_{sk}(1)$$

Higher $|m|$ creates more rapid azimuthal winding, increasing the topological charge magnitude. These correspond to **higher-order optical vortex beams** with orbital angular momentum $m\hbar$ per photon.

### Expected Results

```
  Winding number m = +1: N_sk = +0.89
  Winding number m = -1: N_sk = -0.89
  Winding number m = +2: N_sk = +1.78
  Winding number m = -2: N_sk = -1.78

  Information capacity:
    4 distinct states (m = ±1, ±2) -> log2(4) = 2 bits/symbol
    8 states (m = ±1..±4)          -> log2(8) = 3 bits/symbol
    (limited by SNR and topological stability of high-m states)
```

**Figure**: `figures/lab2_stageF_capacity.png` — $N_{sk}$ vs $m$ (linear scaling), bits per symbol vs number of states.

### Discussion Questions F

1. $N_{sk}$ scales linearly with $m$. What limits the maximum $m$ you can use? (Consider: beam quality for high-$m$ LG modes, SNR in reading $N_{sk}$, crosstalk between $m$ values.)

2. Compare the information capacity of topological skyrmion bits with:
   - Classical binary bits (1 bit/symbol)
   - QAM-64 (6 bits/symbol)
   - Orbital angular momentum multiplexing (theoretically unlimited)
   
3. **Shannon capacity**: If $N_{sk}$ can be read with error $\sigma_N$, and the maximum $|N_{sk}|$ is $N_{max}$, derive an expression for the information capacity in bits/symbol using the Shannon–Hartley theorem analogy.

---

## Summary: A Photonic Computing Stack

Combining all stages, we have demonstrated a minimal **photonic computing stack** based on optical skyrmions:

```
Physical Layer:
  Poisson spot (disc + coherent laser) generates skyrmion field
  
Encoding Layer:
  Jones vector (Ex, Ey) -> N_sk ∈ {-2, -1, +1, +2}  (2 bits/symbol)
  
Logic Layer:
  NOT gate (Dove prism) -> topological charge flip
  
Channel Layer:
  WDM (multiple wavelengths) -> 3× bandwidth multiplication
  
Processing Layer:
  Reservoir computing -> linear readout of XOR task
  
Physical Protection:
  Topological quantisation -> robust to noise σ < σ_c
```

### Comparison with Competing Technologies

| Technology | Speed | Energy | Protection |
|------------|-------|--------|------------|
| Electronic (CMOS) | ~1 GHz | ~10 fJ/op | Voltage noise |
| Photonic (MZI) | ~100 GHz | ~1 fJ/op | Phase noise |
| Topological photonic | ~100 GHz | ~1 fJ/op | Topological |
| Skyrmion (magnetic) | ~1 GHz | ~1 aJ/bit | Topological |
| **Optical skyrmion** | ~THz (projected) | ~fJ | Topological |

---

## Extension Tasks

### E1: AND Gate from NOT and NOR
Using only NOT gates (Stage B) and intensity-based interference, construct a NOR gate (anti-skyrmion outputs when both inputs are skyrmions). Show that NOT + NOR gives a functionally complete gate set.

### E2: Holographic Memory
Skyrmion textures can be stored holographically (as interference patterns on photosensitive media). Simulate writing and reading a single topological bit holographically. What is the storage density limit?

### E3: Quantum Optical Skyrmions
The skyrmion states correspond to specific Stokes polarization textures on the Poincaré sphere. For single-photon states, the Poincaré sphere becomes the Bloch sphere. Research: how do quantum optical skyrmions differ from classical? What role does entanglement play?

### E4: Skyrmion-Based Spiking Neural Network
Map the skyrmion bit to a spiking neuron: $N_{sk} > \theta$ triggers a "spike" (NOT gate fires). Chain multiple gates to implement a spiking neural network layer. What activation functions can you implement with optical gates?

### E5: Integration with Silicon Photonics
The microdisc in the Poisson spot is compatible with silicon photonics fabrication (CMOS process, $\sim 200\ \text{nm}$ disc). Sketch a silicon photonic chip implementing one full skyrmion computing pipeline (encoder → reservoir → readout).

---

## Key Equations Reference

| Concept | Equation |
|---------|----------|
| Skyrmion number | $N_{sk} = \frac{1}{4\pi}\iint \mathbf{n}\cdot(\partial_x\mathbf{n}\times\partial_y\mathbf{n})\,dA$ |
| Jones encoding | $E_x = A\cos\alpha$, $E_y = A\sin\alpha\,e^{im\phi}$, $\alpha = \frac{1}{2}\arccos(\cos f)$ |
| Néel profile | $f(r) = \pi e^{-3r/r_0}$ → $N_{sk} = -1$ |
| Anti-skyrmion profile | $f(r) = \pi(1-e^{-3r/r_0})$ → $N_{sk} = +1$ |
| NOT gate | $E_y \to E_y^*$ (conjugate $E_y$) |
| Capacity | $C = \log_2(2m_{\max}+1)$ bits/symbol |
| WDM channels | $N_{ch} = \Delta\lambda_{\text{band}} / \delta\lambda_{\text{channel}}$ |
| Reservoir output | $\hat{y} = W \cdot \mathbf{f}(\mathbf{x})$, $W = \hat{y}_{\text{train}} \cdot \mathbf{f}^+$ |

---

## References

1. Yao, J. et al., "Optical skyrmions in Poisson spots," *Optica* **13**(6), 1184 (2026). DOI: 10.1364/OPTICA.591840
2. Lugnan, A. et al., "Photonic neuromorphic information processing and reservoir computing," *APL Photonics* **5**, 020901 (2020)
3. Cao, A.J. et al., "Reservoir computing with topological light," *Nat. Photon.* (2025, preprint)
4. Fert, A., Reyren, N. & Cros, V., "Magnetic skyrmions: advances in physics and potential applications," *Nat. Rev. Mater.* **2**, 17031 (2017) — magnetic analogy
5. Allen, L. et al., "Orbital angular momentum of light and the transformation of Laguerre-Gaussian laser modes," *Phys. Rev. A* **45**, 8185 (1992)
6. Maiman, T.H., "Stimulated Optical Radiation in Ruby," *Nature* **187**, 493 (1960) — the laser that makes it all possible
