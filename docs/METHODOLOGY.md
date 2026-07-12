# Methodology

## Measured-only policy

A number appears in this repo's tables only if it is:
1. **Measured on silicon** (`kind: "measured"`) — accuracy scored against real dataset records on
   the board, throughput from **on-chip clock/cycle measurement** (never a host-side loop that
   includes a debug transport), with bitstream hash + tool versions recorded; or
2. a **software reference** (`kind: "reference"`) — the same model/quantization scored by the same
   toolchain on a CPU, present so the silicon number has an honest comparison point.

Estimator outputs (vendor performance estimators, analytical models) and simulation numbers stay
in the implementation repos, clearly labeled. They never enter the results matrix.

## Result levels (inherited from the Agilex 3 repo's convention)

- `PH2` — software reference runs (FP32 and deployed-quantization accuracy on full test sets).
- `PH3` — on-board measured inference.
- `L0…L5` — platform microbenchmarks (MAC arrays, memory bandwidth, quantization sweeps) that
  explain *why* a PH3 number is what it is; they live in the implementation repo and are cited
  from benchmark pages when relevant.

## Throughput provenance rules

- **Engine rate**: computed from the accelerator's own clock-cycle counters and its measured clock
  (e.g. CoreDLA "IP throughput" at the measured `clk_dla`). This is the architecture number.
- **End-to-end rate**: includes the input/output path. Reported separately and labeled with the
  transport (JTAG-hosted numbers are smoke-test signals, not results — record them in `notes`,
  not `metrics`).
- Accuracy runs must state `n_records`; subsets (e.g. 100 of 10 000 CIFAR images) are valid when
  aligned with the reference scoring, but the full-set run is preferred and subsets must be
  labeled.

## Record format

Every record in `results/` validates against `results/schema/result.schema.json`
(fields: `kind, level, subject, date, plan_ref, config, metrics, notes`). Records are copied
**verbatim** from implementation repos — same filename, byte-identical — so a record here can
always be diffed against its origin. Provenance chain: record → `config.bitstream_sha256` +
`config.report_paths` + `config.tool_versions` → implementation repo (submodule pin).
