# Visual wake words — MobileNetV1-0.25 / VWW

MLPerf Tiny visual-wake-words benchmark: MobileNetV1 (width 0.25) on the Visual Wake Words
dataset (person / no-person, 10 962-record test set, top-1 accuracy). Closed-division quality
target: 80 % top-1.

## References (OpenVINO/NNCF toolchain, full test set)

| Precision | Top-1 | n | Record |
|---|---|---|---|
| FP32 | 86.03 % | 10 962 | [`ph2_mobilenetv1-025-vww-fp32_20260704.json`](../../results/agilex3-coredla/ph2_mobilenetv1-025-vww-fp32_20260704.json) |
| INT8 (NNCF PTQ, deployed) | 85.84 % | 10 962 | [`ph2_mobilenetv1-025-vww-int8-deployed_20260710.json`](../../results/agilex3-coredla/ph2_mobilenetv1-025-vww-int8-deployed_20260710.json) |

## Platform results

| Platform | Accuracy | Throughput | Record |
|---|---|---|---|
| Agilex 3 CoreDLA (AXC3000) | *not yet measured on silicon* | — | — |

The INT8 deployed model is ready in the implementation repo (`tiny_bundles/`); the on-board run
follows the same PH3 method proven with resnet8-cifar10.
