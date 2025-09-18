# Probabilistic Traversability Model for Risk-Aware Motion Planning in Off-Road Environments

## 1. Citation Information

``` bibtex
@inproceedings{cai2023probabilistic,
  title     = {Probabilistic Traversability Model for Risk-Aware Motion Planning in Off-Road Environments},
  author    = {Xiaoyi Cai and Michael Everett and Lakshay Sharma and Philip R. Osteen and Jonathan P. How},
  booktitle = {Proceedings of the 2023 IEEE/RSJ International Conference on Intelligent Robots and Systems (IROS)},
  year      = {2023},
  organization = {IEEE},
  url       = {https://github.com/mit-acl/mppi_numba}
}
```

## 2. Clear Abstract

This paper introduces a framework for off-road robot navigation that
explicitly models uncertainty in terrain traction. Instead of assuming
constant or expected traction, the authors learn the full empirical
distribution of traction parameters for a unicycle vehicle model using
self-supervised data. Two risk-aware motion planning costs are proposed:
(1) **CVaR-Cost**, which evaluates candidate trajectories against the
right-tail conditional value at risk of navigation cost across sampled
traction maps, and (2) **CVaR-Dyn**, which plans using the worst-case
expected traction values directly in the dynamics. To detect novel or
out-of-distribution terrains, a Gaussian Mixture Model is fitted to the
latent space of the trained neural network, providing a confidence score
that penalizes uncertain regions during planning. Simulated experiments
with both synthetic semantic environments and real-world traction data
show that the method improves navigation success rate and completion
time compared to approaches that assume no slip or use only expected
traction. Incorporating the density-based terrain confidence further
improves success by up to 30% when deployed in environments unseen
during training.

## 3. Keywords

-   Probabilistic traversability\
-   Risk-aware motion planning\
-   Conditional Value at Risk (CVaR)\
-   Off-road navigation\
-   Neural network traction modeling\
-   Out-of-distribution detection\
-   Gaussian Mixture Model (GMM)\
-   Model Predictive Path Integral (MPPI) control\
-   Self-supervised learning\
-   Unicycle dynamics

## 4. Problem Addressed

Robots operating in natural off-road environments face uncertain terrain
properties: visually similar surfaces can have drastically different
traction, leading to wheel slip and planning failures. Standard planners
either ignore this uncertainty or rely on nominal/expected traction
values, which cannot guarantee safe or efficient navigation when
traction distributions are non-Gaussian or poorly modeled.

## 5. Innovation / Main Contribution

-   **Empirical traction distribution**: Learns the full non-Gaussian
    distribution of terrain traction parameters conditioned on semantic
    and geometric features.\
-   **Risk-aware costs**: Introduces two CVaR-based cost functions to
    explicitly account for worst-case outcomes in motion planning.\
-   **Latent-space novelty detection**: Uses a GMM on neural network
    latent features to identify and avoid out-of-distribution terrains
    at runtime.\
-   **Improved robustness**: Demonstrates superior success rate and
    time-to-goal over methods assuming no slip or expected traction.

## 6. Key Methods / Architecture

-   Learn terrain-dependent traction distributions with a
    self-supervised neural network using camera and LiDAR-based semantic
    and elevation features.\
-   Store predicted traction distributions in a spatial map for
    planning.\
-   Estimate epistemic uncertainty by fitting a PCA-reduced latent-space
    Gaussian Mixture Model to detect out-of-distribution (OOD)
    terrains.\
-   Plan motions using Model Predictive Path Integral (MPPI) control
    with:
    -   **CVaR-Cost**: CVaR of trajectory costs from sampled traction
        maps.\
    -   **CVaR-Dyn**: CVaR of traction parameters used directly in the
        dynamics.

## 7. Datasets & Experimental Setup

-   **Training Data**: Collected by manually driving a Clearpath Husky
    in a forest while generating a semantic octomap from RGB + LiDAR,
    and estimating ground truth velocities using LiDAR odometry and
    IMU.\
-   **Simulated Environments**: Grid-world semantic maps with "dirt" and
    "vegetation" cells having known traction distributions; vegetation
    patches introduce high-variance traction.\
-   **Test Environment**: Distinct forest dataset for generalization
    experiments.

## 8. Evaluation Metrics & Results

-   **Metrics**: Navigation success rate (goal reached within time
    limit), time-to-goal.\
-   **Results**:
    -   CVaR-based planners consistently outperform baselines that
        assume no slip or use expected traction.\
    -   CVaR-Dyn achieves similar success rates to CVaR-Cost with lower
        computation.\
    -   Density-based OOD detection improves success rate by up to 30%
        in unseen environments.\
    -   Auxiliary penalties for OOD terrain further reduce time-to-goal
        while maintaining success.

## 9. Limitations

-   **Sample Efficiency & Conservativeness**: CVaR-Cost is
    computationally expensive and can be overly conservative for low
    risk tolerances.\
-   **Hardware Validation**: Experiments are primarily in simulation
    with limited real-world robot trials.\
-   **Relation to Our Project**:
    -   The work focuses on traction uncertainty rather than *human
        preference* or explicit semantic elevation fusion.\
    -   It does not integrate bird's-eye semantic mapping or
        human-centric cost functions.\
    -   MPC + CBF safety constraints are not explored.

## 10. Similarity / Relevance to Our Work

-   **Overlap**:
    -   Shared interest in off-road navigation and terrain-aware
        planning.\
    -   Uses semantic and elevation features to inform
        traversability---similar inputs to our BEV semantic and
        elevation mapping.\
    -   Emphasizes risk-aware planning, which complements our MPC + CBF
        safety framework.\
-   **Divergence**:
    -   Focuses on stochastic traction modeling, not human comfort or
        social norms.\
    -   Does not encode explicit human-preference cost layers.

## 11. Potential Integration Points

-   Incorporate CVaR-based risk metrics into our MPC to handle uncertain
    elevation or traction estimates.\
-   Apply latent-space OOD detection to flag novel terrain types in our
    BEV semantic map.\
-   Use their self-supervised traction-learning pipeline to enrich our
    human-preference cost map with physical feasibility scores.

## 12. Open Questions / Future Work

-   Explore more sample-efficient CVaR optimization methods.\
-   Investigate alternative risk metrics beyond CVaR to reduce
    conservativeness.\
-   Integrate normalizing flow models for joint density and traction
    estimation.\
-   Conduct extensive hardware experiments for field validation.

## 13. Reproducibility Notes

-   **Code/Data**: GPU planners and example code released at
    <https://github.com/mit-acl/mppi_numba>.\
-   **Training Data**: Forest traction datasets used in experiments are
    described but not fully released.\
-   Implementation details (e.g., MPPI parameters, network architecture,
    CVaR computation) are clearly specified in the paper.

## 14. Link

[Paper & Project Page](https://github.com/mit-acl/mppi_numba)
