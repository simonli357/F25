### GA-Nav Capability Checklist

| Category | GA-Nav Uses / Has? | Explanation |
|-----------|---------------------|--------------|
| **Uses/has terrain semantics** | ✅ **Yes** | Performs coarse-grained **semantic segmentation** of terrain into navigability groups (smooth, rough, bumpy, forbidden, obstacle, background). |
| **Uses/has physical properties** | ❌ **No** | Does **not** estimate explicit physical properties like friction, compliance, or texture beyond what’s implicit in RGB appearance. |
| **Uses/has geometric properties / elevation** | ⚠️ **Partially** | Incorporates **elevation information indirectly** by fusing its segmentation-based cost map with a separate **elevation cost map** from the TERP planner. The GA-Nav model itself operates purely on **RGB**. |
| **Uses/has proprioception** | ❌ **No** | No proprioceptive sensing (vibration, force, or odometry feedback) is used for terrain classification or navigation. |
| **Has planning or control loop** | ✅ **Yes** | Integrated with the **TERP** planner to generate trajectories; evaluated on real robots (Jackal, Husky). |
| **Uses/has classical mapping** | ✅ **Yes (via TERP)** | Uses classical **cost map** fusion: GA-Nav segmentation projected to ground plane via **homography**, then **added to elevation map** for planning. |
| **Uses/has learning-based mapping** | ✅ **Yes** | The segmentation itself is a **learning-based mapping** (deep neural network, transformer encoder–decoder). |
| **Needs LiDAR** | ⚠️ **Partially** | The GA-Nav segmentation network itself needs **only RGB**, but the full navigation system uses **LiDAR for localization and elevation map generation**. |
| **Works on quadrupeds** | ⚠️ **Potentially (not tested)** | Evaluated only on **Clearpath Jackal and Husky (wheeled)**; however, the approach is general and could apply to quadrupeds if camera geometry and projection are adapted. |

