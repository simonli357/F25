# Wait, That Feels Familiar: Learning to Extrapolate Human Preferences for Preference-Aligned Path Planning — Review

## 1. Citation Information

```bibtex
@inproceedings{karnan2024patern,
  title        = {Wait, That Feels Familiar: Learning to Extrapolate Human Preferences for Preference-Aligned Path Planning},
  author       = {Haresh Karnan and Elvin Yang and Garrett Warnell and Joydeep Biswas and Peter Stone},
  booktitle    = {Proceedings of the 2024 IEEE International Conference on Robotics and Automation (ICRA)},
  year         = {2024},
  month        = may,
  address      = {Yokohama, Japan},
  publisher    = {IEEE},
  doi          = {10.1109/ICRA57147.2024.10611475},
  url          = {https://elvout.github.io/patern/}
}
```

## 2. Clear Abstract

Robots often need to follow a human operator’s terrain preferences (e.g., prefer sidewalk over grass), but real-world lighting or novel surfaces can break vision-only systems. The paper proposes **PATERN**, which learns how different terrains *feel* from the robot’s own inertial, joint, and foot-contact signals. When the camera sees a new-looking surface, PATERN finds the most similar *feel* in this learned proprioceptive space and **extrapolates** the operator’s known preference from that similar terrain. It then fine-tunes the visual model with these extrapolated labels, so the planner can choose paths that match operator preferences—even on novel terrains or at night. Experiments on a Spot robot outdoors show improved preference-aligned navigation and robustness to lighting and unseen terrains.

## 3. Keywords

- Preference-aligned navigation  
- Proprioception-guided extrapolation  
- Terrain representation learning  
- Cross-modal nearest-neighbor transfer  
- Out-of-distribution terrain adaptation  
- Visual encoder fine-tuning  
- Legged robot (Boston Dynamics Spot)  
- Off-road navigation  
- Preference utility learning  
- Homography BEV patch extraction

## 4. Problem Addressed

How to keep robot navigation **aligned with human terrain preferences** when the robot encounters **visually novel** surfaces or strong **appearance shifts** (e.g., day→night) without constantly recollecting/labeling data or relying on brittle hand-coded costs.

## 5. Innovation / Main Contribution

- **Preference Extrapolation via Proprioception:** Learn a proprioceptive latent space and, for novel visual terrains, **transfer** the preference of the nearest known proprioceptive cluster—no new human labels at deployment time.  
- **Self-supervised adaptation loop:** Use the extrapolated labels to **retrain** the visual encoder and preference utility, yielding a post-adaptation model (**PATERN+**) that handles new terrains/lighting.  
- **Cross-modal novelty trigger:** Detect visual OOD by the **mismatch** (MSE) between visual and proprioceptive utilities.

## 6. Key Methods / Architecture

- **Two-step cost model for planning**  
  - Visual encoder (CNN) → visual utility function.  
  - Proprioceptive encoder (MLP) → proprioceptive utility function.  
- **Novelty detection:** Visual–proprio utility mismatch flags OOD.  
- **Preference extrapolation:** Nearest-neighbor transfer in proprio space.  
- **Retraining (PATERN+):** Fine-tune visual encoder and utility with extrapolated labels.  
- **Planning stack:** GRAPHNAV with preference cost term.

## 7. Datasets & Experimental Setup

- **Robot & Sensors:** Spot with RGB, LiDAR, IMU, joint sensors.  
- **Known terrains:** concrete, grass, marble rocks.  
- **Novel conditions:** pebble pavement, bushes, shadows, night.  
- **Planner:** GRAPHNAV with added preference utility.

## 8. Evaluation Metrics & Results

- **Metrics:** Hausdorff distance to human reference path; % preference alignment.  
- **Findings:** PATERN+ succeeds on OOD terrains & night, approaching fully supervised performance with fewer labels.  
- **Large-scale test:** 3-mile trail hike with 1 manual intervention.

## 9. Limitations

- Relies on proprio similarity; fails if no known analog.  
- Requires physical traversal (potentially risky).  
- Focused on flat-ish terrains.

## 10. Similarity / Relevance to Our Work

- **Overlap:** Both aim for terrain-aware cost integration.  
- **Differences:** PATERN lacks explicit semantics/physical properties, requires contact.  
- **Usefulness:** Their OOD handling & preference extrapolation could complement our unified framework.

## 11. Potential Integration Points

- Add PATERN-style preference utility as map layer.  
- Use cross-modal utility mismatch as OOD detector.  
- Few-shot proprio “probes” to adapt visual models.  
- Combine with semantic layers to avoid mis-labels.  
- Integrate into MPC cost shaping.

## 12. Open Questions / Future Work

- Extrapolation when no proprio analog exists.  
- Avoiding unsafe contact (vision→proprio distillation).  
- Extending to wheeled robots & urban terrains.  
- Contextual preferences (conditional utilities).

## 13. Reproducibility Notes

- Architectures and training details given.  
- Open-source GRAPHNAV base; project site and video provided.  
- No public code/dataset for PATERN itself (reproduction requires Spot).

## 14. Link

- [Project page](https://elvout.github.io/patern/)  
- [YouTube demo](https://youtu.be/j7159pE0u6s)  
- DOI: 10.1109/ICRA57147.2024.10611475

## 15. Recap

- Collect data on known terrains + preferences.  
- Train visual & proprio encoders with utilities.  
- At deployment, detect OOD via cross-modal mismatch.  
- Extrapolate preferences to novel terrains via proprio similarity.  
- Fine-tune visual encoder with extrapolated labels → PATERN+.  
- Plan paths with preference-aware GRAPHNAV.  
- Evaluate: strong performance day/night, novel terrains, long trail test.

---

### ✅ **PATERN Capability Checklist**

| Category | Uses / Has? | Explanation |
|-----------|--------------|-------------|
| **Terrain semantics** | ❌ No | Does not use explicit terrain class labels or semantics (e.g., “grass,” “road”) for planning—only implicit latent features. |
| **Physical properties** | ✅ Yes | Learns from inertial, tactile, and joint signals—captures terrain *feel* (stiffness, vibration, traction). |
| **Geometric properties / Elevation** | ⚠️ Partial | Incorporates geometric costs via GRAPHNAV (obstacle avoidance, goal distance), but not explicit 3D elevation modeling. |
| **Proprioception** | ✅ Yes | Central to PATERN—proprioceptive signals drive the preference extrapolation and latent-space similarity search. |
| **Planning or control loop** | ✅ Yes | Implements a planning loop using GRAPHNAV, integrating preference utility into the cost function. |
| **Classical mapping** | ⚠️ Partial | Uses classical navigation (GRAPHNAV) but not mapping in the SLAM or topological sense; only local cost-based planning. |
| **Learning-based mapping** | ✅ Yes | Visual and proprioceptive encoders learn terrain representation spaces used for preference mapping. |
| **Needs LiDAR** | ⚠️ Partial | LiDAR used for geometric obstacle detection, but not essential for preference learning; could function with RGB+IMU. |
| **Works on quadrupeds** | ✅ Yes | Experiments run on Boston Dynamics **Spot**, demonstrating proprioceptive legged locomotion compatibility. |

