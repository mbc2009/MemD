# MemD — Code & Data Companion

Machine-learning study of the mass- and heat-transfer behavior of graphene aerogel (GA)
membranes for membrane distillation. This repository is the code/experimental companion to
the manuscript *"Transformer-Enhanced 3D CNN for Predicting Structural and Transport
Properties of Porous Materials for Membrane Distillation."*

## Top-level layout

| Path | Description |
|---|---|
| `Lmp_ML/` | The machine-learning pipeline — data preprocessing, the Transformer-enhanced 3D CNN (TRM-CNN), training, and evaluation. **Documented in detail below.** |
| `EXP_DCMD/` | Experimental direct-contact membrane-distillation hardware: 3D-printable CAD (CATIA `.CATPart`/`.CATProduct`, `.STL`, `.form`) for DCMD chambers and molds. |
| `LICENSE` | Apache-2.0. |

---

## `Lmp_ML/` — file structure

```
Lmp_ML/
├── README.md                   # Project synopsis: application, inputs, outputs, model, data source
├── Data_pretreatment.ipynb     # Preprocessing: LAMMPS atomic data → 128×128×128 3D density matrices
├── Model_trainning.ipynb       # Training scaffold for the 3D CNN + Transformer model
├── LmpGP.xlsx                  # Tabular dataset: scalar inputs (len, sigma, temp) + scalar targets
│
├── Trials/                     # Full-dataset experiments (model-architecture iterations)
│   ├── Trial_1.ipynb
│   ├── Trial_2.ipynb
│   ├── Trial_3.ipynb
│   ├── Trial_3.5.ipynb
│   ├── Trial_4.ipynb
│   └── Trial_5.ipynb
│
└── Trials_LessData/            # Reduced-dataset experiments — the branch used for the paper
    ├── Trial_3.ipynb
    ├── Trial_3.5.ipynb
    ├── Trial_3_index.ipynb                # ★ Paper-of-record: TRM-CNN, overall R² = 0.7412
    ├── Trial_3_index_CNN_Benchmark.ipynb  #   Baseline 3D-CNN (identical split) for benchmarking
    └── Trial_3_percentage.ipynb
```

### What each file does

| File / folder | Role |
|---|---|
| `README.md` | One-page summary of the study (application background, input/output specification, model, and the LAMMPS-derived data parameters). |
| `Data_pretreatment.ipynb` | Converts each GA model (LAMMPS full-atom data file) into a standardized 200 Å supercell and then into a 128³ voxel grid encoding carbon-atom density per unit volume — the structural input tensor for the network. |
| `Model_trainning.ipynb` | Reference/scaffold notebook defining the 3D CNN + Transformer (TRM-CNN) architecture and training loop. (The values reported in the paper come from the `Trials_LessData/Trial_3_index.ipynb` run — see below.) |
| `LmpGP.xlsx` | The scalar dataset: per-sample input parameters (`len`, `sigma`, `temp`) and target properties used for training/evaluation. |
| `Trials/` | The earlier, **full-dataset** experiment series. Each `Trial_N` is an iteration of the architecture / training setup. |
| `Trials_LessData/` | The **reduced-dataset** experiment series. This branch contains the deterministic index-based split, the CNN benchmark, and the run that produced the paper's reported metrics. |

### The pipeline

1. **Data generation** *(external, LAMMPS — not in this folder)* — high-throughput molecular-dynamics simulations produce GA structures and their measured properties.
2. **Preprocessing** — `Data_pretreatment.ipynb` voxelizes each structure into a 128³ density matrix.
3. **Training** — a Trial notebook trains the TRM-CNN (and, for benchmarking, a plain 3D CNN) with 10-fold cross-validation on the training split.
4. **Evaluation** — predictions are scored (MAE, MSE, RMSE, EVS, PCC, R²) overall and per target.

### Inputs / outputs / model *(per `Lmp_ML/README.md`)*

- **Inputs:** testing temperature `T`, and the 128³ 3D matrix of the GA (carbon-atom density per voxel).
- **Targets:** water-vapor diffusivity and thermal conductivity, plus morphological parameters — density, porosity, tortuosity, pore radius, specific surface area, and water flux.
- **Model:** Transformer-enhanced 3D CNN (TRM-CNN) = two 3D-conv layers → subregion tokenization → Transformer encoder → fully connected head, with the temperature concatenated late.
- **Dataset design:** 836 samples = 19 `len` (graphene sheet mean length) × 11 `sigma` (inclusion zero-potential distance) × 4 `temp`.

### Which notebook reproduces the paper

The manuscript's reported results correspond to **`Trials_LessData/Trial_3_index.ipynb`**:

- It outputs the paper's overall **R² = 0.7412**.
- Its data split matches the manuscript exactly: 836 total → test = indices for `len` 10–13 (176 samples), train = `len` 2–8 ∪ 15–20 (572 samples) with `len` 9 & 14 excluded as a leakage buffer; ×4 rotation augmentation → 2288, minus 1 % → 2266 *(the manuscript states 2265 — a harmless off-by-one)*.

Its sibling **`Trial_3_index_CNN_Benchmark.ipynb`** uses the identical split and is the baseline 3D-CNN against which the TRM-CNN's gains (MSE −56.78 %, R² +16.80 %, training time −28.22 %) are computed.

### Notebook naming conventions

- `Trial_N` / `Trial_N.5` — successive architecture/training iterations.
- `_index` — deterministic, **index-range (structure-aware) split** by graphene-sheet length (used in the paper).
- `_percentage` — percentage-based (random) split variant.
- `_CNN_Benchmark` — the plain 3D-CNN baseline (no Transformer).
- `Trials/` vs `Trials_LessData/` — full-dataset vs reduced-dataset experiment series.

---

**License:** Apache-2.0 &nbsp;|&nbsp; **Ownership:** Honglin Liu (MSE Ph.D., Georgia Tech)
&nbsp;|&nbsp; **Note:** Please do not copy or use without the author's permission.
