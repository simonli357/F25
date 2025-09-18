# Legged Locomotion in Challenging Terrains using Egocentric Vision — Structured Report

## 1. Citation Information
```bibtex
@inproceedings{agarwal2022egocentric,
  title     = {Legged Locomotion in Challenging Terrains using Egocentric Vision},
  author    = {Agarwal, Ananye and Kumar, Ashish and Malik, Jitendra and Pathak, Deepak},
  booktitle = {6th Conference on Robot Learning (CoRL)},
  address   = {Auckland, New Zealand},
  year      = {2022},
  url       = {https://vision-locomotion.github.io},
  eprint    = {2211.07638},
  archivePrefix = {arXiv},
  primaryClass  = {cs.RO}
}
```

## 2. Clear Abstract 
This paper presents an end-to-end visual locomotion system for quadruped robots that uses only a single front-facing depth camera plus proprioception. Instead of constructing metric elevation maps or relying on pre-programmed gaits, the authors train a recurrent control policy that directly outputs joint targets from egocentric depth, enabling robust traversal of stairs, curbs, stepping stones, gaps, and uneven outdoor terrain. Training proceeds in two stages. First, a reinforcement-learning “teacher” policy learns from inexpensive scandot height probes that summarize local terrain geometry. Second, a student policy is distilled via supervised learning to operate purely from onboard sensing (depth + proprioception), using a convolutional encoder and GRU memory to retain information about terrain that passes out of view. Experiments in simulation and on a Unitree A1 robot show large gains over “blind” policies and approaches that depend on noisy elevation maps. The method achieves near-perfect success on stairs, curbs, and gaps, and high success on stepping stones, while running in real time on modest onboard compute. The work demonstrates that tightly coupling egocentric vision with memory and control can produce agile, versatile, and hardware-efficient legged locomotion in challenging terrains.

## 3. Keywords
- Legged locomotion
- Egocentric depth vision
- Reinforcement learning (PPO)
- Policy distillation / DAgger
- Recurrent neural networks (GRU)
- Rapid Motor Adaptation (RMA) variant
- Sim-to-real transfer / domain randomization
- Unitree A1
- IsaacGym / legged-gym
- Stepping stones / gaps / stairs

## 4. Problem Addressed
Enable robust, agile quadruped locomotion over challenging, discontinuous, or cluttered terrain **without** relying on elevation maps, precise localization, or pre-programmed gaits—using only egocentric depth and proprioception.

## 5. Innovation / Main Contribution
- **End-to-end egocentric depth→joint policy** that forgoes elevation maps and foothold optimization yet handles stairs, gaps, and stepping stones.
- **Two-phase training**: RL with cheap scandots → supervised distillation to a depth-based student (with theory showing bounded suboptimality under assumptions).
- **Recurrent memory** to remember terrain that moved out of view (e.g., under hind feet).
- **Hardware efficiency**: real-time control on a small, low-compute Unitree A1; emergent specialized gaits (e.g., hip abduction) without gait priors.

## 6. Key Methods / Architecture
- **Sensing:** single forward depth camera (Intel RealSense) + proprioception; 50 Hz control.
- **Phase 1 (Teacher):** PPO policy trained in IsaacGym using scandots (sampled terrain heights) + proprioception; both monolithic GRU and RMA-style variants explored.
- **Phase 2 (Student):** depth processed by CNN → latent; GRU fuses latent + proprioception to predict joint targets; trained with DAgger to imitate teacher.
- **RMA variant:** learn separate latents for terrain geometry (\hat{\gamma}) and extrinsics (\hat{z}) feeding a feedforward base policy.
- **Robustness:** heavy domain randomization (latency, friction, pushes, mass/COM, controller gains) and observation noise; no gait priors.

## 7. Datasets & Experimental Setup
- **Simulator:** IsaacGym + legged-gym; 100 sub-terrains (slopes, stairs, discrete obstacles, stepping stones, gaps, fractal roughness) with curriculum over difficulty.
- **Robot:** Unitree A1 (12 DOF) with front RealSense depth camera; Jetson NX + UP board; PD low-level loop at 400 Hz.
- **Real-world courses:** indoor/outdoor stairs, curbs (up to ~26 cm), gaps (≤ 26 cm), stepping stones (≈ 30 cm wide, 15 cm spacing), ramps, rocky trails, beaches.

