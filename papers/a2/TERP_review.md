# Paper Review: TERP — Reliable Planning in Uneven Outdoor Environments using Deep Reinforcement Learning

## 1. Citation Information (.bib)
```bibtex
@article{Weerakoon2022TERP,
  title   = {TERP: Reliable Planning in Uneven Outdoor Environments using Deep Reinforcement Learning},
  author  = {Kasun Weerakoon and Adarsh Jagan Sathyamoorthy and Utsav Patel and Dinesh Manocha},
  journal = {arXiv preprint arXiv:2109.05120},
  year    = {2022},
  eprint  = {2109.05120},
  archivePrefix = {arXiv},
  primaryClass  = {cs.RO},
  url     = {https://arxiv.org/abs/2109.05120}
}
```

## 2. Clear Abstract 
The paper presents **TERP**, a hybrid navigation framework for ground robots operating on uneven outdoor terrain. TERP builds a local **elevation map** from LiDAR, then uses a deep reinforcement learning network with **Convolutional Block Attention Modules (CBAM)** to extract an **attention mask** that highlights reduced-stability regions given the robot’s pose and goal. This attention is fused with the normalized elevation map to form a **navigation cost map** that encodes a measure of terrain planarity and likely stability (roll/pitch safety). TERP then computes **locally least-cost waypoints** with Dijkstra search over the cost map and follows them with **DWA-RL**, a controller that enforces kinodynamic feasibility and collision avoidance. Unlike end-to-end DRL policies that directly output velocities, TERP **decouples perception from control**, improving transfer from simulation to real robots and providing local optimality guarantees on waypoints. In high-elevation and curb scenarios, TERP increases success rates and reduces elevation-gradient exposure relative to DWA, ego-graph baselines, and an end-to-end Attn-DRL policy. The authors demonstrate real-time performance (10–12.5 Hz) and successful deployment on a Clearpath Husky with a Velodyne VLP-16 LiDAR in real outdoor environments.

## 3. Keywords
- uneven terrain navigation  
- elevation maps  
- attention (CBAM)  
- deep reinforcement learning (DDPG)  
- cost-map planning  
- Dijkstra waypointing  
- DWA-RL control  
- stability-aware planning (roll/pitch)  

## 4. Problem Addressed
Plan reliable, dynamically-feasible robot trajectories through **uneven outdoor terrain** where elevation changes and slopes can destabilize a ground robot. TERP seeks to avoid reduced-stability regions while still progressing efficiently toward the goal.

## 5. Innovation / Main Contribution
- Uses a **DRL-trained attention** (CBAM) purely for **perception** to emphasize terrain features tied to instability—**without human semantic labels**.  
- Constructs a **stability-oriented cost map** by fusing attention with **normalized elevation** (robot-ground-clearance aware).  
- Computes **locally least-cost waypoints** with guarantees to avoid unsafe regions (infinite costs) and then **tracks with DWA-RL** to ensure kinodynamic feasibility.  
- Demonstrates **sim-to-real robustness** compared to an end-to-end DRL baseline and classical planners.

## 6. Key Methods / Architecture
- **Inputs:** local elevation grid (n=40), robot pose, goal direction, heading gradient vector.  
- **Attn-DRL (perception):** DDPG with CNN trunk + **CBAM** → intermediate feature map → **attention mask**.  
- **Cost Map:** elementwise product of normalized elevation and attention; hard-thresholding to ∞ for robot-unreachable regions.  
- **Waypoint Selection:** arc-based candidate set on a goal-facing circle; **Dijkstra** cumulative cost → pick least-cost, goal-most-progressing waypoint; adaptive exploration radius based on local mean cost.  
- **Controller:** **DWA-RL** generates collision-free, kinodynamically-feasible velocities to follow the planned path.  
- **Reward shaping:** promotes goal progress, heading alignment, **low roll/pitch**, and avoidance of high elevation gradients.

## 7. Datasets & Experimental Setup
- **Simulation:** Unity + ROS Melodic; Clearpath Husky model with **Velodyne VLP‑16** 3D LiDAR; Grid-Map & OctoMap for elevation grids.  
- **Real Robot:** Clearpath Husky + VLP‑16; onboard laptop (Intel i9, RTX 2080).  
- **Scenarios:** Low (≤1 m), Medium (1–2 m), High (≥3 m) elevation, and **City-curb** (curbs above ground clearance).  
- **Baselines:** DWA, ego‑graph (with and without goal-distance cost), and **end‑to‑end Attn‑DRL**.

## 8. Evaluation Metrics & Results
- **Metrics:** Success Rate, **Cumulative Elevation Gradient (CEG)**, Normalized Trajectory Length, Goal Heading Deviation.  
- **Headlines:** In **high‑elevation** scenarios, TERP achieves **73%** success vs. DWA (**61%**), ego‑graph (**62%**), end‑to‑end Attn‑DRL (**60%**). In **city‑curb**, TERP reaches **62%** vs. DWA (**39%**) and ego‑graph (**29%**).  
- **Stability Exposure:** TERP lowers **CEG** (e.g., **2.09** in high‑elevation vs. DWA **2.49**, ego‑graph **2.83**).  
- **Runtime:** ~**0.08–0.10 s** per cycle (10–12.5 Hz), ~4× faster than a semantic segmentation baseline (GANav: **0.32–0.39 s**).

## 9. Limitations (esp. vs. our project)
- **No human preference modeling** (comfort/social norms); focuses on geometric stability.  
- **LiDAR-only elevation cues** can miss boundaries (e.g., surface transitions), and **low reflectance** around steep ditches reduces effective points.  
- No explicit BEV semantic fusion; cost is purely elevation/attention-derived.  
- **Control** relies on DWA‑RL, not **MPC + CBF** safety constraints.  

## 10. Similarity / Relevance to Our Work
- **Overlap:** elevation mapping; local cost-map planning; sim‑to‑real emphasis; attention to stability.  
- **Differences:** we target **human‑preference, terrain semantics (BEV), and elevation fusion** + **MPC/CBF** constraints; TERP uses attention from DRL instead of semantic labels and controls with DWA‑RL.

## 11. Potential Integration Points
- **Attention‑guided elevation costs:** Use TERP’s attention mask (learned stability salience) as a **modulator** on our BEV semantic cost map—e.g., down‑weight “preferred” semantics if attention signals instability (curbs, steep grass).  
- **Reward terms → CBF shaping:** Adapt **Rstable** (low roll/pitch) and gradient penalties into **barrier functions** or soft constraints within our MPC.  
- **Arc‑based waypointing:** Reuse their **goal‑facing arc + Dijkstra** scheme to escape local minima while retaining our semantic/elevation cost.  
- **Torque-aware hard constraints:** Map “infinite‑cost” unsafe regions to **hard CBF barriers** based on our platform’s torque/clearance limits.  
- **Runtime path:** Their lightweight pipeline suggests a real‑time budget compatible with our camera+LiDAR fusion.

## 12. Open Questions / Future Work (from authors)
- Improve sensing around **steep ditches / low reflectance**; integrate additional depth sensors.  
- **Fuse RGB with point clouds** to better delineate surface boundaries.  
- Extend generalization across environments/robots; tighten global guarantees beyond local least‑cost waypointing.

## 13. Reproducibility Notes
- Implementation details (ROS, Unity, PyTorch, Grid‑Map/OctoMap) and network architecture/reward functions are described.  
- No explicit public code/data link is provided in the paper; replication is **feasible but non‑trivial** (requires sim setup, DRL training, and platform‑specific torque/clearance tuning).

## 14. Link
Paper: https://arxiv.org/abs/2109.05120
