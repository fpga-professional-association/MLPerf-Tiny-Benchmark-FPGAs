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
| **Agilex 3 CoreDLA** (AXC3000, HyperRAM, no DDR) | 86.0 % top-1 (100-image aligned subset) | **409.3 fps** | 200.0 MHz | [`ph3_resnet8-cifar10-hyperram-onboard_20260711.json`](../../results/agilex3-coredla/ph3_resnet8-cifar10-hyperram-onboard_20260711.json) |

Notes: engine rate is CoreDLA's on-chip-clock-measured IP throughput; the JTAG-hosted loop
(~12 fps, 83.8 ms) is input-path bound and recorded as a note, not a metric. Follow-ups tracked in
the implementation repo: full 10 k-image accuracy run; HyperRAM-resident feed; Ethernet feed via
[fpga-ai-endpoint](https://github.com/fpga-professional-association/fpga-ai-endpoint) targeting
≥ 400 fps end-to-end.