## 8. Evaluation Metrics & Results
- **Simulation metrics:** average forward displacement and mean time-to-fall across terrains. The monolithic and RMA students **outperform blind and noisy-elevation baselines by ~60–90%** in aggregate distance and stability, with especially large gains on stepping stones (≈ 20 m vs. ≈ 0–1 m).
- **Real-world success:**
  - **Upstairs:** 100% (blind: fails to complete)
  - **Downstairs:** 100% (blind: 100% but with damaging high-impact gait)
  - **Gaps (≤ 26 cm):** 100% (blind: 0%)
  - **Stepping stones:** **94%** (blind: 0%)
- **Capability highlights:** stairs up to ~24 cm height; curbs/obstacles up to ~26 cm; robust to pushes and slips; emergent hip abduction for small-robot stair ascent.

## 9. Limitations
- **Sim-to-real brittleness** when real scenes differ from randomized training; failures require engineering the new case into simulation and retraining.
- **No explicit terrain semantics** or human-preference modeling—policy optimizes locomotion feasibility/energy, not social comfort norms.
- **Single-camera FOV** can miss side/rear geometry; memory mitigates but not eliminates this.

## 10. Similarity / Relevance to Our Work (Human-Preference Terrain-Aware Navigation)
- **Overlap:** Both approaches leverage **camera perception** and aim for safe, natural traversal in semi-structured outdoor environments (curbs, sidewalks, rough terrain). The paper’s **egocentric memory** complements our plan to track terrain desirability over short horizons.
- **Divergence:** Their objective is **pure locomotion feasibility** (end-to-end control), whereas our goal is **human-preference navigation** (semantic/elevation fusion + preference costs + MPC/CBF). They avoid maps/semantics; we explicitly build BEV semantics and preference-weighted cost maps.
- **Takeaway for us:** Their depth-GRU pipeline is a strong **low-level locomotion layer** we can place under our higher-level preference-aware planner to ensure feasible foot/whole-body movement where we ask the robot to go.

## 11. Potential Integration Points
- **Hybrid stack:** Use their **Phase-2 student controller** (depth→joint targets) as the **controller backend** under our MPC/CBF that tracks preference-optimal paths; CBF handles safety envelopes while the learned controller guarantees foot-level agility on curbs/steps.
- **Geometry latent → planner:** Tap their RMA **terrain-geometry latent (\hat{\gamma})** as a lightweight traversability signal to **augment our BEV cost map** where semantics are uncertain (e.g., snow/grass edges).
- **Training recipe:** Adopt their two-phase **scandot→depth distillation** and domain-randomization (including **latency/pushes/friction**) to harden our **preference-cost estimators** and **CBF tracking** against timing jitter and sensing holes.
- **Failure recovery:** Borrow their **recurrent memory** design to help our planner maintain short-term awareness of curbs/steps that pass out of view between BEV updates.

## 12. Open Questions / Future Work (from the paper)
- How to **reduce sim-to-real gaps** without manually re-authoring new failure cases?
- Can real-world data be **continuously incorporated** for online improvement of both vision and control?
- How well would the approach generalize to **different platforms/sensors** (e.g., RGB-only, stereo, fisheye, multi-camera)?
- Can the latent representations (\hat{\gamma}, \hat{z}) be **interpreted or exposed** to upstream planners for better task-level coordination?

## 13. Reproducibility Notes
- **Code/data:** The project page provides videos and detailed implementation notes; explicit code release is not stated in the paper text.
- **Implementation specifics provided:** hardware (Unitree A1, Jetson NX + UP board), control rates, depth preprocessing, reward shaping, curriculum terrains, domain-randomization ranges, and pseudo-code for phase-2 training—sufficient to **reimplement** with IsaacGym/legged-gym.

## 14. Link
- Project page & videos: https://vision-locomotion.github.io
