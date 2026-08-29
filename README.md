# MLPerf Tiny Benchmarks on FPGAs

Measured MLPerf™ Tiny benchmark results on FPGA platforms — one canonical results tree, one
schema, per-platform implementation repos pinned as submodules. **Measured-silicon-only policy**
for headline numbers: estimator and simulation figures may appear in implementation repos, but a
number lands in this repo's tables only when it was measured on hardware (or is the clearly-labeled
software reference used to validate it).

## Results matrix

Software references (full test sets, OpenVINO/NNCF toolchain unless noted):

| Benchmark | Model / dataset | n | FP32 reference | INT8 deployed reference | MLPerf quality target |
|---|---|---|---|---|---|
| **IC** — image classification | ResNet-8 / CIFAR-10 | 10 000 | 87.19 % top-1 | 86.64 % top-1 | 85 % |
| **KWS** — keyword spotting | DS-CNN / Speech Commands | 4 890 | 91.86 % top-1 | 91.72 % top-1 | 90 % |
| **VWW** — visual wake words | MobileNetV1-0.25 / VWW | 10 962 | 86.03 % top-1 | 85.84 % top-1 | 80 % |
| **AD** — anomaly detection | FC-AutoEncoder / ToyCar | 2 459 files | 0.8760 AUC | 0.7811 AUC ⚠ | 0.85 AUC |

Measured on silicon — one row per dev kit (quality on the stated test-set size · engine
rate; engine rates are on-chip-counter measurements that exclude the host data path by
construction):

