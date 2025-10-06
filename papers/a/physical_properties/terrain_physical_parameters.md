## Capability Checklist

| Category | Uses/Has | Notes |
|-----------|-----------|-------|
| **Terrain semantics** | ❌ | The system does not classify terrain semantically (e.g., grass, rock); it focuses on physical properties instead. |
| **Physical properties** | ✅ | Explicitly predicts *friction* and *stiffness* maps (physical parameters). |
| **Geometric properties / elevation** | ✅ | Uses local height samples around each foot for decoder input; geometry indirectly included through DINOv2 features. |
| **Proprioception** | ✅ | Physical decoder trained on proprioception (joint/body states, leg phase, commands). |
| **Planning or control loop** | ✅ | Contains a closed learning–control loop: decoder informs real-world motion; online Mission Graph updates visual model during operation. |
| **Classical mapping** | ✅ | Mission Graph stores spatial data (poses, features, labels) → forms a structured map used for online updates. |
| **Learning-based mapping** | ✅ | Vision network learns dense mapping (DINOv2 + MLP) from vision to friction/stiffness; online self-supervision refines it. |
| **Needs LiDAR** | ❌ | Uses only camera (vision) and proprioception; LiDAR not mentioned. |
| **Works on quadrupeds** | ✅ | Implemented and validated on ANYmal D quadruped robot. |

