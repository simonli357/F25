# Evidential Semantic Mapping in Off-road Environments with Uncertainty-aware Bayesian Kernel Inference

## 1) Citation Information (BibTeX)
```bibtex
@inproceedings{Kim2024EvidentialSemanticMapping,
  author    = {Junyoung Kim and Junwon Seo and Jihong Min},
  title     = {Evidential Semantic Mapping in Off-road Environments with Uncertainty-aware Bayesian Kernel Inference},
  booktitle = {2024 IEEE/RSJ International Conference on Intelligent Robots and Systems (IROS)},
  year      = {2024},
  month     = {October},
  address   = {Abu Dhabi, UAE},
  pages     = {1420--1427},
  doi       = {10.1109/IROS58592.2024.10802766},
  note      = {Project site: https://bit.ly/EvSemMap}
}
```

## 2) Clear Abstract
The paper improves 3D **semantic mapping** in messy, off-road scenes where segmentation networks are often uncertain. It trains an **evidential** segmentation model that outputs both class probabilities and an uncertainty score (in one forward pass). Those outputs are fused with a **Bayesian Kernel Inference (BKI)** mapper that’s modified to (a) use full class probability vectors instead of one-hot labels and (b) adapt its spatial kernel based on the prediction’s uncertainty—expanding influence for confident points and down-weighting or discarding uncertain ones. Across off-road datasets, the method yields more accurate, smoother semantic maps and better uncertainty estimates than prior BKI-based approaches.

## 3) Keywords
- Bayesian Kernel Inference (BKI)
- Evidential Deep Learning (EDL)
- Continuous semantic mapping
- Uncertainty-aware fusion
- Off-road autonomy
- Dirichlet / Continuous Categorical likelihood
- Sparse kernel / adaptive kernel length
- RELLIS-3D, RUGD, OffRoad datasets

## 4) Problem Addressed
Off-road scenes cause high perception uncertainty; standard DNN segmenters can be unreliable, and typical semantic BKI mappers compress predictions to one-hot labels and ignore uncertainty. This leads to noisy, discontinuous, or overconfident maps. The authors aim to **incorporate calibrated per-point semantic uncertainty directly into continuous 3D semantic mapping**.

## 5) Innovation / Main Contribution
- **Evidential segmentation for mapping:** Integrates **EDL** into a semantic segmentation net to obtain **probabilities + uncertainty** per pixel in a single pass.
- **Uncertainty-aware BKI:** Extends semantic BKI to use **Continuous Categorical** likelihoods (probability vectors) and an **adaptive sparse kernel** whose **length-scale grows for confident predictions** and shrinks (or gates out) uncertain ones.
- **End-to-end framework** that produces both a **semantic map** and a **variance/uncertainty map** with improved calibration.

## 6) Key Methods / Architecture
- **Backbone:** DeepLabV3 trained with **Evidential Deep Learning** loss (Dirichlet evidence; KL regularizer). Outputs per-pixel Dirichlet parameters ⇒ class probs and an uncertainty score u.
- **2D→3D projection:** Project image-space predictions to paired LiDAR point clouds before mapping.
- **Semantic BKI refresher:** Dirichlet prior over voxel semantics; recursive updates using a **sparse kernel** over local neighborhoods.
- **Likelihood change:** Replace one-hot labels with **probability vectors** using the **Continuous Categorical** likelihood.
- **Uncertainty-aware kernel:** Thresholding and adaptive length-scale so confident points influence a wider neighborhood, uncertain points a smaller one.
- **Outputs:** Semantic voxel map + **variance map** as an uncertainty layer.

## 7) Datasets & Experimental Setup
- **Datasets:** RELLIS-3D, RUGD, and a new OffRoad dataset collected by the authors.
- **Segmentation:** DeepLabV3 + EDL, 100 epochs; KL weight ramps with epoch.
- **Mapping hyperparameters:** Provided for each dataset (resolution, kernel parameters, uncertainty thresholds).

## 8) Evaluation Metrics & Results
- **Metrics:** per-class IoU, mIoU, overall voxel semantic Accuracy, Brier Score for uncertainty calibration.
- **Headline results:** Highest mIoU and Accuracy among continuous methods on RELLIS-3D and OffRoad datasets. Good zero-shot robustness and improved uncertainty calibration.

## 9) Limitations
- Possible computational overhead despite BKI efficiency.
- Dependence on accurate sensor pose and paired RGB–LiDAR.
- Evaluation limited to 7-class off-road settings; scalability to finer urban classes untested.

## 10) Similarity / Relevance to Our Work
- Strong overlap with our goal of a unified terrain-aware map: both build continuous semantic maps and model uncertainty.
- Diverges by focusing solely on semantic + uncertainty, not terrain physical properties or planning/control.
- We currently lack pixel-level uncertainty propagation; this method could enhance our risk-aware planning.

## 11) Potential Integration Points
- Add their **variance map** as an uncertainty layer in our multi-layer terrain map.
- Use **EDL** for our semantic segmentation to get calibrated Dirichlet outputs.
- Apply **adaptive kernel fusion** for both semantic and traversability layers.
- Feed uncertainty into our cost map and planning MPC.

## 12) Open Questions / Future Work
- Faster geometric models to improve runtime.
- Better uncertainty calibration techniques.
- Multi-modal uncertainty fusion.
- Active exploration leveraging the uncertainty map.

## 13) Reproducibility Notes
- Project website listed, but no public code repository confirmed.
- Datasets: RELLIS-3D, RUGD, plus proprietary OffRoad set with details in paper.
- Hyperparameters and training details included for re-implementation.

## 14) Link
- Project page: https://bit.ly/EvSemMap
- DOI: 10.1109/IROS58592.2024.10802766

## 15) Recap (Step-by-Step)
- Train a DeepLabV3 segmentation net with Evidential Deep Learning to output probabilities + uncertainty.
- Project predictions onto LiDAR to get 3D semantic points with uncertainty.
- Fuse points into a 3D map using BKI extended to handle probability vectors.
- Make kernel uncertainty-aware by dropping uncertain points and adapting influence radius.
- Output a semantic 3D map and a variance map.
- Evaluate on multiple off-road datasets showing better accuracy and uncertainty calibration than prior work.
