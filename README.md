Use Ctrl + d or /bye to exit.
>>> **Rotational lines** → microwave regionexplain this part and shpw somee maths --**Rotational lines** → microwave region
# Rotational Spectroscopy: Microwave Region

## Physical Picture

Molecules aren't rigid — they **rotate** in space. When a molecule rotates, it has angular momentum, and (just like a person on a rotating stool) it has 
**quantized energy levels**. 

When a photon has *exactly* the right energy to bump the molecule from one rotational level to the next, the photon is **absorbed**. These energy gaps are 
tiny (compared to vibrations or electronic transitions), so the photons have low frequency and long wavelength → the **microwave region** (~1 GHz to 1000 
GHz, i.e., ~0.3 mm to 30 cm).

> ☝️ **Key rule**: A molecule absorbs microwaves only if it has a **permanent electric dipole moment** (heteronuclear diatomic molecules like HCl, CO, NO; 
not symmetric molecules like N₂, O₂, CO₂). The rotating dipole acts like a tiny antenna, coupling to the electromagnetic field.

---

## Deriving the Energy Levels

Treat the molecule as a **rigid rotor** — two masses $m_1$ and $m_2$ separated by a fixed distance $r$, rotating about their center of mass.

**Step 1: Reduce to a single particle**

The system can be replaced by a single particle of mass **reduced mass** $\mu$ at distance $r$ from the origin:

$$
\mu = \frac{m_1 m_2}{m_1 + m_2}
$$

**Step 2: Classical rotational energy**

The moment of inertia is

$$
I = \mu r^2
$$

so the rotational energy of a classical rotor is

$$
E_{\text{rot}} = \frac{L^2}{2I}
$$

where $L$ is the angular momentum.

**Step 3: Quantize angular momentum**

In quantum mechanics, the square of the angular momentum is quantized:

$$
L^2 = \hbar^2 \, J(J+1), \quad J = 0, 1, 2, \ldots
$$

Plugging this in:

$$
\boxed{\,E_J = \frac{\hbar^2}{2I} \, J(J+1)\,}
$$

where $J$ is the **rotational quantum number**. This is one of the cleanest results in all of quantum mechanics.

Define a convenient constant — the **rotational constant** in wavenumber units:

$$
B \equiv \frac{\hbar^2}{2I h c} \quad \left[\text{cm}^{-1}\right]
$$

so that

$$
E_J/hc = B\, J(J+1)
$$

---

## Selection Rule

For a molecule with a permanent dipole, the rotational selection rule from the dipole interaction is

$$
\boxed{\,\Delta J = \pm 1\,}
$$

So absorption goes from $J \to J+1$ (emission would be the reverse).

---

## The Microwave Absorption Frequencies

The energy gap between adjacent levels is

$$
\Delta E = E_{J+1} - E_J = \frac{\hbar^2}{2I}\Big[(J+1)(J+2) - J(J+1)\Big]
$$

The algebra simplifies beautifully:

$$
(J+1)(J+2) - J(J+1) = (J+1)\big[(J+2) - J\big] = 2(J+1)
$$

So

$$
\boxed{\,\Delta E = \frac{\hbar^2}{I}(J+1)\,}
$$

The **frequency** of the absorbed photon is

$$
\nu_J = \frac{\Delta E}{h} = \frac{\hbar}{2\pi I}(J+1) = 2B(J+1)\,c
$$

Or in **wavenumber** form (the usual unit in spectroscopy):

$$
\boxed{\,\tilde{\nu}_J = 2B(J+1) \quad \left[\text{cm}^{-1}\right]\,}
$$

with $J = 0, 1, 2, \ldots$ labeling the **initial** state.

---

## The Spectrum Looks Like This

| Transition | $J \to J+1$ | Frequency $\tilde{\nu}$ |
|:----------:|:-----------:|:-----------------------:|
| 1 | $0 \to 1$ | $2B$ |
| 2 | $1 \to 2$ | $4B$ |
| 3 | $2 \to 3$ | $6B$ |
| 4 | $3 \to 4$ | $8B$ |

**The spectrum is a series of equally-spaced lines**, separated by $2B$ cm⁻¹. The equal spacing is a signature of the rigid rotor model (a real molecule 
shows slightly decreasing spacing because of centrifugal stretching — the *non-rigid* rotor correction).

### Numerical example: HCl

- $r \approx 1.29 \times 10^{-10}$ m
- $\mu \approx 1.63 \times 10^{-27}$ kg
- $I = \mu r^2 \approx 2.7 \times 10^{-47}$ kg·m²
- $B \approx 10.6$ cm⁻¹

So lines appear at $\tilde{\nu} \approx 21, 42, 63, 84, \ldots$ cm⁻¹, which is the far-infrared / microwave region. ✓

---

## Why This Matters

1. **Measuring $B$ → bond length.** Since $B = \hbar^2/(2Ihc) = \hbar^2/(2\mu r^2 hc)$, the line spacing directly gives $r$. Microwave spectroscopy 
determines bond lengths to ~4 significant figures.
2. **Measuring $\mu$ → isotope ratios.** Different isotopologues (e.g., $^{35}$Cl vs. $^{37}$Cl) have different reduced masses, hence different spacings.
3. **Astronomy.** CO, NH₃, H₂O are routinely observed in interstellar clouds by their rotational fingerprints.
4. **Confirming quantum mechanics.** The $J(J+1)$ ladder is one of the earliest, sharpest confirmations that angular momentum is quantized.

---

**Want me to extend this to:**
- the **non-rigid rotor** (centrifugal distortion, real spectra)?
- the **relative line intensities** (Boltzmann distribution, $2J+1$ degeneracy)?
- **vibrational-rotational** coupling (going from pure microwaves into the infrared)?

