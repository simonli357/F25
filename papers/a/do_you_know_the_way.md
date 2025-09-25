# Do You Know the Way? Human-in-the-Loop Understanding for Fast Traversability Estimation in Mobile Robotics

## 1) Citation Information

``` bibtex
@article{Schreiber2025DoYouKnowTheWay,
  title   = {Do You Know the Way? Human-in-The-Loop Understanding for Fast Traversability Estimation in Mobile Robotics},
  author  = {Andre M. Schreiber and Katherine Driggs-Campbell},
  journal = {IEEE Robotics and Automation Letters},
  volume  = {10},
  number  = {6},
  pages   = {5863--5870},
  year    = {2025},
  month   = {June},
  doi     = {10.1109/LRA.2025.3563819},
  url     = {https://doi.org/10.1109/LRA.2025.3563819},
  note    = {Code: https://github.com/andreschreiber/CHUNGUS}
}
```

## 2) Clear Abstract

The paper presents **CHUNGUS**, a human-in-the-loop (HiL) method for
visual **traversability estimation**. Instead of pre-labeling lots of
data or relying on risky self-supervision from robot experience, the
system detects when a camera image looks **novel** and quickly asks a
human for a few **sparse, relative annotations** (which of two pixels is
more/less/equally traversable). It builds on **DINOv2** features with
**FeatUp** to get dense per-pixel predictions, retrains its small head
online in seconds, and fuses predictions into an elevation map for
navigation. The approach achieves **state-of-the-art** prediction and
strong navigation performance in simulation and works well on real
images, despite very little human labeling.

## 3) Keywords

-   Traversability estimation\
-   Human-in-the-loop (HiL) learning\
-   Online/continual learning\
-   DINOv2 features & FeatUp upsampling\
-   Novelty / out-of-distribution detection (Faiss)\
-   Elevation mapping (BEV fusion)\
-   Model Predictive Path Integral (MPPI) control\
-   Sim-to-Real transfer\
-   Field robotics navigation

## 4) Problem Addressed

Robots in unstructured environments must know **where they can drive**.
Pure geometry or self-supervised vision approaches can miss nuances,
require risky data collection, or fail under **domain shift**. The work
addresses **fast, adaptable traversability learning during deployment**
using minimal, safe human input.

## 5) Innovation / Main Contribution

-   **HiL traversability with sparse, relative pixel labels** collected
    **on demand** when novelty is detected.\
-   **Foundation-model features (DINOv2) + FeatUp** to enable **rapid
    from-scratch retraining** of a lightweight MLP head in **seconds**
    while deployed.\
-   A **class-token Faiss novelty detector** that triggers annotation
    only for unfamiliar images.\
-   **End-to-end integration**: online learning → BEV fusion in an
    elevation map → **MPPI** navigation; validated in photorealistic sim
    and real data with **state-of-the-art** results.

## 6) Key Methods / Architecture

-   **Backbone**: DINOv2 ViT-S (frozen) extracts 14×14-patch features.
    **FeatUp** upsamples to per-pixel 384-D features; linear
    interpolation restores input resolution.\
-   **Head**: Small MLP (384→128→1 with sigmoid) outputs per-pixel
    traversability; side branch reconstructs features for
    **uncertainty/novelty** (MSE).\
-   **Loss**: LRIZZ on **pairwise ordinal labels** (−1/0/1) +
    reconstruction loss (γ=0.1).\
-   **Novelty detection**: DINOv2 **class token** indexed in **Faiss**
    (L2); request HiL labels if distance ≥ μ + ασ (α tunable).\
-   **Annotation policy**: for a novel image, collect (i) intra-image
    pair, (ii) cross-image pair (novel pixel with highest recon-error
    vs. known pixel with lowest).\
-   **Online learning**: cache all annotations and **retrain from
    scratch** quickly to avoid catastrophic forgetting; **\~0.1 s per
    retrain**, **7.7 Hz** inference on RTX-class GPU.\
-   **Mapping & Control**: fuse per-pixel traversability into
    **multi-modal elevation map** → **BEV** traversability layer →
    **MPPI** controller (unicycle model).

## 7) Datasets & Experimental Setup

-   **Simulator (Unreal Engine 4.26)** with photoreal assets;
    environments: **forest**, **warehouse**, **lot**, **dark lot**
    (night).\
-   **Robot**: Jackal-like diff-drive; camera images 240×424 (resized to
    224).\
-   **Training sets for novelty**: 4,000 sim images (train) + 858 (val)
    across the four environments.\
-   **Real-world dataset**: 7,342 images from Clearpath Jackal +
    RealSense D435i (three areas: semi-urban campus, park/grove, wooded
    tall-grass). Train 4,000 / val 3,342.\
-   **Controller**: MPPI sampling **N=1024** trajectories; mapping @10
    Hz; control @10 Hz.

## 8) Evaluation Metrics & Results

-   **Navigation success rate** and **median completion time** across 10
    trials per environment; analysis includes **labeling time** when HiL
    is active.\
