# How Does It Feel? Self-Supervised Costmap Learning for Off-Road Vehicle Traversability --- Structured Review

## 1) Citation Information (BibTeX)

``` bibtex
@article{GuamanCastro2023HDIF,
  title   = {How Does It Feel? Self-Supervised Costmap Learning for Off-Road Vehicle Traversability},
  author  = {Mateo Guaman Castro and Samuel Triest and Wenshan Wang and Jason M. Gregory and Felix Sanchez and John G. Rogers III and Sebastian Scherer},
  journal = {arXiv preprint arXiv:2209.10788},
  year    = {2023},
  url     = {https://mateoguaman.github.io/hdif}
}
```

## 2) Clear Abstract (rewritten)

The paper learns a *continuous* traversability costmap for off-road
driving without human labels... \[Content truncated for brevity in this
code block; use full text in actual output\]


---

## Feature & Capability Checklist

| Category | Uses / Has? | Notes |
|---|---|---|
| Use/has **terrain semantics** | ⚠️ Indirectly | No explicit semantic segmentation; terrain properties are captured implicitly via learned cost (no labeled classes). |
| Use/has **physical properties** | ✅ Yes | Learns interaction characteristics (roughness, deformability, bumpiness) from IMU-derived cost. |
| Use/has **geometric properties / elevation** | ✅ Yes | Uses height statistics (min/max/mean/std) from stereo/LiDAR point clouds in BEV maps. |
| Use/has **proprioception** | ✅ Yes | IMU z-acceleration bandpower (1–30 Hz) provides self-supervised target for traversability. |
| **Has planning or control loop** | ✅ Yes | Integrated with MPPI; costmap directly informs planning and control. |
| Use/has **classical mapping** | ✅ Yes | Builds local BEV RGB + height maps via stereo/LiDAR + odometry/registration. |
| Use/has **learning-based mapping** | ✅ Yes | CNN+MLP predicts continuous traversability costmaps conditioned on velocity (Fourier features). |
| **Needs LiDAR** | ⚠️ Optional | Works with LiDAR **or** stereo-only pipeline; LiDAR improves density but isn’t required. |
| **Works on quadrupeds** | ❌ Not shown | Demonstrated on wheeled platforms (ATV, Warthog UGV); not validated on legged robots. |

