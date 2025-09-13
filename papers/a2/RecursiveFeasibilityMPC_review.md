# Review: *Control Invariant Sets for Neural Network Dynamical Systems and Recursive Feasibility in Model Predictive Control*

## 1. Citation Information
```bibtex
@misc{li2025cisnnds,
  title         = {Control Invariant Sets for Neural Network Dynamical Systems and Recursive Feasibility in Model Predictive Control},
  author        = {Xiao Li and Tianhao Wei and Changliu Liu and Anouck Girard and Ilya Kolmanovsky},
  year          = {2025},
  eprint        = {2505.11546},
  archivePrefix = {arXiv},
  primaryClass  = {eess.SY},
  url           = {https://arxiv.org/abs/2505.11546}
}
```

## 2. Clear Abstract (150–200 words)
The paper develops a practical way to construct **control invariant sets (CIS)** for discrete‑time systems whose dynamics are represented by **neural networks** (ReLU MLPs). The state space is quantized into hyperboxes and an iterative set‑recursion is applied to find a subset of the safe set that is guaranteed to be forward invariant under an admissible feedback policy. The core subproblems—one‑step returnability, set propagation through ReLU layers, and feasibility checks—are reduced to interval/box reachability and mixed‑integer linear constraints, which makes the overall procedure computationally tractable and provably **finite‑terminating**. The authors then embed the resulting CIS into a **model predictive control** (MPC) problem, yielding a controller that is **recursively feasible** and respects state and input constraints by construction. A lane‑keeping case study trains a compact NN model of the vehicle’s lateral dynamics and shows that the offline CIS synthesis converges in a manageable number of iterations, while the online mixed‑integer MPC solves in **millisecond‑scale** time and keeps the vehicle within lane and control bounds. The work demonstrates a bridge between NN modeling and certified safety through invariant sets.

## 3. Keywords
- Control invariant set (CIS)
- Neural network dynamical system (NNDS)
- State‑space quantization (SSQ)
- ReLU MILP/MILC encoding
- Recursive feasibility
- Model predictive control (MPC)
- Interval/box reachability
- Lane keeping

## 4. Problem Addressed
Guaranteeing **safety** (state/input constraints) and **recursive feasibility** for MPC when the system dynamics are modeled by neural networks—nonlinear models that are challenging to analyze and certify.

## 5. Innovation / Main Contribution
- A **finite‑termination** algorithm to synthesize CIS on a quantized state space for NN‑based dynamics.
- **Mixed‑integer encodings** for ReLU networks to propagate set bounds and verify one‑step returnability.
- Integration of the CIS as **hard constraints** in MPC, providing **recursive feasibility** with warm‑starts from the CIS‑induced policy.
- Empirical validation on a lane‑keeping task with compact NN dynamics and real‑time feasible solve times.

## 6. Key Methods / Architecture
- **Closed‑loop NNDS model**: ReLU MLP takes state (and possibly control) as input.
- **State‑space quantization**: partition safe set into hyperboxes; represent sets as unions of boxes.
- **Set recursion**: iterate one‑step returnability/containment over boxes until a fixed point yields the CIS.
- **ReLU reachability via MILP/MILC**: decision‑dependent interval bounds and mixed‑integer constraints to over‑approximate NN mapping.
- **MPC formulation**: MIQP/MILP with CIS constraints; warm‑start using the CIS policy to improve solve time.

## 7. Datasets & Experimental Setup
- **Task**: simulated **lane‑keeping** at fixed speed with steering control.
- **Dynamics model**: compact NN (small MLP) trained on data generated from a bicycle‑type vehicle model.
- **Environment**: offline CIS construction using mathematical programming; online MPC solved with a commercial MI(L)P solver (e.g., YALMIP + Gurobi), achieving real‑time feasibility on desktop‑class hardware.
- **Safety set**: lane boundaries and steering limits expressed as box constraints.

## 8. Evaluation Metrics & Results
- **CIS synthesis** converges in a limited number of iterations on the chosen grid resolution.
- **Online MPC** solves on the order of **milliseconds per step**, stays within lane and control bounds, and keeps the closed‑loop trajectory inside the CIS.
- Case study plots show accurate tracking and constraint satisfaction under the learned NN dynamics.

## 9. Limitations (relative to our project on human‑preference, terrain‑aware navigation with BEV semantic maps, elevation mapping, and MPC + CBF)
- Evaluated on a **low‑dimensional lane‑keeping** problem; does not address high‑dimensional perception (BEV semantics) or 3‑D **elevation** reasoning.
- Assumes a reliable NN dynamics model; **model uncertainty** and disturbances are not explicitly handled.
- Mixed‑integer formulations may scale poorly with larger horizons/networks typical of full perception‑to‑control stacks.
- Does not include **CBF**-style continuous‑time guarantees; safety is enforced via invariant‑set constraints within MI(L)P.

## 10. Similarity / Relevance to Our Work
- Shares our emphasis on **safety‑aware** MPC and certified constraint satisfaction.
- Complements our focus on **human‑preference** and **terrain semantics** by supplying a dynamics‑side **safety envelope** (CIS) we could enforce while optimizing human‑preferred costs.

## 11. Potential Integration Points
- Use a **CIS as a safety envelope** in the reduced lateral‑dynamics space while our planner optimizes a human‑preference cost over BEV/elevation maps.
- **Warm‑start** our MPC using a feasible policy derived from the CIS to improve reliability and latency.
- Treat **BEV cells** as discrete boxes and explore CIS‑style set recursion over a coarser terrain state (e.g., curb/sidewalk regions) to define certified “stay‑on‑preferred‑terrain” sets.
- Reuse **ReLU MILP encodings** if we adopt compact NN surrogates for dynamics or cost‑to‑go.

## 12. Open Questions / Future Work (from the paper)
- Scaling CIS synthesis to **higher‑dimensional** systems.
- Incorporating **uncertainty and disturbances** into the invariant‑set construction.
- Extending beyond lane‑keeping to richer tasks and control objectives.

## 13. Reproducibility Notes
- Paper provides detailed formulations and a lane‑keeping case study; **no public code repository** is linked.
- Uses standard toolchains (e.g., MATLAB/YALMIP and Gurobi for MILP; PyTorch for NN training), suggesting moderate effort to replicate with similar solvers.

## 14. Link
- Paper: https://arxiv.org/abs/2505.11546