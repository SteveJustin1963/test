# Lab 1: Optical Skyrmions via the Poisson Spot

**2nd Year University Physics Lab**  
Based on: Yao et al., *"Optical skyrmions in Poisson spots"*, Optica 13(6), 1184 (2026).  
DOI: 10.1364/OPTICA.591840

---

## Overview

In 2026, researchers at Nanyang Technological University demonstrated that the 200-year-old **Poisson spot** (bright spot at the centre of a disc's shadow) can simultaneously generate four distinct types of **optical skyrmions** — topologically protected vortex structures in light's vector fields.

This lab builds that experiment in simulation, from first principles, across three stages:

| Stage | File | Topic |
|-------|------|-------|
| 1 | `lab1_fresnel_poisson.py` | Fresnel diffraction and the Poisson spot |
| 2 | `lab1_skyrmion_topology.py` | Vectorial fields, Stokes parameters, skyrmion topology |
| 3 | `lab1_faultfinding.py` | Fault-finding: 8 deliberately broken functions |

**Run all stages:**
```bash
python lab1_fresnel_poisson.py
python lab1_skyrmion_topology.py
python lab1_faultfinding.py
```

All figures are saved to `figures/`.

---

## Prerequisites

```
numpy scipy matplotlib
```

Install with: `pip install numpy scipy matplotlib`

**Python version**: 3.9+. Note: `np.trapz` was removed in NumPy ≥ 2.0; this lab uses `scipy.integrate.trapezoid` throughout.

---

## Background Theory

### 1. The Poisson Spot (Fresnel Diffraction)

When coherent laser light illuminates a small circular opaque disc, Fresnel diffraction around the disc edge produces **constructive interference on-axis** — a bright spot at the centre of the geometric shadow. This is the Poisson (Arago) spot.

The complex field amplitude at an observation point is given by the **Fresnel-Kirchhoff diffraction integral**:

$$U(P) = \frac{i}{\lambda} \iint_{\Sigma} U_0(\xi, \eta) \frac{e^{ikr}}{r} \left( \frac{1 + \cos\theta}{2} \right) d\xi\, d\eta$$

where $\lambda$ is wavelength, $k = 2\pi/\lambda$, $r$ is the distance from aperture element to $P$, and $\Sigma$ is the unobstructed region around the disc.

**Babinet's Principle** provides a complementary view:

$$U_{\text{disc}} + U_{\text{aperture}} = U_{\text{free}}$$

The field behind a disc equals the free-space field minus the field from an equivalent open aperture. This lets us compute disc diffraction as the difference of two aperture propagations.

#### Fresnel Number

$$N_F = \frac{a^2}{\lambda z}$$

where $a$ is disc radius, $z$ is propagation distance. The regime depends on $N_F$:

| $N_F$ | Regime | Effect |
|--------|--------|--------|
| $\gg 1$ | Geometric optics | Sharp shadow, no spot |
| $\sim 1{-}5$ | Fresnel diffraction | **Poisson spot forms** |
| $\ll 1$ | Fraunhofer (far field) | Broad diffuse pattern |

For $a = 0.5\ \text{mm}$, $\lambda = 532\ \text{nm}$, $z = 23.5\ \text{cm}$: $N_F = 2.0$ — ideal for a bright Poisson spot.

#### Angular Spectrum Method (ASM)

The ASM is the numerically stable way to propagate an optical field:

1. Compute the 2D FFT of the input field: $\tilde{E}(f_x, f_y)$
2. Multiply by the free-space transfer function:

$$H(f_x, f_y) = \exp\!\left(i k_z z\right), \quad k_z = \sqrt{k^2 - (2\pi f_x)^2 - (2\pi f_y)^2}$$

3. Inverse FFT to get the propagated field.

Evanescent components ($k_x^2 + k_y^2 > k^2$) are removed by zeroing the imaginary part of $k_z$.

---

### 2. Optical Skyrmions

A **skyrmion** is a topologically protected, particle-like configuration of a vector field that wraps the unit sphere an integer number of times. Originally from nuclear physics (Tony Skyrme, 1962), they appear in magnetic materials and — as demonstrated in 2026 — in light's vector fields.

In optics, the vector field $\mathbf{n}(x,y)$ can be:
- **Spin**: direction of $\mathbf{s} \propto \mathbf{E}^* \times \mathbf{E}$ (spin angular momentum density)
- **Stokes**: normalised Stokes vector $(s_1, s_2, s_3) = (S_1, S_2, S_3)/S_0$ on the Poincaré sphere
- **E-field**: direction of the electric field vector
- **B-field**: direction of the magnetic field vector

#### Skyrmion Number (Topological Charge)

$$N_{sk} = \frac{1}{4\pi} \iint \mathbf{n} \cdot \left( \frac{\partial \mathbf{n}}{\partial x} \times \frac{\partial \mathbf{n}}{\partial y} \right) dx\, dy$$

This is an integer counting how many times $\mathbf{n}$ covers the unit sphere. It is a **topological invariant** — conserved under continuous deformations, making skyrmions robust to perturbations.

Typical values:
- $N_{sk} = -1$: Néel skyrmion (radial polarization winding)
- $N_{sk} = +1$: Anti-skyrmion
- $N_{sk} = 0$: Trivial (uniform) state

#### Skyrmion Taxonomy

A skyrmion texture is parameterised by:

$$\mathbf{n} = \begin{pmatrix} \sin f(r) \cos\psi(\phi) \\ \sin f(r) \sin\psi(\phi) \\ \cos f(r) \end{pmatrix}$$

- **Profile** $f(r)$: $f(0) = \pi$ (centre points down), $f(\infty) = 0$ (boundary points up)
- **Helicity** $\psi(\phi) = m\phi + \gamma$:
  - $\gamma = 0$: Néel (radial) type
  - $\gamma = \pi/2$: Bloch (tangential/vortex) type
  - $m = -1$: Anti-skyrmion ($N_{sk} = +1$)

#### Stokes Parameters

For a light field with Jones vector $(E_x, E_y)$:

$$S_0 = |E_x|^2 + |E_y|^2 \quad \text{(total intensity)}$$

$$S_1 = |E_x|^2 - |E_y|^2 \quad \text{(linear H/V)}$$

$$S_2 = 2\,\mathrm{Re}(E_x E_y^*) \quad \text{(linear ±45°)}$$

$$S_3 = 2\,\mathrm{Im}(E_x E_y^*) \quad \text{(circular)}$$

These satisfy the Stokes identity for fully polarized light:

$$S_0^2 = S_1^2 + S_2^2 + S_3^2$$

The normalised Stokes vector $(s_1, s_2, s_3) = (S_1, S_2, S_3)/S_0$ lives on the **Poincaré sphere** — the unit sphere of polarization states. A Stokes skyrmion is a full covering of this sphere within the beam cross-section.

---

## Stage 1: Fresnel Diffraction and the Poisson Spot

**File**: `lab1_fresnel_poisson.py`

### Setup

The simulation uses:
- **Laser**: $\lambda = 532\ \text{nm}$ (green)
- **Disc**: radius $a = 0.5\ \text{mm}$
- **Grid**: $512 \times 512$ pixels, $2\ \mu\text{m/pixel}$ → $1.024\ \text{mm}$ field of view
- **Beam**: Gaussian with $w_0 = 1\ \text{mm}$ (2× disc radius for strong spot)

### Checkpoint 1: Grid and Mask Sanity

Verifies that:
- Centre pixel is blocked (`mask = 0` at $r=0$)
- Corner pixels are open (`mask = 1` far from disc)
- Centre intensity of the input beam is near 1 (before blocking)

**Expected output:**
```
✓ CHECKPOINT 1 PASSED: Grid and mask are correct
```

### Checkpoint 2: Single Propagation at Optimal Distance

Propagates the blocked Gaussian beam to $z = 23.5\ \text{cm}$ ($N_F = 2.0$).

Verifies: **on-axis intensity > ring-region mean intensity** (Poisson spot is brighter than surroundings).

**Key parameter**: Fresnel number $N_F = a^2/(\lambda z) = (0.5\times10^{-3})^2 / (532\times10^{-9} \times 0.235) = 2.0$.

**Expected output:**
```
✓ CHECKPOINT 2 PASSED: Poisson spot detected (on-axis / ring = X.XX)
```

Saved figure: `figures/stage1_poisson_spot.png`

### Checkpoint 3: Multi-z Sweep

Sweeps propagation distance from $z = 5\ \text{cm}$ to $z = 50\ \text{cm}$ (5 values) and plots the Poisson spot intensity versus $z$, alongside the Fresnel number.

**Expected output:**
```
✓ CHECKPOINT 3 PASSED: Poisson spot present at multiple distances
```

Saved figure: `figures/stage1_z_sweep.png`

**Discussion question**: At what Fresnel number does the Poisson spot appear/disappear? What happens at very small $z$ (large $N_F$)?

### Checkpoint 4: Babinet's Principle

Verifies Babinet's principle by comparing two calculations:

1. **Direct**: propagate blocked Gaussian beam
2. **Babinet**: $U_{\text{disc}} = U_{\text{free}} - U_{\text{aperture}}$

RMS difference should be $< 1\%$.

**Expected output:**
```
✓ CHECKPOINT 4 PASSED: Babinet verified (RMS diff < 0.01)
```

**Mathematical note**: Babinet's principle is exact for scalar diffraction. Verify it numerically — any significant discrepancy reveals a coding error in the ASM propagation.

### Checkpoint 5: Disc Radius Parameter Sweep

Sweeps disc radius from $0.1\ \text{mm}$ to $0.7\ \text{mm}$ (keeping $z = 23.5\ \text{cm}$ fixed) and plots how the Poisson spot intensity varies with both disc radius and Fresnel number.

**Expected output:**
```
✓ CHECKPOINT 5 PASSED: Disc radius sweep complete
```

Saved figure: `figures/stage1_disc_sweep.png`

**Discussion question**: How does Poisson spot intensity scale with disc radius? At what radius does it vanish (explain physically)?

---

## Stage 2: Vectorial Fields and Skyrmion Topology

**File**: `lab1_skyrmion_topology.py`

### Setup

Extends Stage 1 with vectorial (polarized) light. The incident beam is a **radially polarized Laguerre-Gaussian beam**:

$$E_x = u(r,\phi)\cos\phi, \quad E_y = u(r,\phi)\sin\phi$$

where $u(r,\phi) = (r/w_0)^{|l|} e^{-r^2/w_0^2} e^{il\phi}$ is the LG mode amplitude with orbital angular momentum $l=1$.

### Checkpoint 1: Stokes Identity Verification

Tests the Stokes identity $S_0^2 = S_1^2 + S_2^2 + S_3^2$ for five polarization states:

| State | $E_x$ | $E_y$ | Expected $S_3$ |
|-------|-------|-------|----------------|
| H | 1 | 0 | 0 |
| V | 0 | 1 | 0 |
| D | $1/\sqrt{2}$ | $1/\sqrt{2}$ | 0 |
| R | $1/\sqrt{2}$ | $i/\sqrt{2}$ | +1 |
| L | $1/\sqrt{2}$ | $-i/\sqrt{2}$ | -1 |

**Expected output:**
```
✓ CHECKPOINT 1 PASSED: Stokes identity holds for all polarisation states (max error = 0.00e+00)
```

### Checkpoint 2: Vectorial Poisson Spot Maps

Computes the Stokes parameters, spin angular momentum $s_z \propto S_3$, Poincaré sphere coverage, and polarization ellipse quiver plot for the vectorial Poisson spot.

Saved figure: `figures/stage2_vectorial_maps.png`

**What to look for**: The polarization state should vary spatially across the beam — a prerequisite for a non-trivial Stokes skyrmion texture.

### Checkpoint 3: Skyrmion Number Computation

Two parts:

**Part A** — Analytical verification:  
Creates a textbook Néel skyrmion profile:

$$f(r) = \pi\,e^{-3r/r_{\max}}, \quad \psi(\phi) = \phi$$

and verifies $N_{sk} \approx -1$ (within 5%).

**Expected output:**
```
[Analytical Neel skyrmion] N_sk = -0.9962  (expected ≈ -1)
✓ Skyrmion number formula confirmed
```

**Part B** — Propagated field:  
Computes $N_{sk}$ for the propagated vectorial Poisson spot. Values between −1 and 0 are typical for the simplified Gaussian beam geometry; the student task is to optimise the beam profile and propagation distance to approach $|N_{sk}| \rightarrow 1$.

**Expected output:**
```
[Propagated field] N_sk = -0.XXX
  -> Fractional values expected: the propagated LG beam approximates
     but does not perfectly realise an ideal skyrmion texture.
     Task: adjust w0, z, beam mode to maximise |N_sk|.
✓ CHECKPOINT 3 PASSED
```

### Checkpoint 4: Four Simultaneous Skyrmion Types

Demonstrates the four skyrmion types observed in the NTU experiment:

| Type | Vector field | Physical meaning |
|------|-------------|-----------------|
| Stokes | $(s_1, s_2, s_3)$ | Polarization texture on Poincaré sphere |
| Spin | $(s_x, s_y, s_z)$ | Spin angular momentum density |
| E-field | $(\mathrm{Re}(E_x), \mathrm{Re}(E_y), I-I_0)$ | Electric field direction |
| B-field | $(-\mathrm{Re}(E_y), \mathrm{Re}(E_x), I_0-I)$ | Magnetic field (approx.) |

Saved figure: `figures/stage2_four_skyrmions.png`

### Checkpoint 5: Topology vs Propagation Distance

Sweeps $z$ from $5\ \text{cm}$ to $50\ \text{cm}$ and plots $N_{sk}$ vs $z$ and $N_F$.

Saved figure: `figures/stage2_topology_vs_z.png`

**Discussion question**: Is $N_{sk}$ constant with propagation distance? What does this imply about the topological protection of optical skyrmions?

---

## Stage 3: Fault-Finding Exercises

**File**: `lab1_faultfinding.py`

Eight functions have been deliberately broken. The checker at the bottom of the file tells you which exercises currently fail (bugs present) and which pass.

Run the checker:
```bash
python lab1_faultfinding.py
```

**Initial state**: Exercises 1–7 should **FAIL** (bugs present), Exercise 8 should **PASS** (correct).

### Exercise 1: Angular Spectrum Propagation (Wrong Sign)

**The bug**: The ASM transfer function uses the wrong sign:
```python
H = np.exp(-1j * kz * z)  # BUG: should be +1j
```

**How to find it**: The propagated field intensity is wrong — it appears to "collapse" rather than diffract outward. Compare with a free-space propagation of a Gaussian; the beam should expand, not shrink.

**Fix**: Change `exp(-1j * ...)` to `exp(+1j * ...)`.

**Why it matters**: The sign of the phase factor determines the direction of propagation. A negative sign propagates *backwards* in time (convergent wave), not forward.

### Exercise 2: Missing FFT Shift

**The bug**: The ASM is missing `fftshift`/`ifftshift`:
```python
E_prop = ifft2(fft2(E_pad) * H)  # BUG: missing fftshift/ifftshift
```

**How to find it**: The propagated intensity shows an artefact — a four-quadrant cross pattern, or the Poisson spot appears off-centre. This is the classic "DC corner" artefact from unshifted FFTs.

**Fix**:
```python
E_prop = fftshift(ifft2(fft2(ifftshift(E_pad)) * H))
```

**Why it matters**: `fft2` places zero frequency at the array corner. `fftshift` moves it to the centre, which is required for the transfer function $H(f_x, f_y)$ to be correctly aligned.

### Exercise 3: Wrong Stokes Parameter S2

**The bug**: $S_2$ uses the imaginary part instead of the real part:
```python
S2 = 2 * np.imag(Ex * np.conj(Ey))  # BUG: should be np.real
```

**How to find it**: The Stokes identity $S_0^2 = S_1^2 + S_2^2 + S_3^2$ fails. For diagonal polarization ($E_x = E_y = 1/\sqrt{2}$), $S_2$ should be $+1$ but gives $0$.

**Fix**: Change `np.imag` to `np.real`.

**Why it matters**: $S_2$ measures the projection onto $\pm 45°$ linear polarization; $S_3 = 2\,\mathrm{Im}(E_x E_y^*)$ measures circular polarization. Mixing them gives incorrect Poincaré sphere coordinates and wrong skyrmion topology.

### Exercise 4: Missing Normalisation Factor in N_sk

**The bug**: The $1/(4\pi)$ prefactor is missing from the skyrmion number:
```python
N_sk = np.sum(density) * dx**2  # BUG: missing 1/(4*pi)
```

**How to find it**: For a Néel skyrmion, the result is $\approx -4\pi \approx -12.57$ instead of $-1$.

**Fix**: Add `N_sk = (1/(4*np.pi)) * np.sum(density) * dx**2`.

**Why it matters**: The $4\pi$ factor normalises the solid angle of the full unit sphere. Without it, $N_{sk}$ is not an integer and loses its meaning as a winding number.

### Exercise 5: Wrong Fresnel Number Formula

**The bug**: Uses disc radius $a$ instead of $a^2$:
```python
return a / (wavelength * z)  # BUG: should be a**2
```

**How to find it**: For $a=0.5\ \text{mm}$, $\lambda=532\ \text{nm}$, $z=23.5\ \text{cm}$, the correct answer is $N_F = 2.0$. The buggy version gives $N_F = a/(\lambda z) = 0.5\times10^{-3}/(532\times10^{-9}\times0.235) \approx 3990$.

**Fix**: Change `a` to `a**2`.

**Why it matters**: $N_F = a^2/(\lambda z)$ has units of (m²)/(m·m) = dimensionless. Using $a$ gives wrong units and wrong regime classification.

### Exercise 6: Not Normalising Stokes Vector for N_sk

**The bug**: Computes $N_{sk}$ using raw (unnormalised) Stokes parameters:
```python
nx, ny, nz = S1, S2, S3  # BUG: should be S1/S0, S2/S0, S3/S0
```

**How to find it**: With non-uniform intensity ($S_0 \neq 1$), the unnormalised vector does not lie on the unit sphere. The result $N_{sk}$ is wrong. The checker tests with $S_0 = 1 + 0.5\sin(f)$.

**Fix**: Normalise: `nx, ny, nz = S1/S0, S2/S0, S3/S0`.

**Why it matters**: The skyrmion number formula requires $|\mathbf{n}| = 1$ everywhere. Without normalisation, the integrand $\mathbf{n} \cdot (\partial_x\mathbf{n} \times \partial_y\mathbf{n})$ is not the solid angle density.

### Exercise 7: Gradient Axis Swapped

**The bug**: The $x$-gradient is computed along the wrong axis:
```python
grad_x = np.gradient(field, dx, axis=0)  # BUG: axis=0 is y
grad_y = np.gradient(field, dx, axis=1)  # axis=1 is x
```

**How to find it**: The skyrmion number sign is wrong for non-symmetric textures. For an azimuthal texture (Bloch skyrmion), swapping $x$ and $y$ gradients flips the cross product and changes $N_{sk}$ sign.

**Fix**: Change `axis=0` to `axis=1` for `grad_x`, and `axis=1` to `axis=0` for `grad_y`.

**Why it matters**: NumPy arrays are row-major: `axis=0` iterates over rows (y direction), `axis=1` over columns (x direction). This is a classic and subtle numpy indexing trap.

### Exercise 8: Grid Centre (Correct)

This function is **not buggy**. For an even-sized grid, the centre pixel is at index $N/2$, which is correct. The checker should **PASS** for this exercise.

**Purpose**: Trains students to distinguish real bugs from correct-but-unusual code. Just because something looks odd (integer division for grid centre) doesn't mean it's wrong.

---

## Expected Checker Output

When all bugs are present (initial state):

```
=== FAULT-FINDING CHECKER ===
Ex 1: FAIL - Negative sign propagates wave backwards
Ex 2: FAIL - FFT shift artefact detected
Ex 3: FAIL - S2 error for diagonal polarisation
Ex 4: FAIL - N_sk = -12.57 (missing 1/4pi factor)
Ex 5: FAIL - Fresnel number wrong by factor a
Ex 6: FAIL - N_sk wrong with non-uniform intensity
Ex 7: FAIL - Wrong gradient axis
Ex 8: PASS - Grid centre is correct for even N
Score: 1/8 correct (target after fixing all bugs: 8/8)
```

After fixing all bugs:
```
Ex 1: PASS
Ex 2: PASS
...
Ex 8: PASS
Score: 8/8 correct
```

---

## Discussion Questions

1. **Poisson spot intensity**: For a plane wave, the on-axis intensity equals that of the unobstructed beam. Why is the ratio lower for a Gaussian beam?

2. **Topological protection**: The skyrmion number $N_{sk}$ is an integer (topological invariant). What kinds of perturbations can change it? What cannot?

3. **Four simultaneous skyrmions**: The NTU experiment observes Stokes, Spin, E-field, and B-field skyrmions simultaneously. Are these independent? How are they coupled through Maxwell's equations?

4. **Babinet's principle**: We verified Babinet's principle numerically. Can you derive it analytically from the Fresnel-Kirchhoff integral?

5. **Fresnel number and topology**: From Checkpoint 5 of Stage 2, how does $N_{sk}$ depend on $N_F$? At what Fresnel number does the skyrmion texture become well-defined?

---

## Extension Tasks

### E1: Structured Incident Beams
Replace the Gaussian beam with:
- A Laguerre-Gaussian beam (vortex, OAM $l = \pm 1, \pm 2$)
- A circularly polarized Gaussian
- A vector beam (azimuthal polarization)

How do these change the skyrmion types and numbers observed?

### E2: Multiple Discs (Skyrmion Arrays)
Place two or more discs in the beam path. Do the skyrmion numbers add? Can you create a skyrmion lattice?

### E3: Temporal Dynamics
Simulate a pulsed laser (replace the monochromatic field with a Gaussian pulse in frequency). How do the skyrmion textures evolve with the pulse envelope?

### E4: Spatial Light Modulator (SLM) Control
In experiment, an SLM can impose an arbitrary phase pattern on the input beam. Simulate different phase masks (spiral, binary grating, etc.) and see which maximise $|N_{sk}|$.

### E5: 3D Skyrmions (Hopfions)
3D topological structures called **Hopfions** (Hopf fibration) generalise skyrmions to full 3D space. Research: how would you simulate a 3D optical hopfion? What propagation distances are needed?

---

## Key Equations Reference

| Symbol | Formula | Units |
|--------|---------|-------|
| Fresnel number | $N_F = a^2/(\lambda z)$ | dimensionless |
| Wavenumber | $k = 2\pi/\lambda$ | rad/m |
| ASM transfer function | $H = e^{ik_z z}$, $k_z = \sqrt{k^2 - k_x^2 - k_y^2}$ | — |
| Stokes $S_0$ | $\|E_x\|^2 + \|E_y\|^2$ | (intensity) |
| Stokes $S_1$ | $\|E_x\|^2 - \|E_y\|^2$ | (intensity) |
| Stokes $S_2$ | $2\,\mathrm{Re}(E_x E_y^*)$ | (intensity) |
| Stokes $S_3$ | $2\,\mathrm{Im}(E_x E_y^*)$ | (intensity) |
| Stokes identity | $S_0^2 = S_1^2 + S_2^2 + S_3^2$ | — |
| Skyrmion number | $N_{sk} = \frac{1}{4\pi}\iint \mathbf{n}\cdot(\partial_x\mathbf{n}\times\partial_y\mathbf{n})\,dx\,dy$ | integer |
| Néel profile | $f(r) = \pi e^{-r/r_0}$, $\psi=\phi$ → $N_{sk}=-1$ | — |
| Anti-skyrmion | $f(r) = \pi(1-e^{-r/r_0})$, $\psi=-\phi$ → $N_{sk}=+1$ | — |

---

## References

1. Yao, J. et al., "Optical skyrmions in Poisson spots," *Optica* **13**(6), 1184 (2026). DOI: 10.1364/OPTICA.591840
2. Born & Wolf, *Principles of Optics*, Chapter 8 (Fresnel diffraction)
3. Goodman, *Introduction to Fourier Optics*, Chapter 4 (Angular Spectrum)
4. Shen, Y. et al., "Optical vortices 30 years on," *Light Sci. Appl.* **8**, 90 (2019)
5. Skyrme, T.H.R., "A unified field theory of mesons and baryons," *Nucl. Phys.* **31**, 556 (1962)
6. Tourrette & Garnett, *Photonic Skyrmions: Topological Light Fields* (review, 2024)
