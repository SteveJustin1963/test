# CEREBRA-NEXUS — Deep Project Plan
**Folder:** `~/cerebra-nexus/` **Version:** 1.0

## 0. Principles

1. **Simulate first, solder second** — Octave/Python before protoboard (not breadboard: `memR/CLAUDE.md` parasitic <10kHz).
2. **Deterministic assemblies** — all concept→neurons via `blake2b(salt+concept)` `jeannie/concepts.py:33` or `0xBEEF/0xCAFE` `EchoMind:184,196` — reproducible across restarts.
3. **LLM as codec only** — `jeannie_alive_next` rule applies globally: supervisor gates all persistent writes.
4. **One wire, many neurons** — FDM `memR/lissajous_hardware_design.m:428` / STFT phase-wrap `EchoMind:158` is how we scale.
5. **Frequency discipline** — `ω >> ω0` (10-100×) `memR/CLAUDE.md`: DC = training, 1-10kHz = inference. Never mix.

---

## Phase 0 — Scaffolding (Week 1) — GATE: repo + tests green

**Deliverables:** `~/cerebra-nexus/` structure, `pyproject.toml`, CI local, vendored contracts.
- Create `pyproject.toml` (requires-python >=3.11, deps: numpy, pydantic, typer, rich, textual, fastapi, uvicorn, psutil, pyserial, bleak, httpx; optional cupy, transformers, torch for brain2qwerty; dev: pytest, ruff).
- Copy contracts: `jeannie/contracts.py` -> `src/cerebra_nexus/contracts.py` (`NeuralStimulus`, `FieldSnapshot` `kernel.py:12`).
- Vendor `EchoMind/echo_mind.py:1` -> `src/cerebra_nexus/reservoir.py`, `jeannie/neural/*` -> `src/cerebra_nexus/field/`, `gpu_brain2/brain_core.py` sheet -> `src/cerebra_nexus/sheet/`.
- Reproduce gates: `python EchoMind/run_tests.py` 30 OK (done 0.074s), `pytest jeannie_alive_next/tests/test_neural.py -k checkpoint` pass `test_neural.py:15`, Octave `run_sim_windowed.m` hysteresis PNG.
- Freeze `brain2qwerty` pins `neuralset==0.2.2` `brain2qwerty/pyproject.toml`.
- **Exit gate:** `pytest -q` green, `python -m cerebra_nexus demo --dry` prints snapshots, no GPU required.

## Phase 1 — Reservoir Front-End (Weeks 2-3) — GATE: STFT→reservoir→RLS predicts sine 50-step MAE <0.08

- Task 1.1: Wrap `EchoMind:145` `encode_stft(signal,sr=16000,n_fft=256,hop=64)` -> `encode_stft_standalone()` `echo_mind.py:425` as input bus. Threshold 70th percentile `echo_mind.py:165`, bounded [-1,1] `echo_mind.py:171`.
- Task 1.2: Adapt `step_encoded()` `echo_mind.py:92` `r=tanh(W·tanh(r)+signal[:N])` to drive both `reservoir.py` and `field/kernel.py:113` `stimulate()` pending injection `kernel.py:122`.
- Task 1.3: Unify encoders: `encode_neuro_vector()` `echo_mind.py:175` 5-D arousal/attention/reward/surprise/curiosity + `encode_text()` `echo_mind.py:189` D=32 mean-pool -> same `NeuralStimulus(concepts, strength, salience, valence)` `kernel.py:113`.
- Task 1.4: RLS readout `train_step()` `echo_mind.py:106` `W_out+=outer(e,g)` online, `novelty_threshold=2.5` `echo_mind.py:41` triggers replay `echo_mind.py:133` cooldown 20 `echo_mind.py:69`.
- Task 1.5: Bench: `demo()` `echo_mind.py:433` 300 steps sine `avg error last 50` <0.08, entropy 2-4 bits `echo_mind.py:413`, STFT nonzero <70% `test_echo.py`.
- **Risk:** Overfitting char-LM templates `echo_mind.py:274` - mitigate by template gate `is_degenerate()` `echo_mind.py:325`.

## Phase 2 — Persistent Field (Weeks 4-5) — GATE: checkpoint roundtrip + growth stable <5% firing

