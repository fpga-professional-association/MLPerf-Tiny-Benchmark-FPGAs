# Adding a platform (or a new benchmark result)

## New platform

1. **Implementation repo**: keep the platform's full engineering history (RTL/build scripts/logs/
   estimator runs) in its own repo under the org. It must produce result JSONs that validate
   against `results/schema/result.schema.json` and follow
   [`METHODOLOGY.md`](METHODOLOGY.md) (on-chip throughput provenance, bitstream hash, tool
   versions, `n_records`).
2. **Submodule**: `git submodule add <repo-url> implementations/<repo-name>` — pin the commit that
   produced the results you're publishing. Bump the pin when publishing newer records.
3. **Records**: copy the relevant JSONs **byte-identical, same filename** into
   `results/<platform-slug>/` (slug: lowercase, e.g. `agilex3-coredla`). Minimum set per
   benchmark: the deployed-quantization software reference (PH2) and the measured silicon record
   (PH3). FP32 reference recommended.
4. **Tables**: add/extend the platform section + matrix row in the top README, and the per-platform
   row in each relevant `benchmarks/*/README.md`.

## New benchmark result on an existing platform

Steps 3–4 only, plus bump the submodule pin to the commit containing the new records.

## Validation

```
python3 - <<'EOF'
import json, glob, jsonschema  # pip install jsonschema
schema = json.load(open('results/schema/result.schema.json'))
for f in glob.glob('results/*/*.json'):
    jsonschema.validate(json.load(open(f)), schema); print('ok', f)
EOF
```

(CI for this check is a welcome addition once the repo grows.)

## Ground rules

- No estimator/simulation numbers in the matrix — implementation repos only.
- Never edit a copied record; if the origin record was wrong, fix it there, re-copy, bump the pin.
- Subset accuracy runs must be labeled with `n_records`; prefer full test sets.
- JTAG-hosted end-to-end numbers are notes, not metrics.
