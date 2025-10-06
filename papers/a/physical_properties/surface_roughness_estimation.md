# Surface Roughness Estimation for Terrain Perception — Structured Review
... (same as previous detailed markdown content above) ...


## Checklist: Method Properties (appended 2025-10-06)

| Category | Uses/Has? | Notes |
|---|---|---|
| use/has terrain semantics | ✅ Yes | Roughness task and annotations are **semantic-aware** and **edge-aware** (uses semantic masks alongside geometry). |
| use/has physical properties | ⚙️ Partial | Infers terrain **roughness** from vision/geometry; does not directly sense physical parameters like friction/tactile properties. |
| use/has geometric properties/elevation | ✅ Yes | Explicitly uses **depth, surface normals, point clouds**, and **plane coefficients** to model geometry/elevation. |
| use/has proprioception | ❌ No | The framework is **exteroceptive** (RGB/depth/point cloud); no proprioceptive sensing used. |
| has planning or control loop | ⚙️ Partial / Indirect | Outputs inform **traversability/foothold analysis**; the paper/framework itself does **not** implement a planning or control loop. |
| use/has classical mapping | ⚙️ Partial | Employs **classical plane fitting (LS/RANSAC/DSAC)** and surface reconstruction, but does not build a full 2D/3D map. |
| use/has learning-based mapping | ✅ Yes | **Learning-based** CNN/ResUNet modules produce pixel-wise roughness maps. |
| needs lidar | ❌ No | Designed for **RGB** and optionally depth/point clouds; **LiDAR is not required** (multi-modal support possible). |
| works on quadrupeds | ⚙️ Potential | Aimed at **legged robot** navigation (bipeds/quadrupeds); not demonstrated on a specific quadruped in this README. |

