# Review Report — *Multimodal dynamics modeling for off-road autonomous vehicles*

## 1. Citation Information

```bibtex
@article{tremblay2021multimodal,
  title   = {Multimodal dynamics modeling for off-road autonomous vehicles},
  author  = {Tremblay, Jean-Fran{\c{c}}ois and Manderson, Travis and Noca, Aur{'e}lio and Dudek, Gregory and Meger, David},
  journal = {arXiv preprint arXiv:2011.11751},
  year    = {2021},
  archivePrefix = {arXiv},
  eprint  = {2011.11751},
  primaryClass = {cs.RO},
  url     = {https://arxiv.org/abs/2011.11751}
}
```

## 2. Clear Abstract (150–200 words)

Accurately predicting how an off-road robot will move is difficult because terrain, traction, and vegetation interact in complex ways and sensors can fail or be unreliable. This paper proposes a **Multimodal Recurrent State-Space Model (MRSSM)** that learns robot dynamics from **vision, LiDAR, and proprioception** and remains robust when some modalities are missing at test time. The method combines a recurrent state-space model (for long-horizon prediction) with a **multimodal variational autoencoder** fused via a **product-of-experts**. A new training objective extends the ELBO so the model learns to reconstruct and predict **held-out modalities**, improving performance under modality dropouts. In simulation with controlled friction changes, the model uses vision to anticipate traction shifts and reduces position error during terrain transitions. On a challenging real-world forest dataset (17 trajectories), the model predicts future pose over multi-second horizons and **outperforms baseline** dynamics predictors, including when only vision or only proprioception is available. Results highlight the value of leveraging multiple sensors for off-road dynamics modeling and suggest that such models could enable planning and control that account for terrain-induced dynamics without manual terrain labels.

## 3. Keywords

- Multimodal variational autoencoder (MVAE)  
- Recurrent state-space model (RSSM)  
- Product-of-experts fusion  
- Off-road robot dynamics  
- Long-horizon prediction  
- Missing-modality robustness  
- Visual odometry from learned latent states  
- Model-based RL / latent dynamics  
- Traction change prediction

## 4. Problem Addressed

Predict long-horizon robot motion in **unstructured, forested** environments where **terrain and traction** vary and **sensors intermittently fail**, using a single learned model that **fuses multiple modalities** and remains accurate when some are missing.

## 5. Innovation / Main Contribution

- **MRSSM**: A dynamics model that merges RSSM with MVAE for **multimodal, time-series** prediction.  
- **New ELBO**: Training objective that maximizes likelihood of **held-out modalities**, improving prediction when sensors drop out.  
- **Robustness**: Demonstrates accurate predictions **even with only vision or only proprioception** at test time.  
- **Off-road focus**: First application (claimed) of MVAEs to **challenging outdoor robotics** data for dynamics modeling.

## 6. Key Methods / Architecture

- **Latent time-series model (RSSM)** with hybrid deterministic–stochastic transitions for long-horizon rollouts.  
- **Per-modality encoders/decoders** (vision CNN, LiDAR CNN, MLPs for low-D proprioception).  
- **Product-of-experts** to fuse Gaussian posterior “experts” (vision, LiDAR, IMU/velocities) with a **prior expert** from the transition model.  
- **Training with random modality subsets** plus **importance-weighted ELBO** so the prior learns from the **full-modality posterior** while optimizing reconstruction of **all** modalities, including those held out.  
- **0.1 s timestep**; angular velocity encoded as **quaternion** over the last 0.1 s; images/heightmaps at **64×64**.  
- **Prediction metric**: integrate predicted linear/angular velocities to a final pose over a horizon; evaluate Euclidean translation error.

## 7. Datasets & Experimental Setup

- **Simulation (Unreal Engine)**: 4.38 hours, 684 trajectories with 3 terrain types (different friction), obstacles (trees/rocks), average speed ~5 m/s; modalities: camera + proprioception.  
- **Real-world (Montmorency forest)**: 17 trajectories across 4 sites; **Maple grove** subset used for validation (8 trajectories, **126 minutes**). Sensors: front RGB camera, **LiDAR height map** (4 m² grid with mean z, z-std, intensity), linear & angular velocities, **tri-axial accelerometer**; platform is skid-steered, **max 1 m/s**.  
- **Baselines**: (i) **Control**—assume robot achieves commanded velocity; (ii) RSSM with **concatenated encoders** (no missing-modality support).  
- **Ablations**: All modalities vs **vision-only** vs **proprioception-only**; compare **new ELBO** vs prior MVAE ELBO.