- Task 2.1: Integrate `NeuralFieldKernel` `kernel.py:27` `n=2048` default, `exc_fraction 0.80` `kernel.py:51`, `edges_per_neuron 10` local `growth.py:build_local_edges`, 4 types `kernel.py:20`.
- Task 2.2: Wire neuromod `neuromod.py` DA/NE/5HT `pulse()` `kernel.py:124` -> `inhibitory_scale`, `gain`, `learning_rate_scale`; adaptation `kernel.py:168` `+0.08 decay 0.92`.
- Task 2.3: Plasticity `plasticity.py`: STP `step_stp()` `kernel.py:142`, traces `step_traces()` `kernel.py:170`, delta `compute_weight_delta()` `kernel.py:173` BCM, clip exc [0.001,0.22] inh [-0.22,-0.001] `kernel.py:180`, homeostasis `thresholds+=0.0004*(rate_ema-0.035)` `kernel.py:185` clamp [0.60,1.80] `kernel.py:186`.
- Task 2.4: Geometry+growth `geometry.py:11` + `growth.py` - centroid spawn `kernel.py:204` `SPAWN_PER_ASSEMBLY`, prune silent `growth.step()` `kernel.py:196`, `checkpoint()` `kernel.py:340` `state_version 4` + JSON meta `kernel.py:369`.
- Task 2.5: Expose `snapshot()` `kernel.py:271` firing_rate/overload/underactivity + `heatmap_snapshot(columns=64)` `kernel.py:293` values/cell_types/spikes.
- **Exit:** `test_neural_checkpoint_roundtrip` `tests/test_neural.py:15` + legacy `test_legacy_checkpoint...` pass, firing 0.2-3.5% sustained.

## Phase 3 — GPU Sheet + Router (Weeks 6-7) — GATE: heatmap 60fps, `/train a|b` distills to sheet readout

- Task 3.1: Port `gpu_brain2/brain_core.py:18` `DEFAULT_GRID 200` sheet + `Brain(brain_core.py)` adapt `CUPY_ACCELERATORS="" CUB_DISABLED=1` `brain_core.py:2`. Keep CPU fallback (Jeannie kernel) when no CUDA.
- Task 3.2: Patches: input top-left 50x50, output bottom-right 50x50 `BRAIN:126`, `BLOB_SIZE=5` stable blobs `gpu_brain2/brain_core.py:22`, `fan_in 24 radius 8` `brain_core.py:19`. Homeostasis ported.
- Task 3.3: SuperBrain `super_brain.py:11` routing `Hybrid(>=0.85)->Facts->Smart->GPU` `super_brain.py:32` markers `FALLBACK_MARKERS` `super_brain.py:7` + `_maybe_distill()` teaching `brain_paths.npz` readouts.
- Task 3.4: TUI `tui.py:500` 3-pane (menu 20% | heatmap top-right | input 20%) `gpu_brain2/tui.py`, `ascii_heatmap.py:184` 40×? grid `_heatmap_tick` per step, keep Jeannie `heatmap_snapshot` unified.
- Task 3.5: Merge `BRAIN/train_brain.py:6` corpus (greetings/math/facts) into `trainers.py:230`, verify `/good /bad /reward /temp` `brain.py`.
- **Risk:** CUDA mix `brain_paths.npz` 6.4M shape mismatch `BRAIN:371` -> guard `expected_shape` check.

## Phase 4 — Analog/Photonic Fabric (Weeks 8-10) — GATE: Octave FDM 4 tones 1-4kHz FFT demux BER <1e-3

- Task 4.1: Simulate `memR/lissajous_hardware_design.m:428` 4 neurons FDM + `memristor_lissajous_transfer_function.m:252` phase shift, `SIMULATE_MEMRISTOR_WINDOWED.m:132` 4-panel.
- Task 4.2: Implement bus `Output=Σ A sin(ωt+φ)` `CLAUDE.md` on protoboard (not breadboard), RC bandpass `README.md:602` f0=1/(2π√RC), envelope diode+C `README.md:636`, PLL phase decode `README.md:551`.
- Task 4.3: Z80 bridge `memristor_interface.asm:10` `OUT (0x10) MUX, OUT (0x11) WRITE, IN A,(0x12)` identical for all levels `CLAUDE.md` universal flow. Arduino `tty` translates to piezo `~$2` or thermo-optic `30-50` for fiber `level 4`.
- Task 4.4: Build 2×2 MZI `photonic-neural-networks/.../02-mzi-mesh.md` 1550nm laser $30-50, 50:50 couplers $20-40, InGaAs PD $10-20, single-mode fiber. Target 1ns inference `CLAUDE.md` Level4.
- Task 4.5: Map `trained φ` from `lissajous_logic_gates.m:321` to hardware via same `brain_core` training export.
- **Gate:** FFT demux distinguishes 1-4kHz with guard bands `N_max=(fmax-fmin)/(Spacing×Safety)` `lissajous_hardware_design.m` -> 50-80 neurons @99kHz.

