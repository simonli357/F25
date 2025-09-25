# TerraPN --- Paper Review & Research Relevance Report

## 1. Citation Information (BibTeX)

``` bibtex
@article{Sathyamoorthy2022TerraPN,
  title   = {TerraPN: Unstructured Terrain Navigation using Online Self-Supervised Learning},
  author  = {Adarsh Jagan Sathyamoorthy and Kasun Weerakoon and Tianrui Guan and Jing Liang and Dinesh Manocha},
  journal = {arXiv preprint arXiv:2202.12873},
  year    = {2022},
  archivePrefix = {arXiv},
  eprint  = {2202.12873},
  primaryClass = {cs.RO},
  url     = {http://gamma.umd.edu/terrapn/}
}
```

## 2. Clear Abstract (150--200 words)

TerraPN introduces a self-supervised approach for outdoor robot
navigation that learns terrain "surface costs" directly from
robot--terrain interaction, rather than relying on human-labeled
semantic datasets. The system ingests RGB image patches and recent robot
velocities, and it learns to predict a four-element label vector derived
from IMU vibration features and odometry errors---quantities that
capture traction, bumpiness, and deformability. These predictions are
combined into a per-pixel surface cost map using non-uniform patch
sampling that places finer patches near terrain boundaries for speed and
fidelity. For planning, TerraPN adapts the Dynamic Window Approach (DWA)
by adding a surface-cost term to the objective and by modulating
allowable accelerations based on predicted cost along the forward
trajectory, producing collision-free, dynamically feasible motions that
slow on rough or slippery ground. On a Clearpath Husky in real outdoor
scenes, TerraPN trains online in about 20--25 minutes for five surfaces
and achieves higher success rates, lower vibration costs, and reduced
speeds on high-cost terrain compared to DWA, TERP (elevation-based), and
segmentation-driven baselines. These results show that a lightweight,
robot-specific, self-supervised model can outperform semantic
segmentation for traversability-aware navigation in unstructured
environments.

## 3. Keywords

-   Self-supervised terrain learning\
-   Surface cost map / navigability cost\
-   Non-uniform patch sampling\
-   Dynamic Window Approach (DWA) adaptation\
-   IMU vibration features (PCA)\
-   Odometry error--based labels\
-   Unstructured outdoor navigation\
-   Acceleration limit modulation\
-   Clearpath Husky platform

## 4. Problem Addressed

Learning terrain navigability that truly reflects **robot-specific**
interaction (traction, bumpiness, deformability) without expensive human
labeling, and using that information to plan **dynamically feasible,
low-vibration** trajectories in unstructured outdoor environments.

## 5. Innovation / Main Contribution

-   **Online self-supervised cost learning** from robot
    IMU/odometry---no human annotations; trains in \~20 minutes for five
    surfaces.\
-   **Non-uniform image patch sampling** around surface boundaries to
    cut inference time by **47.27%** vs. uniform sampling.\
-   **Navigation that adapts DWA** with a surface-cost term and
    cost-based acceleration limits, improving success rates (up to
    **+35.84%**), lowering vibration (up to **−21.52%**), and reducing
    speed on high-cost terrain (up to **−46.76%**).

## 6. Key Methods / Architecture

-   **Inputs**: RGB patch + sequence of recent linear/angular
    velocities.\
-   **Labels (self-supervised)**:
    -   IMU (6-DoF) reduced via PCA → variances along top two components
        (bumpiness/vibration).\
    -   Odometry errors w.r.t. LiDAR odometry (LEGO-LOAM): distance and
        heading errors (traction/slip).\
-   **Surface cost map**: Weighted norm of predicted 4-vector; tiled
    back to image resolution.\
-   **Non-uniform patch sampling**: weak segmentation using Sobel + GMM
    (BIC) + watershed; large patches for homogeneous regions, small near
    boundaries.\
-   **Planning**: DWA search space unchanged, but objective becomes G2 =
    α·head + β·dist + γ·vel − δ·sur; allowable accelerations scaled by τ
    = cos(future cost).

