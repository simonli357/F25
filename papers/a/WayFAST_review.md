# Review: *WayFAST — Navigation With Predictive Traversability in the Field*

## 1. Citation Information

```bibtex
@article{Gasparino2022WayFAST,
  title   = {WayFAST: Navigation With Predictive Traversability in the Field},
  author  = {Gasparino, Mateus V. and Sivakumar, Arun N. and Liu, Yixiao and Velasquez, Andres E. B. and Higuti, Vitor A. H. and Rogers, John and Tran, Huy and Chowdhary, Girish},
  journal = {IEEE Robotics and Automation Letters},
  volume  = {7},
  number  = {4},
  pages   = {10651--10658},
  year    = {2022},
  doi     = {10.1109/LRA.2022.3193464}
}
```

## 2. Clear Abstract

The authors propose **WayFAST**, a modular, self-supervised system that learns where a wheeled robot can drive with good traction in unstructured outdoor environments. Instead of hand-crafted labels, WayFAST estimates traction online using a **nonlinear moving horizon estimator (NMHE)** combined with an **EKF**, and projects those estimates into the camera image to create training targets. A CNN called **TravNet** fuses RGB and depth to predict a dense, per-pixel “traversability coefficient.” The controller then plans within the camera’s field of view using **MPPI** with a kinodynamic model, seeking paths that both reach the goal and maintain high predicted traction. Field tests across varied terrain (forests, beaches, snow) show the method avoids both geometric obstacles and visually subtle hazards (e.g., snow) that LiDAR geometry alone misses. In comparative studies, WayFAST performs on par with a LiDAR-based baseline in forests, and succeeds on snow where LiDAR planning fails. The approach is data-efficient, requiring only a short data-collection run to bootstrap labels. Limitations include reliance on camera **FOV** and depth range and ambiguity when visually similar terrains have different traction. Overall, WayFAST demonstrates a practical path to robust outdoor navigation without manual labeling.

## 3. Keywords

- Self-supervised traversability  
- Traction estimation (μ)  
- NMHE + EKF  
- RGB-D perception / TravNet (ResNet-18)  
- MPPI sampling control  
- Skid-steer kinodynamics  
- Outdoor unstructured navigation  
- Snow/semantic terrain hazards

## 4. Problem Addressed

Enable waypoint-free, **goal-directed navigation** in outdoor, unstructured environments by predicting **where the robot can obtain traction**, not just where geometry is free—overcoming the gap between geometric “free space” and true traversability (e.g., snow, sand, tall grass).

## 5. Innovation / Main Contribution

- **Self-supervised labels from state estimation**: NMHE estimates traction-related parameters (μ, ν) online; projected into images to train a traversability network without human labels or heuristics.  
- **RGB-D TravNet**: ResNet-18 encoder with depth fusion predicts dense traversability coefficients in image space.  
- **Traction-aware model-based control**: MPPI plans within the camera FOV using the predicted μ(x,y) to bias toward safe, high-traction paths.

## 6. Key Methods / Architecture

- **Robot & sensors:** TerraSentia skid-steer; Intel RealSense D435i RGB-D (usable depth ~5 m); Jetson Xavier NX onboard compute.  
- **State estimation & labels:** NMHE solves a finite-horizon optimization to estimate states and μ,ν, feeding an EKF; μ,ν ∈ [0,1] and serve as traversability coefficients.  
- **Training data creation:** Project robot’s future trajectory and estimated μ onto images to create sparse label masks; address label imbalance via distribution smoothing over μ bins.  
- **TravNet:** RGB-D encoder–decoder; output is a single-channel traversability image (≈424×240).  
- **Controller:** NMPC/MPPI minimizes tracking error, control effort, terminal cost, and adds a **clearance term** that samples nearby points to maximize local traversability margin.  
- **Real-time:** >4000 sampled trajectories at ~10 Hz on GPU.

## 7. Datasets & Experimental Setup

