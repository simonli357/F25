# Report on *A Sim2Real Deep Learning Approach for the Transformation of Images from Multiple Vehicle-Mounted Cameras to a Semantically Segmented Image in Bird’s-Eye View (Cam2BEV)*

## 1. Citation Information (BibTeX)

```bibtex
@article{Reiher2020Cam2BEV,
  title   = {A Sim2Real Deep Learning Approach for the Transformation of Images from Multiple Vehicle-Mounted Cameras to a Semantically Segmented Image in Bird’s Eye View},
  author  = {Lennart Reiher and Bastian Lampe and Lutz Eckstein},
  journal = {arXiv preprint arXiv:2005.04078},
  year    = {2020},
  note    = {© 2020 IEEE. Personal use permitted; permissions required for other uses.},
  url     = {https://arxiv.org/abs/2005.04078}
}
```

## 2. Clear Abstract (150–200 words)

This paper presents **Cam2BEV**, a method to convert images from multiple vehicle-mounted cameras into a **semantically segmented bird’s-eye-view (BEV)** map that corrects the distortions introduced by classic **Inverse Perspective Mapping (IPM)**. Instead of learning from photorealistic images—which exacerbates the simulation-to-reality gap—the authors train entirely on **synthetic data** while feeding networks **semantic segmentation masks** as inputs. Two architectures are explored: (i) a **single-input DeepLabv3+** that takes a stitched, IPM-warped “homography image” and learns to fix its errors; and (ii) **uNetXST**, a multi-input U-Net variant with **Spatial Transformer** blocks that project intermediate features from each camera into a common BEV frame before fusion. The pipeline explicitly models **occlusions** by adding an “unknown/occluded” class. Trained on Virtual Test Drive (VTD) with ~33k samples and evaluated using **Mean IoU**, both approaches **substantially outperform** raw IPM; uNetXST is best overall while using fewer parameters than DeepLab-Xception. Qualitative tests on real street scenes—using a separate model to generate input semantics—show reasonable generalization to real data. Code and data are available publicly. The approach yields BEV maps suitable not only for free-space but also for **dynamic object localization** and occlusion awareness, providing a practical bridge from multi-camera perception to downstream planning.

## 3. Keywords

- Bird’s-Eye View (BEV)
- Inverse Perspective Mapping (IPM)
- Semantic segmentation
- Multi-camera fusion
- Spatial Transformer Networks (STN)
- U-Net / DeepLabv3+
- Sim-to-Real (domain gap)
- Occlusion reasoning
- Surround view perception
- Automated driving

## 4. Problem Addressed

How to obtain an **accurate, semantically segmented BEV** of the surrounding scene from **multiple monocular cameras**—**without** the geometric distortions of IPM and **without** requiring manually labeled BEV data—so that dynamic objects and occlusions are represented reliably for automated driving.

## 5. Innovation / Main Contribution

- **Sim2Real via semantic inputs:** Train only on **synthetic** BEV labels while using **semantic camera inputs** to reduce texture-induced reality gaps.
- **Two effective architectures:**
  - **DeepLabv3+ (single-input):** Learns to **correct** a stitched IPM “homography image.”
  - **uNetXST (multi-input):** Introduces **in-network projective alignment** using fixed homographies as **Spatial Transformers**, then fuses features across cameras.
- **Explicit occlusion class** with principled rules, enabling the BEV to encode **unknown/blocked** areas.
- **No manual BEV labeling** is needed; training labels come from simulation’s top-down “drone” view.

## 6. Key Methods / Architecture

- **Inputs:** Per-camera **semantic masks** (not RGB), optionally pre-warped by IPM (single-input variant).
- **Projective preprocessing:** Derive homographies from camera intrinsics/extrinsics; build a stitched **homography image** covering the 360° footprint.
- **uNetXST:**
  - Separate encoder per camera stream.
  - **Spatial Transformer** blocks apply fixed homographies to **feature maps** at each scale to align to BEV.
  - Concatenate aligned features and decode via a shared U-Net decoder with skip connections.
- **Loss/training:** Class-weighted cross-entropy (log-frequency weights), Adam (lr=1e-4), input resized to 512×256, batch size 5.
- **Metrics:** Per-class **IoU** and **Mean IoU (MIoU)**.

## 7. Datasets & Experimental Setup

- **Simulator:** **Virtual Test Drive (VTD)**.
- **Sensors:** Four wide-angle virtual cameras (front/rear/left/right) → 360° surround; also a separate single-front-camera dataset.
- **Resolution:** Original 964×604; training with center-crop and resize to **512×256**.
- **Sampling:** ~**33k** train / **3.7k** val samples at **2 Hz** (multi-camera); ~**32k/3.2k** for the single-camera dataset.
- **Classes:** 9 visible (road, sidewalk, person, car, truck, bus, bike, obstacle, vegetation) + **occluded**.
- **Ground truth:** BEV “drone” labels (~70 m × 44 m FoV).

