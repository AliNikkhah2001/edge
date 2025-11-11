---
layout: default
title: Fisheye Tracking
nav_order: 6
---

# Fisheye Tracking

Fisheye cameras offer an ultra-wide field of view (FOV) for applications such as autonomous driving and surveillance, yet their **strong optical distortion** breaks the translational invariance assumption of standard convolutional neural networks (CNNs).  
Recent research has advanced two complementary directions:

- **Rectified Convolutions (RectConv)** — adapt pretrained CNNs to fisheye imagery *without retraining*.  
- **Fisheye Multiple Object Tracking (FEMOT)** — track and detect objects *without dewarping* via distortion-aware learning.

---

## 1. Rectified Convolutions (RectConv)

**Paper:** *Adapting CNNs for Fisheye Cameras Without Retraining* (Griffiths & Dansereau, 2024):contentReference[oaicite:2]{index=2}
![RectConv vs Conv kernel deformation](/assets/images/fisheye/rectconv_fig2.png)
*Figure 1. Standard convolution vs RectConv on fisheye imagery (paper Fig. 2).*

### Overview
RectConv introduces a *training-free* method to adapt any pretrained CNN (for segmentation, detection, etc.) to work natively on non-perspective, distorted images.  
Instead of rectifying the **input image**, RectConv **rectifies the convolution kernels** based on the camera calibration model.

> *See Fig. 2 (p. 2)* — illustrates how kernel shapes deform to match local distortion, letting the network “see” a rectified patch while still using the full fisheye FOV.

### Method
1. **Camera Model:**  
   Each pixel is projected into 3D space and re-sampled on a planar grid tangent to the surface at that point.  
   The back-projection defines **offset fields** applied to each convolution kernel.

2. **Conversion:**  
   Every standard convolutional layer (`Conv`) in a pretrained model is replaced by a `RectConv` layer using pre-computed offsets.  
   Weights and biases remain unchanged.

3. **Compatibility:**  
   Works best on **fully convolutional networks (FCNs)** such as DeepLabV3 or FCN-ResNet, which accept arbitrary input sizes.

![RectConv projection pipeline](/assets/images/fisheye/rectconv_fig3.png)
*Figure 2. Projection and resampling pipeline (paper Fig. 3).*
### Experimental Results
- **Datasets:** Woodscape (fisheye driving) and PIROPO (omnidirectional people tracking).  
- **Tasks:** Semantic segmentation and object detection.

| Method | Additional Training | Field of View | Performance |
|---------|--------------------|---------------|--------------|
| Conv (Distorted) | No | Full | Poor (features misaligned) |
| Conv (Rectified) | No | Cropped | Moderate (dead zones) |
| Conv (Patches) | Yes | Partial | Good but slow |
| **RectConv (Ours)** | **No** | **Full** | **Best overall** |

> *See Table 1 (p. 6)* — RectConv achieved highest mean-IoU and pixel accuracy across all models while maintaining complete FOV coverage.

- **Inference time:** ≈ +60 % over standard convs (vs +180 % for patch-based).  
- **Bias shift:** Small distribution drift observed due to interpolation (Fig. 4 p. 4).

### Key Takeaways
- No retraining or re-annotation required.  
- Adapts CNNs directly to new camera geometries using only calibration.  
- Enables deployment of perspective-trained networks on fisheye and panoramic sensors.

---

## 2. Fisheye Multiple Object Tracking (FEMOT)

**Paper:** *Fisheye Multiple Object Tracking by Learning Distortions Without Dewarping* (Chen et al., ICIP 2023):contentReference[oaicite:3]{index=3}

### Motivation
Traditional MOT pipelines fail under fisheye distortion because:
- Kalman filters assume **linear motion** on a flat plane.
- Dewarping adds calibration errors and computational cost.

FEMOT overcomes these issues by **learning distortions directly**, avoiding any dewarping.

