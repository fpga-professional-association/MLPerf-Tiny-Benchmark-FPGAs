# MLPerf Tiny Benchmarks on FPGAs

Measured MLPerf™ Tiny benchmark results on FPGA platforms — one canonical results tree, one
schema, per-platform implementation repos pinned as submodules. **Measured-silicon-only policy**
for headline numbers: estimator and simulation figures may appear in implementation repos, but a
number lands in this repo's tables only when it was measured on hardware (or is the clearly-labeled
software reference used to validate it).

## Results matrix

| Benchmark | Model / dataset | FP32 reference | INT8 deployed reference | On-silicon (measured) |
|---|---|---|---|---|
| **Image classification** | ResNet-8 / CIFAR-10 | 87.19 % top-1 (10 000) | 86.64 % top-1 (10 000) | **86.0 % top-1, 409.3 fps engine rate** — Agilex 3 CoreDLA, [details ↓](#agilex-3--coredla-arrow-axc3000) |
| **Keyword spotting** | DS-CNN / Speech Commands | 91.86 % top-1 (4 890) | 91.72 % top-1 (4 890) | — |
| **Visual wake words** | MobileNetV1-0.25 / VWW | 86.03 % top-1 (10 962) | 85.84 % top-1 (10 962) | — |
| **Anomaly detection** | FC-AutoEncoder / ToyCar | 0.8760 AUC (2 459) | 0.7811 AUC (2 459) ⚠ | — |

⚠ The AD INT8 per-tensor deployment loses significant AUC vs FP32 (0.876 → 0.781, below the
MLPerf Tiny closed-division 0.85 target); per-channel and INT4 quantization sweeps exist in the
Agilex 3 repo (`results/l5_quant-sweep_ad-toycar-*.json`) — see
[benchmarks/anomaly-detection](benchmarks/anomaly-detection/README.md).

References were produced with the same toolchain that deploys to the FPGA (OpenVINO/NNCF PTQ), so
the reference column is the honest comparison point for the silicon column. Full records with
provenance (bitstream SHA-256, tool versions, report paths) live in [`results/`](results/).

## Platforms

### Agilex 3 + CoreDLA (Arrow AXC3000)

Implementation: [`implementations/agilex_3_ai_benchmarks`](https://github.com/fpga-professional-association/agilex_3_ai_benchmarks)
(submodule) — Altera FPGA AI Suite **CoreDLA** on the ~$129 Arrow **AXC3000**
(Agilex 3 `A3CY100BM16AE7S`, no HPS, **no DDR** — fed from a 16 MB HyperRAM).

First measured result (2026-07-11, `resnet8-cifar10` INT8):

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

## Repository layout

```
benchmarks/               one page per MLPerf Tiny benchmark: model, dataset, references, per-platform status
  image-classification/   keyword-spotting/   visual-wake-words/   anomaly-detection/
results/                  canonical result records (JSON, schema-validated) + the schema
  schema/result.schema.json
  agilex3-coredla/        records copied verbatim from the implementation repo (provenance preserved)
implementations/          per-platform implementation repos as git submodules
  agilex_3_ai_benchmarks/ Agilex 3 CoreDLA + HyperRAM on the AXC3000
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
