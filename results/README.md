# Canonical result records

Records are copied **byte-identical, same filename** from implementation repos (see the submodule
pins under `implementations/`); never edit them here — fix at the origin, re-copy, bump the pin.
Every record validates against [`schema/result.schema.json`](schema/result.schema.json)
(schema originates from the Agilex 3 implementation repo).

| Directory | Platform | Origin |
|---|---|---|
| `agilex3-coredla/` | Altera Agilex 3 + CoreDLA, Arrow AXC3000 (HyperRAM, no DDR) | [`implementations/agilex_3_ai_benchmarks`](../implementations/agilex_3_ai_benchmarks) `results/` |

Naming (origin convention): `ph2_*` = software references (FP32 / deployed INT8, full test sets) ·
`ph3_*` = on-board measured (HyperRAM-fed build) · `ph4_*` = on-board measured (Nios-less
JTAG-hosted build; on the origin repo's
[`main`](https://github.com/fpga-professional-association/agilex_3_ai_benchmarks) since
[PR #75](https://github.com/fpga-professional-association/agilex_3_ai_benchmarks/pull/75)
@ `2a5486b2`, `results/`, which is what the submodule pin now tracks — the ph4 copies here carry
that repo's rebased `arch_file`/`report_paths`) · levels explained in
[`docs/METHODOLOGY.md`](../docs/METHODOLOGY.md).