### Architecture Overview
FEMOT consists of two core modules (see *Fig. 2 p. 2*):
![FEMOT pipeline diagram](/assets/images/fisheye/femot_fig2.png)
*Figure 3. FEMOT pipeline with DFIA and HDA (paper Fig. 2).*
![DFIA augmentation examples](/assets/images/fisheye/femot_fig3.png)
*Figure 4. DFIA examples with increasing distortion (paper Fig. 3).*
1. **Distorted Fisheye Image Augmentation (DFIA)**  
   - Synthesizes fisheye distortions from standard perspective datasets using geometric projection equations.  
   - Enables training of detectors/re-identifiers *without collecting new fisheye data*.  
   - Controlled by distortion parameter φ (Eq. 1).

   *Example augmentations on Fig. 3 p. 3 show how φ = 0.5 / 0.9 increases curvature.*

2. **Hybrid Data Association (HDA)**  
   - Couples the fisheye and “pseudo-perspective” domains via a twin model.  
   - Performs IoU similarity and motion prediction using a **fisheye-aware Kalman filter**.  
   - Tracks objects directly in distorted space, using both fisheye and converted bounding boxes.

### Implementation
- **Detector:** PRBNet [1]  
- **Re-ID model:** OSNet [2]  
- **Tracker:** BoT-SORT-R [10]  
- **Dataset:** FisheyeMOT (9 k frames, 228 k boxes, 8 vehicle classes)

### Results

| Method | HOTA ↑ | IDF ↑ | MOTA ↑ | IDs ↓ |
|---------|---------|--------|---------|-------|
| SORT | 22.1 | 24.1 | 27.9 | 48 201 |
| FairMOT | 37.2 | 45.8 | 46.2 | 32 597 |
| BoT-SORT-R | 41.2 | 52.1 | 50.0 | 19 566 |
| **FEMOT (ours)** | **48.9** | **59.2** | **59.1** | **15 892** |

(*Table 1 p. 4*)

Ablation studies (Tables 2–3 p. 4):
- DFIA alone → + 3–5 HOTA points  
- HDA alone → + 4 HOTA points  
- DFIA + HDA → **+ 8 HOTA** over baselines

### Key Takeaways
- Learns distortion directly; no calibration or warping step needed.  
- DFIA enables reuse of perspective datasets.  
- HDA integrates geometric and learned cues for robust MOT.  
- State-of-the-art accuracy on FisheyeMOT dataset.

---

## 3. Comparative Insights

| Aspect | RectConv (2024) | FEMOT (2023) |
|--------|-----------------|---------------|
| **Goal** | General CNN adaptation to distorted imagery | MOT under fisheye distortion |
| **Strategy** | Modify convolution kernels via camera model | Learn distortion mapping via DFIA + HDA |
| **Training** | None (training-free) | Data augmentation with standard datasets |
| **Geometry Dependence** | Requires camera calibration | Implicitly learned; no dewarping |
| **Output Tasks** | Segmentation / Detection | Detection / Tracking |
| **Computation** | +60 % inference overhead | Real-time feasible |
| **Strength** | Generalizable to any network | High MOT accuracy on fisheye datasets |

---

## 4. Updated Strategies for Fisheye Tracking

- **Dewarp to rectilinear or bird’s-eye** (baseline; loses coverage)  
- **Rotated overlapping sub-windows** for local detection  
- **DFIA (Distorted Fisheye Image Augmentation)** to synthesize training data  
- **RectConv** to adapt pretrained CNNs without retraining  
- **HDA (Hybrid Data Association)** for motion tracking in distorted views  
- **End-to-end fisheye pipelines** combining RectConv + DFIA + HDA for unified segmentation + tracking

---

## 5. References

1. Griffiths, R., & Dansereau, D. (2024). *Adapting CNNs for Fisheye Cameras Without Retraining*. arXiv:2404.08187 [cs.CV].  
2. Chen, P.-Y., Hsieh, J.-W., Chang, M.-C., et al. (2023). *Fisheye Multiple Object Tracking by Learning Distortions Without Dewarping*. Proc. ICIP 2023.

---
