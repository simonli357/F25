# Identifying Terrain Physical Parameters from Vision — Report

## 1. Citation Information

```bibtex
@article{Chen2024TerrainParamsFromVision,
  title   = {Identifying Terrain Physical Parameters from Vision: Towards Physical-Parameter-Aware Locomotion and Navigation},
  author  = {Jiaqi Chen and Jonas Frey and Ruyi Zhou and Takahiro Miki and Georg Martius and Marco Hutter},
  journal = {IEEE Robotics and Automation Letters},
  note    = {Accepted August 2024; preprint version. arXiv:2408.16567},
  year    = {2024},
  url     = {https://arxiv.org/abs/2408.16567},
  howpublished = {\url{https://bit.ly/3Xo5AA8} (project page)}
}
```

## 2. Clear Abstract (rewritten)

The paper introduces a two-stage, self-supervised framework that teaches robots to **see** how the ground **feels**. First, a **physical decoder** is trained in simulation to infer per-foot **friction** and **stiffness** from robot proprioception plus local terrain geometry. Then, in the real world, those per-foot estimates are **reprojected into camera images** to label pixels and train a **visual network** that predicts dense maps of friction and stiffness directly from images. An **anomaly detector** masks out unreliable pixels. On ANYmal robots, the decoder outperforms a strong baseline; the visual model produces accurate, confidence-masked friction/stiffness maps and adapts online to new scenes.

## 3. Keywords

- Terrain friction estimation
- Terrain stiffness estimation
- Self-supervised cross-modal learning
- DINOv2 visual features
- Legged locomotion (ANYmal)
- Digital twin validation
- Online adaptation / mission graph
- Anomaly / OOD detection (GMM)
- Reprojection labeling
- Physical-parameter-aware navigation

## 4. Problem Addressed

Robots often fail on **non-geometric hazards**—e.g., slippery or soft ground—because these physical properties are hard to sense **before** contact and hard to simulate exactly. The work aims to infer **simulation-aligned physical parameters** (friction, stiffness) from **vision**, enabling policies trained in sim to transfer better to the real world.

## 5. Innovation / Main Contribution

- A **two-stage cross-modal pipeline**: (1) train a **twin** physical decoder in simulation to predict per-foot friction/stiffness; (2) use those labels to **self-supervise** a visual model that outputs **dense** friction/stiffness with a **confidence mask**.
- **Online learning** via a **Mission Graph**, allowing continuous adaptation during deployment.
- **Anomaly detection** using a **GMM** over reconstruction errors, yielding **hyperparameter-free**, per-image thresholds.
- A **digital-twin** study linking predicted parameters in reality to similar motion in sim, supporting the use of sim-parameters as meaningful labels.

## 6. Key Methods / Architecture

- **Physical decoder (interaction pipeline)**
  - Inputs: proprioception (commands, joint/body, leg phase) + local height samples around each foot.
  - Architecture: **twin GRU** stacks (friction/stiffness) + **self-attention** + **MLP heads** with a **gating** mechanism combining proprio-only and fused paths.
  - Outputs: **per-foot** friction and stiffness each timestep.

- **Visual network (vision pipeline)**
  - Features: **DINOv2** pixel embeddings.
  - Head: encoder–decoder MLP producing **reconstructed features** (for OOD) + **per-pixel** friction & stiffness.
  - Losses: **L_final = 0.9·L_recon + 0.1·L_pred** (MSE).
  - **Reprojection labeling**: project foot contacts with predicted parameters into past/future frames to create sparse supervision masks.
  - **Anomaly/OOD**: fit a **2-component GMM** to reconstruction-error histograms per image to threshold confidence.
  - **Mission Graph**: stores feature maps, camera poses, and supervision masks; drives **continuous online training** during operation.

## 7. Datasets & Experimental Setup

- **Simulation**: Legged Gym with procedurally generated terrains and soft-terrain contact model; friction ∈ [0,1], stiffness ∈ [1,10]. **18.3 h** sim data for training, **5 h** for testing decoder. ANYmal with robust locomotion policy.
- **Real world**:
  - **Friction**: high-friction ground vs **water-covered whiteboard** (slippery).
  - **Stiffness**: rigid ground vs **foam board**.
  - **Vision training**: indoor set with **22 images**; outdoor set with **82 images** for anomaly-detection analysis.
  - Hardware: **ANYmal D**, RTX 4080, i7-12700H.

## 8. Evaluation Metrics & Results

- **Physical decoder (sim)**
  - **Friction MAE**: **0.15** (vs baseline 0.21).
  - **Stiffness MAE**: **0.46** over range [1,10].
  - Quick **per-foot** transitions; ~0.4 s delay when entering new patches (contact needed).

- **Physical decoder (real)**
  - On slippery **whiteboard** vs ground, decoder distinguishes feet on low- vs high-friction areas; baseline predicts same across feet.
  - **Quantitative LH foot error**: WB **0.02±0.08** vs baseline **0.33±0.11**; Ground **0.00±0.00** for both.

- **Digital twin**
  - Lowest-error friction ranges: **WB ~ (0.0–0.2)**, **Ground ~ (0.3–1.0)**, supporting parameter interpretability.

