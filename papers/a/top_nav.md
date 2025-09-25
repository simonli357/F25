# TOP-Nav — Paper Review

## 1. Citation Information

```bibtex
@article{Ren2024TOPNav,
  title   = {TOP-Nav: Legged Navigation Integrating Terrain, Obstacle and Proprioception Estimation},
  author  = {Junli Ren and Yikai Liu and Yingru Dai and Junfeng Long and Guijin Wang},
  year    = {2024},
  journal = {arXiv preprint arXiv:2404.15256},
  note    = {8th Conference on Robot Learning (CoRL 2024), Munich},
  url     = {https://top-nav-legged.github.io/TOP-Nav-Legged-page/}
}
```

## 2. Clear Abstract
The paper presents **TOP-Nav**, a legged robot navigation system that blends **vision** (terrain & obstacle perception) with **proprioception** (motion feedback) in a closed loop. A planner selects waypoints using a combined cost map—terrain traversability, obstacle costs, proprioceptive “disturbance” cues, and goal progress. A learned locomotion controller tracks waypoints and provides a value-function–based motion score used to (i) detect invisible obstacles/disturbances and (ii) **online-correct** vision-based terrain estimates, especially on **novel terrains** or under poor visuals. Experiments in Isaac Gym and on Unitree hardware show higher success rates, more stable motion, and lower energy than prior navigation baselines.

## 3. Keywords
- Legged navigation  
- Terrain traversability  
- Proprioception / value function  
- Cost-map planning  
- Vision-based terrain classification (BEV patches)  
- Online uncertainty & entropy  
- Reinforcement learning controller  
- Isaac Gym simulation  
- Unitree Go1/Go2 hardware

## 4. Problem Addressed
Vision-only legged navigation often fails on **unseen terrains**, changing lighting, or **invisible obstacles**. The paper tackles: **How to integrate proprioception and vision to plan safer, more traversable paths and adapt online to novel conditions**.

## 5. Innovation / Main Contribution
- **Closed-loop integration** of vision and proprioception **at both planning and control levels**.  
- **Proprioception Advisor:** uses an RL **critic/value estimate** (from the locomotion policy) as a real-time motion evaluation to detect disturbances and inject costs into the planner—**no extra training/sensors** beyond the controller’s learned critic.  
- **Online terrain correction:** adjusts vision-derived terrain costs using (a) classifier uncertainty (entropy) and (b) nearest-neighbor matching in the classifier’s **latent feature space** to previously traversed patches with recorded proprioceptive cost—**adapts to novel terrains without on-the-fly retraining**.

## 6. Key Methods / Architecture
- **Hierarchical System**
  - **Path planner** over a local cost map \(M_C\) combining:
    - Terrain traversability \(M_T\) (vision + online corrections)
    - Obstacle occupancy \(M_O\) (from depth → point cloud → SDF)
    - Proprioception advice \(M_P\) (critic-based motion score projected forward/laterally)
    - Goal approaching \(M_G\) (distance / heading-based)
  - Dynamic weights for \(M_T, M_G\) based on distance/time.  
- **Proprioception Advisor**
  - Uses normalized **critic value \(c_t\)** from the RL controller to raise costs ahead/around when motion quality drops (e.g., slips/collisions), also triggers simple recovery behaviors.  
- **Vision Terrain Estimator**
  - RGB → BEV; discretize into patches.
  - Lightweight **MobileNet** classifier outputs class probabilities; entropy gives **confidence**.
  - Map classes to reference costs via prior walking experience.  
- **Online Corrections for Novel Terrains**
  - Cache latent features \(L\) and local proprioceptive cost traces \(T_P\).
  - For a new patch, find nearest previously seen feature by cosine similarity and blend \(M_T\) with proprioceptive prior **weighted by confidence**.  
- **Locomotion Controller**
  - RL (PPO) policy with a learned value function; tracks velocity & yaw to follow waypoints.  
- **Real-time**
  - Controller 50 Hz; planner ~3 Hz; onboard Jetson NX; RealSense D435i.

## 7. Datasets & Experimental Setup
- **Simulation:** NVIDIA Isaac Gym; 8×8 grid of 5 m×5 m cells, random start-goal; obstacles & terrains with graded difficulty.  
- **Hardware:** Unitree **Go1/Go2**; **RealSense D435i** RGB-D; onboard **Jetson NX**.  
- **Terrain prior data:** ~5 min teleop per terrain to collect critic values & RGB for class cost mapping; classes include bush/brick/snow/gravel/slab/cement/paved/grass.

## 8. Evaluation Metrics & Results
**Metrics:**  
- **SR**: success rate (within 0.5 m in 20 s)  
- **TD**: average traversed terrain cost (lower = simpler terrains)  
- **UT**: fraction of time with \(|\text{roll}|>0.15\) or \(|\text{pitch}|>0.15\)  
- **VFT**: velocity tracking failures  
- **AEC**: energy (\u03c4·q̇)  

**Simulation (Table 1):**
- **TOP-Nav:** **SR 73.62%**, **TD 22.65%**, **UT 27.21%**, **VFT 18.14%**, **AEC 92.08**  
- **VP-Nav:** SR 65.62%, TD 47.18%, UT 36.18%, VFT 26.09%, AEC 101.8  
- **Obstacle-only:** SR 52.81% …  
→ Terrain awareness + proprioadvisor raise SR by ~8% vs VP-Nav and halve TD; also reduce UT/VFT/AEC.

**Real-world (Table 2; novel terrains):**
- **Unknown gravel:** TOP-Nav SR 5/5; UT 14.30; AEC 57.46; Time% 62.72  
- **Slippy tarpaulin:** TOP-Nav SR 5/5; UT 1.19; AEC 83.40; Time% 67.62  
→ Outperforms VP-Nav/Sterling by maintaining stability and avoiding challenging patches.