## 7. Datasets & Experimental Setup

-   **Platform**: Clearpath Husky with Velodyne VLP-16 LiDAR and Intel
    RealSense camera; compute: Intel i9 + RTX 2080 laptop. LiDAR used
    for initial odometry + 2D obstacle scans.\
-   **Training regime**: autonomous data collection over multiple
    surfaces; online training completes in \~20--25 minutes for five
    surfaces.\
-   **Scenarios**: five real outdoor settings mixing trained and
    untrained surfaces (e.g., tiles/concrete/mud/grass; highly
    unstructured fallen leaves/branches/mud).

## 8. Evaluation Metrics & Results

-   **Metrics reported**: success rate, normalized trajectory length,
    vibration cost, mean velocity.\
-   **Headline results** (vs. DWA, TERP, OCRNet/PSPNet):
    -   Success rate: TerraPN best across scenarios (e.g.,
        100/88/85/72%).\
    -   Vibration cost: lowest in 3/4 scenarios; TERP wins once due to
        curb elevation cues.\
    -   TerraPN slows appropriately on rough/slippery surfaces; others
        often drive at near-max speed.\
    -   Aggregate claims: **+35.84%** success, **−21.52%** vibration,
        **−46.76%** mean velocity on high-cost surfaces.

## 9. Limitations (with respect to our project)

-   Must **drive on a surface to learn its cost**; not applicable to
    fully non-traversable terrain (e.g., swamp).\
-   Sensitive to **lighting changes** affecting RGB appearance.\
-   No depth/elevation → **cannot detect subtle height changes** (e.g.,
    curbs), which impacted performance vs. TERP.\
-   Limited analysis on the **number of surfaces** the model can learn
    effectively.

*Relevance:* Our work explicitly uses **BEV semantics + elevation
mapping + MPC/CBF**. TerraPN's reliance on RGB and lack of elevation
makes it less aligned with our elevation-aware safety aims, but highly
aligned with **learning robot-specific comfort/roughness costs** we care
about.

## 10. Similarity / Relevance to Our Work

-   **Overlap**: learns a **terrain cost** that reflects human-salient
    comfort/safety proxies (vibration, slip), and **modulates speed**
    accordingly---useful for "human-preference" navigation.\
-   **Differences**: no **BEV semantic map**, no **elevation fusion**,
    and planner is **DWA-based** rather than MPC+CBF; costs reflect
    physical traversability more than social norms.

## 11. Potential Integration Points

-   Use TerraPN's **self-supervised labeling** (IMU PCA + odom error) to
    **learn our preference weights** for terrain classes in the BEV
    map---turning semantics into **data-driven human-comfort costs**.\
-   Plug learned **surface cost layer** into our **MPC+CBF** as an
    additional state-dependent **soft cost** (MPC objective) and **hard
    barrier** (CBFs for high-cost zones).\
-   Adopt **non-uniform patch sampling** (Sobel+GMM+watershed) to
    sharpen **BEV boundary costs** where terrain transitions occur.\
-   Borrow **cost-based acceleration scaling** as a CBF-compatible
    constraint (limit forward acceleration when predicted cost ahead is
    high).\
-   Use short **online fine-tuning** (\~20--25 min) when deploying to
    new sites to quickly adapt BEV cost weights.

## 12. Open Questions / Future Work (from the authors)

-   **Multi-sensor fusion** (add depth/elevation) to better capture
    geometric hazards and lighting robustness.\
-   **Scalability** to many terrain types and broader conditions
    (illumination/seasonality).

## 13. Reproducibility Notes

-   **Project page** provides supplemental tech report/video; the PDF
    does **not** reference a public code release.\
-   Hardware/sensors and compute are specified (Husky, VLP-16,
    RealSense, i9 + RTX 2080), aiding replication; training is feasible
    on a laptop GPU; LiDAR used for odometry and 2D obstacle scans.\
-   Inference efficiency aided by **non-uniform sampling**; reported
    **47.27%** speedup vs. uniform patching.

## 14. Link

Project page: http://gamma.umd.edu/terrapn/
