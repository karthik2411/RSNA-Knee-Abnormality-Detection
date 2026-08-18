# RSNA Knee Abnormality Detection

Multimodal deep-learning models that detect **twelve clinically important abnormalities** on knee MRI examinations — 2026 RSNA Knee Abnormality Detection AI Challenge (Kaggle).

> A single knee scan can reveal a dozen different problems. The task: predict the per-study probability of each of twelve findings from MRI scans, learning from images paired with original radiology reports.

**Status:** active · **Deadline:** 22 Oct 2026 · **Metric:** macro-averaged AUC ROC ·

---

## The problem

The knee is the most commonly injured and imaged joint in the body. ACL/MCL tears, meniscal damage, cartilage loss, and fractures can be subtle, and access to musculoskeletal radiologists is limited — leading to delays and inconsistent diagnoses. A reliable model acts as decision support: accurate, consistent, and fast.

## The twelve findings

Each is an independent binary label (`0` / `1`):

| Group | Labels |
|---|---|
| Ligaments | `ACL`, `MCL` |
| Meniscus | `Medial Meniscus`, `Lateral Meniscus` |
| Osteoarthritis | `Medial OA`, `Lateral OA`, `PF OA` |
| Other | `Effusion`, `Synovitis`, `Baker's`, `Contusion`, `Fracture` |

A healthy knee = all twelve are `0`. There is no separate "normal" label.

## Evaluation

Score = the mean AUC ROC across the twelve targets:

```
Final Score = (1/12) · Σ AUCᵢ
```

Each finding is graded independently, so rare findings count as much as common ones.

### Efficiency Prize (our target track)

A second track scores **accuracy and runtime together**, since accurate models are often heavy:

```
Efficiency = AUC / (Benchmark − maxAUC) + RuntimeSeconds / 32400
```

Lower is better. `Benchmark` = the all-0.5 sample submission's score, `maxAUC` = best private-leaderboard AUC, `RuntimeSeconds` = inference time (32400 s = 9 h). Accuracy and speed are weighted roughly equally by design — so a lean, fast model can beat a slow, brilliant one. This is the realistic prize target for a small team.

---

## The data

- **569.76 GB**, ~819,640 files, DICOM + CSV.
- **4,407 training studies.** Each study = one knee, one session.
- A study contains several **series** (scan runs: sagittal / coronal / axial, different fluid/fat settings).
- Each series contains ~20–45 **slices** (median 30) — one `.dcm` per slice, each a cross-section at a different depth.

### The key twist

Only **58 of 4,407 studies (~1%) carry the 12 labels.** The rest come with a **free-text radiology report** (multilingual — Spanish, Dutch, etc.). The labels for those studies must be **derived from the reports.** At test time the report is **not** provided — so the final model reads images only.

### Files

```
train.csv            one row per study — StudyInstanceUID, Report, 12 labels
train_series.csv     one row per series — plane, fluid-sensitive, fat-suppression
train_series/        DICOMs: <StudyInstanceUID>/<SeriesInstanceUID>/<SOPInstanceUID>.dcm
test.csv             example test IDs (swapped for hidden ~1,300 studies at scoring)
test_series.csv      test series descriptors
test_series/         test DICOMs
sample_submission.csv  valid submission, all labels = 0.5
```

The shared `StudyInstanceUID` links labels ↔ series info ↔ image folders.

---

## Approach

Because only ~1% of studies are directly labeled, the plan is **mine labels from reports first, then train an image model** on the result.

```
1 · Setup + explore        environment, read studies / series / slices / labels
2 · Baseline submission    all-0.5 file → proves the pipeline (on the board ~0.50)
3 · Mine labels            read reports → derive 12 yes/no answers for ~4,349 studies
4 · Image model v1         train a CNN on the mined labels
5 · Make it efficient      shrink + speed up for the efficiency track
6 · Final submission       select best, submit
```

The 58 gold labels are held out as a **validation set** — the only human-verified ground truth, used to check both the label-mining and the model.

## Progress

- [x] **1 · Setup + explore**
- [x] **2 · Baseline submission** — on the leaderboard at ~0.50
- [ ] **3 · Mine labels from reports** ← current
- [ ] **4 · Train image model**
- [ ] **5 · Make it efficient**
- [ ] **6 · Final submission**

---

## Notes

- `.dcm` pixel values are not 0–255 — they need normalizing before display or training.
- Findings are fairly balanced (rarest `MCL` ≈ 15%, most 20–60%), so no exotic imbalance handling needed for v1.

---
