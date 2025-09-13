# TMPU: A Framework for Terrain Traversability Mapping and Planning in Uneven and Unstructured Environments — Structured Review
fileciteturn0file0

---

## 1. Citation Information

```bibtex
@inproceedings{Bi2023TMPU,
  author    = {Qingchen Bi and Xuebo Zhang and Shiyong Zhang and Zhangchao Pan and Runhua Wang},
  title     = {TMPU: A Framework for Terrain Traversability Mapping and Planning in Uneven and Unstructured Environments},
  booktitle = {Proceedings of the 42nd Chinese Control Conference (CCC)},
  address   = {Tianjin, China},
  year      = {2023},
  pages     = {4685--4691},
  publisher = {IEEE}
}
```

---

## 2. Clear Abstract (150–200 words)

This paper presents **TMPU**, an autonomous navigation framework for ground robots operating in uneven, unstructured terrain. TMPU first builds a 3D OctoMap using LiDAR-based SLAM, then converts it into a **low-resolution 2D occupancy map** through plane fitting and analysis. Each local plane is evaluated by **slope** and **roughness** thresholds (set to a robot’s capabilities) to label traversable cells. Planning proceeds in two stages: an **A\*** search on the 2D map generates a **high-resolution traversable corridor**, after which a global path is computed using an **improved lattice planner (“H-lattice”)** that explicitly checks inter-plane **height differences** to avoid raised and negative obstacles (e.g., drop-offs) that may be missed at low resolution. A standard local planner (ROS move_base) tracks the path. Simulation studies in Gazebo across slope, step, and uneven scenes using a Scout 2.0–like platform show substantially reduced planning time compared with a prior plane-fitting approach (PUTN), and safer behavior near cliffs and unknown areas. Reported average compute-time reductions range from **57.6% to 97.1%**, with qualitative improvements in path optimality and obstacle avoidance.

---

## 3. Keywords

- Terrain traversability mapping  
- Plane fitting & analysis  
- OctoMap / 3D SLAM (LOAM)  
- A* traversable corridor  
- H-lattice global planning  
- Negative/raised obstacle avoidance  
- ROS move_base local planning  
- Uneven & unstructured environments

---

## 4. Problem Addressed

Enable **safe and efficient navigation** for ground robots in **uneven, unstructured 3D environments** where naïve 2D planners or purely geometric methods either over-process point clouds or miss hazardous terrain features (e.g., cliffs, steps).

---

## 5. Innovation / Main Contribution

- **Tire-footprint–aware plane fitting**: fits planes sized to the projected **tire rectangle**, improving traversability assessment per wheel.  
- **Three-metric plane analysis**: **slope**, **roughness**, and **height**; uses slope/roughness for map labeling and height for planning safety checks.  
- **H-lattice planning**: modifies lattice expansion by **angle and height-difference thresholds** to avoid **negative** and **raised** obstacles not captured at low 2D resolution.  
- **Two-stage planning**: fast **A\*** on low-res map to form a **traversable corridor**, then high-res global planning within that corridor.

---

## 6. Key Methods / Architecture

- **Perception & Mapping**
  - LOAM-style **3D SLAM** → **OctoMap** environment representation.  
  - **Plane fitting (SVD)** over tire-footprint windows.  
  - **Metrics**: plane slope (s_p), roughness (r_p = d_max - d_min), and plane height (h_p).  
  - Thresholds (s_t, r_t) set per robot capability; traversable planes → **free** cells in a **2D occupancy map**.  
- **Planning**
  - **A*** on low-res 2D map → **high-res traversable corridor**.  
  - **H-lattice**: extend between lattice cells only if plane **angle diff** < threshold; then require **height diff** < threshold; otherwise reject.  
  - **Local planner**: ROS **move_base**–compatible controller for path tracking; alternatives (e.g., TEB) possible.

---

## 7. Datasets & Experimental Setup

- **Simulator**: Gazebo; **scenes**: two uneven terrains, one slope, one step.  
- **Robot**: Scout 2.0–style ground robot; modeled as **two-wheel differential** for experiments.  
- **Software**: ROS Melodic; OctoMap; LOAM-based SLAM.  
- **Hardware**: Laptop with **Intel i7-11800H**, **16 GB RAM**.  
- **Baselines**: **PUTN** (plane-fitting uneven terrain navigation).

---

## 8. Evaluation Metrics & Results

- **Compute time** (terrain assessment + global planning):  
  - Scene 1: 618.34 ms (PUTN) → **262.13 ms** (TMPU) (**57.6%** ↓)  
  - Scene 2: 479.51 ms → **141.97 ms** (**70.4%** ↓)  
  - Slope: 536.85 ms → **15.68 ms** (**97.1%** ↓)  
  - Step: 372.41 ms → **143.22 ms** (**61.5%** ↓)  
- **Qualitative safety**: avoids **cliffs** and **negative obstacles** where PUTN may fail; produces **smoother** and more **optimal** paths within the corridor.

---

## 9. Limitations (relative to our BEV-semantics + Elevation + MPC+CBF project)

- **No semantic understanding**: traversability uses only **slope/roughness**, not terrain classes or human preference (e.g., sidewalk vs. grass).  
- **LiDAR-centric**: depends on LOAM + OctoMap; lacks **camera-only BEV** mapping.  
- **Heuristic thresholds**: (s_t, r_t) and height/angle thresholds are **manually set**, not learned or preference-aware.  
- **Planner form**: lattice + move_base, not **MPC + CBF** with formal safety guarantees.  
- **Evaluation scope**: **simulation-only** on Gazebo; no real-world trials, no human-preference metrics.

---

## 10. Similarity / Relevance to Our Work

- **Overlap**: focus on **uneven terrain safety**, explicit handling of **negative/raised obstacles**, and a clear **elevation/height-discontinuity** signal—useful for our **elevation map** fusion and CBF constraints.  
- **Divergence**: they optimize **geometric traversability and speed**, while our aim is **human-preference terrain costs** with **BEV semantics** and **MPC+CBF** control.

---

## 11. Potential Integration Points

- **Height-discontinuity guard**: incorporate TMPU’s **plane-height difference check** as a **CBF** (hard constraint) to prevent curb/edge descent even if the semantic cost is low.  
- **Corridor-first planning**: use A\*-derived **traversable corridors** as **guidance tubes** for our **MPC** to reduce search volume and improve real-time performance.  
- **Tire-footprint plane windows**: adopt **wheel-scale surface checks** to derive **local safety margins** for contact-relevant CBFs.  
- **Unknown-area handling**: their rule to treat a goal-adjacent **unknown** as potentially **negative obstacle** can be adapted as a **risk prior** in our cost map.  
- **Hybrid map**: fuse **elevation jumps** (from LiDAR/stereo) with our **BEV semantics** to down-weight desirability near potential drop-offs.

---

## 12. Open Questions / Future Work (from the paper)

- Quantify **threshold selection** (slope/roughness/height) and sensitivity across platforms.  
- Extend from **simulation to real-world** deployments with sensor noise and occlusion.  
- Compare against **learning-based** traversability or **semantic** planners.  
- Provide **formal safety** or stability guarantees beyond heuristic checks.

---

## 13. Reproducibility Notes

- **Code/data**: not provided.  
- **Implementation details**: ROS Melodic, LOAM, OctoMap, Gazebo scenes, and hardware specs are given; H-lattice rules are described at a high level (angle/height thresholds) but no parameter values or code. **Moderate effort** would be required to re-implement faithfully.

---

## 14. Link

- **Supplementary video** (from the paper): https://youtu.be/rvYoyR3imEk