**Additional hardware (Table 6; known terrains):**
- **TOP-Nav:** SR 15/15; **UT 7.79**; **AEC 59.07**; **Time% 65.03**  
→ Better stability/energy than GA-Nav/VP-Nav/Obstacle-only.

## 9. Limitations
- Vision odometry vulnerable to lighting; suggests adding **LiDAR-odometry**.  
- Costmap local planning can cause **waypoint jitter** when several low-cost options exist; propose a **diffusion-based** task planner for smoother sequences.

## 10. Similarity / Relevance to Our Work
**Overlap with our idea (unified geometry–semantics–traversability):**
- Uses **vision semantics → terrain classes & costs** (though fairly coarse).
- Integrates **proprioceptive “terrain property” signal** via value function (implicitly captures friction/roughness/slip).
- Builds a **multi-layer cost map** for planning (terrain, obstacles, proprio, goal).

**Divergence:**
- No explicit **geometric elevation mapping**; obstacle map from depth but not a fused 3D elevation layer.
- Terrain properties are **implicit** via critic and class cost; no explicit **friction/roughness** estimates per cell.
- Focused on **legged** robots and controller-coupled value; our plan targets **platform-agnostic** (wheeled/legged) high-level navigation.

**What it suggests for us:**
- **Value-function–as-traversability** is powerful: a single score summarizing stability, slip, tracking errors → useful third layer alongside semantics & geometry.
- **Online correction using classifier uncertainty + latent NN** is a light, data-efficient way to handle **novel surfaces** (snow, mud, detergent).  
- Closed-loop **proprioception → planner** feedback can detect **invisible obstacles** (e.g., glass).

**Our current limitations vs. this work:**
- We don’t yet feed **proprioceptive performance** back into the planner in real time.
- Our friction layer may be vision-predicted; TOP-Nav shows **on-the-fly adjustment** via motion outcomes, reducing reliance on offline priors.

## 11. Potential Integration Points
- **Add a Proprioception Layer:** Use robot-agnostic indicators (IMU roll/pitch variance, slip ratio, velocity tracking error, energy) or a **learned critic/value head** to populate a **proprio-cost map \(M_P\)** that temporarily “blocks” disturbed directions.
- **Uncertainty-Gated Fusion:** Compute **semantic uncertainty** (MC dropout/augment entropy) to **blend** vision-derived traversability with proprio-derived costs, **more weight to proprio** when confidence is low.
- **Latent Memory for Novel Terrain:** Maintain a small **feature–cost cache** (e.g., CLIP/segmentation backbone features + recent proprio cost) and **nearest-neighbor** retrieval to adjust costs for visually-similar patches encountered earlier in the run.
- **Invisible-Obstacle Handling:** When motion score drops abruptly, inject a **fan-shaped high-cost band** forward to trigger sidestep/replan.
- **Dynamic weighting:** Increase/decrease terrain vs. goal weights based on **distance/time** to avoid excessive detours near the target.

## 12. Open Questions / Future Work
- How well does value-based proprio cost **generalize across robots** or when control policies change?  
- Can the terrain estimator move beyond discrete classes to **continuous properties** (friction, stiffness)?  
- Replace local costmap waypoint jitter with **sequence-planning** (e.g., diffusion or MPC over waypoints).  
- Robustness under extreme visual degradation; potential **LiDAR fusion** for geometry and odometry.

## 13. Reproducibility Notes
- **Code/data:** Not explicitly released in the paper; a **project page** is provided.  
- Implementation details include **network sizes**, **training steps**, **hyperparameters**, and **hardware frequencies**, plus **reward terms** and **metric thresholds**, which is helpful but still requires reimplementation.

## 14. Link
- Project page: https://top-nav-legged.github.io/TOP-Nav-Legged-page/

## 15. Recap (step-by-step)
- **Train an RL locomotion policy** (actor–critic) for a quadruped; the **critic’s value** is a compact measure of motion quality.  
- **Build four local maps** around the robot: terrain \(M_T\), obstacles \(M_O\), proprioception \(M_P\), and goal \(M_G\).  
- **Terrain \(M_T\):**  
  - RGB → BEV patches → lightweight classifier → class → **reference traversability cost**.  
  - Compute **entropy-based confidence**; when low (novel/uncertain), **correct** the cost using a **nearest-neighbor** in the classifier’s latent space to a **previously traversed** patch with recorded **proprio cost**.  
- **Proprio \(M_P\):**  
  - Read the **critic value** online; if it drops (slip/collision), raise directional costs (a band in front & laterally) to discourage continuing straight.  
- **Obstacle \(M_O\):**  
  - Depth → point cloud → SDF → obstacle costs.  
- **Fuse to \(M_C\)** with dynamic weights (terrain weight decreases near goal; goal weight increases over time).  
- **Select next waypoint** by integrating costs along rays to the costmap edge; **controller tracks** velocity/yaw to reach it.  
- **Repeat online:** proprio feedback continuously adjusts costs; terrain costs are **corrected on-the-fly** for novel surfaces.  
- **Results:** higher success/stability/lower energy vs. VP-Nav/GA-Nav/Sterling in sim and on Unitree robots.

### How this informs our unified framework
- Add a **proprio-feedback layer** and **uncertainty-gated fusion** to our multi-layer map (geometry, semantics, **physical**).  
- Use a **value function** or robot-agnostic performance metrics as an online **terrain property proxy**, complementing our explicit friction/roughness estimates.  
- Implement **latent-memory corrections** so our planner adapts in minutes, not model retrains.