## Phase 5 — Biological Input (Weeks 11-12) — GATE: offline MEG decode CER <30% on BCBL sample

- Task 5.1: Install `brain2qwerty v1` `brain2qwerty_v1/pl_module.py` pins `pyproject.toml` python>=3.12, download `SpanishBCBL` `README.md:39` small subset, run `scripts/extract_predictions.py` ngram decode `ngram_decoding.py`.
- Task 5.2: Adapter: MEG spectrogram -> `encode_stft_standalone(N=256)` `echo_mind.py:425` -> `NeuralStimulus` -> field. Do not retrain v2 (embargo).
- Task 5.3: Optional: wire `polar_h10/hr_stream.py` BLE HR -> `encode_neuro_vector` arousal channel `echo_mind.py:175`.
- **Note:** Read-only input; no closed-loop organoid `Organoid intelligence/readme.md` wet-lab.

## Phase 6 — Supervisor, Memory, Safety (Weeks 13-14) — GATE: audit log + deterministic replay

- Task 6.1: Port `jeannie/memory/*` SQLite WAL FTS5 scoring `jeannie_alive_next/README.md`, `jeannie/safety/*` allowlist, `jeannie/research/*` boundary disabled default.
- Task 6.2: Global workspace: `snapshot().dominant_concepts` `kernel.py:287` + `overload/underactivity` `kernel.py:278` gate replay `EchoMind:131` cooldown.
- Task 6.3: REST `jeannie/api/server.py` localhost, `jeannie/cli.py` `typer`, `jeannie/tui/*` textual. EBM-CoT optional `Energy Based Models EBM/ebm-cot` as verifier `<10% latency` `ebm-cot/README.md`.
- Task 6.4: `dampen(0.85)` `kernel.py:334` emergency + `privacy-kill` files `jeannie_alive_next README`.

## Phase 7 — Integration, Benchmark, Release (Weeks 15-16)

- Task 7.1: End-to-end: mic/STFT or MEG -> reservoir -> field -> sheet readout (`gpu_brain2` vocab 8192 `BRAIN:140` + `smart_brain.py:349` fallback) -> LLM codec `qwen3.5:9b` `jeannie_alive_next README` -> TUI.
- Task 7.2: Metrics: reservoir prediction MAE, firing homeostasis 2-5%, novelty precision, MEG CER, latency (<1ms CPU path, <10ms GPU, ~1ns fiber path isolated), power.
- Task 7.3: Docs: `README_FLOWCHART.txt`-style map `memR/README_FLOWCHART.txt`, presentation `create_presentation_final.py` pattern.
- **Release:** tag `v0.1.0-nexus`, `MANIFEST.sha256`, systemd unit `jeannie_alive_next/systemd`.

## Timeline Summary

```
W1  ■ Phase0 scaffold
W2-3  ■■■ Phase1 reservoir
W4-5  ■■■ Phase2 field
W6-7  ■■■ Phase3 sheet+router
W8-10 ■■■■ Phase4 photonic
W11-12■■ Phase5 MEG
W13-14■■ Phase6 supervisor
W15-16■■ Phase7 release
```

## Risk Register

- CUDA fragility (`cupy 14 + cu13 headers`) -> keep `CUB_DISABLED` + CPU Jeannie fallback always on.
- Memristor decay (hours) -> use `MCP4131` emulator `memR/README.md` for dev, fiber for prod.
- Template collapse `EchoMind:325` -> enforce `is_degenerate` + `_infer_template` `EchoMind:274` + last-template fallback.
- Topic folder noise -> `.gitignore` `*/*` allowlist only `src docs flows`.
