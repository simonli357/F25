### Traversability and Planning Feature Checklist

| Category | Present / Used | Notes |
|-----------|----------------|-------|
| **Uses/Has Terrain Semantics** | ✅ Yes | Semantic classifiers (e.g., PointRend on RUGD dataset) are used to identify terrain types like vegetation, grass, and mulch for traversability modeling. |
| **Uses/Has Physical Properties** | ✅ Yes | Models terrain traction as a physical property derived from commanded vs. achieved velocities; directly represents terrain–wheel interaction physics. |
| **Uses/Has Geometric Properties / Elevation** | ✅ Yes | Incorporates elevation maps and semantic octomaps for geometric context in traversability estimation. |
| **Uses/Has Proprioception** | ✅ Yes | Employs IMU and lidar odometry for proprioceptive sensing to estimate true motion and traction (accounts for wheel slip). |
| **Has Planning or Control Loop** | ✅ Yes | Implements a **Model Predictive Path Integral (MPPI)** control loop for risk-aware navigation with receding horizon planning. |
| **Uses/Has Classical Mapping** | ✅ Yes | Builds semantic octomaps and 2D elevation maps—classical mapping approaches fused with semantic and geometric data. |
| **Uses/Has Learning-Based Mapping** | ✅ Yes | Uses a neural network to learn traction distributions (self-supervised) and GMM-based latent-space confidence mapping. |
| **Needs LiDAR** | ✅ Yes | Relies on lidar for odometry, environment mapping, and semantic fusion; essential for both data collection and localization. |
| **Works on Quadrupeds** | ⚠️ Primarily Wheeled | Designed for wheeled platforms (unicycle model). Quadruped applicability discussed in related work but not directly implemented. |

