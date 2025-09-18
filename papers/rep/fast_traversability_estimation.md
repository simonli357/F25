# Fast Traversability Estimation for Wild Visual Navigation — Structured Report

## 1. Citation Information
```bibtex
@inproceedings{Frey2023WVN,
  author    = {Jonas Frey and Matias Mattamala and Nived Chebrolu and Cesar Cadena and Maurice Fallon and Marco Hutter},
  title     = {Fast Traversability Estimation for Wild Visual Navigation},
  booktitle = {Proceedings of Robotics: Science and Systems (RSS)},
  address   = {Daegu, Republic of Korea},
  date      = {2023-07-10/2023-07-14},
  year      = {2023},
  url       = {https://bit.ly/3M6nMHH}
}
```

## 2. Clear Abstract 
Robots often misinterpret natural clutter like tall grass and undergrowth as rigid obstacles, causing conservative routes or failures. This paper proposes **Wild Visual Navigation (WVN)**, an **online, self-supervised**, **vision-only** system that rapidly learns **platform-specific traversability** from a short human teleoperation demo. WVN extracts dense semantic cues using **self-supervised ViT (DINO-ViT)** features and compresses computation via **SLIC superpixels**, then couples two lightweight MLP heads: one that regresses traversability and another that reconstructs features to estimate **confidence/anomaly**. A **Supervision Graph** records recent footprints and a **Mission Graph** stores features; reprojection from the footprint generates sparse labels continuously, enabling on-the-fly learning. WVN integrates with a local mapping pipeline that raycasts image-space traversability into a 2.5D elevation map and uses an SDF-based local planner to command motion. On an ANYmal quadruped, WVN adapts within minutes and supports closed-loop navigation in forests, parks, and grasslands, including **1.4 km** autonomous footpath following with minor interventions. Compared to purely geometric methods, WVN correctly treats high grass as traversable when appropriate, improving path quality. While demonstrated on legged hardware, the approach generalizes to other ground robots.

## 3. Keywords
- Online self-supervised learning
- Traversability estimation
- DINO-ViT features
- Anomaly/confidence modeling
- Supervision & Mission Graphs
- SLIC superpixels
- Elevation mapping & SDF planning
- Legged robot (ANYmal)
- Visual navigation
- Preference-aware path following

## 4. Problem Addressed
Accurately estimating **which natural terrain is safe to traverse** is hard with geometry alone (e.g., high grass appears as obstacles). Large, labeled semantic datasets are expensive and not portable across environments/robots. WVN seeks **fast, in-field adaptation** of traversability using **vision only**, minimal supervision, and lightweight online learning.

## 5. Innovation / Main Contribution
- **Online, self-supervised vision-only traversability** that **adapts in minutes** from brief human demonstration.
- **Feature-efficient learning**: uses **self-supervised ViT** features with **superpixel pooling** and **two-head MLP** for traversability + reconstruction-based confidence.
- **Dual-graph supervision mechanism** (Supervision & Mission Graphs) that **reprojects footprints** to generate sparse labels during operation.
- **Closed-loop integration** with visual traversability projected onto a local elevation map and **SDF-based planning** for autonomous navigation.

## 6. Key Methods / Architecture
- **Perception & Features**
  - Monocular RGB → **DINO-ViT** dense features → **SLIC** (~100 segments) → per-segment embeddings.
- **Self-Supervision Signal**
  - **Velocity-tracking error** (difference between commanded and actual body velocity) smoothed and mapped via sigmoid → traversability score.
- **Graphs for Label Propagation**
  - **Supervision Graph** (ring buffer of recent footprints + scores).
  - **Mission Graph** (images, segments, features).
  - **Reproject** footprints into mission images to assign per-segment labels; unlabeled segments default conservative (0).
- **Learning**
  - **Two-head MLP** shares hidden layers:
    - **Reconstruction head** learns to reconstruct traversed features → **confidence** via Gaussian fit to reconstruction loss.
    - **Traversability head** trained with **confidence-weighted MSE** and an online **ROC-based threshold** at bounded FPR.
- **Closed-Loop Control**
  - Raycast image traversability into **2.5D elevation map**; median filter → **SDF**; local SE(2) twist planner; locomotion controller for rough terrain.

## 7. Datasets & Experimental Setup
- **Platform:** ANYbotics **ANYmal C** with Jetson **Orin AGX**; main sensing: single wide-FOV **RGB** (Sevensense Alphasense Core); LiDAR/depth only for local mapping.
- **Environments:** Forests, parks, grasslands; additional offline datasets **Hilly**, **Forest**, **Grass** with dedicated train/val/test splits.
- **Procedure:** Teleop for ~**2 minutes** to seed supervision; then autonomous local navigation and long-range path following.

## 8. Evaluation Metrics & Results
- **Headline results**
  - **1.4 km** autonomous footpath following (3 runs: **0.55 km**, **0.50 km**, **1.4 km**), with parameterized FPR (0.15–0.3) and minor/major interventions mainly at intersections/muddy patches.
  - **Fast adaptation:** Good accuracy within ~**200 optimization steps** from scratch due to per-image segment multiplication.
- **Ablations (offline)**
  - **Scene adaptation table:** training/testing across Hilly/Forest/Grass shows best performance in-scene; notable drops cross-scene → motivates online adaptation.