- **Data collection:** ~2 hours of teleop driving in forest-like & semi-urban areas → ~15k images; labels from NMHE+EKF. Images downsampled to 2 fps.  
- **Evaluation environments:** US Midwest & Puerto Rico; forests, urban corridors, beaches; snow-covered grass fields.  
- **Baselines:** BADGR (self-supervised), LiDAR-based geometric traversability, and “Blind Pursuit” goal-seeking. All use the same controller except for perception.

## 8. Evaluation Metrics & Results

- **Success rate (reach goal) across trials/starts** is the primary metric.  
- **Forest-like environment:** WayFAST reached the goal **as many times as LiDAR**, outperforming BADGR and Blind Pursuit.  
- **Snow terrain:** LiDAR **0/5** successes (stuck in snow); WayFAST **4/5** successes, with one near-goal failure.  
- **Generalization (urban):** With RGB+Depth, **5/5** successes; RGB-only, **3/5** successes.  
- **Uneven terrain (RGB-only):** **3/4** close-to-goal; **1/4** stuck.

## 9. Limitations (vs. Our Project: Human-Preference BEV Semantics + Elevation + MPC+CBF)

- **Short horizon & narrow FOV**: Camera FOV and ~5 m depth limit planning horizon; difficult to route around large untraversable regions—our BEV + elevation mapping aims for longer-horizon context.  
- **No BEV semantic map**: Predictions live in **image space**; no explicit top-down semantics or class-specific costs.  
- **No explicit elevation modeling**: Depth is used, but no elevation grid or curb/slope reasoning as in our plan.  
- **No human-preference modeling**: Optimizes traction, not social/comfort norms.  
- **Control safety guarantees**: Uses MPPI; no CBF-style hard safety constraints as in our MPC+CBF plan.  
- **Ambiguity in visually similar terrains** (e.g., variable snow): may require extra sensing (e.g., force, thermal) or history.

## 10. Similarity / Relevance to Our Work

- **Shared goals**: terrain-aware outdoor navigation, avoiding subtle hazards not captured by geometry alone.  
- **Key difference**: WayFAST optimizes **traction-predicted traversability**; our work optimizes **human-preference costs** (sidewalk ≺ grass ≺ snow), **BEV semantics**, and **elevation-aware safety** with MPC+CBF.

## 11. Potential Integration Points

- **Self-supervised label pipeline**: Use **NMHE-style traction estimation** to auto-label “comfort/risk” or even to **learn preference weights** for our BEV cost map—turn μ into an additional BEV layer.  
- **Clearance term in cost**: Borrow their **sampling-based clearance** around candidate states to inflate safety margins in our MPC stage.  
- **RGB-D fusion tricks**: TravNet’s simple yet effective **ResNet-18 + depth fusion** could inform our BEV semantics network.  
- **Fast MPPI backend**: Consider MPPI (>4000 samples @~10 Hz) as a fallback/planning inner loop under our MPC+CBF outer safety constraints.

## 12. Open Questions / Future Work

- **Sensing breadth**: How to overcome FOV/range limits—multi-camera, fisheye, or LiDAR+vision fusion?  
- **Terrain ambiguity**: Detecting when visually similar surfaces (e.g., snow) differ in traversability; possibly require new modalities.  
- **Generalization & data**: How robust is μ across speeds/loads/weather—and can online adaptation mitigate drift?

## 13. Reproducibility Notes

- **Hardware**: Commercial platform (TerraSentia), RealSense D435i, Jetson NX—accessible but not identical to all labs.  
- **Algorithm detail**: NMHE/EKF/MPPI and training process described with sufficient equations and steps; success counts for several scenarios are provided in text.  
- **Code/data**: No public repository link is listed in the paper text; replication is feasible but requires re-implementation and data collection following their procedure.

## 14. Link

- DOI: https://doi.org/10.1109/LRA.2022.3193464
