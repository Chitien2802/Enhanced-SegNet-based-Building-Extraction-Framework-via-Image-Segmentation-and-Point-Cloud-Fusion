# Enhanced-SegNet-based-Building-Extraction-Framework-via-Image-Segmentation-and-Point-Cloud-Fusion
Modified SegNet fusing airborne orthomosaics with photogrammetric point clouds for automatic building extraction. SMRF separates ground/non-ground points; DBSCAN and normal-vector region growing recover complete structures. 2D masks and 3D clusters fuse in image space to refine boundaries. On ISPRS Vaihing precision 0.87→0.94, F1 0.86→0.87, IoU 76%
This work presents a modified SegNet-based framework that fuses airborne orthomosaics and photogrammetric point clouds for automatic building extraction in dense urban scenes. First, a modified SegNet with an ImageNet-initialized VGG16-Batch Normalization encoder produces airborne orthophotos masks from ISPRS Vaihingen dataset. In parallel, photogrammetric point clouds are filtered using the Simple Morphological Filter to separate ground and non-ground points, clustered with DBSCAN, and refined via normal-vector-based region growing to recover complete building structures and suppress noise. The 2D segmentation and 3D clusters are then fused in image space, where geometric attributes refine building boundaries, remove spurious detections, and confirm structurally consistent building.
Quantitative experiments on Vaihingen Area 35 show that the fused framework improves over a 2D only modified SegNet baseline, raising precision from 0.87 to 0.94 and F1 from 0.86 to 0.87, with IoU reaching 0.77. In addition, our modified SegNet improves IoU from 67.98% reported by a recent satellite-based method to 76%. Qualitative comparisons further highlight sharper, less fragmented building footprints that better match the ground truth. Overall, the proposed SegNet-based building extraction pipeline demonstrates that combining lightweight deep segmentation with simple geometric post-processing yields accurate and robust building masks suitable for urban mapping, 3D city modelling, and digital twin applications.


Requirements

| Component | Version |
|-----------|---------|
| Python | 3.10 |
| CUDA | 11.8 |
| Anaconda | 2023+ |
| PyTorch | 2.0.1 |

---

## 🚀 Installation
```bash
git clone https://github.com/YOUR_USERNAME/ISPRS-Building-Reconstruction.git
cd ISPRS-Building-Reconstruction
conda env create -f environment.yml
conda activate Pointcloud_Building2026
```

---

## 📂 Data

Download **ISPRS Vaihingen** dataset:
👉 https://www.isprs.org/education/benchmarks/UrbanClassification/vaihingen.aspx

Place files under `data/area35/`. See `data/README.md` for full structure.

---

## 🔄 Pipeline
```
RGB Orthophoto (.tif)
      │
      ▼
[Step 0] SegNet Inference → prediction_binary.tif
      ▼
[Step 1] Extract Building Boundaries → buildings.json
      ▼
[Step 2] SMRF: Ground / Non-Ground separation (.las)
      ▼
[Step 3] Filter Non-Ground by mask → building points
      ▼
[Step 4] DBSCAN Clustering + colorization
      ▼
[Step 5] Fusion & Localization → final building LAS
```

---

## ▶️ Usage

### Step 0 — SegNet Inference
Edit paths in `src/test_af_train_segnetmodel.py`, then:
```bash
python src/test_af_train_segnetmodel.py
```

### Step 1 — Boundary Extraction
```bash
python src/1_ISPRS_boundaryinto_json.py \
  -i data/area35/prediction_binary.tif \
  -o data/area35/buildings.json \
  --min-area-px 9 --connectivity 8
```

### Step 2 — Ground Separation (SMRF)
Edit `INPUT_LAS`, `OUTPUT_GROUND_LAS`, `OUTPUT_OBJ_LAS` in script, then:
```bash
python src/2_ISPRS_extract_Ground-Non_Ground.py
```

### Step 3 — Filter by Mask
```bash
# Option A: filter by TIF mask
python src/3_1_ISPRS_convert_mask_intoonly_point.py

# Option B: filter by JSON boundaries
python src/3_ISPRS_extract_Building_points.py
```

### Step 4 — DBSCAN Clustering
```bash
python src/4_ISPRS-Clustering_from_lasfile.py \
  -i data/area35/building_points.las \
  -o results/clustered.las \
  --min-samples 15
```

### Step 5 — Localize Clusters
```bash
python src/5_ISPRS_LOC_CLUSTER.py
```

---

## 📊 Results

Training curves (200 epochs) are in `results/training_curves/`.
Per-building IoU metrics: `results/evaluation/iou.xlsx`.


## 📁 Project Structure
```
├── src/                          # All pipeline scripts (Steps 0–5)
├── data/
│   ├── area35/                   # ISPRS Area 35 input files
│   ├── train/ & test/            # Tiled training/test patches
│   └── README.md
├── results/
│   ├── training_curves/          # accuracy, iou, loss, precision, recall, dice
│   └── evaluation/iou.xlsx
├── environment.yml
└── README.md
```