-   **Headline findings**:
    -   **Big CHUNGUS** (pretrained on 4,858 sim images) **succeeds in
        all attempts**, outperforming others.\
    -   **HiL CHUNGUS (α=1 or 2)**: **higher success** than online
        **WVN** without pre-teleop, and **comparable to WVN(+5 min
        teleop)**---but with only seconds of quick HiL labels; α trades
        more labels for slightly slower runs.\
    -   **LiDAR occupancy** often misses low obstacles; **Blind**
        fails.\
    -   **Timing**: HiL pauses add \~**30--60 s**; **retrain ≈0.1 s**
        per update; total train time 1--2 s; **inference 7.7 Hz** vs
        **WVN 5.7 Hz**.\
-   **Novelty detection metrics**: report **Accuracy**, **F1**,
    **ROC-AUC**, **PR-AUC**; **L2 \> cosine** but both high.\
-   **Sim2Real**: **HDR** (Human Disagreement Rate) at thresholds
    {0.1,0.25,0.5}; sim-trained model shows only modest HDR increases on
    real data and even **lower HDR\_{0.5}** than real-trained in table.

## 9) Limitations

-   Requires **human annotator** availability; albeit seconds per label,
    it still pauses the robot.\
-   Traversability is **vision-only**; geometry enters via elevation
    mapping but no active 3D sensing in the head itself.\
-   Novelty is approximated via **class-token distance**; could misfire
    on subtle conditions or illumination shifts.\
-   Evaluations use a **photorealistic simulator** plus real images (no
    full, long-horizon real robot trials reported).

## 10) Similarity / Relevance to Our Work

**Overlap with our unified mapping & navigation idea** - Uses
**multi-modal elevation mapping** and fuses traversability into a **BEV
layer**---directly aligned with our multi-layer map concept.\
- Emphasizes **semantics-adjacent** visual cues (via DINOv2 features)
and **traversability prediction** for planning---matching our
terrain-aware cost-mapping.

**Divergences** - Focuses on **visual traversability**; does not
explicitly model **terrain physical properties** (friction/roughness) or
semantic classes as separate layers, which are central to our plan.

**What it teaches us** - **HiL on-demand labeling + online retraining**
can robustly handle **domain shift** without risky self-supervision.\
- **Relative, sparse labels** are highly efficient and sufficient for
strong performance---useful for bootstrapping our maps in new locales.\
- **FeatUp+DINOv2** gives crisp per-pixel predictions with tiny
trainable heads---ideal for **real-time** multi-layer mapping.

**Our limitations vs. this work** - We currently rely on integrating
multiple subsystems; this work shows an elegant, **minimal data** path
to practical traversability that we may not yet match in **online
adaptability**.

## 11) Potential Integration Points

-   **HiL trigger** in our stack: adopt their **Faiss class-token
    novelty detector** to request operator input only when needed.\
-   **Labeling scheme**: add **pairwise relative pixel labels** UI to
    quickly seed/adjust our traversability layer in unseen conditions.\
-   **Backbone & upsampling**: reuse **DINOv2 + FeatUp** for dense
    features across our **semantic** and **terrain-property** heads
    (friction, roughness).\
-   **Fusion**: inject their per-pixel traversability as an additional
    **cost map layer** combined with our **geometry + semantics +
    properties**.\
-   **Planner hook**: weight MPPI (or our MPC) with their traversability
    layer; prefer routes with high predicted μ while still respecting
    geometry and semantics.

## 12) Open Questions / Future Work

-   How to **reduce/automate** the remaining HiL effort (e.g., combine
    with **self-supervision**)?\
-   How well does HiL CHUNGUS scale across **more robots/platforms** and
    **long real deployments**?\
-   Could novelty detection incorporate **temporal cues** or
    **multi-modal signals** (LiDAR, IMU) for finer triggers?\
-   Can the approach estimate **physical terrain parameters**
    (friction/roughness) jointly with visual traversability?

## 13) Reproducibility Notes

-   **Code available**: public GitHub (**CHUNGUS**).\
-   **Supplemental materials** linked via DOI.\
-   Training/retraining details (frozen DINOv2, tiny MLP head,
    cache-and-retrain) are clearly specified; reported **\~0.1 s**
    retrain, **7.7 Hz** inference on commodity GPUs.

## 14) Link

-   Paper: https://doi.org/10.1109/LRA.2025.3563819\
-   Code: https://github.com/andreschreiber/CHUNGUS

## 15) Recap --- Step-by-Step

-   **See a new image** from the robot camera.\
-   **Compute DINOv2 features** and **FeatUp** them to per-pixel
    descriptors.\
-   **Check novelty**: compare the image's **class token** to a Faiss
    index of past images.\
-   **If novel → ask human**: show two crosshairs; annotator says which
    pixel is **more/less/equally traversable** (plus one cross-image
    pair).\
-   **Retrain fast**: update a **tiny MLP head** (seconds) using all
    cached pairwise labels; DINOv2 stays frozen.\
-   **Predict per-pixel traversability** and an **uncertainty/novelty**
    score.\
-   **Fuse into elevation map** as a **BEV traversability layer**.\
-   **Plan with MPPI**: penalize low-traversability cells, generate safe
    controls.\
-   **Repeat online**: keep operating; only pause briefly when new
    labels are required.
