# TVMC Pipeline Analysis and Enhancement - Final Project README

## 📌 Overview
This project investigates and builds upon the **TVMC: Time-Varying Mesh Compression using Volume-tracked Reference Meshes** framework. Our goal was to reproduce the original pipeline, apply it to the **DFAUST dataset**, analyze its strengths and weaknesses, and implement multiple improvements to both reconstruction quality and compression performance.

---

## 🏗️ Original Pipeline Replication

We successfully replicated the TVMC pipeline from the original codebase. The primary steps included:

- **Mesh Preprocessing:** Converting DFAUST meshes into `.xyz` and `.obj` formats, decimating the base mesh.
- **Arap Tracking:** Using the provided ARAP solver to compute volume centers for each frame, tracking local deformations.
- **Dual Quaternion Tracking:** Calculating transformations between frame centers using `get_transformation.py`.
- **Displacement Calculation:** Finding displacements from each deformed frame back to a reference mesh.
- **Draco Compression:** Encoding displacements using the Draco library.
- **Evaluation:** Running `evaluation.py` on selected sequences (shake_arms, chicken_wings) to compute D1, D2, MSE, PSNR, bitrate, and decoding time.

### ✅ Datasets Used
We applied this pipeline to DFAUST sequences:
- `50004_shake_arms`
- `50004_chicken_wings`

For each sequence, we selected **10 frames** for evaluation (typically frames 11–20), following the original methodology.

---

## 🔎 Critical Analysis

### Strengths
- **Self-contact Handling:** Volume center tracking avoids self-occlusion artifacts.
- **Compression Modularity:** Mesh and displacements are encoded separately.
- **Self-contact Heuristics:** Frame selection uses simple, interpretable metrics.

### Weaknesses
- **Hard-Coded Frame Dropping:** Risk of discarding valuable motion.
- **No Learning Component:** Fully hand-crafted pipeline.
- **Poisson Reconstruction Limitations:** Assumes watertightness.
- **Euclidean Matching:** Ignores mesh topology.
- **Independent Compression:** Ignores temporal redundancy.

---

## 🧪 Observed Issues with DFAUST

When applied to DFAUST sequences, we found:

- Severe **spikiness and distortion** in raw outputs.
- **Incorrect displacements** due to one-way KD-Tree matches.
- **Open3D normal estimation failures** in frames with bad geometry.
- **High variance in results** across frames due to lack of smoothing.

---

## 🔧 Fixes and Enhancements Implemented

- **Displacement Clamping:** Limited max vector length to 0.02 units.
- **Laplacian Smoothing:** Applied 3 iterations before computing normals.
- **Fallback Normal Estimation:** Added fallback logic to Open3D.
- **Displacement Norm Debugging:** Logged magnitude stats per frame.
- **Bidirectional KD-Tree Matching:** Averaged forward and reverse displacement queries.
- **Confidence Weighting:** Used match distance to down-weight noisy updates.

---

## 📉 Compression Comparison: Draco vs TVMC

We compared:
- **Draco-Only Compression** (raw mesh input)
- **TVMC+Draco Compression** (displacement-based)

| Method         | D1 (mm) | D2 (mm) | log10 MSE | PSNR (dB) | Bitrate (Mbps) | Decode Time (ms) |
|----------------|---------|---------|------------|-----------|----------------|------------------|
| Draco Only     | 45.8    | 52.2    | -2.68      | 31.6      | 0.23           | 5.8              |
| TVMC + Draco   | 38.0    | 42.6    | -3.23      | 35.1      | 1.6            | 6.3              |

**TVMC outperforms Draco in accuracy (lower D1/D2, higher PSNR)** but has higher bitrate due to displacement encoding.

---

## 📊 Visual Analysis
We generated before/after reconstructions for DFAUST sequences.
- **Raw TVMC outputs** show distortion in limbs and face.
- **Post-fix reconstructions** show improved smoothness and fewer spikes.

---

## ⚠️ Limitations of the Current Pipeline

- Improvements worked well for shake_arms and chicken_wings, but **generalization to all DFAUST sequences failed**.
- High-motion or topologically complex regions still yield poor correspondences.
- **Mesh self-intersections** and occlusions disrupt nearest-neighbor matching.

---

## 💡 Proposed Future Work

- **Learned Deformation Prediction:** Train a neural network to replace raw displacement mapping.
- **Temporal Feature Tracking:** Incorporate time-aware consistency for improved displacement coherence.
- **Geometry-Aware Blending:** Use geodesic or feature-based matching instead of raw Euclidean distances.
- **End-to-End Framework:** Jointly train compression with decoding and correspondence estimation.

---

## 📁 Folder Structure (Replicated Setup)
```
TVMC/
├── arap-volume-tracking/
│   ├── data/50004_shake_arms/*.obj
│   ├── get_transformation.py
│   └── ...
├── tvm-editing/
│   ├── TVMEditor.Test/bin/Release/net5.0/Data/
│   │   └── shake_arms_2000/
│   │       ├── meshes/
│   │       ├── reference_mesh/
│   │       └── ...
├── draco/
│   └── build/draco_encoder-1.5.7
├── evaluation.py
├── draco_compression.py
└── README.md (this file)
```

---

## ✅ Key Takeaways

- TVMC improves compression and fidelity for structured datasets.
- Proper smoothing, filtering, and correspondence refinement are crucial for generalization.
- Integrating data-driven models may significantly enhance robustness for human mesh sequences.


