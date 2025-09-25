# EVORA — Deep Evidential Traversability Learning for Risk-Aware Off-Road Autonomy

## 1) Citation Information
```bibtex
@article{Cai2024EVORA,
  title   = {EVORA: Deep Evidential Traversability Learning for Risk-Aware Off-Road Autonomy},
  author  = {Xiaoyi Cai and Siddharth Ancha and Lakshay Sharma and Philip R. Osteen and Bernadette Bucher and Stephen Phillips and Jiuguang Wang and Michael Everett and Nicholas Roy and Jonathan P. How},
  journal = {arXiv preprint arXiv:2311.06234},
  year    = {2024},
  url     = {https://xiaoyi-cai.github.io/evora},
  note    = {v2, March 31, 2024}
}
```

## 2) Clear Abstract
The paper presents **EVORA**, a full pipeline that (1) **learns terrain traction** from data while explicitly modeling **two kinds of uncertainty**—natural variability (aleatoric) and “not seen before” (epistemic)—and (2) **plans** robot motions that hedge against these risks. It uses an evidential neural network to predict a **distribution over traction** (how much commanded velocity turns into actual motion) and a **density over latent features** to detect out-of-distribution terrain. A new **uncertainty-aware Earth Mover’s Distance** loss improves learning. For planning, EVORA simulates trajectories using **worst-case expected traction (left-tail CVaR)** and penalizes routes through terrain with **high epistemic uncertainty**. In simulation and on wheeled and legged robots, EVORA yields faster, more reliable navigation than methods assuming no slip, using expected traction only, or optimizing worst-case cost directly.

## 3) Keywords
- Off-road autonomy
- Traversability / traction modeling
- Aleatoric & epistemic uncertainty
- Evidential deep learning (Dirichlet over categorical)
- Normalizing flows / latent density
- Earth Mover’s Distance (EMD^2) loss
- CVaR (Conditional Value at Risk) planning
- Out-of-distribution (OOD) detection
- Risk-aware model predictive control (MPPI)
- Semantic elevation mapping / octomap

## 4) Problem Addressed
Robots must move quickly off-road while avoiding **non-geometric hazards** (mud, slippery vegetation) that aren’t well handled by geometry-only maps. Learning-based traversability helps, but **uncertainty** in learned models is under-addressed: prior work typically models either aleatoric or epistemic uncertainty—not both—and often doesn’t propagate uncertainty into planning. EVORA tackles **uncertainty-aware traversability learning** and **risk-aware motion planning** jointly.

## 5) Innovation / Main Contribution
- **Dual-uncertainty traversability model:** Predicts **discrete traction distributions** (aleatoric) and **latent feature densities** via a **normalizing flow** (epistemic).
- **New closed-form, uncertainty-aware EMD^2 loss** for ordered traction bins, improving prediction, OOD detection, and downstream navigation.
- **Risk-aware planning via left-tail CVaR of traction (CVaR-Dyn):** Simulates rollouts with worst-case expected traction; adds penalties to avoid high-epistemic-uncertainty terrain. Cheaper and competitive vs CVaR-of-cost (CVaR-Cost).

## 6) Key Methods / Architecture
- **Inputs & features:** local **elevation patches** + **semantic labels** (from a semantic octomap).
- **Evidential predictor:** NN outputs parameterize a **Dirichlet** over a **categorical traction distribution** (linear & angular traction bins), capturing multimodal traction.
- **Epistemic via latent density:** **Normalizing flow** models density of encoder features; low density ⇒ OOD; define a **confidence score** and threshold for avoidance.
- **Training loss:** Combined **UCE** + **uncertainty-aware EMD^2** + entropy regularization; closed-form expressions provided.
- **Planner:** Sampling-based MPC (MPPI). Two risk models:
  - **CVaR-Dyn:** rollouts use **left-tail CVaR** of traction at each step.
  - **CVaR-Cost:** minimizes **right-tail CVaR** of total cost via Monte Carlo over traction maps.
  - Add **auxiliary costs** to avoid OOD terrain.

## 7) Datasets & Experimental Setup
- **Synthetic 3D terrains** to benchmark losses: controlled elevation/slope/semantics (dirt/vegetation), limited training coverage (samples along a path). Evaluated EMD^2, KL, AUC-ROC/PR.
- **Perception stack:** **Semantic octomap** built by fusing LiDAR with image segmentation (PointRend trained on **RUGD** 24 classes).
- **Robots / platforms:**
  - **RC car** indoor race track (fake vegetation). Planners at **20 Hz**, **5 s** horizon, **1024 rollouts**; CVaR-Cost uses **400** traction map samples; V_max=**1.5 m/s**, max steering **30°**.
  - **Outdoor legged robot** experiments demonstrating aleatoric & epistemic handling.

