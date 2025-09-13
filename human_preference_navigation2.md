# Research Plan: Human-Preference Terrain-Aware Robot Navigation

## Research Question

**How can a mobile robot plan and execute navigation in semi-structured
outdoor environments so that its chosen route reflects *human terrain
preferences*---not merely physical traversability---while guaranteeing
safety and dynamic feasibility?**

## Key Problem

-   **Gap in current navigation systems:**
    -   Conventional planners rely on **occupancy grids or
        traversability maps**, which primarily encode *physical
        feasibility* (e.g., slope, roughness, friction).
    -   These methods **do not capture human-like preferences** (such as
        choosing a clear sidewalk over a shorter snowy path) and often
        produce shortest-path routes that are uncomfortable, unsafe, or
        socially undesirable.
-   **Need:** A unified framework that fuses **semantic understanding**
    and **elevation awareness** with **formal safety guarantees**, so
    that robots behave more like people making terrain choices.

## Main Contribution / Innovation

-   A **camera-based semantic BEV mapping system** fused with elevation
    data to create a **human-desirability terrain cost map**, which
    differs from classical occupancy/traversability maps.
-   Integration of this cost map into **Model Predictive Control (MPC)
    enhanced by Control Barrier Functions (CBF)** to ensure the robot
    follows **human-preferred routes** while remaining dynamically
    feasible and provably safe.
-   Demonstration that a robot can navigate **semi-structured city-like
    environments** (sidewalks, grass, snow, curbs) in a way that
    reflects **human decision-making**, not just geometric or physical
    constraints.

## Pipeline Components

1.  **Bird's-Eye-View (BEV) Semantic Terrain Map**
    -   Monocular RGB camera → semantic segmentation → inverse
        perspective mapping (IPM).
    -   Produces a top-down map where each pixel encodes terrain
        category (road, grass, sidewalk, snow, etc.).
2.  **Human-Preference Terrain Cost Map**
    -   Fuse BEV semantics with learned/heuristic *preference weights*
        (e.g., sidewalk \< dry grass \< snow).
    -   Creates a spatial cost layer for planning that reflects human
        choices, not just free vs. occupied space.
3.  **Elevation Mapping** *(in progress)*
    -   LiDAR/stereo depth → 3-D elevation grid.
    -   Detects slopes, curbs, drop-offs and fuses with the semantic
        cost map.
4.  **Model Predictive Control (MPC) + Control Barrier Functions (CBF)**
    *(in progress)*
    -   Plans and tracks safe, dynamically-feasible trajectories while
        enforcing barrier constraints to stay on preferred terrain and
        avoid unsafe regions.

## Difference from Conventional Traversability / Occupancy-Map Approaches

  --------------------------------------------------------------------------
  Conventional Occupancy / Traversability Map               This Work
  --------------------------------------------------------- ----------------
  Represents terrain as **binary or scalar traversability** Represents
  (free / occupied, or cost from slope, roughness).         terrain with
                                                            **rich
                                                            semantics**
                                                            (sidewalk, snow,
                                                            grass, asphalt)
                                                            and **human
                                                            preference
                                                            weights**.

  Cost usually based on **physical feasibility** (friction, Cost explicitly
  slope).                                                   encodes **social
                                                            and comfort
                                                            preferences**
                                                            (e.g., prefer
                                                            sidewalk even if
                                                            grass is
                                                            traversable).

  Typically uses 3-D LiDAR or stereo to estimate elevation  Adds
  and obstacles.                                            **camera-based
                                                            semantic BEV +
                                                            elevation
                                                            fusion**, so the
                                                            map is both
                                                            geometrically
                                                            and semantically
                                                            aware.

  Goal: ensure the robot can drive anywhere it is           Goal: ensure the
  physically able.                                          robot moves
                                                            **where a person
                                                            would choose**,
                                                            balancing
                                                            safety,
                                                            stability, and
                                                            human-like
                                                            norms.
  --------------------------------------------------------------------------

## Implementation Status

-   ✅ **Implemented**
    -   BEV generation from monocular RGB (segmentation + IPM).
    -   Human-preference terrain cost map construction.
-   🔄 **In Progress**
    -   Elevation mapping and semantic fusion.
    -   MPC + CBF control.

## Planned Platform & Simulator

-   **Robot platform:** Quadruped or high-mobility ground robot (ANYmal,
    Unitree, Spot, Clearpath Husky).
-   **Simulator:** Isaac Sim for photorealistic sensors, diverse
    terrain, and ROS 2 integration.

## Known Challenges in Isaac Sim

-   Accurate terrain physics (friction/material tuning).
-   Quadruped contact stability (solver tuning).
-   Multi-sensor synchronization (camera/LiDAR/IMU).
-   High GPU/CPU requirements.
-   Steep learning curve for USD asset creation and physics tuning.
    *(Mitigations: procedural scene generation, careful sensor
    calibration, multi-resolution elevation mapping, decoupled control
    loops.)*

## Proposed Timeline

**September** - Set up Isaac Sim, build minimal terrain environments
(sidewalks, curbs, grass), attach sensors.

**October** - Integrate BEV + terrain-cost pipeline into Isaac. -
Implement elevation mapping and begin MPC + CBF control.

**November** - Conduct structured experiments and ablations: -
Segmentation backbone comparisons. - With/without elevation fusion. -
MPC vs. MPC + CBF safety evaluation. - Human-preference
vs. shortest-path planners.

**Late November -- December** - Finalize experiments, visuals, and
quantitative metrics. - Draft and polish the conference paper.
