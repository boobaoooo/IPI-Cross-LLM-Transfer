# Beyond Payload Presence: Evaluating Whether Behavioral Detection of Indirect Prompt Injection Transfers Across LLMs

Entry point for the code and data behind this paper (IEEE ISPA 2026, accepted). This repository holds **no code or data of its own**. It links the component repositories at the exact commits used for the paper and shows which file produced which reported number.

## 1. Component repositories (frozen commits)

| Stage | Repository | Commit | What it contains |
|---|---|---|---|
| 1. Benign controls and baseline-template IPI runs | [IPI-Exp1-Baseline-InjecAgent](https://github.com/boobaoooo/IPI-Exp1-Baseline-InjecAgent) | `d0d2720` | 1,183 raw records (270 benign, 913 IPI with the baseline template). All benign controls used in the paper come from here. |
| 2. IPI runs with the important-instructions payload | [IPI-Exp2-ImportantInstructions](https://github.com/boobaoooo/IPI-Exp2-ImportantInstructions) | `e679bf6` | 890 accepted IPI runs, the runner (`p2runner/`), helper scripts, `SHA256SUMS.txt`. |
| 3. Ground-truth labels | _to be added_ | _to be added_ | Action-level `ipi_affected` labels (CSV). |
| 4. Behavioral feature extraction | [person3_behavioral_processing](https://github.com/Pimpavee08/person3_behavioral_processing) | `f38cd6e` | Feature schema `p3-features-1.2` (77 primary features, frozen), code `p3` 1.2.1, and a README section describing how the 1,131-trajectory feature file was built. |
| 5. Detectors, evaluation, revision analyses | [IPI-Cross-LLM-Research-Archive](https://github.com/a3troooooo/IPI-Cross-LLM-Research-Archive) | `a917177` | Feature tables, 72 trained models (`outputs/models`), `P4_TERMINAL_PIPELINE`, `P4_REVISION_OUTPUT`. |

Commits were recorded on 15 Nov 2026. To match the paper, check out the commit listed for each repository.

## 2. Pipeline order

1. **Run agents** in AgentDojo (stages 1 and 2). Every raw record stores the runner source hash `runner_source_sha256 = 32395aeafbdd3316f621c1c984876d4e823c2ec9a3f0b56b19d012faee35b1eb`.
2. **Label trajectories** (stage 3).
3. **Extract blinded behavioral features** (stage 4). From the root of the stage 4 repository:
   `python -m p3 --raw ../trajectories_sanitized/trajectories_sanitized.jsonl --out output_pii_1131_p3-1.2 --expected 1131`
   - Input `trajectories_sanitized.jsonl`: 1,131 trajectories (873 IPI + 258 benign), sha256 `14b1c8f662b947445f3f86d10217c193e56c41a1cf59fb1c534ff1c4fcdacfac`. This file is not yet included in the stage 4 repository.
   - Output `features.csv`: 1,131 rows (`trajectory_id` + 77 features), sha256 `6bb8c94c5964d24117a7d832a2c05ad7fa0b911d0e301a33802df09e6ded414e`. The same file is stored as `features_p3_77_1131.csv` in the stage 5 repository (hash compared, identical).
   - Reported by the feature author: run 2026-10-30, Python 3.12.7, standard library only, repeated run gave identical bytes; the pipeline refuses to run if the frozen schema or code hash differs; the feature stage received no labels.
4. **Train and evaluate** LR / RF / GB detectors on the hash-locked test set (stage 5).
5. **Post-unseal threshold analysis** (stage 5, `P4_REVISION_OUTPUT`).

## 3. Where reported numbers come from

| Paper item | Source file |
|---|---|
| Table 3 (validation vs. test operating points) | `P4_TERMINAL_PIPELINE/P4_REVISION_OUTPUT/16_R2_R3_paper_tables.md`, built from `14_R2_original_threshold_table.csv` and `15_R3_strict_fpr_table.csv` |
| Mean test FPR 0.37 (0.07–0.74) and recall 0.93 over the 72 V_all cells | `P4_REVISION_OUTPUT/02_original_threshold_validation_vs_test.csv`, rows with `feature_regime = V_all` |
| Other tables and figures | Source files are being traced and will be listed here. |

## 4. Reproducibility notes

- All LLMs ran under Ollama on one RTX 3060 GPU at default 4-bit quantization with `temperature 0.0` and `seed 42`. Rerunning the agents gave identical tool-call sequences in only 45 of 120 repeated conditions (see the paper's limitations).
- Checksums use LF-normalised files (relevant on Windows). `SHA256SUMS.txt` in stages 1 and 2 verifies with `sha256sum -c`.
- The agent runs and the feature and detector pipelines were **not** re-run when preparing this release; only file hashes and reported tables were checked.

## 5. Data availability

Ground-truth labels will be published in this repository or a linked one (see stage 3). Other items not included in the component repositories, such as the sanitized trajectories, are available from the authors on reasonable request.

## 6. Citation

_To be added after publication._
