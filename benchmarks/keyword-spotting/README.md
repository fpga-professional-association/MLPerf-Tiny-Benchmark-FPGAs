# Keyword spotting — DS-CNN / Speech Commands

MLPerf Tiny keyword-spotting benchmark: DS-CNN (depthwise-separable CNN) on Google Speech
Commands (12-class, 4 890-record test set, top-1 accuracy). Closed-division quality target: 90 % top-1.

## References (OpenVINO/NNCF toolchain, full test set)

| Precision | Top-1 | n | Record |
|---|---|---|---|
| FP32 | 91.86 % | 4 890 | [`ph2_ds-cnn-kws-fp32_20260704.json`](../../results/agilex3-coredla/ph2_ds-cnn-kws-fp32_20260704.json) |
| INT8 (NNCF PTQ, deployed) | 91.72 % | 4 890 | [`ph2_ds-cnn-kws-int8-deployed_20260710.json`](../../results/agilex3-coredla/ph2_ds-cnn-kws-int8-deployed_20260710.json) |

## Platform results

| Platform | Accuracy | Throughput | Record |
|---|---|---|---|
| Agilex 3 CoreDLA (AXC3000) | *not yet measured on silicon* | — | — |

The INT8 deployed model is ready in the implementation repo (`tiny_bundles/`); the on-board run
follows the same PH3 method proven with resnet8-cifar10.
