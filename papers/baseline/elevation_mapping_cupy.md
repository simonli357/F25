# Elevation Mapping for Locomotion and Navigation using GPU — Structured Review

## 1. Citation Information

```bibtex
@article{miki2022elevationmappingcupy,
  title   = {Elevation Mapping for Locomotion and Navigation using GPU},
  author  = {Takahiro Miki and Lorenz Wellhausen and Ruben Grandia and Fabian Jenelten and Timon Homberger and Marco Hutter},
  journal = {arXiv preprint arXiv:2204.12876},
  year    = {2022},
  url     = {https://github.com/leggedrobotics/elevation_mapping_cupy}
}
```

## 2. Clear Abstract

The paper introduces a fast, GPU-accelerated elevation-mapping pipeline for ground/legged robots. It turns depth sensor point clouds and robot pose into a 2.5D height map and adds practical features—like height drift compensation, visibility cleanup via ray casting, an upper-bound height layer, overlap clearance for multi-floor scenes, and optional filters (inpainting, smoothing, plane segmentation). A lightweight CNN computes per-cell geometric traversability on-GPU. The system runs in real time on embedded platforms and has been validated in extensive hardware experiments, including deployments during the DARPA Subterranean Challenge. The implementation is open source and ROS-integrated.

## 3. Keywords

- Elevation mapping (2.5D)
- GPU / CuPy acceleration
- Ray casting & visibility cleanup
- Height drift compensation
- Upper-bound layer (virtual surfaces)
- Geometric traversability CNN
- ROS / GridMap integration
- Legged locomotion (ANYmal)
- DARPA SubT deployment
- Real-time mapping on embedded compute

## 4. Problem Addressed

How to generate high-quality, real-time elevation maps from dense point clouds for rough-terrain navigation and legged locomotion, while mitigating practical issues (state-estimate drift, overhangs, dynamic obstacles, multi-level artifacts) that degrade map utility for planning and control.

## 5. Innovation / Main Contribution

- GPU-native pipeline (CuPy + custom CUDA kernels) for point transformation, height updates, ray casting, normals, and a CNN traversability head—all on device memory.
- Robustness features: height drift compensation, exclusion zones to reject ceilings/overhangs, visibility cleanup at sensor rate, overlap clearance near the robot, and an upper-bound height layer to reason about occlusions.
- Locomotion-oriented post-processing: inpainting, smoothing, plane segmentation; seamless ROS/GridMap publication at user-defined rates.
- Extensive real-world validation incl. DARPA SubT; open-sourced implementation.

## 6. Key Methods / Architecture

- Inputs: depth/ LiDAR point clouds + robot pose (SLAM/odometry).
- On-GPU steps:
  - Transform points to map frame and compute z-error for drift.
  - Kalman-style cell update with distance-dependent sensor noise; Mahalanobis outlier rejection.
  - Ray casting for visibility cleanup and upper-bound computation.
  - Exclusion region to ignore overhangs near the robot.
  - Overlap clearance to erase stale upper/lower floor cells near the robot.
  - Per-cell traversability CNN (lightweight, PyTorch on the same GPU memory).
  - Surface normals and optional filtering.
- On-CPU: publish GridMap layers at a chosen rate; optional additional filters for locomotion planners.

## 7. Datasets & Experimental Setup

- Platforms: ANYmal quadruped robots.
- Sensors: Two RoboSense RS-Bpearl LiDARs or Intel RealSense depth cameras.
- Environments: Rough terrain and subterranean settings (DARPA SubT).
- Baselines: CPU elevation mapping (Fankhauser et al. 2018) for timing/quality comparisons.

## 8. Evaluation Metrics & Results

- Throughput / latency:
  - Jetson Xavier example (~6.86 ms per frame for ~43k points with all features; traversability head ~4.10 ms dominates).
  - Update frequencies on Xavier: RealSense filtered ~49.4 Hz, RealSense raw ~16.1 Hz, Bpearl ~20.0 Hz.
- Quality (qualitative):
  - Drift compensation reduces elevation discontinuities.
  - Visibility cleanup at sensor rate removes “ghost walls” faster than baseline.
  - Exclusion handling prevents overhangs from becoming false obstacles.
- Use in autonomy: Provided traversability and upper-bound layers for local navigation; provided height samples/planes for locomotion controllers; successfully used in DARPA SubT missions.

## 9. Limitations

- 2.5D assumption: cannot fully represent multi-level structures; overlap clearance is a mitigation, not a full 3D solution.
- No semantic understanding: traversability is geometric; classes like grass/asphalt are not modeled.
- Local mapping focus: global consistency is out of scope (compared to global elevation mapping works).
- Reliance on pose quality: still sensitive to state-estimation errors; drift compensation is simple and height-only.
- Benchmarks: heavy emphasis on timing and qualitative maps; limited quantitative navigation/locomotion success metrics.

