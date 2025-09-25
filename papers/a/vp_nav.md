# Coupling Vision and Proprioception for Navigation of Legged Robots — Structured Review

## 1. Citation Information
```bibtex
@misc{fu2022vpnav,
  title        = {Coupling Vision and Proprioception for Navigation of Legged Robots (VP-Nav)},
  author       = {Zipeng Fu and Ashish Kumar and Ananye Agarwal and Haozhi Qi and Jitendra Malik and Deepak Pathak},
  howpublished = {Manuscript/PDF provided by the user; project page with videos},
  note         = {Legged point-goal navigation that couples a visual planner with proprioceptive safety advisors},
  url          = {https://navigation-locomotion.github.io},
}
```

## 2. Clear Abstract
VP-Nav is a point-goal navigation system for quadruped robots that combines vision and proprioception. A depth-camera–based visual planner builds an occupancy map and computes a cost map and desired velocities. In parallel, a proprioception-driven **Safety Advisor** detects invisible obstacles (e.g., glass) and predicts near-term fall risk, updating the map and capping speed accordingly. A velocity-conditioned locomotion policy (learned in simulation and transferred to hardware) executes the commands. The system outperforms vision-only and wheeled baselines in simulation and on a real Unitree A1 across rough, slippery, and cluttered settings.

## 3. Keywords
- Legged navigation
- Proprioception
- Vision-based mapping
- Safety advisor (collision detection, fall prediction)
- Fast Marching Method (FMM) planning
- Rapid Motor Adaptation (RMA)
- Occupancy and cost maps
- Sim2real transfer (Unitree A1)
- Velocity-conditioned locomotion policy

## 4. Problem Addressed
Enable reliable point-goal navigation for **legged** robots in environments where vision alone misses critical terrain properties (slipperiness, softness) or obstacles (glass), by tightly coupling high-level planning with proprioceptive feedback reflecting the robot’s true interaction with the world.

## 5. Innovation / Main Contribution
- Tight coupling of navigation and locomotion via a proprioceptive **Safety Advisor** that online (i) injects “missed” obstacles into the map when collisions are sensed and (ii) adapts a safe speed limit based on predicted fall risk.
- Continuous-space visual planner that fuses FMM goal distance with obstacle distance (SDF) and outputs smooth linear/angular velocity commands matched to the locomotion policy’s feasible region.
- Sim2real pipeline combining a velocity-conditioned policy and RMA-style adaptation, running **fully onboard** a low-cost Unitree A1.

## 6. Key Methods / Architecture
- **Perception / Mapping**: Realsense D435 depth → point clouds → top-down 2D occupancy map.
- **Cost Map**: FMM geodesic distance to goal + inverse SDF obstacle term.
- **Planning / Control**: PD heading control, line-search forward speed, clipped by Safety Advisor.
- **Locomotion**: Velocity-conditioned RL policy with RMA extrinsics adaptation.
- **Safety Advisor**: Collision detector updates map; fall predictor sets speed limits.

## 7. Datasets & Experimental Setup
- **Sim**: Gibson & Matterport3D scenes in Habitat → RaiSim. Variants: rough terrain, invisible obstacles, slippery patches.
- **Hardware**: Unitree A1 with RealSense D435 & T265; fully onboard compute.

## 8. Evaluation Metrics & Results
- Metrics: Success rate, SPL, time-to-goal, energy.
- Simulation: Collision detection boosts success up to +15 pts with many invisible obstacles. Fall predictor adds ~+7 pts success on rough/slippery terrain. Continuous actions ~27% faster than discrete. VP-Nav ~95% success vs wheeled baselines at ~15–69%.
- Real world: 100% avoidance of glass, robust speed adaptation on slippery floors.

## 9. Limitations
- Locomotion not vision-conditioned; cannot climb or jump.
- No explicit semantic or friction/roughness mapping.
- Mapping is 2D only.

## 10. Similarity / Relevance to Our Work
Overlap:
- Both seek terrain-aware navigation beyond pure geometry.
- Uses proprioception for traversability, akin to our goal.

Differences:
- Our plan includes multi-layer semantics & physical properties mapping.
- VP-Nav uses only 2D occupancy and implicit physical cues.

Informative aspects:
- Shows effective coupling of proprioception and planning.
- Demonstrates continuous command space and onboard operation.

## 11. Potential Integration Points
- Proprioceptive collision detector to mark unseen obstacles.
- Fall-risk predictor to cap speed using our friction/roughness layer.
- Continuous-space planner with FMM+SDF cost shaping.
- RMA-style extrinsics as a physical-properties layer.

## 12. Open Questions / Future Work
- Vision-conditioned locomotion for climbing.
- Mapping of physical properties.
- Outdoor urban and mixed-mode navigation.
- Learning semantics-to-traversability priors.

## 13. Reproducibility Notes
- Mapping code based on librealsense; datasets specified.
- No full code release, but methodology detailed for re-implementation.

## 14. Link
- Project/videos: https://navigation-locomotion.github.io

## 15. Recap
- Train velocity-conditioned locomotion and adaptation in sim.
- Train proprioceptive safety heads for collision & fall prediction.
- Onboard: build 2D occupancy map from depth.
- Compute continuous commands from cost map gradient and line-search.
- Couple with proprioception: inject obstacles and cap speed.
- Execute on Unitree A1; evaluate vs baselines in sim & real.
