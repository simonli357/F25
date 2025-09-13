# Guaranteed Obstacle Avoidance for Multi-Robot Operations With Limited Actuation: A Control Barrier Function Approach

## 1. Citation Information
```bibtex
@article{chen2021guaranteed,
  title={Guaranteed Obstacle Avoidance for Multi-Robot Operations With Limited Actuation: A Control Barrier Function Approach},
  author={Yuxiao Chen and Andrew Singletary and Aaron D. Ames},
  journal={IEEE Control Systems Letters},
  volume={5},
  number={1},
  pages={127--134},
  year={2021},
  publisher={IEEE},
  doi={10.1109/LCSYS.2020.3000748}
}
```

## 2. Clear Abstract
This paper presents a decentralized method for ensuring collision-free operation of multiple robots that have limited actuation. Using **control barrier functions (CBFs)** combined with a “backup strategy,” each robot can guarantee obstacle avoidance without requiring centralized coordination. The approach applies to general nonlinear dynamics and scales to many agents. Each robot computes a control barrier function from a simple trajectory that leads to a stable equilibrium, checking if this trajectory remains safe. If it does, the CBF supervisor ensures the robot stays in a safe set for all time. A simple broadcasting mechanism allows multiple “backup strategies” per robot to improve mobility. Experiments on ground robots (Dubins car model) and high-dimensional quadrotor simulations demonstrate the method’s ability to maintain safety while allowing decentralized control and limited communication.

## 3. Keywords
- Control Barrier Functions (CBF)  
- Decentralized multi-robot control  
- Obstacle avoidance  
- Limited actuation / kinodynamic planning  
- Backup strategy  
- Quadratic programming (QP)  
- Dubins car  
- Quadrotor dynamics  
- Multi-agent systems  
- Safety-critical control

## 4. Problem Addressed
How to **guarantee obstacle avoidance** for multiple robots with **limited actuation** and **general nonlinear dynamics**, without centralized control or heavy communication, while ensuring scalability to large teams.

## 5. Innovation / Main Contribution
- **Backup-strategy CBF:** Constructs a control barrier function from a trajectory that converges to a safe equilibrium, ensuring a provably invariant safe set.
- **Decentralized feasibility:** Robots need only neighbor states; no global communication is required.
- **Multiple backup strategies with broadcasting:** Enhances mobility by allowing robots to switch among several safe maneuvers while maintaining safety guarantees.

## 6. Key Methods / Architecture
- Formulate system dynamics: \( \dot{x} = f(x,u) \) with limited actuation.
- Define safe set \( C \) and small control-invariant subset \( S_0 \).
- **Backup strategy:** Predefined controller bringing each agent to a stable equilibrium.
- Construct CBF \( h(x) \) via forward simulation of the backup trajectory.
- Solve a **quadratic program (QP)** to minimally adjust a nominal controller while enforcing \( \dot{h} + \alpha(h) \ge 0 \).
- Decentralized implementation: each agent solves its own QP using neighbor states.
- Optional multi-backup strategy with a lightweight broadcast scheme.

## 7. Datasets & Experimental Setup
- **Robotarium experiments**: 3–6 differential-drive ground robots (Dubins car model) patrolling between waypoints.  
- **Quadrotor simulation**: 17-dimensional ROS/C++ simulator with realistic quadrotor dynamics and PD backup controllers.

## 8. Evaluation Metrics & Results
- **Safety guarantees:** Zero collisions in all trials.  
- **Scalability:** Works with up to 6 robots and high-dimensional quadrotor models.  
- **Feasibility:** QP always solvable when initial conditions satisfy CBF condition.  
- **Mobility:** Multi-backup strategy allows swerving maneuvers while avoiding obstacles.

## 9. Limitations
- **No terrain semantics or elevation reasoning:** Focuses on collision avoidance, not human-preferred terrain selection.  
- **Backup strategy design critical:** Performance depends on well-chosen backups.  
- **Finite sampling / discretization:** Requires robustness to discretization of continuous constraints.  
- Does not address **human preference costs** or **BEV semantic mapping**, so integration with our BEV-based terrain-aware navigation would require additional perception layers.

## 10. Similarity / Relevance to Our Work
- **Overlap:**  
  - Uses **Control Barrier Functions (CBFs)** and a QP-based MPC-like supervisory layer—directly relevant to our planned **MPC + CBF** safety module.  
  - Decentralized multi-agent safety can inspire our approach if we deploy multiple robots.
- **Difference:**  
  - Their focus is pure obstacle avoidance with minimal dynamics, not human-preference or terrain semantics.  
  - No BEV semantic map, elevation mapping, or human-like cost functions.

## 11. Potential Integration Points
- Apply their **backup strategy-based CBF** to guarantee collision avoidance in our MPC controller.  
- Use their decentralized QP formulation for multi-robot scenarios in shared outdoor environments.  
- Borrow their method for ensuring feasibility of CBF constraints under limited actuation.

## 12. Open Questions / Future Work
- How to select or learn effective backup strategies automatically.  
- Extensions to dynamic or moving obstacles with complex terrain.  
- Combining CBF safety with high-level preference-based planning (as in our human-preference navigation).  
- Real-world validation beyond small robot teams.

## 13. Reproducibility Notes
- **Code availability:** Quadrotor simulation code released at [https://github.com/DrewSingletary/uav_sim_ros](https://github.com/DrewSingletary/uav_sim_ros).  
- Detailed mathematical formulations and solver details (OSQP) provided.  
- Robotarium experiments reproducible through public Robotarium testbed.

## 14. Link
[IEEE Xplore Paper](https://doi.org/10.1109/LCSYS.2020.3000748)