## 8) Evaluation Metrics & Results
- **Learning metrics:** **EMD^2** and **KL** for traction distributions; **AUC-ROC/PR** for OOD detection. The **hybrid UCE + UEMD^2** objective yields **best prediction accuracy** and **more stable OOD performance** as data increases.
- **Planner performance (simulation):** CVaR-Dyn shows **equal or better success rate and time-to-goal** than CVaR-Cost at moderate risk levels; both outperform “nominal” and “expected-traction” baselines.
- **Indoor RC car:** Over multiple races, **CVaR-Dyn (α=0.8)** achieved the **best time-to-goal with 100% success**, while CVaR-Cost grew over-conservative near obstacles; “expected-traction” (WayFAST) and nominal-traction baselines under-steered and collided more.
- **OOD avoidance (outdoor-style sim):** Thresholding latent density (confidence) and **avoiding OOD** increased **success rate by up to ~30%**; **soft penalties** for OOD terrain improved time-to-goal with similar success.

## 9) Limitations
- Uses **2D robot models**; 6-DoF modeling needed for rougher terrain.
- Relies on **semantic segmentation**; large domain shifts (lighting/seasonal) can degrade performance—perception uncertainty not explicitly handled.
- Requires **empirical traction distributions** during training; scaling to high-dimensional raw RGB supervision needs reassessment of EMD^2 benefits.
- CVaR-Dyn’s worst-case-traction rollouts can become **too conservative** (local minima) at very low risk tolerance.

## 10) Similarity / Relevance to Our Work
**Overlap**
- EVORA’s **semantic+elevation** map inputs and **risk-aware planner** align with our intent to combine semantics, geometry, and traversability.
- Their **traction (friction-like) modeling** complements our desire to embed terrain physical properties.

**Divergence**
- EVORA focuses on **traction distributions** learned from proprioception-aligned signals and **does not explicitly fuse global geometric costs** like slope/roughness layers or long-horizon global planning.
- Semantics are used as **features** into the traction predictor, not as an explicit policy-level prior.

**How it informs us**
- Strong recipe for **uncertainty calibration** (evidential + latent density) and **CVaR-based planning** that we can port into our multi-layer map.
- Demonstrates **practical improvements** showing that **risk modeling pays off in real robots**.

**Our limitations vs EVORA**
- If we currently don’t **quantify** both uncertainties or propagate them into planning, our system may **over-commit** in ambiguous terrain.

## 11) Potential Integration Points
- **Evidential traction head:** Add a **Dirichlet-parameterized categorical** head to our vision/LiDAR fusion to output **per-cell friction/traction distributions**.
- **Latent density OOD gate:** Train a **normalizing flow** on encoder features; define a **confidence threshold** to **mask or up-cost** OOD cells in our costmap.
- **Loss function:** Replace/augment CE with the **closed-form uncertainty-aware EMD^2** for ordered traction bins.
- **Planner:** Use **CVaR-Dyn** for local MPC rollouts, plus **aux penalties** for OOD.
- **Multi-layer fusion:** Treat EVORA’s traction CVaR as an **additional layer** in our unified cost.

## 12) Open Questions / Future Work
- Extend beyond **2D models** to full 6-DoF on extreme terrain.
- Reduce dependence on **semantic segmentation** and explicitly account for **perception uncertainty**.
- Revisit **data collection** burden for traction distributions; explore **uncertainty-guided sampling**.
- Generalize CVaR-of-traction idea to **other parameters/tasks**; integrate **online adaptation** and **global planning**.

## 13) Reproducibility Notes
- Paper includes **closed-form losses** and **planner specs** (rollouts, rates, horizons).
- **Project page** is provided; explicit public code release isn’t stated in the PDF.

## 14) Link
- **Project website:** https://xiaoyi-cai.github.io/evora
- **arXiv:** arXiv:2311.06234 (v2, 31 Mar 2024)

## 15) Recap
- **Collect data** by driving robots and logging **achieved vs commanded velocities** to estimate **traction**, while building **semantic elevation maps**.
- **Form training labels** as **empirical traction histograms** per terrain cell (captures multi-modality).
- **Train an evidential NN** that outputs a **Dirichlet** over traction bins (aleatoric) and a **normalizing-flow density** over latent features (epistemic).
- **Optimize** with a **combined uncertainty-aware EMD^2** + **UCE** loss (closed-form), improving prediction & OOD calibration.
- **Detect OOD** at test time via **latent density threshold** and **penalize/avoid** those cells in planning.
- **Plan with risk:** Use MPPI; **simulate rollouts under left-tail CVaR traction (CVaR-Dyn)** and compare to **CVaR-of-cost (CVaR-Cost)** and baselines.
- **Validate** in synthetic benchmarks, simulation planners, **indoor RC car** races, and **outdoor legged robot** runs. Show **faster times**, **higher success**, and **+~30% success** when avoiding OOD via confidence.

