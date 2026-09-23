# Reproducibility package (organised edition)

Archival package for the manuscript *Subclass-level damage detection on concrete dams:
difficulty follows imaging conditions, not class frequency* (Arabian Journal for Science and
Engineering). It contains the derived annotations, result files, the full experiment pipeline,
the modified framework files, configurations, trained weights and reading samples.

## Contents

| Folder | What it is |
|---|---|
| `derived_labels/` | YOLO labels for all five dataset variants, incl. skeleton pseudo-labels (`DamCrack-Sub5/skeleton/`). Images come from the cited sources. |
| `results_json/` | The 28 result files (JSON/CSV) every reported number traces to; see its `README.md` for the file → paper mapping. |
| `scripts_damsegment/` | The complete pipeline: data construction, training, evaluation, report/figure generation, plus the patch-level few-shot baselines (`fewshot/`). |
| `framework/` | The seven modified/added Ultralytics files, the change log (`改动说明.md`) and an incremental patch (`changes.patch`). |
| `configs/` | Dataset descriptors, model definitions and the recorded per-run `args.yaml` files. |
| `weights_and_samples/` | Trained checkpoints and representative image/label samples. |
| `data_manifest/` | Dataset statistics, merge manifest and provenance notes. |

See `00_索引.md` (Chinese) for a one-page navigation guide.

## Data sources and licences

1. **DamCrack source dataset.** Gharehbaghi, V., Li, J. (2026), Zenodo DOI [`10.5281/zenodo.17274707`](https://doi.org/10.5281/zenodo.17274707), CC BY 4.0. The derived labels and split manifests are distributed under CC BY 4.0 with attribution.
2. **DSI source dataset.** Hong, K., Wang, H., Yuan, B., Wang, T. (2023), GitHub [`GITSHOHOKU/DSI-Data-set`](https://github.com/GITSHOHOKU/DSI-Data-set), MIT licence; article DOI [`10.3390/buildings13020285`](https://doi.org/10.3390/buildings13020285), CC BY 4.0.
3. **Framework code.** The modified Ultralytics files derive from an AGPL-3.0 codebase; the project-level licence remains AGPL-3.0.

## Reproducing the reported results

```bash
# 1) environment: Python 3.10, torch 2.5.1+cu121, and the base Ultralytics 8.4 tree
#    with framework/ applied (see framework/README.md)
# 2) place the images next to derived_labels/<dataset>/ and run, e.g.:
python scripts_damsegment/detection/19_full_seeds.py --seeds 42,123,456 --name e6full
python scripts_damsegment/detection/49_method_experiments.py --only M1 --seed 42 --workers 2
python scripts_damsegment/report/55_render_figures_en.py
python scripts_damsegment/report/56_build_ajse_tables.py
```

The figure and table scripts read the result files, so figures can be regenerated without
retraining (copy `results_json/` back into a `runs/` tree, see `results_json/README.md`).

## Contact

AUTHOR\_INPUT\_NEEDED: insert the corresponding author, institutional e-mail address and the
public repository DOI/URL before publication.
