# Review: *Learning robust perceptive locomotion for quadrupedal robots in the wild*

## 1. Citation Information
```bibtex
@article{miki2022learning,
  title   = {Learning robust perceptive locomotion for quadrupedal robots in the wild},
  author  = {Miki, Takahiro and Lee, Joonho and Hwangbo, Jemin and Wellhausen, Lorenz and Koltun, Vladlen and Hutter, Marco},
  journal = {arXiv preprint arXiv:2201.08117},
  year    = {2022},
  note    = {Compiled Jan 21, 2022}
}
```

## 2. Clear Abstract 
The paper tackles a long-standing challenge in legged robotics: using vision robustly for fast, reliable locomotion in the wild, where depth sensing often fails on snow, vegetation, water, reflective surfaces, or under poor lighting. The authors propose a learned locomotion controller that fuses exteroceptive (terrain) and proprioceptive (body/foot) inputs through an attention-gated recurrent belief encoder trained end-to-end. Training follows a privileged-learning scheme: a teacher policy learns with perfect terrain knowledge in simulation; a student policy then imitates the teacher while seeing only noisy, incomplete elevation data plus proprioception, learning to trust vision when useful and fall back to proprioception when not. Deployed zero-shot on ANYmal, the controller achieved rapid, smooth traversal across alpine, forest, underground, and urban terrains without falls, including an hour-long mountain hike at human-comparable time. Controlled tests show higher success over steps and obstacles, and higher maximum speeds and turn rates than a proprioceptive baseline. Overall, the method demonstrates that learned multi-modal fusion can deliver both speed and robustness for quadrupeds operating in challenging real-world environments.

## 3. Keywords
- Quadrupedal locomotion  
- Exteroceptive–proprioceptive fusion  
- Privileged learning / teacher–student  
- Recurrent belief encoder (GRU, gated attention)  
- Elevation mapping (robot-centric 2.5D)  
- Sim-to-real transfer  
- Rough terrain traversal  
- Robust control under perception failure

## 4. Problem Addressed
How to robustly leverage unreliable, incomplete exteroceptive terrain perception to increase speed and stability of quadrupedal locomotion in diverse, real-world environments—while retaining the safety of proprioceptive control when vision fails.

## 5. Innovation / Main Contribution
- An **attention-gated recurrent belief encoder** that integrates noisy elevation samples with proprioception, learning when and how much to trust vision.  
- A **privileged learning** pipeline (teacher with perfect terrain, student with noisy observations) that yields a **zero-shot, real-world policy** robust to perception failures.  
- Extensive hardware validation demonstrating **speed**, **obstacle capability**, and **zero falls** across seasons and terrains.

## 6. Key Methods / Architecture
- **Teacher policy (PPO)** trained in simulation with privileged ground-truth terrain, friction, contacts.  
- **Student policy** with a **GRU-based belief encoder** + **attentional gate** that fuses proprioception and sampled elevation heights; trained via **behavior cloning + reconstruction** losses to mimic teacher and reconstruct hidden factors.  
- **Elevation mapping** supplies terrain as height samples; the belief encoder learns to down-weight untrustworthy exteroception.  
- **Domain randomization + noise modeling** (offsets, outliers, occlusions) to simulate mapping failures and pose drift.  
- **Zero-shot deployment** on ANYmal: policy 50 Hz; elevation map 20 Hz.

## 7. Datasets & Experimental Setup
- **Platform:** ANYmal C quadruped.  
- **Sensors:** either two **Robosense Bpearl LiDARs** or four **Intel RealSense D435** cameras; robot-centric 2.5D elevation map built from point clouds.  
- **Environments:** alpine/forest/urban/underground; seasonal conditions (snow, ice, mud, reflective surfaces).  
- **Simulation:** RaiSim with randomized terrains (slopes, steps, stairs), disturbances, friction.  
- **Frequencies:** elevation mapping 20 Hz; control policy 50 Hz.

## 8. Evaluation Metrics & Results
- **Obstacle course** (inclined/raised platforms, stairs, block pile): proposed controller completed smoothly unaided; proprioceptive baseline required human help.  
- **Max speed** on flat: **1.2 m/s** vs **0.6 m/s** (forward & lateral). Over a 20 cm step, ours maintained speed; baseline stalled. **Turn rate**: **3 rad/s** vs **0.6 rad/s**.  
- **Step-up capability:** reliable up to ~**30.5 cm** (baseline failed earlier).  
- **Field robustness:** **zero falls** across varied natural/urban sites and sensing degradations.  
- **Mountain hike:** 2.2 km, 120 m gain, up to 38% grade; finished in **78 min** (human guide ~76 min), summit reached in **31 min** (< signage 35 min).

## 9. Limitations (esp. vs. our BEV-semantic, elevation + MPC + CBF project)
- **No semantic terrain understanding:** uses only **2.5D elevation**; cannot encode human comfort/social preferences (e.g., prefer sidewalk over grass) central to our work.  
- **Implicit, not explicit, uncertainty:** the belief state handles unreliability implicitly; no calibrated uncertainty for downstream planners/CBFs.  
- **Local, reactive control:** focuses on gait/footing; **no high-level human-preference cost map** or long-horizon routing.  
- **Mapping representation limits:** 2.5D elevation fails for overhangs/transparent/soft terrain; semantics could help disambiguate.  
- **No MPC/CBF guarantees:** safety is empirical, not constraint-enforced as in our planned MPC + CBF stack.

## 10. Similarity / Relevance to Our Work
- **Shared goal:** robust, terrain-aware locomotion under perception failures.  
- **Common inputs:** elevation mapping; exposure to snow/grass/stairs aligns with our mixed-terrain focus.  
- **Key difference:** they optimize **motor policy** for agility/robustness; we optimize **route selection and control** for **human-preference** costs using BEV semantics and MPC + CBF.

## 11. Potential Integration Points
- **Reliability gating for semantics:** borrow the **attention-gated belief encoder** to produce a *per-cell reliability weight* for BEV semantic/elevation layers; feed this into our cost map and CBF safety sets.  
- **Noise curricula:** adopt their **exteroception noise model** (offsets, occlusions, outliers) to **stress-test BEV mapping** and train preference-learning to be robust to mapping artifacts.  
- **Friction/terrain property inference:** extend the belief decoder idea to estimate **friction/softness** and modulate **human-preference weights** (e.g., avoid wet grass/snow more strongly).  
- **Hierarchical stack:** keep their **low-level locomotion controller** (or ideas from it) beneath our **MPC + CBF** planner for safer, human-preferred trajectories with agile footstep adaptation.

## 12. Open Questions / Future Work (from authors)
- **Explicit uncertainty modeling** in the belief state to avoid risky steps at cliffs/stepping-stones.  
- **Ingest raw sensory data** (not just elevation maps) and **jointly train pose estimation**.  
- **Learn occlusion models** to reason about unseen terrain.  
- **Handle non-walking maneuvers** (e.g., extrication from holes, high ledge climbs).

## 13. Reproducibility Notes
- **Public artifact status:** paper claims all data to evaluate conclusions are in the paper/Supplement; cites **RSLGym** and provides detailed hyperparameters in Supplementary Tables.  
- **Hardware dependence:** replicating full results requires **ANYmal C** (or similar) and either **Bpearl LiDAR** or **RealSense D435**.  
- **Implementation details:** mapping runs at **20 Hz** on GPU; policy at **50 Hz**—important real-time constraints for reproduction.

## 14. Link
- Paper (arXiv): https://arxiv.org/abs/2201.08117
