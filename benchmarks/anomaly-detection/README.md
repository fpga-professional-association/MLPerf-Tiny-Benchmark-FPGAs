# Anomaly detection — FC-AutoEncoder / ToyCar

MLPerf Tiny anomaly-detection benchmark: fully-connected autoencoder on ToyADMOS ToyCar machine
sounds (2 459-record test set, metric = AUC over reconstruction error). Closed-division quality
target: 0.85 AUC.

## References (OpenVINO/NNCF toolchain, full test set)

| Precision | AUC | n | Record |
|---|---|---|---|
| FP32 | 0.8760 | 2 459 | [`ph2_ad-toycar-fp32_20260704.json`](../../results/agilex3-coredla/ph2_ad-toycar-fp32_20260704.json) |
| INT8 (NNCF PTQ, deployed) | **0.7811** ⚠ | 2 459 | [`ph2_ad-toycar-int8-deployed_20260710.json`](../../results/agilex3-coredla/ph2_ad-toycar-int8-deployed_20260710.json) |

⚠ **Known issue — quantization sensitivity**: this model loses ~0.095 AUC under the deployed
INT8 quantization, landing below the 0.85 closed-division target. The implementation repo carries
the diagnostic sweep (`results/l5_quant-sweep_ad-toycar-int8-per-channel*.json`,
`…int8-per-tensor*.json`, `…int4-weight-only*.json`) — resolving the deployed recipe (per-channel
scales are the usual fix for AE reconstruction error) is a prerequisite for a meaningful silicon
run of this benchmark.

## Platform results

| Platform | AUC | Throughput | Record |
|---|---|---|---|
| Agilex 3 CoreDLA (AXC3000) | *blocked on quantization recipe (above)* | — | — |
| **Efinix Ti180 TinyML** (Ti180 J484 Dev Kit, Efinix INT8 quantization, on-device MSE scoring, JTAG data feed) | **0.8188 mean AUC / 0.672 pAUC (full 2,459 files, 481,964 slices)**; same-model CPU ref 0.8401 — FC-engine rounding costs 0.021 AUC (score corr 0.857), software-FC ablation in the implementation repo | 342.4 fps/slice @ 250 MHz (2.921 ms) | [`ad1_hw_accuracy`](../../results/efinix-ti180-tinyml/ad1_hw_accuracy.json) · [`ad1_hw_performance`](../../results/efinix-ti180-tinyml/ad1_hw_performance.json) |

This benchmark is also the reference workload for the vendor-neutral open path in
[fpga-ai-endpoint](https://github.com/fpga-professional-association/fpga-ai-endpoint) (pure-FC
model on a bit-exact int8 MLP engine), which will produce an independent FPGA result for this
table.
