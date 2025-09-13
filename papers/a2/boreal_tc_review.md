# Paper Review — *Proprioception Is All You Need: Terrain Classification for Boreal Forests* (IROS 2024)

## 1. Citation Information

```bibtex
@inproceedings{LaRocque2024ProprioceptionBorealTC,
  author    = {Damien LaRocque and William Guimont{-}Martin and David{-}Alexandre Duclos and Philippe Gigu{\`e}re and Fran{\c{c}}ois Pomerleau},
  title     = {Proprioception Is All You Need: Terrain Classification for Boreal Forests},
  booktitle = {Proceedings of the 2024 IEEE/RSJ International Conference on Intelligent Robots and Systems (IROS)},
  address   = {Abu Dhabi, UAE},
  pages     = {11686--11693},
  month     = {Oct},
  year      = {2024},
  doi       = {10.1109/IROS58592.2024.10801407},
  url       = {https://doi.org/10.1109/IROS58592.2024.10801407},
  note      = {Code \& dataset: https://github.com/norlab-ulaval/BorealTC}
}
```

## 2. Clear Abstract (150–200 words)

The paper tackles terrain classification for mobile robots operating in boreal forests—environments where snow, ice, and low light degrade camera and lidar performance. The authors release **BorealTC**, a public proprioceptive dataset recorded with a Clearpath Husky A200 that includes IMU (100 Hz), wheel odometry and motor currents (6.5 Hz) across five terrains: asphalt, flooring, ice, silty loam, and snow (≈116 min total). They evaluate two learning approaches for classification from proprioception alone: a spectrogram-based **CNN** that improves prior baselines using a Hamming window to reduce spectral leakage, and a **Mamba** state-space model that processes raw time series and scales linearly with sequence length. Using BorealTC, a prior dataset (Vulpi), and a combined set, they show that CNNs excel on smaller datasets while Mamba benefits more from additional data. Results include per-class metrics, an ablation on training-set size, and a t-SNE analysis suggesting clusters align with physical terrain properties and revealing domain shift when merging datasets. The work underscores proprioception’s robustness for winter navigation and provides open resources to spur standardized evaluations in challenging off-road settings.

## 3. Keywords

- Proprioceptive terrain classification  
- Boreal forests / winter navigation  
- IMU, motor current, wheel odometry  
- Spectrogram (STFT) + Hamming window  
- Convolutional neural network (CNN)  
- State Space Model (SSM) / Mamba  
- Domain shift / dataset merging  
- Skid-steer UGV (Husky A200)

## 4. Problem Addressed

How to reliably classify terrain types for off-road, winter conditions—where visual and lidar sensing are unreliable—using **proprioceptive signals** alone to support resilient autonomous navigation.

## 5. Innovation / Main Contribution

- **BorealTC**: a public, significantly larger proprioceptive dataset emphasizing winter terrains (snow, ice, silty loam) on a Husky A200.  
- **Improved CNN pipeline**: STFT spectrograms with **Hamming windowing**, yielding better accuracy than prior proprioceptive baselines.  
- **Mamba for TC**: first exploration of SSM/Mamba for terrain classification from raw sequences; shows stronger gains as data increases.  
- **Merging-datasets study**: empirical analysis (t‑SNE) of cross-dataset latent space and the implications of domain shift.

## 6. Key Methods / Architecture

- **Data partitioning**: trajectories split into 5 s partitions; 1.7 s samples for training; class rebalancing by oversampling.  
- **CNN path**: STFT spectrogram per channel (window 0.4 s, overlap 0.2 s) with **Hamming** window; early 1×1 conv (per‑bin MLP), BN → 3×3 conv → BN → FC.  
- **Mamba path**: two branches (IMU; wheel signals) → per-branch FC projection → **Mamba block** (causal SSM) → concatenate final states → FC.  
- **Normalization**: channel-wise min–max from training data.  
- **Cross‑validation**: 5‑fold; additional ablations for data size.  
- **Combined-dataset handling**: downsample IMU to 50 Hz and wheel data to 6.5 Hz for the CNN; **no resampling needed for Mamba**.

## 7. Datasets & Experimental Setup

- **Platform**: Clearpath **Husky A200** (skid‑steer), MDL‑BDC24 motor drivers, encoders, Xsens MTi‑30 IMU; ROS 2 logging.  
- **Sensors/rates**: IMU 100 Hz; wheel velocities & motor currents 6.5 Hz.  
- **BorealTC terrains** (location): Asphalt (UL campus), Flooring (UL), Ice (ice rink at UL), Silty Loam & Snow (Forêt Montmorency).  
- **Terrain counts (5 s partitions)**: Asphalt 111; Flooring 423; Ice 450; Silty Loam 126; Snow 281.  
- **Vulpi dataset**: Concrete 24; Dirt Road 16; Ploughed 60; Unploughed 56 (Husky platform, different rates).  
- **Assumptions**: mostly flat surfaces (pitch < 5°); nine classes when combined.

