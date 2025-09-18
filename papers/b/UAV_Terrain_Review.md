# UAV-Assisted Self-Supervised Terrain Awareness for Off-Road Navigation --- Report

*Source:
UAV-Assisted_Self-Supervised_Terrain_Awareness_for_Off-Road_Navigation.pdf*

------------------------------------------------------------------------

## 1. Citation Information (.bib)

``` bibtex
@inproceedings{Fortin2025UAVSelfSupervisedTerrain,
  author    = {Jean{-}Michel Fortin and Olivier Gamache and William Fecteau and Effie Daum and William Larriv{'e}e{-}Hardy and Fran{\c{c}}ois Pomerleau and Philippe Gigu{\`e}re},
  title     = {UAV-Assisted Self-Supervised Terrain Awareness for Off-Road Navigation},
  booktitle = {2025 IEEE International Conference on Robotics and Automation (ICRA)},
  year      = {2025},
  month     = {May},
  address   = {Atlanta, USA},
  pages     = {11340--11346},
  publisher = {IEEE},
  isbn      = {979-8-3315-4139-2},
  doi       = {10.1109/ICRA55743.2025.11128050}
}
```

------------------------------------------------------------------------

## 2. Clear Abstract (150--200 words)

This paper proposes a self-supervised method to predict terrain
properties for off-road navigation using an overhead camera on a
hovering UAV. Instead of relying on geometric traversability heuristics
or terrain classes, the authors train a lightweight CNN (ResNet-18 +
MLP) to regress three directly useful signals for a UGV: vertical
vibration (from IMU (a_z) spectra), "bumpiness" (norm of roll/pitch
angular velocities), and power consumption (time-integrated
(I`\cdot `{=tex}V)). Image patches (1.5 m × 1.5 m) aligned to the
robot's path are extracted both from the UGV's BEV-projected front
camera and from the UAV's nadir videos, with labels generated purely
from the UGV's proprioception. On a 2.8 km forest/quarry dataset (∼13.5k
ground images; ∼12.9k aerial images), aerial patches yield substantially
lower RMSE than ground BEV patches---on average 21% improvement overall
and 37% in high vegetation---largely due to reduced occlusion and
uniform ground sampling distance. Ablations show prediction errors for
ground BEV grow monotonically with patch distance, implicating
projection blur as a key limiter; simulating blur on aerial patches
reproduces the trend. A field demo scouts an unseen area by drone,
builds per-metric cost maps, and plans a smoother UGV route than a
shortest-path baseline. The approach highlights UAV--UGV collaboration
as a practical way to anticipate terrain risks from a distance.

------------------------------------------------------------------------

## 3. Keywords

-   Off-road navigation\
-   Self-supervised learning\
-   UAV--UGV collaboration\
-   Bird's-eye-view (BEV) mapping\
-   Terrain property prediction\
-   Proprioception-derived labels\
-   Vibration/bumpiness/power metrics\
-   Aerial scouting\
-   Patch-based regression

------------------------------------------------------------------------

## 4. Problem Addressed

Predict terrain properties *ahead of the robot* in unstructured,
off-road environments where onboard, oblique cameras suffer occlusions
and rapidly decreasing ground resolution---hurting long-range planning
and comfort/safety prediction.

------------------------------------------------------------------------

## 5. Innovation / Main Contribution

-   Uses a hovering UAV's top-down view to train and infer
    **self-supervised** terrain property predictors (vibration,
    bumpiness, energy) aligned to the UGV path.\
-   Demonstrates **consistent performance gains vs. ground BEV** inputs,
    especially in high vegetation (occlusions).\
-   Provides quantitative ablations isolating **projection blur** and
    distance as key failure modes for ground-mounted cameras.\
-   Shows a **real-world scouting → mapping → planning** loop that
    improves path selection on an unseen site.

------------------------------------------------------------------------

## 6. Key Methods / Architecture

-   **Data capture:** Clearpath Warthog UGV with ZED X camera, Xsens
    IMU, battery current/voltage, RTK GNSS; DJI Mavic 3E hovering ∼10 m
    overhead. ArUco on UGV for scale/heading.\
-   **BEV & patching:**
    -   Ground: front camera → BEV via flat-ground assumption;
        time-varying extrinsics from IMU roll/pitch; 1.5 m patches
        sampled every 0.3 m along path.\
    -   Aerial: nadir frames; GSD from ArUco perimeter; same patch
        footprints co-registered to path.\
-   **Self-supervised labels:**
    -   (M_z): bandpower of (a_z) (1--30 Hz, Welch), log-transformed.\
    -   (M\_`\omega`{=tex}):
        \|,\[`\omega`{=tex}\_x,`\omega`{=tex}\_y\]\^ op\|.\
    -   (M_p): (`\sum `{=tex}I_b V_b `\Delta `{=tex}t) over a
        patch-length window.\
    -   Gaussian smoothing applied to each metric.\
-   **Model:** ResNet-18 backbone → 3-layer MLP → scalar regression;
    min-max labels; sigmoid output. 5-fold CV; batch 256; lr (5 imes
    10\^{-5}).\
-   **Inference & planning:** Sliding-window patch grid → metric maps →
    Dijkstra shortest path per metric.

------------------------------------------------------------------------

## 7. Datasets & Experimental Setup

-   **Environments:** Montmorency research forest: quarry, trails,
    gravel, moss, high vegetation. 45 min driving; 2.8 km; 7,536 unique
    terrain patches.\
-   **Images:** 13,484 ground; 12,935 aerial.\
-   **UAV altitude/GSD:** ∼10 m; ≈3.66 mm/pixel (uniform).\
-   **Controls:** Ground patches limited to ≤5 m distance for fair
    comparisons; additional ablations vary distance out to 10 m.\
