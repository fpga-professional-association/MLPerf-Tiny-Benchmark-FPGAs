# Image classification — ResNet-8 / CIFAR-10

MLPerf Tiny image-classification benchmark: ResNet-8 (v1, 8-layer residual CNN) on CIFAR-10
(10 000-image test set, top-1 accuracy). Closed-division quality target: 85 % top-1.

## References (OpenVINO/NNCF toolchain, full test set)

| Precision | Top-1 | n | Record |
|---|---|---|---|
| FP32 | 87.19 % | 10 000 | [`ph2_resnet8-cifar10-fp32_20260704.json`](../../results/agilex3-coredla/ph2_resnet8-cifar10-fp32_20260704.json) |
| INT8 (NNCF PTQ, deployed) | 86.64 % | 10 000 | [`ph2_resnet8-cifar10-int8-deployed_20260710.json`](../../results/agilex3-coredla/ph2_resnet8-cifar10-int8-deployed_20260710.json) |

## Platform results

| Platform | Accuracy | Throughput (engine) | Clock | Record |
|---|---|---|---|---|
| **Agilex 3 CoreDLA** (AXC3000, Nios-less JTAG-hosted, on-chip params) | **86.33 % top-1 (full 10 000)** | 1,895.5 fps | 300 MHz | [`ph4_resnet8-cifar10-niosless-jtag-full10k_20260822.json`](../../results/agilex3-coredla/ph4_resnet8-cifar10-niosless-jtag-full10k_20260822.json) |
| **Agilex 3 CoreDLA** (same design, silicon-ceiling clock) | 84.0 % top-1 (official ic01 200-image subset; logits bit-identical to the row above) | **2,178.4 fps** (2,270.7 pipelined) | 340 MHz | [`ph4_resnet8-cifar10-niosless-jtag-340mhz_20260822.json`](../../results/agilex3-coredla/ph4_resnet8-cifar10-niosless-jtag-340mhz_20260822.json) |
| Agilex 3 CoreDLA (AXC3000, HyperRAM, no DDR — first on-silicon proof) | 86.0 % top-1 (100-image aligned subset) | 409.3 fps | 200.0 MHz | [`ph3_resnet8-cifar10-hyperram-onboard_20260711.json`](../../results/agilex3-coredla/ph3_resnet8-cifar10-hyperram-onboard_20260711.json) |
| **Efinix Ti180 TinyML** (Ti180 J484 Dev Kit, ResNet-8-only 16×8 accelerator config, JTAG data feed) | **88.0 % top-1 (n=1000)** | **148.6 fps** | 250 MHz | [`ph7_hw_accuracy`](../../results/efinix-ti180-tinyml/ph7_hw_accuracy.json) · [`ph7_hw_performance`](../../results/efinix-ti180-tinyml/ph7_hw_performance.json) |
| Efinix Ti180 TinyML (same board, stock six-app accelerator config — baseline) | 88.0 % top-1 (n=100) | 106.6 fps | 250 MHz | [`ph5_hw_accuracy`](../../results/efinix-ti180-tinyml/ph5_hw_accuracy.json) · [`ph5_hw_performance`](../../results/efinix-ti180-tinyml/ph5_hw_performance.json) |

Notes: engine rate is CoreDLA's on-chip-clock-measured IP throughput (the ph4 records use the
`jobs_active`-gated `CLOCKS_ACTIVE` CSR, which excludes the JTAG data plane by construction); the
JTAG-hosted loops (~12 fps in ph3, ~57–79 img/s in ph4) are input-path bound and recorded as notes,
not metrics. Model provenance differs across records: ph3 deploys the OpenVINO/NNCF-PTQ INT8 model
(reference 86.64 %), ph4 deploys the MLPerf Tiny pretrained INT8 TFLite (reference band 86.5–87.0 %
on the 200-image set). The full-10 k accuracy follow-up is DONE (ph4, 86.33 %). Remaining follow-ups
tracked in the implementation repo: Ethernet feed via
[fpga-ai-endpoint](https://github.com/fpga-professional-association/fpga-ai-endpoint) targeting
≥ 400 fps end-to-end.

Ti180 notes: the deployed model is **Efinix's INT8 quantization** of the same MLCommons fp32
checkpoint (quantized with the MLPerf calibration set in the Efinix model zoo), *not* the
OpenVINO/NNCF model the references above use — argmax agreement with the OpenVINO-INT8 CPU
reference is 956/1000 on the measured slice. Latency provenance is the RISC-V CLINT counter
around `Invoke()` only (JTAG/UART transport excluded); throughput was cross-checked over 1000
*distinct* images vs a constant-input loop (within 1 %, no caching flattery). The 88.0 % n=1000
figure carries ~±1 % standard error vs the full-10k runs in the Agilex rows; a full-10k Ti180 run
(~16 min of JTAG batches) is the natural follow-up.
