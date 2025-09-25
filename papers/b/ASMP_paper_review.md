# Review: *Learning to See Physical Properties with Active Sensing Motor Policies*

## 1. Citation Information
```bibtex
@inproceedings{Margolis2023ASMP,
  title     = {Learning to See Physical Properties with Active Sensing Motor Policies},
  author    = {Gabriel B. Margolis and Xiang Fu and Yandong Ji and Pulkit Agrawal},
  booktitle = {7th Conference on Robot Learning (CoRL 2023)},
  address   = {Atlanta, USA},
  year      = {2023},
  url       = {https://gmargo11.github.io/active-sensing-loco},
  eprint    = {2311.01405},
  archivePrefix = {arXiv},
  primaryClass  = {cs.RO},
  note      = {Also available as arXiv:2311.01405}
}
```

## 2. Clear Abstract 
The paper tackles how to infer terrain *physical* properties—like friction and roughness—from color images so robots can plan better. Instead of asking humans to label these properties, the authors train a quadruped to *actively* gather labels through its own movements. In simulation, a controller (the **Active Sensing Motor Policy**, ASMP) is rewarded for actions that make friction/roughness easier to estimate from proprioception (e.g., briefly dragging or swiping a foot). The robot then walks in the real world; its proprioceptive estimates become sparse but trusted labels that supervise a vision module predicting per-pixel physics from RGB images. Separately, the authors learn task-specific cost functions in simulation that map physical parameters to performance (e.g., speed) under different operating modes such as free walking vs. dragging a payload. At test time, the vision model can run on onboard or aerial images to produce physics maps, which are converted to cost maps for A* path planning. Experiments on a Unitree Go1 show that ASMP improves friction estimation versus a passive baseline and that the learned vision model generalizes to novel viewpoints. Planned paths reflect task needs—for instance, preferring sidewalks when dragging a payload but cutting across grass during free locomotion.

## 3. Keywords
- Active sensing  
- Self-supervised terrain perception  
- Legged locomotion  
- Friction & roughness estimation  
- Vision–proprioception coupling  
- Costmap learning  
- Aerial–ground teaming  
- Path planning (A*)  
- Sim-to-real transfer  
- Quadruped robotics

## 4. Problem Addressed
Robots often ignore terrain *material* properties (slip, softness) and rely on geometry/depth only. Labeling physical properties in images is hard for humans, limiting supervised approaches. The paper seeks a scalable way to learn vision that predicts per-pixel physical parameters to improve planning across tasks.

## 5. Innovation / Main Contribution
- **Active Sensing Motor Policies (ASMP):** A controller trained to take information-gathering actions that increase the accuracy of proprioceptive estimates of terrain physics.  
- **Self-supervised visual physics:** Use those proprioceptive labels to train a vision head that densely predicts friction/roughness from RGB, generalizing to new viewpoints (including drone footage).  
- **Task-aware cost learning:** Learn cost functions in simulation that map physics to performance for different operating modes (e.g., payload dragging), enabling planning directly from images.

## 6. Key Methods / Architecture
- **ASMP reward shaping:** Add an estimation bonus (penalize |e − ê|²) so the policy learns foot-drag/swipe behaviors that expose friction.  
- **Data collection:** Real-world traversals with a quadruped; project traversed ground pixels into camera frames and attach proprioceptive physics labels to those pixels.  
- **Vision module:** Frozen pretrained segmentation backbone → linear classifier over 20 discretized physics bins to produce per-pixel friction/roughness.  
- **Cost functions:** In sim, roll out policies across terrains spanning physics values; measure performance (e.g., realized velocity) to fit mode-specific traversal costs.  
- **Planning:** Convert predicted physics → cost map (per operating mode) → plan with A* from onboard or overhead imagery.

## 7. Datasets & Experimental Setup
- **Robot:** Unitree Go1 (12-DOF) with NVIDIA Jetson Xavier NX.  
- **Cameras:** Insta360 X3 (training images), DJI Mini 3 drone (overhead test images).  
- **Environments/terrains:** Grass, gravel, dirt, pavement, stairs; ~15 minutes of real-world traversal.  
- **Ground truth checks:** Dynamometer measurements of friction for terrain classes.  
- **Simulation:** Friction µ ∈ [0.25, 3.0]; 50 parallel agents × 20 s rollouts; evaluate free locomotion vs. dragging a 1 kg payload.  
- **Training details:** PPO for policies; linear head trained over 20 bins; hyperparameters and reward terms listed in appendices.

