# Review: VERN — Vegetation-aware Robot Navigation in Dense Unstructured Outdoor Environments

## 1. Citation Information

```bibtex
@article{Sathyamoorthy2023VERN,
  title   = {VERN: Vegetation-aware Robot Navigation in Dense Unstructured Outdoor Environments},
  author  = {Sathyamoorthy, Adarsh Jagan and Weerakoon, Kasun and Guan, Tianrui and Russell, Mason and Conover, Damon and Pusey, Jason and Manocha, Dinesh},
  journal = {arXiv preprint arXiv:2303.14502},
  year    = {2023},
  archivePrefix = {arXiv},
  eprint  = {2303.14502},
  primaryClass = {cs.RO}
}
```

## 2. Clear Abstract 

The paper presents VERN, a navigation system for legged robots operating in dense outdoor vegetation where traditional planners either freeze or collide because sensors flag all plants as hard obstacles. VERN learns to distinguish **pliable, traversable** vegetation (e.g., tall grass) from **non-pliable, non-traversable** obstacles (e.g., bushes, trees) using a **few-shot, siamese classifier** trained on a few hundred RGB images. At run time, camera images are split into quadrants and compared to class exemplars to obtain both a class label and a confidence score. These outputs are fused with **multi-height LiDAR cost maps** to estimate vegetation height and produce a **vegetation-aware traversability cost map** that explicitly accounts for classification confidence and height. A DWA-based local planner then prefers low-cost (pliable) regions, reduces speed in uncertain areas, and triggers **holonomic recovery behaviors** when freezing or entrapment is detected. Experiments on a Boston Dynamics Spot in multiple real-world scenarios show large gains over baselines (Spot autonomy, DWA, GA-Nav, GrASPE) in success rate, freezing rate, trajectory efficiency, and false positives. VERN demonstrates that combining lightweight few-shot perception with height-aware cost shaping and recovery behaviors enables reliable navigation through dense vegetation without requiring dangerous negative-experience training.

## 3. Keywords

- vegetation-aware navigation
- pliability classification
- few-shot / siamese networks
- MobileNetV3 backbone
- multi-height LiDAR cost maps
- cost-map clearing / homography
- Dynamic Window Approach (DWA)
- cautious behaviors & recovery
- outdoor unstructured environments

## 4. Problem Addressed

Robots in dense, mixed vegetation often **freeze, collide, or get entrapped** because lidars treat all flora as solid obstacles and semantic models lack labels for *pliability/traversability*. The core question: **How can a robot reliably move through traversable vegetation while avoiding non-pliable obstacles in real time?**

## 5. Innovation / Main Contribution

- **Few-shot vegetation pliability classifier** (RGB, siamese with MobileNetV3) trained with only ~hundreds of images per class.
- **Vegetation-aware cost map (CVA)** combining class, **confidence**, and **height** from **multi-view (low/mid/high) LiDAR cost maps**; analytic “clearing” function guarantees PV (pliable) < NPV (non-pliable) costs.
- **Cautious navigation + recovery behaviors** layered on DWA to reduce speed in uncertain regions and escape freezing/entrapment.

## 6. Key Methods / Architecture

- **Image quadrant pipeline:** split RGB into 4 quadrants; compute similarity to reference sets for {sparse grass, dense grass, bush, tree}; output class + confidence (κ = e^(−α·d)).
- **Homography projection:** map image quadrants onto cost-map coordinates.
- **Multi-height LiDAR maps:** build Clow, Cmid, Chigh from point cloud slices; sum to form **Ccrit** to infer obstacle height.
- **Cost-map clearing function:** height- and confidence-aware rule that **reduces** costs for PV (with linear height term) and **raises** costs for NPV (with sinusoidal height term); analytically ensures PV costs < NPV costs.
- **DWA planning over CVA:** obstacle term = sum of CVA along simulated trajectories; **velocity space scaled by κ** for cautious motion.
- **Holonomic recovery:** detect freezing/entrapment, backtrack to stored safe poses, and mark unsafe locations with high cost.

## 7. Datasets & Experimental Setup

- **Platform & sensors:** Boston Dynamics **Spot**, Velodyne **VLP-16** LiDAR, Intel **RealSense L515** RGB, onboard Intel NUC 11 (i7 + RTX 2060).
- **Training data:** ≈600 images **per class** (UMD campus), prepared via quick pairing; ~6 hours training for 55 epochs (TensorFlow 2).
- **Scenarios (10 trials each):**
  1) sparse tall grass + trees,
  2) dense tall grass + trees (close proximity),
  3) trees + bushes + dense grass,
  4) dense grass + trees + **unseen human** obstacle.

