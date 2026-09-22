# CEREBRA-NEXUS — Unified Brain Code Project
**Path:** `~/cerebra-nexus/` **Status:** Plan / Architecture v1.0 **Date:** 2026-09-03

A synthesis of 5 mature brain-code systems in `~/` into one persistent, hardware-mappable mind. Not a new model — a nervous system that routes, grows, and remembers outside any single LLM.

> LLM is organ, not identity. Waves are computation. Growth is memory.

---

## 1. What We Fuse (provenance)

| Subsystem | Source `~/` | LOC / Tests | What it gives Nexus |
|-----------|-------------|------------|---------------------|
| **EchoMind** | `EchoMind/echo_mind.py:17` 542 LOC, 30 tests OK 0.074s | Mackey-Glass `N=256` reservoir `echo_mind.py:54` `rho=0.99`, RLS `echo_mind.py:106`, STFT `echo_mind.py:145`, char-LM `echo_mind.py:218` | Temporal front-end, phase-coded encoder, fastest test loop, fixed-point ready `DESIGN.md:6` |
| **gpu_brain2** | `gpu_brain2/brain_core.py:1` 801 LOC, 6.4M `brain_paths.npz` | 200×200 LIF sheet `brain_core.py:18`, 20% inhibitory, homeostasis+global inhib `BRAIN:118`, eligibility `BRAIN:94`, GPT2 patches `BRAIN:126`, SuperBrain router `super_brain.py:11`, ASCII heatmap `ascii_heatmap.py:184` | GPU spiking substrate + demo UX + distillation |
| **BRAIN** | `BRAIN/brain_core.py:77` 509 LOC | 500×500 previous gen, training sets `train_brain.py:6` | Training corpus, stability fixes (retired into gpu_brain2) |
| **Jeannie** | `jeannie_alive_next/src/jeannie/neural/kernel.py:27` 470 LOC kernel, ~10k total | 4-type LIF `kernel.py:20`, STP+BCM+STDP `plasticity.py`, 3D geometry `geometry.py:11`, growth/pruning `growth.py`, hash assemblies `concepts.py:33`, checkpoint `kernel.py:340`, runner `kernel.py:434`, supervisor/drives/memory/workspace | Persistent self, embodiment, safety |
| **memR** | `memR/*.m` 2229 LOC Octave, `CLAUDE.md` | `Output=Σ A sin(ωt+φ)` `CLAUDE.md`, φ=weight `memristor_vs_lissajous.m:240`, Z80 port map `memristor_interface.asm:10` `0x10/0x11/0x12`, Level 4 MZI fiber `photonic-neural-networks/.../02-mzi-mesh.md` 1ns | Analog/photonic compute fabric |
| **brain2qwerty** | `brain2qwerty/brain2qwerty_v2/` Meta 2026 | Conv+Transformer v1, CTC+contrastive+LLM v2, `neuralset==0.2.2`, HuggingFace BCBL `README.md:39` | MEG→text decoder as biological input channel |

4472 other `~/` topic stubs are ignored — no code.

---

## 2. Goal / Non-Goals

**Goal:** One process that (a) ingests audio/MEG/text/HR, (b) phase-encodes, (c) spikes through reservoir→sheet→field, (d) commits to SQLite+checkpoints, (e) speaks via frozen local LLM codec, (f) runs on CPU now, fiber later, with same Z80 interface.

**Non-Goals:** new foundation model training, human iPSC wet lab (use `brain2qwerty` datasets), beating GPT-4 fluency (beat it on persistence, temporal novelty, watt).

---

## 3. Quick Start (future)

```bash
cd ~/cerebra-nexus
python -m venv .venv && source .venv/bin/activate
pip install -e ".[dev]"        # numpy, textual, fastapi, uvicorn, httpx
pip install cupy-cuda12x transformers  # optional GPU sheet
pytest -q                     # 30 EchoMind tests + neural checkpoint tests
python -m cerebra_nexus demo --mode reservoir
python -m cerebra_nexus tui   # 3-pane: menu | heatmap | chat
```

---

## 4. Docs Map

- `docs/PLAN.md` — 7 phases, tasks, gates, timeline
- `docs/ARCHITECTURE.md` — layers, contracts, schemas
- `flows/FLOWCHART.txt` — single complete ASCII system flow
- `flows/FLOWCHART_DETAILED.txt` — layer-expanded flow with timing/buses
- `src/` — (to be populated Phase 0)