## 8. Evaluation Metrics & Results
- **Proprioceptive estimation quality:** ASMP yields *lower* friction estimation error than passive estimation; behaviors like brief foot swipes emerge.  
- **Agreement with physical measurements:** ASMP’s friction estimates align better with dynamometer readings and preserve terrain ordering (grass>pavement>dirt≈gravel by µ).  
- **Vision generalization:** The visual model matches proprioceptive distributions on train images and maintains reasonable variance on *test* images and new viewpoints (drone).  
- **Planning outcomes:** Using overhead images, **dragging payload** prefers sidewalk (45 ± 1 s) over grass (48 ± 1 s); **free locomotion** prefers cutting across grass (23 ± 1 s) over sidewalk (26 ± 0 s).  
- **Mode-dependent costs:** With payload, realized speed decreases as friction increases; without payload, controller maintains target speed except at very low friction.

## 9. Limitations (esp. vs. our BEV-semantics + elevation + MPC+CBF project)
- **Physics-only preference:** Optimizes for task performance via physics; does *not* encode human comfort/social norms—central to our human-preference objective.  
- **Limited terrain state:** Focuses on friction/roughness; no explicit **elevation** or slope/curb detection that our mapping emphasizes.  
- **Mapping scope:** Uses image-space prediction; lacks fused BEV semantic/elevation layers we rely on.  
- **Control layer:** Demonstrated with A* and teleoperated execution; no **MPC + CBF** safety constraints or guarantees.  
- **Labeling assumptions:** Needs physics to be simulatable and inferable from proprioception; relies on visible surfaces (occlusions untreated).  
- **Sim-to-real gap:** Small mismatch remains between estimates and dynamometer measures.

## 10. Similarity / Relevance to Our Work
- **Overlap:** Vision-based, per-pixel maps used to build **cost maps**; emphasis on **terrain-aware** planning; useful **aerial-ground** viewpoint generalization.  
- **Divergence:** We target **human-preference** costs and **semantics + elevation fusion** with **MPC+CBF** constraints; this work targets **physical affordances** (friction/roughness) with A*.  
- **Takeaway:** Their active self-supervision pipeline can complement our semantic/elevation maps by contributing physics layers and better labels.

## 11. Potential Integration Points
- **Use ASMP for data gathering:** Train brief “probing” maneuvers (safe micro-slides/taps) to collect richer labels for our terrain classes and elevation hazards.  
- **Physics-augmented cost map:** Add friction/roughness layers to our BEV map; combine with human-preference weights to reflect comfort *and* risk of slip.  
- **Mode-aware costs:** Learn cost mappings for our tasks (e.g., stability, energy, step-down risk) and blend with human-preference objectives.  
- **Overhead fusion:** Apply their viewpoint-robust visual physics to drone/overhead imagery to extend our planning horizon.  
- **Lightweight head on frozen backbone:** Replicate their linear-head training over discretized physics bins for fast adaptation in new locales.

## 12. Open Questions / Future Work
- Can physics prediction incorporate **uncertainty** and update online with new proprioceptive evidence?  
- How to extend beyond friction/roughness to **softness, compliance, wetness, sinkage**?  
- How to **fuse** physics, semantics, and **elevation** in one coherent BEV representation?  
- Can ASMP be constrained for **human-safe probing** and integrated with **CBFs**?  
- How stable is cross-robot generalization (legged → wheeled, different foot materials)?  
- Handling **occlusions** and dynamic conditions (e.g., rain turning grass muddy) in real time.

## 13. Reproducibility Notes
- **Artifacts:** Project page with qualitative results; paper includes reward tables, hyperparameters, and detailed procedures.  
- **Hardware/skills required:** Quadruped (Unitree Go1 or similar), 360° camera (Insta360 X3), drone (DJI Mini 3), GPU workstation; RL training expertise.  
- **Data:** Real-world traversals (~15 min) used; no public dataset explicitly specified.  
- **Complexity:** Moderate–high due to sim-to-real policy training and synchronized labeling pipeline.

## 14. Link
- **Project page:** https://gmargo11.github.io/active-sensing-loco  
- **arXiv:** https://arxiv.org/abs/2311.01405