| Dev kit — platform | IC ResNet-8 | KWS DS-CNN | VWW MobileNetV1-0.25 | AD FC-AutoEncoder |
|---|---|---|---|---|
| **Arrow AXC3000** — Agilex 3 CoreDLA ([details ↓](#agilex-3--coredla-arrow-axc3000)) | **86.33 %** (full 10 000) · 2 178 fps @ 340 MHz | — | — | — |
| **Efinix Ti180 J484 Dev Kit** — TinyML platform ([details ↓](#efinix-titanium-ti180--tinyml-ti180-j484-dev-kit)) | **88.0 %** (n=1 000) · 148.6 fps @ 250 MHz | **91.80 %** (full 4 890) · 116.7 fps | **85.98 %** (full 10 962) · 98.4 fps | **0.8188 AUC** (full 2 459 files) · 342.4 fps/slice — 0.8416 AUC with software FC ([why ↓](#efinix-titanium-ti180--tinyml-ti180-j484-dev-kit)) |

Cell notes: the Ti180 KWS/AD rows deploy Efinix's INT8 quantizations of the MLPerf
checkpoints (their own CPU references: KWS 91.86 %, AD 0.8401 AUC); the Ti180 VWW row
deploys MLCommons' reference `vww_96_int8.tflite`; the Agilex IC row deploys the MLPerf
pretrained INT8 TFLite. Quality cells are directly comparable to the reference table
above; n is stated wherever a run is not the full test set.

⚠ The AD INT8 per-tensor OpenVINO deployment loses significant AUC vs FP32
(0.876 → 0.781, below the 0.85 target); per-channel and INT4 sweeps exist in the
Agilex 3 repo (`results/l5_quant-sweep_ad-toycar-*.json`) — see
[benchmarks/anomaly-detection](benchmarks/anomaly-detection/README.md). The Efinix
quantization of the same checkpoint holds 0.8401 (CPU) / 0.8416 (silicon, software FC).

Each reference was produced with a toolchain that deploys to an FPGA in this repo, so the
reference table is the honest comparison point for the silicon matrix. Full records with
provenance (bitstream SHA-256, tool versions, report paths) live in [`results/`](results/).

## Platforms

### Agilex 3 + CoreDLA (Arrow AXC3000)

Implementation: [`implementations/agilex_3_ai_benchmarks`](https://github.com/fpga-professional-association/agilex_3_ai_benchmarks)
(submodule) — Altera FPGA AI Suite **CoreDLA** on the ~$129 Arrow **AXC3000**
(Agilex 3 `A3CY100BM16AE7S`, no HPS, **no DDR** — fed from a 16 MB HyperRAM).

Latest measured result (2026-08-22, `resnet8-cifar10` INT8, Nios-less JTAG-hosted build — on
[`main`](https://github.com/fpga-professional-association/agilex_3_ai_benchmarks) since
[PR #75](https://github.com/fpga-professional-association/agilex_3_ai_benchmarks/pull/75)
@ `2a5486b2`): no soft CPU — the host PC drives a JTAG-to-Avalon master via System Console; CoreDLA
k16/c8 FP12AGX (24 DSPs in INT9 tensor mode, 128 int8 MACs/cycle), parameters MIF-baked into
on-chip RAM (no HyperRAM, no external memory); model is the MLPerf Tiny pretrained INT8 TFLite:

| Metric | Measured |
|---|---|
| Top-1 accuracy | **86.33 %** on the full 10 000-image CIFAR-10 test set (95 % CI 85.66–87.00; passes the 85 % closed-division floor; MLPerf reference model band 86.5–87.0 %) |
| Engine latency (single-stream) | **527.57 µs** at 300 MHz (deterministic: 9 748/10 000 jobs bit-exact on the counter) · **459.05 µs** at the 340 MHz silicon ceiling |
| Engine throughput | **1,895.5 fps** @ 300 MHz · **2,178.4 fps** @ 340 MHz · **2,270.7 fps** pipelined (descriptor queue ≥ 8) |
| Compute efficiency | 6.4 fps/MHz · 61.7 % MAC utilization (23.69 of 38.4 peak GMAC/s) |
| End-to-end (JTAG-hosted loop) | ~57–79 img/s — input-write bound, not engine bound |
| Resources (340 MHz build) | 24,024/34,000 ALM (71 %) · 254/262 M20K (97 %) · 24/276 DSP (9 %) |

Latency/fps provenance: the DLA's `jobs_active`-gated `CLOCKS_ACTIVE` CSR delta — device time from
descriptor fetch to result written, excluding JTAG transfer by construction. Logits are bit-identical
across the 300/340 MHz builds on the official ic01 200-image subset. 340 MHz sits 0.9 % under this
-E7S die's Minimum-Pulse-Width limit (Restricted Fmax 342.94 MHz). The 2× k16c16 core (measured
295.7 µs) remains blocked: it fits only via the defective `enable_on_chip_parameters` mode
(input-invariant output — vendor escalation in the implementation repo,
[`docs/fpga_ai_streaming_egress_escalation.md`](https://github.com/fpga-professional-association/agilex_3_ai_benchmarks/blob/main/docs/fpga_ai_streaming_egress_escalation.md))
and needs ~275 M20K with working DDR-served parameters vs the C100's 262. Canonical records:
[`ph4_…full10k`](results/agilex3-coredla/ph4_resnet8-cifar10-niosless-jtag-full10k_20260822.json) ·
[`ph4_…340mhz`](results/agilex3-coredla/ph4_resnet8-cifar10-niosless-jtag-340mhz_20260822.json).

First measured result (2026-07-11, `resnet8-cifar10` INT8, HyperRAM-fed build — superseded as the
headline but preserved as the first on-silicon proof):

| Metric | Measured |
|---|---|
| Top-1 accuracy | **86.0 %** on 100 CIFAR-10 test images (vs 86.64 % INT8 software reference) |
| CoreDLA IP throughput | **409.3 fps** at `clk_dla` = 200.0 MHz (on-chip-clock-measured engine rate) |
| Compute efficiency | 2.05 fps/MHz |
| End-to-end (JTAG-hosted loop) | ~12 fps — input-path bound, not engine bound |

The engine-vs-system gap is the input path (JTAG), not the accelerator; an Ethernet tensor feed
targeting ≥ 400 fps end-to-end is planned in
[fpga-ai-endpoint](https://github.com/fpga-professional-association/fpga-ai-endpoint).
Canonical record: [`results/agilex3-coredla/ph3_resnet8-cifar10-hyperram-onboard_20260711.json`](results/agilex3-coredla/ph3_resnet8-cifar10-hyperram-onboard_20260711.json)
(bitstream SHA-256 `e0e363f2…`, full tool versions inside).

#### FPGA resource utilization (measured build)

Full-system fit of the `coredla_hyperram_ed` platform — CoreDLA engine + HyperRAM controller +
JTAG control plane — on the A3CY100 (`top.fit.summary`, Quartus Prime Pro 26.1.0):

| Resource | Used / available | Utilization |
|---|---|---|
| ALMs | 32,397 / 34,000 | **95 %** |
| M20K RAM blocks | 228 / 262 | **87 %** |
| Block memory bits | 4,133,008 / 5,365,760 | 77 % |
| DSP blocks | 75 / 276 | 27 % |
| Dedicated logic registers | 75,872 | — |
| PLLs | 2 / 11 | 18 % |

Performance in that footprint: **409.3 fps at 200.0 MHz = 2.05 fps/MHz** (engine rate, on-chip
measured). Context from the implementation repo's build findings
(`docs/coredla_agx3_build_findings.md`): **M20K is the binding wall on this die** — the untrimmed
Performance-class DLA config needs 431 M20K vs the C100's 262 (164 %); the fitted DDR-free config
standalone comes in at ALM 88 % / M20K 94 % / DSP 23 % with the DLA datapath closing timing at
~343 MHz in DSP tensor mode, so the deployed 200 MHz build has thermal/timing headroom and DSPs
to spare — the die's memory, not its arithmetic, is what's full.

### Efinix Titanium Ti180 + TinyML (Ti180 J484 Dev Kit)

Implementation: [`implementations/efinix_ti180_ai_benchmarks`](https://github.com/fpga-professional-association/efinix_ti180_ai_benchmarks)
(submodule) — the **Efinix TinyML platform** (Sapphire RISC-V + TinyML layer-engine
accelerator, Efinity 2026.1) on the Efinix **Ti180 J484 Development Kit**
(`Ti180J484C4`, 172.8K XLR, 1 GB LPDDR4x on the hardened controller).

Measured result (2026-08-28, `resnet8-cifar10` INT8, ResNet-8-only accelerator config —
conv 16×8, unused MUL/LR/MIN_MAX/FC engines stripped, FC in software; JTAG bulk data feed
to DDR, UART control only):

| Metric | Measured |
|---|---|
| Top-1 accuracy | **88.0 %** (880/1000, CIFAR-10 test 0–999; ~±1 % SE) — Efinix-quantized INT8 of the MLCommons checkpoint; 956/1000 argmax agreement with the OpenVINO-INT8 reference |
| Engine latency | **6.731 ms** (sd 0.017, n=1000) at 250 MHz (timing-closed, Fmax 264 MHz) |
| Engine throughput | **148.6 fps** over 1000 distinct images (0.59 fps/MHz) — matches constant-input loop within 1 % |
| vs stock accelerator config | 1.41× (stock six-app superset: 9.379 ms / 106.6 fps, same accuracy) |
| Resources (16×8 build) | 141,713/172,800 XLR (82 %) · 556/1,280 RAM (43 %) · 177/640 DSP (28 %) |

Provenance: RISC-V CLINT counter around `Invoke()` only — JTAG/UART transport excluded by
construction; bitstream SHA-256 + tool versions in every record. Optimization findings from
on-silicon per-layer profiling: conv = 89 % of time at ~5 % MAC-array utilization (cost is
channel *passes*, not MACs); parallelism ceiling is empirical (16×12 fails packing at
178,747/172,800 XLRs, 16×16-without-FC at 194,385). Architectural note: unlike CoreDLA,
the TinyML accelerator is layer engines sequenced by TFLite Micro on the RISC-V (closed-lib
custom instructions), so the CPU is part of the engine, not a removable host — a "no-CPU"
build is not possible on this platform. Records:
[`results/efinix-ti180-tinyml/`](results/efinix-ti180-tinyml/) (dialect: these validate
against [`results/schema/efinix-ti180.result.schema.json`](results/schema/efinix-ti180.result.schema.json),
carried from the implementation repo — the canonical `result.schema.json` is currently
AXC3000-specific; schema unification is an open item).

#### All four MLPerf Tiny benchmarks (2026-08-28/29)

One union bitstream (conv 16×8 + FC 640×640 engines, 250 MHz timing-closed, Fmax
275 MHz, 88.9 % XLR / 37.7 % DSP), per-benchmark harness firmware swapped over JTAG,
data bulk-loaded to DDR over JTAG (integrity-verified at 30 MHz TCK), CLINT-only
`Invoke()` timing. KWS/AD deploy Efinix's shipped models, which their notebooks build
from the genuine mlcommons/tiny pipelines; VWW deploys **MLCommons' own
`vww_96_int8.tflite`** (Efinix's person-detect model is a 96×96×1 grayscale
non-MLPerf variant). CPU references (same deployed tflite, host CPU): KWS 91.86 %,
VWW 86.04 %, AD 0.8401 AUC.

| Benchmark | n (full test set) | Measured quality | Engine latency | Engine fps |
|---|---|---|---|---|
| KWS DS-CNN | 4,890 | **91.80 % top-1** (target 90 %) | 8.571 ms | 116.7 |
| VWW MobileNetV1-0.25 | 10,962 | **85.98 % top-1** (target 80 %) | 10.158 ms | 98.4 |
| AD FC-AutoEncoder | 2,459 files / 481,964 slices | **0.8188 mean AUC** (target 0.85; per-id [0.838, 0.884, 0.648, 0.906]) | 2.921 ms/slice | 342.4 |

AD caveat, resolved by ablation: the FC-engine run's per-file scores correlate 0.857
with the CPU reference of the same tflite; forcing FC to software (identical
bitstream, all 481,964 slices re-measured) recovers **0.8416 mean AUC** at 0.9998
score correlation — the 0.023-AUC gap is entirely the FC engine's arithmetic, traded
for 5.1× speed (2.921 vs 15.011 ms/slice). Records:
`results/efinix-ti180-tinyml/{kws1,vww1,ad1,ad_swfc}_hw_*.json`.

## MLPerf Tiny v1.4 comparison dashboard

[`dashboard/mlperf_tiny_v1.4_dashboard.html`](dashboard/mlperf_tiny_v1.4_dashboard.html) — a
self-contained (no-CDN, open-it-locally) interactive dashboard of the **22 official MLPerf Tiny
v1.4 submissions** (efficiency frontier, latency + energy leaderboards, per-benchmark switching),
with our measured Agilex 3 point plotted among them as a starred (★) research entry:
**IC engine latency 0.459 ms** (1000 ÷ 2,178.4 fps, the dashboard's single-stream convention —
the 2026-08-22 record at the 340 MHz silicon ceiling).

Where that lands among the v1.4 FPGA entries on image classification: the fastest FPGA IC point on
the chart — ahead of every closed-division FPGA IC submission (best: Andes AnDLA I370 at 3.87 ms on
a Kintex-7) and ahead of the previously-fastest Versal VCK190 DPU (0.54 ms, an open-division
ResNet-18 run on a much larger device). Caveats stated on the plot itself: our point is
engine-rate-derived, measured on-board but **not an MLPerf submission** (no EnergyRunner harness),
and has no energy number yet.

The official v1.4 submission data is preserved untouched at
[`results/mlperf_tiny_v1.4_all_submissions.csv`](results/mlperf_tiny_v1.4_all_submissions.csv)
(source: `github.com/mlcommons/tiny_results_v1.4`) — our research point lives only in the
dashboard, clearly labeled, never mixed into the official record.

## Repository layout

```
benchmarks/               one page per MLPerf Tiny benchmark: model, dataset, references, per-platform status
  image-classification/   keyword-spotting/   visual-wake-words/   anomaly-detection/
dashboard/                self-contained interactive v1.4 comparison dashboard (+ our starred measured point)
results/                  canonical result records (JSON, schema-validated) + the schema
  schema/result.schema.json
  agilex3-coredla/        records copied verbatim from the implementation repo (provenance preserved)
  efinix-ti180-tinyml/    records copied verbatim from the Efinix Ti180 implementation repo
  mlperf_tiny_v1.4_all_submissions.csv   official v1.4 submission set (untouched)
implementations/          per-platform implementation repos as git submodules
  agilex_3_ai_benchmarks/ Agilex 3 CoreDLA + HyperRAM on the AXC3000
  efinix_ti180_ai_benchmarks/ Efinix TinyML (Sapphire RISC-V + layer engines) on the Ti180 J484 kit
docs/                     METHODOLOGY.md (levels, measured-only policy) · ADDING_A_PLATFORM.md
```

## Adding results

New platform or new benchmark run: see [`docs/ADDING_A_PLATFORM.md`](docs/ADDING_A_PLATFORM.md).
Short version: pin your implementation repo as a submodule, copy the schema-valid result JSONs
into `results/<platform>/`, update the matrix here and the relevant `benchmarks/*/README.md`.

---
*MLPerf™ is a trademark of MLCommons. Results here are research measurements, not official
MLPerf Tiny submissions (no EEMBC EnergyRunner harness); the models, datasets, and quality
metrics follow the MLPerf Tiny benchmark definitions.*
