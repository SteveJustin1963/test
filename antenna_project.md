# From Zepp to SDR: Rebuilding the 1936 Antenna Handbook in Python, Maths, and Hardware

**A 12-Week Second-Year University Engineering Project**

> *"A given amount of time and money spent on increasing antenna efficiency will do more to increase the strength of the distant received signal than increasing the power output of the transmitter several times."*
> — The Radio Antenna Handbook, 1936

---

## Overview

This project takes the 1936 *Radio Antenna Handbook* as a launchpad. You will:

1. **Derive** the physics from Maxwell's equations up through radiation resistance, standing waves, and impedance matching
2. **Simulate** every concept in Python — radiation patterns, Smith charts, transmission line behaviour, array factors
3. **Build** real RF circuits — a dipole antenna, Q-bar matching section, and a 2-element phased array
4. **Measure** everything with a NanoVNA and RTL-SDR, closing the loop between theory and reality

The 1936 engineers had slide rules. You have Python, SDR, and vector network analysers. Let's see how far the physics actually goes.

---

## Required Equipment

| Item | Approx. Cost |
|------|-------------|
| NanoVNA-H4 (or V2) | ~$60 AUD |
| RTL-SDR Blog V3 dongle | ~$45 AUD |
| ~10m 26 AWG enamelled copper wire | ~$8 |
| RG-58 coax, 3m | ~$6 |
| BNC connectors, panel-mount | ~$5 |
| SO-239 / PL-259 connectors | ~$5 |
| Air-core inductors (T50-2 toroid) | ~$8 |
| Variable capacitors (500pF) ×2 | ~$10 |
| Veroboard / FR4 strip | ~$5 |
| Ferrite balun core (FT-50-43) | ~$4 |

**Total ~$156 AUD.** Python stack: numpy, scipy, matplotlib (all free).

---

## Part 1 — The Mathematics (Weeks 1–3)

### 1.1 Maxwell's Equations to the Wave Equation

Start at the bedrock. In SI units:

$$\nabla \times \mathbf{E} = -\frac{\partial \mathbf{B}}{\partial t}, \qquad \nabla \times \mathbf{H} = \mathbf{J} + \frac{\partial \mathbf{D}}{\partial t}$$

$$\nabla \cdot \mathbf{D} = \rho, \qquad \nabla \cdot \mathbf{B} = 0$$

In free space with no sources, taking the curl of Faraday's law and substituting Ampere's:

$$\nabla^2 \mathbf{E} - \mu_0 \varepsilon_0 \frac{\partial^2 \mathbf{E}}{\partial t^2} = 0$$

This is the **wave equation**. The solution is a travelling wave at speed $c = 1/\sqrt{\mu_0 \varepsilon_0} = 3 \times 10^8$ m/s.

The wavelength–frequency relation the 1936 book uses:

$$\lambda = \frac{c}{f} = \frac{300{,}000 \text{ km/s}}{f \text{ (kHz)}} \text{ metres}$$

### 1.2 The Hertz Dipole — Vector Potential Approach

A thin wire of half-wavelength $L = \lambda/2$ carrying a sinusoidal current:

$$I(z) = I_0 \sin\!\left(k\!\left(\frac{L}{2} - |z|\right)\right), \qquad k = \frac{2\pi}{\lambda}$$

The **retarded vector potential** in the far field ($r \gg \lambda$):

