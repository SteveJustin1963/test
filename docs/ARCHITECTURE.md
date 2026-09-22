# CEREBRA-NEXUS — Architecture
**Folder:** `~/cerebra-nexus/` **Refs:** `EchoMind/echo_mind.py:17`, `jeannie_alive_next/src/jeannie/neural/kernel.py:27`, `gpu_brain2/brain_core.py:1`, `memR/CLAUDE.md`, `brain2qwerty/brain2qwerty_v2/`

## 0. Invariant

```
Senses -> Phase Encoders -> Reservoir -> Field -> Sheet -> Codec -> Memory/Safety -> World
                    ^-------------------------------------------------------|
                               Z80-compatible bus (0x10/0x11/0x12) everywhere
```

Same 3 ports drive memristor, piezo-fiber, or pure software: `OUT (0x10) MUX` select cell/λ, `OUT (0x11) WRITE` set φ/W, `IN A,(0x12)` read interference. `memR/memristor_interface.asm:10` universal.

---

## 1. Layer Cake (7 layers, bottom = physics)

```
L0  Physics Fabric .......... Octave sim + protoboard + fiber MZI + optional MCP4131/Si photonics
L1  Sensing Bus ............. mic STFT, MEG, text, HR/BLE
L2  Phase Encoders .......... STFT wrap + neuro-vector + char-emb + concept hash
L3  Reservoir ............... EchoMind Mackey-Glass + STP (temporal memory)
L4  Neural Field ............ Jeannie 4-type LIF field + geometry/growth/neuromod
L5  Cortical Sheet .......... gpu_brain2 LIF sheet + SuperBrain router (choice/fluency)
L6  Cognitive Shell ......... LLM codec + SQLite memory + drives + workspace + supervisor
```

### L0 Physics Fabric
- **Sim:** `memR/SIMULATE_MEMRISTOR_WINDOWED.m:132` hysteresis + `lissajous_hardware_design.m:428` FDM 1-4kHz FFT demux.
- **Rule:** `ω >> ω0` 10-100× `memR/CLAUDE.md` - DC programs (10-100ms/pulse), AC computes (1-10kHz optimal, 100kHz protoboard max, breadboard 10kHz avoid). N_max = BW/(Spacing×Safety) `lissajous_hardware_design.m`.
- **Decode:** PLL phase `memR/README.md:551`, BPF sieve `README.md:584`, envelope `README.md:636`, gating `README.md:651`. Fiber Level4: 1550nm laser/couplers/PD `02-mzi-mesh.md` -> 1ns.
- **Bus:** Z80 code unchanged across L0 variants `memR/CLAUDE.md`.

### L1 Sensing Bus
| Source | Raw | Preproc | Output |
|--------|-----|---------|--------|
| Mic/audio | 16kHz PCM | `encode_stft` `EchoMind:145` Hann 256 hop64 log1p·phase_wrap | `pulse[N]` sparse 30% `EchoMind:165` |
| MEG | `brain2qwerty` MEG | `brain2qwerty_v1/transforms.py` Conv+Transformer | token logits -> concepts |
| Text | UTF-8 | `encode_text` `EchoMind:189` 128×32 emb mean-pool `0xCAFE` | `tanh(proj·agg)` [N] |
| Physiology | BLE HR `polar_h10/hr_stream.py` | `encode_neuro_vector` `EchoMind:175` | `tanh(N×5·[arousal,attention,reward,surprise,curiosity])` |
| All | -> | `NeuralStimulus` `kernel.py:113` | `concepts[:24] + strength[0,0.55] + salience[0,1] + valence[-1,1]` |

### L2 Phase Encoders
- **Determinism:** `ConceptEncoder.assembly(concept)` `concepts.py:33` `blake2b(salt+lower)` -> 24-neuron assembly stable across restarts; `EchoMind:183` `seed 0xBEEF` for neuro-proj, `0xCAFE` for text-proj.
- **Multiplex:** Phase alone = 256 states `φ∈[0,2π)` 8-bit; + amplitude `A` doubles; + `f` 50 channels `99kHz`; + coupling 4 windows = 13M states/wire `memR/README.md:705`.
- **Sparsity:** STFT 70th pct threshold `EchoMind:165`, clip `EchoMind:171`.