- **Visual network (dense prediction)**
  - **Confidence-mask IoU** (floor vs rest): **0.82±0.05**.
  - Mean friction error: **WB 0.03±0.05**, **Ground 0.00±0.01**.
  - **GMM** anomaly detection converges faster and to higher accuracy than Gaussian-threshold baseline.

## 9. Limitations

- Needs **contact** to seed supervision; visual model then generalizes but relies on in-mission data.
- **High-friction disambiguation** (0.6–1.0) is intrinsically hard—motion is similar without slip.
- **Thin soft layers** (e.g., ~5 cm foam) can be misestimated; standing can show non-stationary friction estimates.
- Digital twin currently validated on **flat/rigid** terrain reconstructions; broader geometry needs future work.

## 10. Similarity / Relevance to Our Work

**Overlap with our unified mapping + navigation idea**
- Both seek **terrain-aware navigation** that goes beyond geometry to include **physical properties**.
- Their visual pipeline predicts **friction & stiffness maps**—directly relevant to our **physical-properties layer**.
- OOD/confidence masking aligns with our need for **safe planning** under uncertainty.

**Divergence**
- Their focus is **legged locomotion**, per-foot physical decoding, and **self-supervised labeling**; we target a **unified map** (geometry + semantics + properties) for **planning** applicable to **both wheeled and legged** platforms.
- They do not fuse **semantic classes** or build a persistent multi-layer map for global planning as we propose.

**How it informs us**
- Provides a **practical way** to obtain **dense physical-property priors** from **vision** using **in-mission self-supervision**.
- Their **confidence masking** is a strong idea to gate which map cells affect planning.
- The **twin decoder** design suggests we can combine proprio + geometry to refine property estimates where our visual model is unsure.

**Our current limitations vs this work**
- We currently plan to use “wvn / terrain params from vision”—this paper offers a **newer** DINOv2-based **dense** predictor with **online adaptation** and **validated sim-alignment** that might be more robust.

## 11. Potential Integration Points

- **Property layer**: In our multi-layer map, add **friction** and **stiffness** layers predicted by their **visual network**; update cells **online** via Mission Graph-style self-supervision.
- **Confidence-aware planning**: Use their **GMM OOD mask** to weight costs or require exploration/verification before committing.
- **Cross-modal bootstrapping**: When contact occurs (legged or wheel slip detection), run a **lightweight decoder** to refine local property labels and retrain the visual head.
- **Cost design**: Penalize **low-friction** and **low-stiffness** cells; combine with our **semantic** preferences (e.g., prefer sidewalk if friction ≥ threshold).
- **Simulator tie-in**: Train our planners/controllers in sim with **matching property distributions** to their predicted ranges for better sim2real.

## 12. Open Questions / Future Work (from authors + for us)

- How well do property predictions generalize to **complex 3D geometry** and **non-flat** contact?
- Can the digital twin be expanded beyond flat terrain for **quantitative** stiffness validation?
- What is the right way to fuse **semantics + properties** (e.g., prior friction/stiffness priors per class) without biasing against rare but safe cases?
- How to handle **sparse-contact missions** (e.g., cautious walking, wheeled robots) where labels arrive slowly?

## 13. Reproducibility Notes

- Provides substantial **implementation details** (architectures, losses, hyperparameters, data hours, training rates) and a **project page**.
- Code release is not explicitly stated in the paper text; project page likely hosts media/resources.
- Digital-twin reconstruction currently manual/simple; replicating those results may require careful setup.

## 14. Link

- **Project page**: https://bit.ly/3Xo5AA8
- **arXiv**: https://arxiv.org/abs/2408.16567

## 15. recap — What the paper actually does (bullet flow)

- **Train in sim** a **twin recurrent decoder** that reads robot **proprioception** + local **height samples** to output **per-foot friction & stiffness**.
- **Roll this decoder in the real world** so that whenever a foot contacts the ground, you get an estimated **(friction, stiffness)** for that **foothold**.
- **Project those labeled footholds into the camera image** (past/nearby frames) to create **sparse pixel labels** for where the foot touched.
- **Extract DINOv2 features** for every pixel; **train an encoder–decoder MLP** to predict **dense friction & stiffness maps**, while **reconstructing features** for OOD detection.
- **Fit a 2-component GMM** to reconstruction errors **per image** to create a **confidence mask** over pixels.
- **Maintain a Mission Graph** that stores features, labels, and poses; **keep training online** during the mission so the visual model **adapts** to the environment.
- **Evaluate**:
  - In sim, the decoder beats the baseline on friction MAE (**0.15 vs 0.21**) and yields stiffness MAE **0.46**.
  - In real tests (whiteboard vs ground; foam), it detects low friction/low stiffness per foot and outperforms baseline.
  - **Digital twin** analysis shows predicted friction values correspond to **sim ranges** that reproduce similar motion.
  - The visual model achieves **IoU ~0.82** for confidence masks and low friction errors on labeled regions.

---

### How this maps to our project

Use their **vision→friction/stiffness** layer (with **confidence mask**) as a first-class input to our **unified terrain map**, fuse with **geometry** (elevation_mapping_cupy) and **semantics** (PIDNet/Cityscapes/GOOSE), and let our **planner (A*+MPC)** trade off slope/obstacles with **physical risk** (low μ / soft ground). This gives us a concrete, modern recipe to **unify properties + semantics + geometry + traversability** in one framework.