## 8. Evaluation Metrics & Results

- **Metrics:** Success rate, Freezing rate, Normalized trajectory length (≈1 is best), False Positive Rate (FPR) for pliability classification.
- **Headline outcomes (scenarios 1–4):**
  - **VERN success rate:** **100%, 90%, 70%, 70%** (best overall).
  - **Freezing rate:** **0%, 10%, 20%, 30%** (lowest).
  - **Trajectory length:** **1.11, 1.19, 1.28, 1.23** (shortest/most direct).
  - **FPR:** **0.15, 0.18, 0.12, 0.21** (lowest vs GA-Nav 0.33–0.61; GrASPE 0.25–0.44).
- **Backbone comparison (classification):** MobileNetV3 **Acc 0.957 / F1 0.833** > EfficientNet (0.758/0.632) > ViT (0.689/0.546).

## 9. Limitations (esp. vs. our BEV+Elevation+MPC+CBF project)

- **Planner mismatch:** Method assumes **non-holonomic DWA** action space while Spot is holonomic; our MPC+CBF can better exploit full dynamics.
- **Weak semantics:** PV/NPV is coarse; lacks rich **BEV semantic classes** (e.g., sidewalk vs snow) central to our human-preference costs.
- **Elevation detail:** Uses **multi-height 2D slices** rather than a fused **elevation map**; may miss curbs/drop-offs we care about.
- **Entangled vegetation/thin obstacles:** Assumes classes aren’t intertwined and may miss thin branches → collisions.
- **Generalization & seasonality:** Few-shot approach may drift with lighting/season changes; no domain-shift analysis provided.

## 10. Similarity / Relevance to Our Work

- **Overlap:** Uses camera semantics + geometric cues to modulate a **spatial cost map** and adapt motion behavior—aligned with our idea of human-preference costs over semantically labeled terrain.
- **Divergence:** Focus is **pliability** (physics-based traversability), not **human preference** or social norms; planning with DWA (not MPC+CBF); no explicit **BEV mapping** or **elevation fusion** beyond height slices.
- **Takeaway:** Their **confidence- and height-aware cost shaping** is directly relevant to how we weight terrain categories in our BEV cost map.

## 11. Potential Integration Points

- **Confidence-aware costs:** Incorporate classifier **confidence κ** into our terrain weights and **CBF barrier slack** (e.g., lower max speed where κ is low).
- **Height-aware shaping:** Borrow the **linear vs sinusoidal** height terms to penalize tall NPV and cautiously permit tall PV—adapted to our elevation map.
- **Few-shot metric learning:** Use a **siamese / prototypical** approach to add **new terrain classes** (e.g., snow, mud) with few labels for our BEV segmenter.
- **Holonomic recovery playbook:** Add their **safe-pose memory & retreat** routine as a fallback in our MPC stack.
- **Homography sanity-check:** Use their quadrant→map **homography** trick to quickly bootstrap camera-to-map alignment for BEV.

## 12. Open Questions / Future Work

- How robust is the classifier across **seasonal shifts** and **camera poses**?
- Can PV/NPV be expanded to **richer terrain semantics** to better align with human preferences?
- How to integrate **proprioceptive feedback** (slip, effort) with visual pliability for occluded ground?
- What are the **runtime** and **latency** bounds on embedded compute?
- How does performance scale beyond 4 quadrants (e.g., **patch-level BEV**)?
- Authors suggest adding proprioception and relaxing action constraints to holonomic control—both align with our roadmap.

## 13. Reproducibility Notes

- **Code/data:** No public code or dataset links are provided in the paper; dataset creation is described (≈30–45 min of manual grouping and pairing).
- **Hardware:** Commodity sensors/platform (Spot, VLP-16, L515, NUC 11) are standard; training details (55 epochs, MobileNetV3, contrastive loss) are given.
- **Parameters:** Some crucial weights in the clearing function (e.g., (w_s, w_d, w_{NPV}, b_{NPV}, α)) are not enumerated, so exact replication may require tuning. Overall reproducibility: **moderate**.

## 14. Link

- Paper (arXiv): https://arxiv.org/abs/2303.14502

---

**How this informs our project:** VERN’s **confidence/height-aware** cost shaping and **few-shot terrain recognition** are the most transferable ideas. We can fuse these with our **BEV semantic map + elevation grid** and enforce them within **MPC+CBF** for safety-guaranteed, **human-preference** navigation.