$$A_z(\mathbf{r},t) = \frac{\mu_0}{4\pi} \int_{-L/2}^{L/2} \frac{I(z') e^{j(\omega t - kr)}}{r} dz'$$

After integration, the far-field electric field:

$$E_\theta = j\frac{\eta_0 I_0}{2\pi r} \cdot \frac{\cos\!\left(\frac{\pi}{2}\cos\theta\right)}{\sin\theta} \cdot e^{-jkr}$$

where $\eta_0 = \sqrt{\mu_0/\varepsilon_0} \approx 377\ \Omega$ is the impedance of free space.

The **time-averaged Poynting vector** (power per unit area):

$$S = \frac{|E_\theta|^2}{2\eta_0} = \frac{\eta_0 I_0^2}{8\pi^2 r^2} \cdot \frac{\cos^2\!\left(\frac{\pi}{2}\cos\theta\right)}{\sin^2\theta}$$

### 1.3 Deriving the 73-Ohm Radiation Resistance

Integrate the Poynting vector over a sphere of radius $r$:

$$P_{rad} = \oint S \cdot r^2 \sin\theta \, d\theta \, d\phi = \frac{\eta_0 I_0^2}{4\pi} \int_0^\pi \frac{\cos^2\!\left(\frac{\pi}{2}\cos\theta\right)}{\sin\theta} d\theta$$

The integral evaluates numerically to $\approx 1.218$, giving:

$$P_{rad} = \frac{\eta_0 \cdot 1.218}{4\pi} \cdot I_0^2 \approx 36.57 \cdot I_0^2 \quad \text{(quarter-wave, grounded)}$$

For a **centre-fed half-wave dipole in free space**, both halves contribute:

$$R_{rad} = 2 \times 36.57 = \mathbf{73.14\ \Omega}$$

**This is the number the 1936 book quotes. You just proved it from first principles.**

### 1.4 Transmission Line Theory — The Telegrapher's Equations

A transmission line has distributed parameters: resistance $R$, inductance $L$, conductance $G$, capacitance $C$ per unit length.

$$\frac{\partial V}{\partial x} = -(R + j\omega L) I, \qquad \frac{\partial I}{\partial x} = -(G + j\omega C) V$$

The characteristic impedance:

$$Z_0 = \sqrt{\frac{R + j\omega L}{G + j\omega C}} \xrightarrow{\text{lossless}} \sqrt{\frac{L}{C}}$$

For parallel-wire open-line feeders (spacing $d$, wire radius $a$):

$$Z_0 = \frac{120}{\sqrt{\varepsilon_r}} \cosh^{-1}\!\left(\frac{d}{2a}\right) \approx \frac{276}{\sqrt{\varepsilon_r}} \log_{10}\!\left(\frac{d}{a}\right)\ \Omega$$

For RG-58 coax: $Z_0 = 50\ \Omega$. For the Zepp 6-inch open feeder: $Z_0 \approx 600\ \Omega$.

The **voltage reflection coefficient** at a load $Z_L$:

$$\Gamma = \frac{Z_L - Z_0}{Z_L + Z_0}$$

**VSWR** (Voltage Standing Wave Ratio):

$$\text{VSWR} = \frac{1 + |\Gamma|}{1 - |\Gamma|}$$

For a 73 Ω dipole on 50 Ω coax: $\Gamma = (73-50)/(73+50) = 0.187$, VSWR = 1.46. Barely matters — but unmatched Zepp at end-feed point ($Z_L \approx 2400\ \Omega$) gives VSWR ≈ 48. That's why matching matters.

### 1.5 The Smith Chart as a Bilinear Transform

The Smith chart is the unit disk of the complex $\Gamma$-plane. Every normalised impedance $z = Z/Z_0 = r + jx$ maps to:

$$\Gamma = \frac{z - 1}{z + 1}$$

This is a **Möbius (bilinear) transformation**. Circles of constant $r$ map to circles, circles of constant $x$ map to circles — that's what creates the familiar chart grid.

Moving along a lossless transmission line of electrical length $\theta = \beta \ell$ rotates $\Gamma$ around the chart:

$$\Gamma(\ell) = \Gamma_L \, e^{-j2\beta\ell}$$

A **quarter-wave transformer** (Q-bar) of impedance $Z_1$ rotates $\Gamma$ by 180°, transforming:

$$Z_{in} = \frac{Z_1^2}{Z_L}$$

To match $Z_L = 73\ \Omega$ to $Z_0 = 50\ \Omega$: $Z_1 = \sqrt{73 \times 50} = 60.4\ \Omega$.

### 1.6 Array Factor and the Fourier Connection

For $N$ isotropic elements spaced $d$ apart along the z-axis, with progressive phase shift $\delta$:

$$AF(\theta) = \sum_{n=0}^{N-1} e^{jn(kd\cos\theta + \delta)}$$

This is a **Discrete Fourier Transform** of the element excitations. The radiation pattern of the array is:

$$E(\theta) = E_{element}(\theta) \times AF(\theta)$$

For a 2-element array with $d = \lambda/2$ and $\delta = \pi$ (endfire):

$$AF = 1 + e^{j(\pi\cos\theta + \pi)} = 1 - e^{j\pi\cos\theta}$$

Maximum at $\theta = 0°$ (along the axis), null at $\theta = 90°$ (broadside). That's an **endfire array** — the 1936 book's "end-fire array". The pattern is a Fourier transform of the aperture excitation. Pattern synthesis is inverse Fourier design.

---

## Part 2 — Python Simulation Suite (Weeks 2–5)

Install: `pip install numpy scipy matplotlib`

### Module 1: `dipole.py` — Radiation Pattern & Resistance


> **See [`dipole.py`](./dipole.py)**


### Module 2: `smith_chart.py` — Full Smith Chart Engine


> **See [`smith_chart.py`](./smith_chart.py)**


### Module 3: `transmission_line.py` — Standing Waves Visualised


> **See [`transmission_line.py`](./transmission_line.py)**


### Module 4: `array_factor.py` — Phased Array Beamforming


> **See [`array_factor.py`](./array_factor.py)**


### Module 5: `rhombic.py` — The 1936 Diamond Antenna, Computed


> **See [`rhombic.py`](./rhombic.py)**


### Module 6: `matching_optimizer.py` — Automated Q-bar/L-network Design


> **See [`matching_optimizer.py`](./matching_optimizer.py)**


---

## Part 3 — Hardware Builds (Weeks 5–9)

### Build 1: 20m Half-Wave Dipole with Q-bar Match

**Physical design at 14.075 MHz (FT8/WSPR segment):**

Using the 1936 formula with velocity factor 0.95:

$$L_{feet} = \frac{467.4}{f_{MHz}} = \frac{467.4}{14.075} = 33.21 \text{ ft} = 10.12 \text{ m (total)}$$

Each arm = 5.06 m of 26 AWG copper wire.

**Q-bar construction:**

The Q-bar needs $Z_1 = 60.4\ \Omega$. Construct from coax:
- RG-58 ($Z_0 = 50\ \Omega$) is close enough — VSWR reduces from 1.46 to 1.18
- For exact 60.4 Ω: use 2-wire line, spacing $d$, wire radius $a = 0.5$ mm:

$$d = 2a \cdot \cosh\!\left(\frac{Z_0 \sqrt{\varepsilon_r}}{120}\right) = 2 \times 0.5 \times \cosh\!\left(\frac{60.4}{120}\right) \approx 1.43 \text{ mm spacing}$$

**Quarter-wave length at 14.075 MHz:**

$$\ell_{\lambda/4} = \frac{c \times v_f}{4f} = \frac{3\times10^8 \times 0.659}{4 \times 14.075 \times 10^6} = 3.50 \text{ m of RG-58}$$

where $v_f = 0.659$ for RG-58.

**Circuit diagram:**

```
     Dipole arm 1                    Dipole arm 2
     (5.06 m) ──────────────────────── (5.06 m)
                         │
               ┌─────────┴─────────┐
               │     Feedpoint     │
               │      73 Ω         │
               └─────────┬─────────┘
                         │
              ┌──────────┴──────────┐
              │  Q-bar: 3.50m RG-58 │  Z0=50Ω, λ/4
              │  (or 60Ω twin lead) │
              └──────────┬──────────┘
                         │
                    BNC connector
                    To NanoVNA / transceiver
```

**Construction steps:**
1. Cut two wire arms to 5.06 m each. Strip 5 mm at each end.
2. Solder both wires to centre pin and shield of SO-239 connector respectively (half to centre, half to shield).
3. Cut RG-58 to exactly 3.50 m. Solder connectors both ends.
4. Connect feedpoint of dipole → Q-bar → NanoVNA port 1.
5. Perform open/short/load cal on NanoVNA at dipole feedpoint.
6. Sweep 13–16 MHz. Find resonance dip. Trim wire until $f_0 = 14.075$ MHz.

**Expected NanoVNA results:**
- Resonant frequency: 14.0–14.2 MHz (adjust by trimming)
- R at resonance: 70–80 Ω (±10% depending on height above ground)
- VSWR at resonance via Q-bar: < 1.5

### Build 2: EFHW (End-Fed Half-Wave) with Toroid Transformer

An EFHW is a single-wire antenna fed at one end — exactly the "end-fed Hertz" of the 1936 book. The feedpoint impedance is ~2400 Ω. We match to 50 Ω with a 49:1 toroid transformer (impedance ratio = $2450/50 = 49$, turns ratio $= \sqrt{49} = 7:1$).

**Toroid construction (FT-50-43 core, 14 MHz):**

Winding turns:
- Primary: 3 turns
- Secondary: 21 turns (7× primary for 49:1 impedance ratio)
- Use 26 AWG enamelled wire
- Wind primary through centre hole, secondary over full core

Resonating capacitor across primary:

$$X_C = Z_{primary} = 50\ \Omega \implies C = \frac{1}{2\pi f X_C} = \frac{1}{2\pi \times 14\times10^6 \times 50} \approx 227 \text{ pF}$$

Use a 220 pF NP0 cap across the primary.

**Circuit:**

```
Antenna wire (10.12 m)──────────────────────────────────
                                                        │
                                               ┌────────┴────────┐
                                               │ Toroid 49:1     │
                                               │ FT-50-43        │
                                               │ 3T:21T          │
                                               │ + 220pF cap     │
                                               └────────┬────────┘
                                                        │
                                                   50Ω coax
                                                   to radio
```

**Measurement test:** Connect NanoVNA, sweep 13–15 MHz. Look for:
- Impedance dip near 50 Ω at 14.075 MHz
- VSWR < 2:1 across 14.0–14.35 MHz

### Build 3: 2-Element Phased Endfire Array

Two dipoles spaced $\lambda/2 = 10.12$ m apart, fed with a 90° power splitter. This is the 1936 "end-fire array" but controlled digitally.

**Phase shift using a coax delay line:**

A 90° phase shift at 14 MHz using RG-58 ($v_f = 0.659$):

$$\ell_{90°} = \frac{c \times v_f}{4f} = \frac{3\times10^8 \times 0.659}{4 \times 14.075\times10^6} = 3.50 \text{ m}$$

**Power splitter circuit (hybrid coupler approximation):**

```
              From radio (50Ω)
                    │
         ┌──────────┴──────────┐
         │                     │
    3.50m RG-58           Direct feed
    (90° delay)                │
         │                     │
    Dipole 1              Dipole 2
    (0° phase)            (90° phase)
         │                     │
       Ground               Ground
```

This creates an endfire pattern. Swap delay line to other dipole to reverse beam direction.

**Expected gain:** +4.8 dBd (≈3 dB over a single dipole)

---

## Part 4 — NanoVNA & SDR Measurements (Weeks 8–12)

### 4.1 NanoVNA Measurement Protocol

**Calibration:**
```python
# Standard OSL calibration at dipole feedpoint
# 1. OPEN: disconnect wire, leave connector open
# 2. SHORT: short-circuit the connector with a wire
# 3. LOAD: connect 50Ω reference load (SMA terminator)
# Then connect through: coax + Q-bar to dipole
```

**Export S11 data from NanoVNA:**
- Connect via USB: `pip install nanovna`
- Or use NanoVNA-Saver (GUI) → Export Touchstone .s1p file

**Python analysis of NanoVNA data:**

> **See [`nanovna_analysis.py`](./nanovna_analysis.py)** — NanoVNA S1P parser — reads exported Touchstone files, plots VSWR, R+X, return loss, and S11 on Smith chart


### 4.2 SDR Pattern Measurement

Use RTL-SDR to measure receive signal strength from a known beacon at multiple orientations:

> **See [`sdr_pattern.py`](./sdr_pattern.py)** — RTL-SDR signal strength measurement and measured vs simulated polar pattern comparison


### 4.3 WSPR Spot Logging

Transmit low-power WSPR (0.1W is enough) from the built dipole and receive reception reports from wspr.rocks:

> **See [`wspr_logger.py`](./wspr_logger.py)** — WSPR spot fetching from wsprnet.org and visualisation (spot map + SNR vs distance)


---

## Part 5 — Advanced Extensions (Weeks 10–12)

### 5.1 Method of Moments (MoM) Thin-Wire Solver

The 1936 book's formulas assume sinusoidal current distribution. MoM finds the actual distribution numerically, handling mutual coupling, ground effects, and arbitrary geometries.

> **See [`mom_solver.py`](./mom_solver.py)** — Thin-wire Method of Moments (MoM) solver — computes actual current distribution and input impedance


### 5.2 Real-Time NanoVNA Closed-Loop Tuner

Connect Python to NanoVNA via USB. Read S11, compute required matching, actuate relays:

> **See [`auto_tuner.py`](./auto_tuner.py)** — Closed-loop NanoVNA + GPIO relay auto-tuner — reads S11 in real time and adjusts switched capacitors


---

## Project Deliverables & Timeline

| Week | Task | Deliverable |
|------|------|-------------|
| 1–2  | Derive Maxwell → 73Ω from scratch | Written derivation, LaTeX/PDF |
| 2–3  | Smith chart bilinear transform derivation | Proof + hand-drawn chart |
| 3–4  | `dipole.py` — patterns, Rr numerical | Plots + verified 73Ω |
| 4–5  | `smith_chart.py` + `matching_optimizer.py` | Q-bar/L-net length outputs |
| 5–6  | `transmission_line.py` + `array_factor.py` | Standing wave + beam steering plots |
| 6–7  | Build dipole + Q-bar | Physical antenna |
| 7–8  | Build EFHW + toroid transformer | Second antenna |
| 8–9  | NanoVNA measurements, `parse_s1p.py` | Measured vs theory comparison |
| 9–10 | SDR pattern measurement | Measured vs simulated polar plot |
| 10–11 | MoM solver + WSPR | MoM Z_in, WSPR spot map |
| 11–12 | Build 2-element phased array | Directional system |
| 12   | Final report | 15–20 pages |

---

## Assessment Criteria

| Component | Weight | What examiners want to see |
|-----------|--------|---------------------------|
| Mathematical derivations | 30% | Correct derivation of 73Ω, Smith chart from bilinear transform, array factor as FT |
| Python simulation quality | 25% | Clean code, documented, plots agree with theory |
| Hardware build quality | 20% | Resonates at correct frequency, stable construction |
| Measurement accuracy | 15% | NanoVNA data matches simulation within ±15% |
| Advanced extension | 10% | Any one: MoM, WSPR, auto-tuner, phased array |

---

## Key Equations Reference Card

| Quantity | Formula | 1936 Handbook §  |
|---------|---------|-------------------|
| Half-wave length | $L = 467.4 / f_{MHz}$ ft | Ch. I |
| Radiation resistance | $R_{rad} = 73.14$ Ω | Ch. I |
| Characteristic impedance | $Z_0 = 276 \log_{10}(d/a)$ Ω | Ch. III |
| Reflection coefficient | $\Gamma = (Z_L - Z_0)/(Z_L + Z_0)$ | Ch. III |
| VSWR | $(1+\|\Gamma\|)/(1-\|\Gamma\|)$ | Ch. III |
| Q-bar match | $Z_1 = \sqrt{Z_{src} Z_{load}}$ | Ch. IV |
| Quarter-wave transformer | $Z_{in} = Z_1^2 / Z_L$ | Ch. IV |
| Marconi length | $L = 233/f_{MHz}$ ft | Ch. I |
| Array factor | $AF = \sum_n e^{jn(kd\cos\theta + \delta)}$ | Ch. VI |
| Far-field E-field | $E_\theta \propto \cos(\pi\cos\theta/2)/\sin\theta$ | Ch. I |

---

## Why This Project Matters

The 1936 engineers designed the rhombic antenna, the Q-bar, the Zepp feeder — with only slide rules and physical intuition. The physics they discovered is still exact. The same 73-ohm radiation resistance, the same standing waves, the same Smith chart (invented in 1939, two years after this book).

What's changed is your tools. Python lets you compute in seconds what took Terman a week. A NanoVNA for $60 does what required a $50,000 HP network analyser in 1970. An RTL-SDR dongle receives signals from 500,000 km away.

**The 1936 handbook is not obsolete. It's a foundation. Build on it.**

---

*Synthesised from: The Radio Antenna Handbook (1936), Grok-4, DeepSeek-V3, deepseek-coder-v2:16b, and Claude Sonnet 4 — July 2026*