- **Qualitative comparisons**
  - Versus **geometry-only** heuristics/learned traversability, WVN correctly keeps **high-grass** traversable and trees untraversable, leading to cleaner SDFs for planning.

## 9. Limitations
- Runtime ~**2.5 Hz** on Orin with unoptimized code.
- **Superpixels** can merge semantically different regions; limits mask fidelity.
- **Velocity-error** as supervision is a coarse proxy; may miss nuanced interactions.
- **Single camera** + projection/raycasting can cause FoV and mapping artifacts; filtering required.
- Needs **continual learning** mechanisms for longer-term adaptation.

## 10. Similarity / Relevance to Our Work
Your work targets **human-preference, terrain-aware navigation** that fuses **semantics + elevation** and encodes **comfort/safety/social norms** in the cost map, with **MPC + CBF** for constraint-aware control.

- **Overlap**
  - Both build **semantic BEV/costs** rather than pure geometry and **fuse with elevation** for planning.
  - WVN’s **preference-like behavior** emerges when demos bias traversability (e.g., preferring footpath over muddy grass).
- **Divergence**
  - WVN learns **platform-specific “can I go?”**; it does **not** explicitly encode **human comfort/social norms** or multi-class semantic labels.
  - WVN uses **online self-supervision** via **velocity error**, not human preference annotations or CBF-constrained optimal control.

## 11. Potential Integration Points
- **Replace/augment semantic BEV with WVN traversability** as an additional **data-driven layer** that adapts to local appearance (e.g., seasonal grass/snow) while your **human-preference costs** steer the policy.
- **Confidence-aware fusion:** Use WVN’s **reconstruction-based confidence** to gate updates to your cost map or to **tighten/relax CBFs** when confidence is low/high.
- **Demo-conditioned preferences:** Seed WVN with **brief human teleop over desired substrates** (sidewalk > grass > snow) to quickly align learned traversability with **preference rankings**, then map to your **preference weights**.
- **Adaptive thresholding:** Borrow WVN’s **ROC/FPR-bounded thresholding** to auto-tune traversable masks as scenes change.
- **Compute-aware deployment:** Start with WVN for **rapid field bootstrapping**, then distill into your **MPC+CBF** planner’s costs for sustained operation.

## 12. Open Questions / Future Work (from authors)
- **Faster pipeline** beyond 2.5 Hz.
- **Better segmentation** than SLIC superpixels to avoid mixed segments.
- **Richer supervision signals** beyond velocity error.
- **Multi-camera** predictions for wider coverage.
- **Continual learning** across missions/seasons.

## 13. Reproducibility Notes
- **Project page:** provided (link below).
- **Code/data:** Paper mentions a project page but does **not** explicitly state public code or datasets in the text; experiments describe **hardware specifics** and **parameter settings** (e.g., **kσ=2**, **FPR=0.15** typical) and dataset splits for offline ablations.

## 14. Link
- **Project page:** [https://bit.ly/3M6nMHH](https://bit.ly/3M6nMHH)

## 15. Summary
1️⃣ Motivation & Goal

Natural scenes (forest, grass, brush) confuse geometry-only planners: tall grass looks like a solid wall to LiDAR.

Want a fast, robot-specific traversability estimator that learns on-site with almost no labeled data.

2️⃣ Quick Human Demo

An operator drives the robot for ~2 minutes over the terrain.

During this demo, the system logs camera images and the robot’s actual vs. commanded velocity.

3️⃣ Automatic Label Generation

After each step, it compares commanded velocity to actual body velocity.

If the robot slowed down a lot, the terrain is labeled “hard/untraversable.”

Smooth this signal and convert it to a numerical traversability score.

4️⃣ Visual Feature Extraction

Each RGB image is fed through a self-supervised Vision Transformer (DINO-ViT) → dense feature maps.

Use SLIC superpixels to group pixels into ~100 segments and average features per segment.

5️⃣ Graph-Based Label Propagation

Supervision Graph: stores recent robot footprints with their traversability scores.

Mission Graph: stores all image segments and features.

Project the 3-D footprints back into past images to assign labels to the matching superpixels.

6️⃣ Online Learning

Train a two-head MLP:

Traversability head: predicts score for each segment.

Reconstruction head: tries to reconstruct the segment’s feature; reconstruction error gives a confidence/anomaly measure.

Uses a confidence-weighted loss and automatically sets a threshold to keep a fixed false-positive rate.

7️⃣ Real-Time Mapping

Project per-pixel traversability predictions into a 2.5-D elevation map (built from LiDAR for geometry).

Median-filter and convert to a signed distance field (SDF).

8️⃣ Planning & Control

Local planner searches the SDF for low-cost, collision-free paths.

Sends velocity commands to the ANYmal quadruped’s locomotion controller.

9️⃣ Field Deployment

After the short teleop, the robot drives fully autonomously, adapting online as it encounters new terrain.

Demonstrated on 1.4 km autonomous runs in forests and parks with only minor interventions.

🔟 Key Takeaways

No manual labels, no pretraining for traversability.

Learns “can my robot walk here?” in minutes, purely from RGB video + onboard odometry.

Handles natural obstacles better than geometry-based planners, especially in tall grass.