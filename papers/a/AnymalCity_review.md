# Learning Robust Autonomous Navigation and Locomotion for Wheeled-Legged Robots --- Review

## 1. Citation Information

``` bibtex
@article{lee2024wheeledlegged,
  title   = {Learning Robust Autonomous Navigation and Locomotion for Wheeled-Legged Robots},
  author  = {Lee, Joonho and Bjelonic, Marko and Reske, Alexander and Wellhausen, Lorenz and Miki, Takahiro and Hutter, Marco},
  journal = {Science Robotics},
  volume  = {9},
  number  = {89},
  pages   = {adi9641},
  year    = {2024},
  doi     = {10.1126/scirobotics.adi9641},
  note    = {Accepted version, ETH Zurich / Neuromeka / Swiss-Mile Robotics AG}
}
```

## 2. Clear Abstract

This paper presents a fully integrated navigation and locomotion system
for wheeled-legged robots operating in complex urban settings. The
authors develop a hierarchical reinforcement learning (HRL) framework
that couples a high-level navigation controller with a low-level hybrid
locomotion controller. The locomotion policy, trained via model-free RL
with privileged information, enables seamless transitions between
driving and stepping gaits while remaining robust to rough terrain. The
navigation policy directly outputs velocity commands, using terrain
height scans, position history, and the hidden state of the locomotion
policy to plan locally without explicit path-planning modules. The robot
autonomously executed kilometer-scale missions in Zurich and Seville,
negotiating stairs, narrow corridors, dynamic pedestrians, and diverse
surfaces at speeds up to 1.68 m/s with a mechanical cost of transport 53
% lower than a pure-legged ANYmal baseline. Comparisons with a
sampling-based planner show superior success rates, lower tracking
error, and faster inference. The work demonstrates that combining HRL
with wheeled-legged hardware yields fast, energy-efficient, and robust
urban navigation, offering a promising approach for future last-mile
delivery and large-scale autonomous mobility.

## 3. Keywords

-   Wheeled-legged robot\
-   Hierarchical reinforcement learning (HRL)\
-   Hybrid locomotion\
-   Model-free reinforcement learning\
-   Urban navigation\
-   Dynamic obstacle avoidance\
-   Elevation mapping\
-   Low-level locomotion controller\
-   High-level navigation policy\
-   Kilometre-scale autonomy

## 4. Problem Addressed

Enable a single robot to **navigate long urban routes autonomously**
while **driving fast on flat ground and stepping over complex terrain**,
without relying on handcrafted gait switching or slow sampling-based
planners.

## 5. Innovation / Main Contribution

-   **Hierarchical RL architecture** that tightly couples a memory-based
    high-level navigation policy with a learned low-level locomotion
    policy.\
-   **Privileged training** for the locomotion controller, yielding
    robust hybrid gait switching without explicit gait scheduling.\
-   **Kilometer-scale real-world validation** in two European cities,
    showing high speed and low energy cost relative to state-of-the-art
    legged robots.

## 6. Key Methods / Architecture

-   **Low-Level Controller (LLC):**
    -   Recurrent RL policy controlling joint positions and wheel
        velocities at 50 Hz.\
    -   Privileged information (noiseless states, terrain properties)
        during training; only raw IMU, encoders, and elevation maps at
        deployment.
-   **High-Level Controller (HLC):**
    -   Processes terrain height scans, position history, and LLC latent
        state.\
    -   Generates velocity targets at 10 Hz, replacing conventional
        path-planner + path-follower stack.\
    -   Uses exploration memory to avoid local minima.
-   **Simulation & Training:**
    -   Procedurally generated worlds with Wave Function Collapse
        algorithm and navigation graphs.\
    -   Proximal Policy Optimization for both controllers.\
-   **Perception & Hardware:**
    -   Triple LiDARs, RGB stereo camera, elevation mapping for local
        terrain, camera-based human detection.\
    -   Point-cloud localization against a pre-scanned city map.

## 7. Datasets & Experimental Setup

-   **Real-world sites:** Urban areas in Zurich (Glattpark) and
    Seville.\
-   **Pre-scanned navigation graph:** Dense LiDAR point clouds manually
    annotated for socially acceptable routes.\
-   **Robot:** Wheeled-legged quadruped with actuated wheels, RGB stereo
    camera, three LiDARs, GPS antenna, 5 G router, IMU, and joint
    encoders.\
-   **Training data:** Synthetic navigation worlds with dynamic
    obstacles for RL.

## 8. Evaluation Metrics & Results

-   **Mechanical Cost of Transport (COT):** 0.16 (53 % lower than ANYmal
    baseline).\
-   **Average speed:** 1.68 m/s; peak speed 5 m/s on flat terrain.\
-   **Success metrics:** Higher success rate and lower collision rate
    than a sampling-based planner; average tracking error 0.24 m
    vs. 0.45 m baseline.\
-   **Autonomy:** 8.3 km missions with minimal human intervention;
    reliable obstacle negotiation (stairs, narrow corridors,
    pedestrians).

## 9. Limitations (vs. Your Project)

-   **Minimal semantic reasoning:** Navigation relies almost entirely on
    geometry/elevation; no BEV semantic terrain classification.\
-   **Manual global map creation:** Requires labor-intensive
    pre-scanning and graph building.\
-   **Limited field of view & latency:** Elevation mapping restricts
    perception to \~3 m forward, constraining maximum autonomous speed.\
-   **No explicit human-preference modeling:** Social comfort or terrain
    desirability is not learned beyond static "avoid landscaping"
    annotations.

## 10. Similarity / Relevance to Our Work

-   Shares **terrain-aware local navigation and elevation mapping**, and
    demonstrates large-scale autonomy with dynamic obstacle handling.\
-   Hierarchical control echoes our **MPC + Control Barrier Function
    layered approach**, though they use RL instead of optimization for
    the high level.\
-   Valuable insight into integrating **driving and stepping gaits** and
    using **privileged RL** for robustness---useful for our
    elevation-aware controller.

## 11. Potential Integration Points

-   **Memory-based high-level policy**: could inspire memory modules for
    long-horizon MPC or for reasoning over visited BEV map cells.\
-   **Privileged-information training** for low-level control to improve
    robustness of our MPC-CBF tracking policy.\
-   **Dynamic obstacle augmentation** (camera-detected humans with
    safety margins) directly aligns with human-preference navigation.

## 12. Open Questions / Future Work

-   Incorporating **semantic scene understanding** (e.g., pavement
    vs. grass) for richer decision-making.\
-   Reducing reliance on pre-scanned maps through online mapping or
    vision-only SLAM.\
-   Expanding field of view and perception speed to exploit hardware's
    full locomotion capability (\>6 m/s).\
-   Investigating end-to-end learning vs. hierarchical control
    trade-offs.

## 13. Reproducibility Notes

-   **Code/Data:** Not publicly released; only supplementary videos and
    detailed algorithm descriptions.\
-   **Hardware dependence:** Requires a wheeled-legged quadruped with
    multiple LiDARs; training relies on large-scale simulation
    infrastructure.\
-   **Procedural training worlds** described sufficiently for
    replication by well-resourced labs.

## 14. Link

-   DOI:
    [10.1126/scirobotics.adi9641](https://doi.org/10.1126/scirobotics.adi9641)\
-   Project video: [YouTube Overview](https://youtu.be/vJXQG2_85V0)
