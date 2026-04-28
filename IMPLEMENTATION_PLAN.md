# 4-5 Hour Interview Implementation Plan (RTX 4090)

## Goal
Deliver a clean, demo-first repository that can:
1. Train a 3D brain tumor segmentation baseline on a tiny BraTS subset.
2. Export a 3D mesh (`.stl`/`.obj`) from predicted tumor masks.
3. Showcase hardware-aware optimization and measurable quality.

## Scope Lock (to avoid over-engineering)
- Dataset: 3-5 BraTS cases only.
- Label setup: binary tumor mask (tumor vs background) for speed.
- Model: MONAI 3D UNet baseline (no architecture experiments).
- Validation: Dice + HD95 on the small split (if time allows).
- Deliverables: trained checkpoint, one predicted mask, one smoothed mesh, one GIF.

## Fast Timeline

### 00:00-00:45 - Environment + Data
- Install dependencies (`requirements.txt`).
- Prepare `data/raw/case_xxx/{image.nii.gz,label.nii.gz}`.
- Run a quick integrity check (load shape, spacing, non-empty labels).

Exit criteria:
- At least 3 valid cases load without errors.

### 00:45-02:15 - Training + Quick Validation
- Use `src/transforms.py` with:
  - `Spacingd` to `(1,1,1)`
  - `NormalizeIntensityd`
  - Patch sampling at `(128,128,128)`
- Train via `src/train.py` with AMP enabled.
- Save best checkpoint to `output/checkpoints/best_model.pt`.

Exit criteria:
- Training completes without OOM.
- Validation Dice trends up and best checkpoint is saved.

### 02:15-03:15 - Inference + Mesh Reconstruction
- Run inference on one held-out case using `src/infer.py`.
- Convert predicted mask to mesh via `scripts/export_mesh.py`.
- Apply smoothing (`smooth_iterations=10-30`) and save `.stl`.

Exit criteria:
- Usable mesh file opens in a 3D viewer.

### 03:15-04:15 - Demo Packaging
- Record 10-15s rotating 3D mesh GIF.
- Finalize `README.md` with:
  - Setup commands
  - Pipeline diagram (text-level is enough for now)
  - Hardware acceleration note (RTX 4090 + AMP)
  - Metric note (Dice + why Hausdorff/HD95 matters for geometric safety)

Exit criteria:
- Repo can be cloned and run with clear instructions.

### 04:15-05:00 - Buffer / Polish
- Optional: add HD95 computation and sample output.
- Optional: include one failure case + limitations section.
- Optional: clean commit history into 2-3 meaningful commits.

## Practical Defaults for RTX 4090
- `target_spacing=(1.0,1.0,1.0)`
- `patch_size=(128,128,128)`
- `batch_size=1` (increase only if VRAM allows)
- `epochs=80` for tiny demo subset
- AMP enabled by default

## Risks and Mitigations
- **OOM during training** -> reduce patch size to `(96,96,96)` or disable heavy augmentations.
- **Poor Dice with tiny data** -> frame as "engineering demo", emphasize pipeline completeness.
- **Noisy mesh artifacts** -> threshold and increase smoothing iterations.
- **Time overrun** -> skip advanced metrics first; secure end-to-end demo before extras.

## What Interviewers Usually Care About
- End-to-end execution under constraints.
- Correct handling of medical volume preprocessing (spacing/intensity).
- Geometric post-processing for AR/3D visualization use cases.
- Clear communication of trade-offs and next improvements.
