# VLM-GroNav: Robot Navigation Using Physically Grounded Vision-Language Models in Outdoor Environments

## 1. Citation Information

```bibtex
@article{Elnoor2024VLMGroNav,
  title   = {VLM-GroNav: Robot Navigation Using Physically Grounded Vision-Language Models in Outdoor Environments},
  author  = {Mohamed Elnoor and Kasun Weerakoon and Gershom Seneviratne and Ruiqi Xian and Tianrui Guan and Mohamed Khalid M. Jaffar and Vignesh Rajagopal and Dinesh Manocha},
  journal = {arXiv preprint arXiv:2409.20445},
  year    = {2024},
  url     = {http://gamma.umd.edu/vlm-gronav/}
}
```

## 2. Clear Abstract

The paper proposes **VLM-GroNav**, a navigation system for outdoor robots that mixes **vision-language models (VLMs)** with **proprioceptive sensing** to understand terrain not just by how it looks, but by how it **physically behaves** (e.g., deformable mud, slippery snow). The system uses **in-context learning** to update a VLM’s terrain-traversability estimates with real-time proprioceptive feedback, and feeds those estimates to both a **global planner** (using aerial imagery + VLM reasoning to pick waypoints) and a **local planner** (a DWA variant that prefers safer “frontiers”). Tests on a **legged** (Ghost Vision 60) and a **wheeled** (Clearpath Husky) robot across mixed terrains show higher success rates (up to **100%** in scenarios) and lower vibration/energy than prior methods.

## 3. Keywords

- Vision-Language Models (VLMs)
- Proprioception / physical grounding
- Traversability estimation
- In-context learning
- Global waypoint selection (aerial imagery)
- Dynamic Window Approach (DWA) with frontier cost
- Slippage & deformability estimation
- Outdoor robot navigation
- Legged and wheeled robots
- GPS-guided planning

## 4. Problem Addressed

Outdoor terrains can **look** safe but **behave** unpredictably (deform/slip). Pure vision (RGB/LiDAR) misses key physical properties; pure proprioception is local and cannot foresee hazards ahead. The paper tackles **how to combine wide-area visual context with physically grounded feedback** to plan robust global and local paths.

## 5. Innovation / Main Contribution

- **Physically informed VLM traversability**: Use **proprioceptive signals** (sinkage for legs, slippage for wheels) as **in-context examples** to calibrate a VLM’s terrain judgments on-the-fly.
- **VLM-guided global planner**: Visually **mark** aerial images and prompt a large VLM to choose waypoints that optimize objectives (e.g., avoid slippery/deformable regions).
- **Real-time local planner**: Add a **frontier cost** to DWA, using a compact VLM/CLIP classification + updated traversability to bias toward safer nearby paths.

## 6. Key Methods / Architecture

- **Sensors**: RGB camera, 3D LiDAR, IMU, GPS; plus encoders/forces (legs) or wheel odometry (wheels).
- **Traversability indicators**:
  - **Deformability (legs)** via sum of squared joint forces, normalized to τ_sinkage.
  - **Slippage (wheels)** via discrepancy between LiDAR and wheel odometry (distance & yaw), forming τ_slip.
- **Physically grounded reasoning**: Build an **exemplar pool** pairing images (aerial + front) with time-aligned τ and terrain labels; query VLM with **in-context learning** to estimate current τ_estimate.
- **Global planning**: Overlay candidate markers on aerial image; prompt VLM with objective + τ_estimate to select waypoint sequence; **re-query** as τ updates.
- **Local planning**: CLIP zero-shot terrain classification at **left/center/right** frontiers; modify **DWA** with a **frontier cost** that prefers closer, higher-τ frontiers.

## 7. Datasets & Experimental Setup

- **Robots**:
  - **Ghost Vision 60** (legged): wide-angle RGB, **OS1-32** LiDAR, GPS, Intel NUC 11 (i7 + RTX 2060).
  - **Clearpath Husky** (wheeled): **Velodyne VLP-16**, **RealSense D435i**, GPS, laptop (i9 + RTX 2080).
- **Software/models**: **GPT-4o API** for reasoning/global; **CLIP** for local zero-shot terrain classification.
- **Scenarios (real-world outdoor)**:
  1) Dry grass → muddy grass → concrete (legs)
  2) Dry grass → sand → concrete (legs)
  3) Concrete → dry grass → muddy grass (wheels)
  4) Concrete → snow → muddy grass (wheels)

## 8. Evaluation Metrics & Results