-   **Hardware for demo:** Jetson Orin AGX onboard; UAV mission via DJI
    Pilot 2; RTK base shared by UGV/UAV.

------------------------------------------------------------------------

## 8. Evaluation Metrics & Results

-   **Primary metric:** RMSE of predicted normalized (M_z,
    M\_`\omega`{=tex}, M_p) vs. self-supervised labels.\
-   **Headline:** **Aerial beats ground by \~21.37% on average**, and
    **\~37.35% in high vegetation**.\
-   **Whole-dataset RMSE (UGV → UAV):**
    -   (M_z): 0.0728 → 0.0539 (\~26% ↓)\
    -   (M\_`\omega`{=tex}): 0.0592 → 0.0458 (\~23% ↓)\
    -   (M_p): 0.0374 → 0.0316 (\~16% ↓)\
-   **High-vegetation subset:**
    -   (M_z): 0.1389 → 0.0806 (\~42% ↓)\
    -   (M\_`\omega`{=tex}): 0.1217 → 0.0794 (\~35% ↓)\
    -   (M_p): 0.1285 → 0.0831 (\~35% ↓)\
-   **Ablations:**
    -   Ground BEV error increases monotonically with patch distance;
        performance degrades noticeably beyond \~6 m.\
    -   Simulated blur on aerial patches reproduces the distance-error
        trend → **projection blur** is a principal bottleneck.\
    -   Pixel footprint examples: ground BEV \~2.34 mm at 1 m
        vs. \~14.38 cm at 10 m; aerial at 10 m altitude ≈3.66 mm,
        uniformly.\
-   **Field demo:** UAV-scouted maps identify logs, puddles, and a large
    depression; metric-optimal path ((M_z)) outperforms a
    shortest-feasible baseline qualitatively.

------------------------------------------------------------------------

## 9. Limitations (esp. vs. our BEV semantics + elevation + MPC/CBF project)

-   **No semantics or elevation fused:** predictions are texture-driven;
    lacks class labels (e.g., sidewalk/grass/snow) and 3-D geometry for
    slopes/curbs---both central to our BEV semantic + elevation fusion.\
-   **Patch-based local context:** limited global scene reasoning;
    authors note future work on orthophotos/full-image inputs.\
-   **UAV visibility assumption:** requires unobstructed nadir view;
    under canopy or urban occlusions reduce applicability.\
-   **Planning & control:** uses Dijkstra over static cost; no dynamics,
    MPC, or CBF safety sets.\
-   **Human preference absent:** costs reflect physical effects
    (vibration/energy), not social/comfort norms we target.\
-   **Operational frictions:** manual UAV mission, manual image
    transfer; reliance on precise RTK co-registration.

------------------------------------------------------------------------

## 10. Similarity / Relevance to Our Work

-   **Overlap:** BEV-style top-down mapping; patch-wise cost prediction
    that can populate a terrain cost layer; off-road focus;
    self-supervised labels from proprioception align with our desire to
    reduce annotation.\
-   **Divergence:** No human-preference modeling, no semantic classes,
    no elevation fusion, and no MPC+CBF integration. Their pipeline is
    perception-to-geometric planning; ours is
    perception-to-**preference-aware** control.

------------------------------------------------------------------------

## 11. Potential Integration Points

-   **Cost-layer augmentation:** Add (M_z, M\_`\omega`{=tex}, M_p)
    predictors as extra channels in our BEV cost map. Human-preference
    weights can down-weight paths with high predicted vibration/energy
    even when semantically "allowed."\
-   **UAV scouting for BEV quality:** Use a hovering micro-UAV to
    generate **high-GSD** overhead imagery, mitigating IPM blur and
    occlusion in our BEV semantic map before planning.\
-   **Self-supervised label design:** Reuse their IMU/energy label
    generators to continually calibrate our semantic/elevation costs to
    the platform's actual comfort/effort.\
-   **Barrier functions:** Define CBFs that enforce bounds on predicted
    (M_z)/(M\_`\omega`{=tex}) along candidate trajectories (e.g.,
    barrier sets on discomfort), then couple with our MPC.\
-   **ArUco/AprilTag scale anchoring:** Practical recipe for
    scale/heading alignment between UAV imagery and UGV frame.\
-   **Photogrammetry hook:** Their stated future step (depth from aerial
    photos) dovetails with our elevation mapping---can supply dense DEMs
    to fuse with LiDAR/stereo.

------------------------------------------------------------------------

## 12. Open Questions / Future Work (from the paper)

-   Integrate geometric/depth cues (photogrammetry) with appearance.\
-   Move from patch-based to full image/orthophoto context.\
-   Scale data and benchmark against state-of-the-art off-road planners
    at larger sites.\
-   Reduce reliance on unobstructed overhead views; improve robustness
    under canopy.\
-   Tighter, automated UAV↔UGV data sync and on-board, real-time
    deployment.

------------------------------------------------------------------------

## 13. Reproducibility Notes

-   **Code/data:** No public release mentioned. Reproduction requires
    similar hardware (UGV + UAV + RTK + ArUco) and calibrations.\
-   **Training details provided:** architecture, hyperparameters, CV
    protocol, preprocessing are clearly stated, aiding replication;
    dataset is modest (2.8 km), so results may be sensitive to
    environment.

------------------------------------------------------------------------

## 14. Link

-   DOI:
    [10.1109/ICRA55743.2025.11128050](https://doi.org/10.1109/ICRA55743.2025.11128050)