## 8. Evaluation Metrics & Results

- **Vulpi dataset**:  
  - **CNN accuracy** 94.12% (vs. 91.5% prior baseline; +2.62 pp), aided by **+0.6 pp** from Hamming windowing.  
  - **Mamba accuracy** 86.76%.  
- **BorealTC dataset**:  
  - **CNN accuracy** 93.96%; strong per‑class F1 (e.g., Ice 97.68%, Silty Loam 96.61%).  
  - **Mamba accuracy** 93.68%; notably competitive; Snow F1 90.10%.  
- **Training‑size ablation (combined data)**: CNN better with small data; **Mamba improves faster** and trends to surpass with more data (power‑law behavior).  
- **Latent‑space t‑SNE**: classes cluster by physical similarity (e.g., rigid vs. soft); combined datasets exhibit **domain separation**, impacting merged‑training performance.

## 9. Limitations (esp. vs. our BEV‑semantic + elevation + MPC+CBF project)

- **No exteroceptive semantics**: purely proprioceptive labels; cannot anticipate terrain ahead or encode human/social preferences (sidewalk vs. grass).  
- **No elevation modeling**: experiments are near-flat; curb/slope/step hazards and grade comfort are not represented.  
- **No planning/control integration**: outputs are class labels, not used in a planner or **MPC+CBF** loop; no safety/comfort constraints from predictions.  
- **Robot/sensor specificity**: results tied to Husky geometry & sensor placement; **domain shift** appears when merging datasets.  
- **Limited terrain taxonomy**: five (BorealTC) + four (Vulpi) classes; does not map to nuanced “human‑preference” costs.  
- **Operational aspects**: online latency and closed‑loop benefits (e.g., reduced slip, energy, or path quality) are unmeasured.

## 10. Similarity / Relevance to Our Work

- **Overlap**: terrain awareness for outdoor navigation; snow/ice emphasis; potential to infer **slip/friction** properties relevant to safety and comfort.  
- **Divergence**: our approach centers on **BEV semantic maps, elevation fusion, and human‑preference costs** for planning; this paper stops at classification from ego signals.  
- **Takeaway**: proprioception is a robust **fallback** modality during snow cover/low light and can complement our semantic/elevation stack.

## 11. Potential Integration Points

- **Proprioceptive fallback layer**: run a lightweight Mamba/CNN to detect **slippery/soft** terrain on‑the‑fly; raise **CBF barriers** or increase MPC costs for predicted classes (e.g., ICE, SILTY LOAM).  
- **Cost‑map augmentation**: convert class posteriors into **human‑preference penalties** (e.g., sidewalk preferred; grass acceptable; ice strongly penalized).  
- **Pipeline tips**: adopt **Hamming‑window STFT** if we keep a spectrogram path; or use **Mamba** to avoid resampling and scale to long sequences.  
- **Domain adaptation**: anticipate **dataset/robot shift**; use feature alignment or self‑supervised adaptation when moving between platforms/seasons.  
- **Safety monitors**: trigger cautious speeds or re‑routing when proprioception signals high slip probability, even if BEV semantics look benign (fresh snow cover).

## 12. Open Questions / Future Work

- How to **fuse** proprioceptive TC with BEV semantics & elevation for **anticipatory** planning?  
- Can we learn a **continuous friction/comfort** metric instead of discrete classes, aligned with **human preferences**?  
- Robust **cross‑platform** generalization and standardized logging protocols to mitigate domain shift.  
- **Online** performance: latency/throughput on embedded compute; effect on closed‑loop navigation metrics.  
- Richer terrain sets (curbs, gravel, mud depths), **slopes/steps**, and **seasonal** transitions.

## 13. Reproducibility Notes

- **Open resources**: dataset + training code at https://github.com/norlab-ulaval/BorealTC.  
- **Implementation**: PyTorch Lightning; 5‑fold CV; clear hyperparameters & checkpoints.  
- **Preprocessing**: 5 s partitions; 1.7 s samples; class rebalancing; CNN uses STFT (0.4 s window, 0.2 s overlap, **Hamming**).  
- **Hardware**: authors report RTX A6000 GPU; detailed configs provided.  
- **Caveats**: labels are hand‑assigned; merged‑dataset training requires resampling for CNN; cross‑robot replication may need adaptation.

## 14. Link

- **Paper (DOI)**: https://doi.org/10.1109/IROS58592.2024.10801407  
- **Project / Data / Code**: https://github.com/norlab-ulaval/BorealTC
