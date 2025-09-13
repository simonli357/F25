# Report on *Interactive Multi-Modal Motion Planning with Branch Model Predictive Control*

## 1. Citation Information

``` bibtex
@inproceedings{chen2021branchMPC,
  title     = {Interactive Multi-Modal Motion Planning with Branch Model Predictive Control},
  author    = {Yuxiao Chen and Ugo Rosolia and Wyatt Ubellacker and Noel Csomay-Shanklin and Aaron D. Ames},
  booktitle = {Proceedings of the IEEE/RSJ International Conference on Intelligent Robots and Systems (IROS)},
  year      = {2021},
  note      = {arXiv preprint arXiv:2109.05128},
  url       = {https://github.com/chenyx09/belief-planning}
}
```

## 2. Clear Abstract

This work introduces **Branch Model Predictive Control (Branch MPC)**
for motion planning in environments shared with other, uncontrolled
agents whose reactions are uncertain and multi-modal. Traditional
planners often treat predictions of others' behavior as fixed, leading
to overly conservative or unsafe actions. Branch MPC instead enumerates
a finite set of possible feedback policies for the uncontrolled agent,
forming a *scenario tree* that captures multiple possible futures. The
ego robot then plans a *trajectory tree* with matching topology,
optimizing a feedback policy that adapts to whichever branch of behavior
actually occurs. To balance safety and performance under model
uncertainty, the authors incorporate **risk measures** such as
Conditional Value at Risk (CVaR), allowing the controller to smoothly
tune between risk-neutral and risk-averse behavior. The method is
demonstrated in simulation for autonomous highway driving (overtaking,
lane-change, and merging) and in real experiments on a quadruped robot
interacting with another robot. Results show human-like, proactive
behaviors such as "nudging" to influence others, while maintaining
robust safety.

## 3. Keywords

-   Branch Model Predictive Control (Branch MPC)\
-   Scenario Tree / Trajectory Tree\
-   Risk-Aware Planning (CVaR)\
-   Interactive Motion Planning\
-   Autonomous Vehicles\
-   Quadruped Robots\
-   Multi-Modal Behavior Prediction\
-   Sequential Quadratic Programming (SQP)\
-   Distributionally Robust Optimization

## 4. Problem Addressed

How can an autonomous robot safely and efficiently plan motions when
nearby agents (cars, pedestrians, other robots) respond in
unpredictable, multi-modal ways? Existing planners either ignore these
reactions or treat them as fixed, causing excessive conservatism or
unsafe plans.

## 5. Innovation / Main Contribution

-   **Branch MPC Framework:** Integrates continuous MPC with discrete
    multimodal behavior models via a branching scenario tree.\
-   **Risk-Aware Optimization:** Uses coherent risk measures (notably
    CVaR) to tune the trade-off between performance and robustness.\
-   **Practical Demonstrations:** Validated both in autonomous driving
    simulations and real quadruped-robot interactions, showing
    proactive, human-like behaviors.

## 6. Key Methods / Architecture

-   Represent uncontrolled agent behaviors as a **finite set of feedback
    policies**.\
-   Build a **scenario tree** by forward-propagating these policies.\
-   Solve for a **trajectory tree** (ego robot) with matching topology.\
-   Formulate an optimization that minimizes expected or CVaR-weighted
    cost over all branches.\
-   Implement solution using **Sequential Quadratic Programming** with
    automatic differentiation (CasADi).\
-   Provide risk-tuning parameter α for adjusting robustness.

## 7. Datasets & Experimental Setup

-   **Simulated Highway Driving:**
    -   Tasks: overtaking/lane change, merging.\
    -   Unicycle vehicle model; uncontrolled vehicle with discrete
        policies (maintain speed, slow down, lane change).\
-   **Quadruped Robot Experiment:**
    -   Ego: Unitree A1 with inverse-dynamics trot controller.\
    -   Uncontrolled: Ghost Robotics Vision 60 (tele-operated).\
    -   OptiTrack motion capture for precise state estimation.

## 8. Evaluation Metrics & Results

-   **Metrics:** Collision avoidance, trajectory cost (LQR-style),
    computation time.\
-   **Headline Findings:**
    -   Branch MPC achieved safe overtakes and merges where robust
        single-trajectory MPC remained stuck.\
    -   Risk parameter α controlled behavior: lower α (more risk-averse)
        led to conservative actions similar to robust MPC.\
    -   Real-time performance: \~80--100 ms per solve on standard CPUs.\
    -   Quadruped tests showed successful avoidance and goal reaching
        with multi-branch plans.

## 9. Limitations (vs. Human-Preference Terrain-Aware Navigation)

-   Assumes **finite policy set** for uncontrolled agents; scaling to
    many agents is exponential.\
-   Branching occurs at **fixed time intervals**, not event-triggered.\
-   No explicit **semantic terrain or elevation mapping**---focuses on
    agent interactions rather than terrain preferences.\
-   Uses simplified unicycle dynamics; not directly terrain-aware.\
-   Human preference modeling (comfort/social norms) is absent.

## 10. Similarity / Relevance to Our Work

-   **Overlap:**
    -   Uses MPC with Control-Barrier-like constraints for safety.\
    -   Risk-sensitive optimization is relevant to our MPC+CBF control
        layer.\
-   **Divergence:**
    -   Their emphasis is on *interaction with dynamic agents*, whereas
        our research focuses on *terrain semantics and human-preference
        cost maps*.\
    -   They do not incorporate BEV semantic or elevation mapping.

## 11. Potential Integration Points

-   Incorporate **branching scenario trees** to model *multiple
    human-preference hypotheses* (e.g., different pedestrian path
    preferences).\
-   Use **CVaR-based risk tuning** within our MPC to balance comfort
    vs. aggressive shortcuts.\
-   Sequential Quadratic Programming with automatic differentiation
    could improve solver speed for our elevation-aware MPC.

## 12. Open Questions / Future Work

-   Event-triggered, asynchronous branching.\
-   Incorporating **belief updates and observation models** for better
    uncertainty handling.\
-   Learning reactive models directly from data, especially for
    **multiple uncontrolled agents**.\
-   Combining branch MPC with **sampling-based planners** to avoid local
    minima.

## 13. Reproducibility Notes

-   **Code:** Publicly available at
    <https://github.com/chenyx09/belief-planning>.\
-   Simulation and experiment details (models, parameters) are fully
    described, making replication feasible with similar hardware.

## 14. Link

-   Paper: [arXiv:2109.05128](https://arxiv.org/abs/2109.05128)\
-   Project/Code: <https://github.com/chenyx09/belief-planning>