## 8. Evaluation Metrics & Results

- **Metric**: Final pose translation error after integrating predicted velocities over **1–3 s** horizons.  
- **Simulation**: With terrain transitions, **all-modalities** yields ~**40% lower** translation error vs proprioception-only; benefits persist even without transitions.  
- **Real-world (3 s horizon)**: MRSSM with the **new ELBO** achieves **median error ≈ 11.3 cm** vs **23.6 cm** for Control; improves tails of the error distribution vs baselines.  
- **Missing modalities**: With **vision-only**, MRSSM performs implicit **deep visual odometry** and still **outperforms** both baselines; new ELBO degrades less than prior MVAE ELBO under ablation.

## 9. Limitations (esp. vs. our human-preference, terrain-aware navigation w/ BEV semantics, elevation, MPC+CBF)

- **No explicit BEV semantic map**: Terrain categories are **implicit** in the latent state; no top-down semantic layer aligning with our BEV mapping.  
- **Elevation fusion** is simplified to a **local LiDAR height map**, not a full 3-D elevation grid with slope/curb reasoning like our planned module.  
- **No planning/control**: The work stops at prediction; **MPC/CBF** safety constraints aren’t integrated or certified.  
- **Human preference** not modeled: The cost of “sidewalk vs grass vs snow” is not encoded; dynamics don’t translate to **social/comfort preferences**.  
- **Generalization**: Real-world validation is site-limited (one emphasis site for stats); seasonal/lighting robustness is qualitative.  
- **Reproducibility**: Code modifications are substantial; dataset for dynamics (as used here) may not be readily downloadable.

## 10. Similarity / Relevance to Our Work

- **High relevance** for the **dynamics layer**: Their latent model predicts slip/traction effects from vision+LiDAR—useful to **inform cost and constraints** in our planner.  
- **Sensor robustness** aligns with our outdoor use cases; **vision-only fallback** is valuable for low-cost platforms.  
- **Divergence**: They don’t produce **semantic BEV** or **human-preference** costs; our objective includes social/comfort norms and elevation-aware constraints.

## 11. Potential Integration Points

- **Slip/traction predictor → cost shaping**: Use MRSSM’s predicted velocity/pose error (or uncertainty) to **penalize likely-slip regions** in our BEV cost map.  
- **Missing-modality robustness**: Adopt **product-of-experts** fusion for our perception stack so BEV/elevation pipelines degrade gracefully when **camera or LiDAR** drops out.  
- **Latent visual odometry**: When depth is unreliable (snow, glare), use MRSSM latent dynamics for **short-horizon VO** to stabilize MPC state estimates.  
- **Uncertainty-aware CBFs**: Feed MRSSM predictive variance to **tighten CBF constraints** in low-traction zones.  
- **Self-supervised terrain signals**: Use their held-out-modality ELBO trick to learn **terrain-relevant latents** without manual labels, then **distill** to our BEV semantics as priors.

## 12. Open Questions / Future Work (from the paper & for us)

- How well does the model **generalize** across new forests, seasons, and lighting without finetuning?  
- Can the latent space be made **interpretable** (e.g., factors for slope, softness) to plug into **cost maps** directly?  
- How to **close the loop**: integrate MRSSM with **planning (MPC)** and **safety (CBF)** for end-to-end off-road autonomy?  
- What is the trade-off between **prediction horizon** and control frequency in rough terrain?  
- How to quantify **uncertainty** rigorously for safety-critical constraints?

## 13. Reproducibility Notes

- **Code**: Based on PlaNet and MVAE implementations but “substantial changes”; no explicit public repo is cited.  
- **Data**: Simulation is custom; real-world set references Montmorency forest runs with specific sensors; availability unclear in this paper.  
- **Architectural specifics** (input resolutions, timestep, modality definitions) are given; training hyperparameters are partially described but full configs aren’t exhaustively listed. Expect moderate replication effort.

## 14. Link

- Paper: https://arxiv.org/abs/2011.11751
