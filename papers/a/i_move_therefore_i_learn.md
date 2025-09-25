# I Move Therefore I Learn --- Structured Paper Review

## 1. Citation Information

``` bibtex
@article{deMiguel2025IMoveThereforeILearn,
  title   = {I Move Therefore I Learn: Experience-Based Traversability in Outdoor Robotics},
  author  = {Miguel {\'A}ngel de Miguel and Jorge Beltr{\'a}n and Juan S. Cely and Francisco Mart{\'\i}n and Juan Carlos Manzanares and Alberto Garc{\'\i}a},
  journal = {arXiv preprint arXiv:2507.00882},
  year    = {2025},
  archivePrefix = {arXiv},
  primaryClass  = {cs.RO},
  url     = {https://arxiv.org/abs/2507.00882}
}
```

## 2. Clear Abstract

The paper proposes an experience-driven way for outdoor robots to decide
which terrain is safe to drive on. Instead of requiring labeled data,
the robot first explores under teleoperation and saves compact feature
vectors from small map patches that combine color (RGB) and elevation. A
lightweight VAE (trained only on a generic texture dataset) encodes each
patch; BIRCH clustering summarizes what "safe" terrain looked like
during exploration. Later, in autonomous mode, the robot slides a window
over its live multi-layer grid map and compares each patch to the
learned clusters to produce a traversability score in real time. The
method works across different robots and scenes, adapts online when it
encounters new surfaces, and shows competitive results in simulation
(NEGS-UGV) and in real campus trials with wheeled and legged robots.

## 3. Keywords

-   Traversability estimation\
-   Experience-based learning\
-   Multi-layer grid maps\
-   Variational Autoencoder (VAE)\
-   BIRCH clustering\
-   RGB--LiDAR fusion\
-   Self-supervised navigation\
-   Outdoor robotics\
-   ROS 2 / Nav2 integration\
-   NEGS-UGV benchmark

## 4. Problem Addressed

How can a robot decide, **without task-specific labeled data**, whether
novel terrain is safe to traverse---adapting to new surfaces and
platforms---using only its own prior navigation experience?

## 5. Innovation / Main Contribution

-   **Experience-as-supervision:** Learn traversability from where the
    robot *actually drove* during a teleop phase---no scenario-specific
    labels.\
-   **Patch-level latent encoding:** A tiny VAE trained on **textures
    (DTD)**, not on target scenes, to create generalizable patch
    descriptors from RGB+elevation sub-gridmaps.\
-   **Online summarization with BIRCH:** Streaming clustering keeps only
    representative "safe" feature centers, enabling scalable, adaptive
    comparisons.\
-   **Direct planner coupling:** Produces a traversability **cost
    layer** compatible with Nav2 via occupancy-grid mapping.

## 6. Key Methods / Architecture

-   **Sensing & Map Layers**
    -   Project LiDAR points into camera, populate **multi-layer grid
        map** with RGB (3 channels) + elevation (1 channel).
-   **Patch Encoding**
    -   Slide an *n×n* window (e.g., 16×16 cells) over traversed
        regions; encode each patch with a **3-conv VAE** → 32-D latent
        vector. VAE trained on **DTD textures** only.
-   **Experience Memory**
    -   Store/compress vectors using **BIRCH**; new vectors that fall
        within a cluster radius are discarded as redundant; otherwise
        update subclusters.
-   **Deployment Inference**
    -   For each live patch, compute **Euclidean distance** to nearest
        BIRCH subcluster center; convert to a **center-weighted kernel**
        over the window to fill a **traversability layer**; take max
        across overlapping windows.
-   **Navigation Stack**
    -   Export traversability layer as an **OccupancyGrid** consumable
        by **Nav2**, via an extended map server; implemented in **ROS 2
        Jazzy**.

## 7. Datasets & Experimental Setup

-   **Simulated benchmark:** **NEGS-UGV** (four outdoor scenarios: Park,
    Hill, Forest, Lake) with perfect 2D/3D semantic labels and poses.\
-   **Real robots & sensors:**
    -   **Robotnik Summit XL** (wheeled): Intel NUC + Jetson Xavier AGX;
        **16-plane 3D LiDAR**, **ZED2 RGB**.\
    -   **Unitree Go2** (legged): Jetson Orin NX; **ultra-wide 4D
        LiDAR**, **32-plane 3D LiDAR**, **RealSense D435i**.
-   **Training data for encoder:** **DTD** texture dataset only (no
    target scenes).
-   **Baselines:** **WVN** (vision transformer, self-supervised) and
    **NAEX** (LiDAR-based).
-   **Ablation:** Grid cell sizes 0.05, 0.10, 0.20 m; 0.10 m best
    overall with 16×16 window.

## 8. Evaluation Metrics & Results

-   **Metric:** (F\_{0.5}) emphasizing **precision** (safety-critical:
    avoid false "traversable").
-   **NEGS-UGV headline findings:**
    -   Qualitatively comparable to WVN/NAEX in easy scenes; **more
        conservative** in steep "Hill," initially marking only the
        actually traversed path as safe; performance improves when the
        simulated trajectory includes the steep flanks.