### L3 Reservoir (EchoMind)
- **State:** `r[N]` float64 `EchoMind:51`, `W[N,N]` 5% mask `EchoMind:54` `rho 0.99` `EchoMind:58`, `W_in[N]` `EchoMind:61`, `W_out[n_out,N]` `EchoMind:63`, `P[N,N]` `1e3·I` `EchoMind:64`.
- **Dynamics:** `step(u)` `EchoMind:81` `x_tau=r[-17]; dx=0.2*x_tau/(1+x^10)-0.1*r[0]; r=roll(r,-1); r[-1]+=dx*0.1; r+=u*(W@tanh(r))+u*W_in; r=tanh(r)` tick++ `EchoMind:81-89`. Alt `step_encoded(signal[:N])` `EchoMind:92` `r=tanh(W@tanh(r)+signal)`.
- **Learning:** RLS `EchoMind:106` `y=W_out·r; e=t-y; P_r=P·r; g=P_r/(1+r·P_r); W_out+=outer(e,g); P-=outer(g,P_r)` err `mean|e|` `EchoMind:118`.
- **Novelty:** deque `window 50` `EchoMind:66` median baseline `EchoMind:128` `novelty=err/(median+1e-9)` `EchoMind:129` `threshold 2.5` `EchoMind:41` `cooldown 20` `EchoMind:68` -> `NoveltyEvent` `EchoMind:11` callback `EchoMind:78`.

### L4 Neural Field (Jeannie Kernel)
- **Neurons:** `n=2048` default `kernel.py:50` `exc 80%` `kernel.py:51` types `kernel.py:20` PV@exc, SST@+8%, VIP@+6%.
- **Edges:** `build_local_edges()` `growth.py` `edges_per_neuron 10` `kernel.py:53` 3D-local `geometry.py:11` `positions [0,1]^3` `kernel.py:61`.
- **Step:** `_step_once()` `kernel.py:136`:
  ```
  recurrent = sum_active W[src->dst] * STP(src) * inhib_scale (if src>=exc)
  potential[ready]*=leak 0.93 + recurrent*gain + pending + noise 0.003 - adaptation
  pending*=0.5; adaptation*=0.92 ready
  spikes = ready & (potential >= thresholds[0.60-1.80])
  potential[spikes]=reset 0.0; refractory=2; adaptation+=0.08
  plasticity.step_traces(spikes, encoding_gate) + compute_weight_delta BCM
  W exc clip [0.001,0.22] inh [-0.22,-0.001]
  rate_ema=0.995*rate_ema+0.005*spike; thresholds+=0.0004*(rate_ema-0.035)
  neuromod.step(); geometry.detect_clusters -> growth.step -> _grow/_prune
  ```
- **Plasticity:** `PlasticityEngine` `plasticity.py` pre/post traces, `bcm_threshold`, `stp_r/u_state` `kernel.py:356`.
- **I/O:** `stimulate()` `kernel.py:113` `pending[assembly]+=strength*(0.55+0.45*salience)*gain` `kernel.py:122` + `note_activation()` + valence->DA/NE/5HT `pulse()` `kernel.py:124`. `snapshot()` `kernel.py:271` firing_rate, overload `(fr-0.20)/0.30`, underactivity `(0.002-fr)/0.002`.
- **Persistence:** `checkpoint()` `kernel.py:340` `STATE_VERSION 4` `neural_state.npz` {n,exc_n,cell_type,potential,adaptation,thresholds,refractory,last_spikes,src,dst,weights,traces,rate_ema,bcm,stp_r/u,positions,step_count,spawned/pruned} + json meta `kernel.py:369`. `NeuralRunner` `kernel.py:434` hz loop + checkpoint every 30s.

### L5 Cortical Sheet (gpu_brain2)
- **Sheet:** `GRID 200` `brain_core.py:18` -> 40k (config 500), fan_in 24 radius 8, patch 20 `BLOB_SIZE 5` `brain_core.py:22`. Cupy path with `CUB_DISABLED` `brain_core.py:2`, NumPy fallback = Jeannie field.
- **Readout:** `readout_W [patch², vocab]` `BRAIN:145` vocab 8192 `BRAIN:140`, `logits=x@W+b` `BRAIN:260` softmax grad sparse update `BRAIN:295` only `x>0`.
- **Router:** `SuperBrain` `super_brain.py:11` `respond(text)` name extract -> exact `conf>=0.85` -> keyword -> facts skip fallback -> smart skip `i don't know` -> GPU spikes `super_brain.py:32`. `_maybe_distill` writes GPU readout from reliable Hybrid/Smart replies.
- **TUI:** `tui.py:500` + `ascii_heatmap.py:184` `_heatmap_grids` per tick, `mood.flavor()` `super_brain.py:20`.