## 8. Evaluation Metrics & Results

- **Metric:** IoU (per-class) and **MIoU**.
- **Headline results (validation):**
  - **uNetXST:** **MIoU 71.92%** (best).
  - **DeepLab-Xception (single-input):** **71.35%**.
  - **DeepLab-MobileNetV2:** **66.60%**.
  - **Ablations without IPM/STN:** 45.95–60.13% (notably worse).
  - **Raw IPM homography baseline:** **30.17%**.
- **Class highlights (uNetXST):** Road **98.10%**, Sidewalk **93.36%**, Car **80.90%**; lower for **Person 13.56%** and **Bike 32.43%** due to small object frequency/size.
- **Qualitative real-world tests:** Using a separate semantic segmenter for inputs, both models produce reasonable BEVs; **uNetXST** outputs are smoother and more consistent.

## 9. Limitations (with respect to our project: human-preference, terrain-aware navigation with BEV semantics, elevation mapping, and MPC+CBF)

- **No human preferences or social norms:** Outputs semantic BEV + occlusions only; **no learned terrain desirability** or human comfort weighting.
- **Flat-plane reliance for alignment:** Homographies assume a **planar ground**; elevation changes (curbs/ramps) are not modeled; authors note potential for **miscalibration** under vehicle dynamics.
- **Limited terrain taxonomy:** Focused on **road scenes**; categories like **snow, grass types, mud** (critical for semi-structured outdoor spaces) are not included.
- **No elevation map fusion:** Lacks **3-D elevation** or slope/roughness estimation required for our elevation-aware cost map.
- **No control layer:** Does not integrate with **MPC + CBF**; output suitability for safety constraints is not analyzed.

## 10. Similarity / Relevance to Our Work

- **Strong overlap:** Producing a **camera-only semantic BEV** aligns directly with our **BEV semantic terrain map** module; the **occlusion mask** is also valuable.
- **Divergence:** Their target domain is structured roads; our domain includes **sidewalks/grass/snow** and **human-preference costs** plus **elevation** and **MPC+CBF** for safe, norm-aware motion.
- **Takeaway:** Cam2BEV offers a **robust alternative to naive IPM**, potentially improving our BEV quality and occlusion handling for downstream planning.

## 11. Potential Integration Points

- **Replace/augment our IPM step:** Use **uNetXST** to fuse multi-camera semantics and produce a **distortion-corrected BEV** before cost-map construction.
- **Occlusion-aware planning:** Incorporate their **occlusion class** into our **CBF constraints** (e.g., treat “unknown” as risk-weighted barriers).
- **Sim2Real strategy:** Train on **synthetic BEV labels** while feeding **semantic masks** from real data to narrow the reality gap; this aligns with our **camera-first** philosophy.
- **Class-weighted training & small-object fixes:** Adopt their **log-frequency weighting** and extend with focal/Tversky losses or hard-example mining for rare terrains (e.g., **snow patches**).
- **Dynamic homography in the loop:** Extend STN blocks to ingest **IMU/pose** for **time-varying** alignment; could reduce BEV drift on slopes/curbs.
- **Downstream hooks:** Export BEV semantics + occlusion as layers for our **preference-weighted cost map**, then feed to **MPC+CBF**.

## 12. Open Questions / Future Work (from the paper)

- **Add depth:** Integrate **stereo/monocular depth or LiDAR** to handle non-planarity and improve object dimensions.
- **Full 360° real-world validation:** Deploy the **multi-camera** configuration beyond front-camera demos.
- **Dynamic calibration:** Account for **vehicle pitch/roll** and time-varying extrinsics during inference.

## 13. Reproducibility Notes

- **Code & data:** **Open-source** repository with models and datasets.
- **Training recipe:** Input resize to **512×256**, **Adam (1e-4)**, mini-batch **5**, **class-weighted** loss, semantic-only inputs, ~**33k/3.7k** train/val on VTD.
- **Architectures:** DeepLabv3+ (Xception ~41M params, MobileNetV2 ~2.1M) and **uNetXST (~9.6M)** with **STN** feature warps.
- **Evaluation:** **IoU/MIoU** reporting plus qualitative real-world demos using a separate semantic segmenter for inputs.

## 14. Link

- **Project / code:** https://github.com/ika-rwth-aachen/Cam2BEV
- **Paper (arXiv):** https://arxiv.org/abs/2005.04078
