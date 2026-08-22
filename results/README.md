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
JTAG-hosted build; origin branch
[`codex/evaluation-debug`](https://github.com/fpga-professional-association/agilex_3_ai_benchmarks/tree/codex/evaluation-debug)
@ `54ca686`, `results/` — the submodule pin tracks that repo's `main`, which does not yet contain
the ph4 records) · levels explained in [`docs/METHODOLOGY.md`](../docs/METHODOLOGY.md).
