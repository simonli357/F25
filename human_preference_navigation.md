# Human-Preference Terrain-Aware Navigation: Research Plan & Summary

## Research Summary

- **Core Problem**  
  - Traditional autonomous navigation often chooses the *shortest geometric path*.  
  - In semi-structured outdoor environments—sidewalks, grass, snow, curbs—the shortest route is rarely the *safest* or most *human-preferred*.  

- **Key Insight**  
  - Humans implicitly weigh **comfort, safety, and social norms** when moving across mixed terrain.  
  - A robot should encode *human-like* terrain preferences, not just obstacle avoidance.

## Proposed Solution

- **Human-Preference Terrain-Aware Navigation**  
  - Combine semantic understanding and elevation with explicit human-style cost functions to plan routes that match how people naturally choose paths.

## Pipeline Components

1. **Bird’s-Eye-View (BEV) Semantic Terrain Map**  
   - Monocular RGB camera → semantic segmentation → inverse perspective mapping (IPM).  
   - Produces a top-down map where each pixel encodes terrain category (road, grass, sidewalk, snow, etc.).  

2. **Human-Preference Terrain Cost Map**  
   - Fuse BEV semantics with learned/heuristic *preference weights* (e.g., sidewalk < dry grass < snow).  
   - Creates a spatial cost layer for planning that reflects human choices, not just free vs. occupied space.

3. **Elevation Mapping** *(in progress)*  
   - LiDAR/stereo depth → 3-D elevation grid.  
   - Detects slopes, curbs, drop-offs and fuses with the semantic cost map.

4. **Model Predictive Control (MPC) + Control Barrier Functions (CBF)** *(in progress)*  
   - Plans and tracks safe, dynamically-feasible trajectories while enforcing barrier constraints to stay on preferred terrain and avoid unsafe regions.

## Difference from Conventional Traversability / Occupancy-Map Approaches

| **Conventional Occupancy / Traversability Map** | **This Work** |
|-------------------------------------------------|---------------|
| Represents terrain as **binary or scalar traversability** (free / occupied, or cost from slope, roughness). | Represents terrain with **rich semantics** (sidewalk, snow, grass, asphalt) and **human preference weights**. |
| Cost usually based on **physical feasibility** (friction, slope). | Cost explicitly encodes **social and comfort preferences** (e.g., prefer sidewalk even if grass is traversable). |
| Typically uses 3-D LiDAR or stereo to estimate elevation and obstacles. | Adds **camera-based semantic BEV + elevation fusion**, so the map is both geometrically and semantically aware. |
| Goal: ensure the robot can drive anywhere it is physically able. | Goal: ensure the robot moves **where a person would choose**, balancing safety, stability, and human-like norms. |

## Main Contribution / Innovation

- **A unified perception-to-control framework for *human-preference* terrain navigation** that:  
  - Generates a **camera-only BEV semantic map**, enabling low-cost sensors to achieve rich terrain understanding.  
  - **Fuses semantic terrain categories with elevation** to build a **human-desirability cost map**, beyond standard traversability metrics.  
  - Integrates **Model Predictive Control with Control Barrier Functions** to guarantee safety while following human-like routes.  
- Demonstrates that robots can plan and track **human-preferred** paths in semi-structured outdoor environments—something traditional occupancy-grid or slope-based planners do not address.

## Implementation Status

- ✅ **Completed**  
  - BEV generation from monocular RGB (segmentation + IPM).  
  - Human-preference terrain cost map construction.

- 🔄 **In Progress**  
  - Elevation mapping and fusion.  
  - MPC + CBF control.

## Planned Platform & Simulator

- **Platform:** Legged or high-mobility ground robot (e.g., ANYmal, Unitree, Spot, Clearpath Husky) for mixed terrain.  
- **Simulator:** **Isaac Sim** for photorealistic sensors, ROS 2 integration, and diverse terrain.

## Known Challenges in Isaac Sim

- Realistic terrain physics (friction/material tuning).  
- Quadruped contact stability (solver tuning).  
- Multi-sensor timing (camera/LiDAR/IMU).  
- GPU/CPU performance requirements.  
- Steep learning curve for USD asset creation and physics tuning.  
*(Mitigations: procedural scene generation, sensor calibration, multi-resolution elevation mapping, decoupled control loops.)*

## Proposed Timeline

**September**  
- Set up Isaac Sim and basic terrain scenes.  
- Calibrate RGB/LiDAR/IMU sensors.

**October**  
- Integrate BEV + cost-map pipeline into Isaac.  
- Implement and test elevation mapping.  
- Begin MPC + CBF controller.

**November**  
- Run experiments and ablation studies:  
  - Segmentation backbone comparisons.  
  - With/without elevation fusion.  
  - MPC vs. MPC+CBF.  
  - Human-preference vs. shortest-path planners.

**Late November – December**  
- Finalize results, visuals, and quantitative metrics.  
- Draft and polish the conference paper.