- **Metrics**: Success rate; **normalized trajectory length**; **IMU energy density** (sum of squared accelerations).
- **Baselines**: **DWA**, **GA-Nav**, **CoNVOI**, **ViNT**; plus ablations (**w/o global planner**, **w/o ICL**).
- **Headlines**:
  - **Best success rate in all 4 scenarios**: **100%** (Scen.1 & 2), **90%** (Scen.3 & 4).
  - **Lower IMU energy density** across scenarios vs. baselines, indicating smoother, more stable motion.
  - **Abstract claim**: up to **50%** improvement in success rate over SOTA.

## 9. Limitations

- **GPS dependence** for global planning; performance may drop in GPS-denied settings.
- Does **not** integrate additional modalities (e.g., thermal/hyperspectral) that could help in low-visibility.
- VLM processing adds **latency**; further optimization needed for highly dynamic scenes.

## 10. Similarity / Relevance to Our Work

**Overlap with our unified mapping & navigation idea**
- Shares the goal of **terrain-aware navigation** combining semantics and **physical properties** (deformability/slip).
- Also mixes **vision** (top-down + onboard) with **proprioception** to refine **traversability**.
- Uses a **cost-aware planner** (local DWA + global waypointing) that prefers safer terrain—aligned with our multi-layer cost map ambitions.

**Key divergences**
- VLM-GroNav uses **VLM reasoning + aerial imagery** for **global** waypoint selection, rather than a unified **multi-layer map** (geometry + semantics + physical properties) like we propose.
- Local control is **DWA** with frontier cost; our plan envisions **MPC** on a fused multi-layer map.
- Physical properties are inferred **online via proprioception** (and slippage from odometry mismatch), whereas we also want **vision-based property prediction** at range.

**Our current limitations vs. this work**
- We don’t yet leverage **in-context learning** to **update** terrain properties using proprioceptive feedback.
- We haven’t explored **VLM-guided global waypointing** over aerial imagery.

**How it informs our research**
- Demonstrates a practical recipe to **ground** semantic priors with **measured physics** (sinkage/slip) and feed that into both global and local planning—exactly the coupling our framework seeks.

## 11. Potential Integration Points

- **In-context learning loop**: Use our proprioceptive cues (e.g., friction proxies) to **continually calibrate** vision-predicted terrain properties in our map.
- **Frontier cost for local planning**: Add a **terrain-aware frontier term** to our MPC/A* stack to bias toward safer nearby regions.
- **Aerial-image waypoint prompting**: Overlay candidate waypoints on satellite/aerial tiles and **prompt a VLM** to choose routes consistent with our multi-layer costs (e.g., “prefer sidewalks, avoid low-friction”).
- **Slippage estimator**: Integrate **LiDAR vs. wheel odometry** discrepancy as a live **slip indicator layer** in our unified map.
- **Time-shifted alignment** of τ with camera frames to build a **training exemplar pool** for rapid on-site adaptation.

## 12. Open Questions / Future Work (from authors)

- How to operate robustly in **GPS-denied** environments?
- Can adding **thermal/hyperspectral** improve poor-visibility navigation?
- How to **speed up** VLM reasoning for fast, dynamic scenes?

## 13. Reproducibility Notes

- **Robots/sensors** and high-level software stack are described; **GPT-4o** and **CLIP** are used.
- A **supplemental tech report & video** are advertised; **code release is not stated** in the paper (link below). Overall, replication is feasible with similar hardware but depends on access to VLM APIs and careful prompt/exemplar engineering.

## 14. Link

- Project/Supplemental: <http://gamma.umd.edu/vlm-gronav/>
- arXiv: (as cited) **arXiv:2409.20445**

## 15. Recap — what the paper actually does (start → finish)

- **Look broadly**: Grab **aerial imagery** and the navigation goal.
- **Seed understanding**: Ask a **large VLM** to label terrain types & initial traversability on the aerial view.
- **Mark options**: Overlay candidate **waypoint markers** on the aerial image.
- **Pick a route**: Prompt the VLM (with objectives like “avoid slippery/deformable”) to select an **ordered waypoint list**.
- **Drive locally**: Onboard camera + LiDAR define **left/center/right frontiers**; a **compact VLM/CLIP** labels their terrain types.
- **Measure physics**: From the robot’s **proprioception**, compute **sinkage** (legs) or **slippage** (wheels) to produce a **traversability score τ**.
- **Ground the VLM**: Build an **exemplar pool** (images + aligned τ + labels) and use **in-context learning** so the VLM **updates** its traversability estimates.
- **Refine planning**:
  - **Local**: Run **DWA** with an extra **frontier cost** that prefers closer, higher-τ frontiers to steer around hazards.
  - **Global**: If τ changes materially, **re-query** the VLM on the **marked aerial** to **replan waypoints**.
- **Repeat** until the goal is reached, continually balancing **visual cues** with **measured physical behavior** of the terrain.