-   **Ablation (mean (F\_{0.5})):** 0.05 m: **0.74**; 0.10 m: **0.75**
    (best); 0.20 m: **0.67**.
-   **Real-world (campus) highlights:**
    -   **Scenario 1 (path→grass)**: before grass traversal, method is
        **precise** (e.g., 0.72) versus WVN/NAEX (0.37/0.20); after
        driving on grass, method updates and remains competitive
        (0.92).\
    -   **Scenario 2 (sidewalk→crossing)**: method maintains an edge or
        parity (0.63) and **does not generalize to road** without
        experience, aligning with safety.

## 9. Limitations

-   **Experience-dependence:** Initially conservative; untraversed but
    safe areas remain "unknown" until explored (teleop burden).\
-   **No explicit semantics:** Operates on appearance+elevation
    similarity; cannot explain *why* a patch is risky.\
-   **Ground-truth mismatch:** Static labels penalize an inherently
    **adaptive** definition of traversability.\
-   **Latent ambiguity:** Texture-based encoding could confuse visually
    similar but physically different surfaces (e.g., wet vs. dry
    asphalt) without proprioceptive cues.

## 10. Similarity / Relevance to **Our Work**

-   **Overlap:**
    -   Uses **geometry (elevation)** + **appearance (RGB)** to produce
        a traversability cost layer---directly relevant to our **unified
        multi-layer map**.\
    -   Integrates seamlessly with **Nav2** costmaps, matching our
        planning stack.
-   **Divergence from our vision:**
    -   Lacks **explicit semantics** (no grass/asphalt labels),
        **terrain properties** (friction/roughness), and **preference
        modeling** (e.g., "prefer sidewalk").
-   **What it offers us:**
    -   A **self-supervised traversability prior** that can fill gaps
        when semantics or physics predictors are uncertain.\
    -   A memory mechanism (BIRCH of latent patch features) for **fast
        online adaptation** across sites/robots.
-   **Our current limitations vs. this work:**
    -   We may rely more on supervised semantic/property estimators;
        this approach **reduces annotation demand** and yields safer
        initial behavior.

## 11. Potential Integration Points

-   **Add an "experience layer"** to our map: store VAE-BIRCH
    traversability as another raster alongside
    elevation/semantics/friction; fuse into our planner cost.\
-   **Hybrid cost design:**
    -   Cost = max(experience-risk, geometric-risk, low-friction-risk),
        with tunable weights; default to **experience** when confident,
        fall back to **semantics/physics** otherwise.\
-   **Self-supervision pipeline:** Use traversed paths as **positive
    labels** to fine-tune our friction/semantics heads online.\
-   **Patch footprint tuning:** Match the paper's insight---window size
    (`\approx`{=tex}) robot footprint; auto-select from URDF to
    generalize across Anymal D and wheels.\
-   **Memory management:** Borrow **BIRCH** to keep a compact,
    up-to-date library of "safe looks," enabling lifelong learning
    across runs.

## 12. Open Questions / Future Work (from authors & inferred)

-   How sensitive is performance to **BIRCH threshold**, **window
    size**, and the **distance→kernel** mapping?\
-   Can proprioceptive signals (slip, vibration) augment the latent
    features to disambiguate visually similar but risky surfaces?\
-   How to incorporate **rules/affordances** (e.g., "roads are
    off-limits unless at crosswalks") while staying experience-driven?\
-   Extending from 2.5D to **full 3D traversability** for large
    slopes/steps.\
-   Robustness under **lighting, shadows, weather** and seasonal
    appearance shifts.

## 13. Reproducibility Notes

-   **Code & ROS 2 Jazzy** implementation provided; includes **rosbags**
    and instructions; integrates with **Nav2** via an extended map
    server.\
-   Uses widely available libraries (**grid_map**, ROS 2) and generic
    **DTD** for VAE pretraining.\
-   Benchmarking with **NEGS-UGV** (public).\
-   Overall: **reproducible** for ROS users; online adaptation hinges on
    replicating the teleop exploration.

## 14. Link

-   Paper: <https://arxiv.org/abs/2507.00882>

## 15. recap --- What the paper actually does (bullet walkthrough)

-   Build a **multi-layer grid map** in real time (RGB + elevation from
    camera--LiDAR fusion).\
-   During **teleop exploration**, slide a window over traversed regions
    and **encode each patch** with a tiny **VAE** → 32-D vector.\
-   **Cluster** these vectors online with **BIRCH** to get compact "this
    looked safe when we drove there" centers.\
-   In **deployment**, slide the same window everywhere; for each patch,
    **measure distance** to nearest cluster center.\
-   Convert that distance into a **traversability score** via a
    center-weighted kernel across the window; take the max over overlaps
    to fill a **traversability layer**.\
-   **Export** the layer as an **OccupancyGrid**; let **Nav2** plan
    paths that prefer high-traversability cells.\
-   As the robot **encounters new surfaces**, add/update clusters → the
    map **adapts online**; previously unseen but similar terrain becomes
    allowed.\
-   Validate on **NEGS-UGV** and real campus runs with **wheeled &
    legged** robots; report **(F\_{0.5})**, ablations, and qualitative
    maps showing safe-biased behavior.