## 10. Similarity / Relevance to Our Work

Your goal: a unified mapping & navigation framework combining semantics, geometry, and terrain properties (e.g., friction/roughness) for traversability across urban + off-road settings.

**Overlap**
- Strong geometry layer via high-rate elevation mapping; upper-bound aids reasoning about occlusions; per-cell traversability (geometric) already integrated.
- Robust, field-tested pipeline that can be a foundation for real-time mapping on ANYmal.

**Divergence**
- Lacks semantic classes and learned physical properties (friction/roughness) layers central to your proposal.
- Focuses on rough terrain & subterranean domains rather than urban/semi-structured outdoor scenes.
- Traversability ≈ geometry driven, not unified multi-modal.

**Our limitations vs. this work**
- We don’t yet have a GPU-tight, ROS-native mapping backbone with proven drift/occlusion handling at ~50 Hz.
- Their engineering for artifact suppression (ray-casting cleanup, exclusion zones) is production-grade.

**How it informs us**
- Use this as the geometry & infrastructure layer into which we fuse semantics and terrain-property estimates.
- Borrow upper-bound and overlap clearance to reduce planner conservatism near occlusions and multi-floor artifacts.

## 11. Potential Integration Points

- Adopt elevation_mapping_cupy as our geometric base; keep height, variance, normals, upper-bound, traversability layers.
- Add new layers:
  - Semantic (e.g., sidewalk/grass/gravel/snow) by fusing a segmentation head; publish as categorical + confidence maps.
  - Physical properties (friction, roughness) from vision models (e.g., WVN-style) and/or proprioceptive estimators; fuse into per-cell scalar layers.
- Planner cost fusion: combine slope/roughness (from geometry), semantics preferences, and friction risk into a unified cost map.
- Urban dynamics: leverage visibility cleanup + upper-bound to safely navigate around cars/pedestrians in semi-structured scenes.
- Locomotion interface: expose smoothed planes/polygons to MPC; keep raw map for reactive safety checks.

## 12. Open Questions / Future Work

- Can the traversability head be extended to use multi-modal cues (semantics, friction priors) while keeping real-time performance?
- How to achieve globally consistent elevation maps without sacrificing throughput (e.g., sliding-window pose graph + GPU)?
- More principled drift compensation (full 6D, learned bias models) beyond height shifts.
- Quantitative navigation success metrics across diverse terrains (urban sidewalks/curbs/snow) to complement timing plots.
- Safety under extreme occlusions and moving obstacles—upper-bound helps, but how to reason probabilistically about unknowns?

## 13. Reproducibility Notes

- Code: open-sourced (ROS integration, CuPy/CUDA kernels; PyTorch traversability model).
- Hardware: runs on Jetson Xavier and desktop GPUs; parameters documented in the repo.
- Data: Primarily hardware deployments; no curated dataset release is indicated.
- Ease: High for geometry/traversability layers; replicating DARPA-like environments requires hardware access.

## 14. Link

- Project/Code: https://github.com/leggedrobotics/elevation_mapping_cupy
- Preprint: arXiv:2204.12876

## 15. recap — What the paper is doing, end-to-end

- Takes point clouds + robot pose and ships them straight to the GPU.
- Transforms points to a robot-centric map frame and computes a height error for drift compensation.
- Updates a 2.5D elevation map per cell using a Kalman-style fusion with sensor-distance noise.
- Runs ray casting from the sensor to each hit to:
  - Erase stale/dynamic obstacles (visibility cleanup).
  - Record an upper-bound on ground height in occluded cells (useful for planning).
- Applies exclusion geometry so near-field overhangs/ceilings aren’t falsely mapped as walls.
- Performs overlap clearance near the robot to remove artifacts from multi-level environments.
- Computes surface normals and runs a light CNN on-GPU to produce geometric traversability.
- Optionally applies inpainting/smoothing/plane segmentation for locomotion controllers.
- Publishes a ROS GridMap (multiple layers) at a user-defined rate to downstream planners/controllers.
- Demonstrates real-time performance on embedded hardware and validates in DARPA SubT missions.

---

### Mapping to Our Research Idea

- Use this paper’s GPU elevation mapping as our geometry backbone.
- Add semantic and friction/roughness layers (vision + proprioception) on top.
- Feed all layers into a unified planner (global + local) and MPC that respects slope, semantics, and physics (e.g., avoid low-friction snow, prefer sidewalks).
- The result: a holistic, terrain-aware navigation stack that generalizes from off-road to urban/semi-structured outdoor scenes—novel in integration and scope, building on this robust core.