### L6 Cognitive Shell
- **LLM codec:** Ollama `qwen3.5:9b` main + `qwen3:1.7b` fast `jeannie_alive_next/README.md` via `httpx`, fallback `hermes3:3b`. Vision `riven/smolvlm`.
- **Memory:** SQLite WAL `memory/*` FTS5 relevance/importance/recency/salience, `observation envelopes` live/simulated/imagined/replay `jeannie_alive_next README`.
- **Control:** `orchestrator.py` + `safety/supervisor` allowlist (no shell), `prediction/error` loop, `self_model`, `drives/homeostasis`, `meta/monitoring` overload/novelty/attractor lock.

---

## 2. Data Contracts (key schemas)

```python
NeuralStimulus(concepts: list[str]≤24, strength: float≤0.55, salience: float, valence: float)
FieldSnapshot(captured_at: iso, firing_rate, mean_potential, active_neurons: int[≤512],
              dominant_concepts: list[(str,score)], overload, underactivity, step_count)
HeatmapSnapshot(columns, rows, n, values: float[N], cell_types, spikes: bool[N],
                firing_rate, mean_potential, dominant_concepts, neuromod, growth)
Storage: neural_state.npz (versioned) + neural_state.json + brain_paths.npz (6.4M) + SQLite WAL
Bus: MUX 0x10 row/col, WRITE 0x11 DAC/phase, ADC 0x12 read -> same Python shim `write_phase(cell,phi)`
```

## 3. Cross-Layer Interfaces

```
Audio/MEG/Text/HR --[pulse[N]/concepts]--> Reservoir.step_encoded() --[r[N]]--> Field.stimulate(NeuralStimulus)
Field.step() --[FieldSnapshot/Heatmap]--> Sheet.encode_text_to_input() -> Sheet.step() -> logits -> tokens
Tokens --[codec]--> Ollama -> text -> Memory/Workspace --[proposal]--> Supervisor -> gate -> Memory.neuromod pulse
Supervisor -> dampen(0.85) on overload, checkpoint() every 30s
All layers expose state_snapshot() {tick, pred_error, novelty, replay, entropy} EchoMind:415 + FieldSnapshot
```

## 4. Timing / Buses

- Reservoir tick: per audio hop 64 (~4ms @16kHz) or per text token.
- Field Runner: `hz` configurable `kernel.py:435` default 10-20 Hz, `NeuralRunner._run()` `kernel.py:450` `period=1/hz` `checkpoint_seconds 30`.
- Sheet: per settle `settle_steps 100` `BRAIN:325` + `reply_len 16` `BRAIN:350` loops.
- Fabric: DC 10-100ms/weight training, AC 1-10kHz inference (`ω>>ω0`). Protoboard 100kHz max.
- Backpressure: `replay_cooldown 20` `EchoMind:68` + `growth.silence_counter` prevents oscillation.

## 5. Scaling Paths

- CPU-only: EchoMind (256) + Jeannie (2k->grown) - no GPU - laptop viable.
- GPU: + gpu_brain2 sheet 40-250k, `GRID` knob.
- Fiber: + MZI 2×2 $200-400 `02-mzi-mesh.md` 4-64 neurons 1ns. Same `WRITE` shim.
- Si photonics Level 5A/B `memR/CLAUDE.md` 1k-80k+ via WDM (future).

## 6. Failure Modes + Mitigations

- CUB NVRTC fail -> `CUPY_CUB_DISABLED=1` + CPU field fallback.
- Weight explosion -> `weight_decay 0.9995` `BRAIN:103` + clip `kernel.py:180`.
- Attractor lock -> `meta/monitoring` + `dampen()` `kernel.py:334` `potential*=factor pending*=0.2 thresholds+=0.03`.
- Template degredation -> `is_degenerate()` `EchoMind:325` max_run>6 or |set|<3.
- MEG embargo v2 -> use v1 only, offline.
