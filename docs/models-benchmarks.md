---
layout: default
title: Recommended Models & Benchmarks
nav_order: 2.8
---

# Recommended Models & Benchmarks (Human Detection / Pose)

## Architectural constraints
- DLPU executes **TFLite INT8 built-ins** only; unsupported ops fall back to CPU.
- Prefer lightweight backbones (**MobileNet v2**, **MobileDet**, **EfficientNet-Lite**).  
- ARTPEC-8 prefers **per-tensor** quantization; ARTPEC-9: tensors ≤ 4D, batch=1; prefer **ReLU6**; filters divisible by 16 (per Axis tips).

## Recommended detectors
- **SSD MobileNet v2**
- **SSDLite MobileDet**
- **YOLOv5 (n/s/m)** on ARTPEC-8/9

## Benchmarks (Axis Model Zoo excerpts)
| SoC/Camera | Model | Latency (ms) | mAP/Top-1 | Notes |
|---|---|---|---|---|
| ARTPEC-7 (Q1615 Mk III) | SSD-MobV2 | 17.81 | 25.6 mAP | Fastest on A7 |
|  | SSDLite MobileDet | 31.16 | 32.9 mAP | More accurate |
| ARTPEC-8 (P1465-LE) | SSD-MobV2 | 28.04 | 25.6 mAP | 300×300 input |
|  | SSDLite MobileDet | 38.93 | 32.9 mAP |  |
|  | YOLOv5n-Artpec8 | 100.05 | 23.5 mAP | small YOLO |
| ARTPEC-8 (Q1656-LE) | SSD-MobV2 | 17.67 | 25.6 mAP | newer HW |
|  | SSDLite MobileDet | 28.40 | 32.9 mAP |  |
|  | YOLOv5n-Artpec8 | 55.52 | 23.5 mAP |  |
|  | YOLOv5s-Artpec8 | 69.87 | 32.3 mAP |  |
|  | YOLOv5m-Artpec8 | 95.34 | 37.9 mAP |  |
| ARTPEC-9 (Q1728) | SSD-MobV2 | 10.35 | 25.6 mAP | fastest |
|  | SSDLite MobileDet | 21.00 | 32.9 mAP |  |
|  | YOLOv5n-Artpec9 | 42.59 | 23.3 mAP |  |
|  | YOLOv5s-Artpec9 | 45.54 | 32.2 mAP |  |
|  | YOLOv5m-Artpec9 | 53.22 | 38.1 mAP |  |
| CV25 (M3085-V) | SSD-MobV2 | 5.36 | — | Ambarella DLPU |
|  | EfficientNet-Lite0 (cls) | 6.75 | 71.2% Top-1 | classification |

**Pose estimation**: **MoveNet SinglePose Lightning (192×192)** on ARTPEC-8; 17 keypoints + confidences. Multi-container example (inference + Flask server).

**Docs:**  
- Model Zoo: <https://github.com/AxisCommunications/axis-model-zoo>  
- Pose example: <https://raw.githubusercontent.com/AxisCommunications/acap-computer-vision-sdk-examples/main/pose-estimator-with-flask/README.md>

---

## Vision models and tracking algorithms for human movement on Axis ACAP v12

### Deep-learning framework support and general limitations
- On ACAP v12 devices, the **DLPU** accelerates inference only for **TensorFlow Lite INT8** models (CV25 uses a proprietary format converted from TFLite/ONNX). No custom ops, PyTorch, or raw ONNX on the DLPU; unsupported ops fall back to CPU and run much slower [1].  
- **Model-size & operator constraints**: Edge devices can’t run heavy architectures (e.g., ViTs, deformable convs). Use light backbones like **ResNet-18**, **MobileNet**, **EfficientNet-Lite** [2].  
- **SoC-specific tips**:  
  - **ARTPEC-8**: best with per-tensor quantization, 3×3 conv (stride 2), filter counts divisible by 6, **ReLU** activation [3].  
  - **ARTPEC-9**: tensors ≤ 4D, **batch = 1**, prefer **ReLU6**, filters divisible by 16 [4].

> ✅ **Implication**: Convert TF/Keras models to **quantized TFLite INT8** to leverage the DLPU [1].

### Recommended architectures
Axis recommends SoC-specific yet largely overlapping families [5][6][7][8][28]:
- **Backbone (default)**: **MobileNet v2** on all SoCs.
- **Detection heads**:
  - **SSD MobileNet v2** – light, real-time baseline.
  - **SSDLite MobileDet** – higher accuracy, extra compute.
  - **YOLOv5 (n/s/m)** – supported on ARTPEC-8/9; higher accuracy at higher latency [5][6].
- **ARTPEC-8**: **ResNet-18** is noted as a stronger backbone (no ready-made configs; DIY training) [7].
- **CV25**: MobileNet v2 with SSD/SSDLite [8].

### Supported detection models (human detection)
Axis’s Model Zoo benchmarks relevant human-detection models (see table above). Latencies are measured on **AXIS OS 12.6.85** and reflect DLPU average inference time; mAP values are from off-device evaluations and indicate relative accuracy [9]–[21].

**Key observations** [22][23][17]:
- **SSD MobileNet v2**: best real-time performance (≈10–30 ms) with moderate accuracy (~25–26 mAP).  
- **SSDLite MobileDet**: ~33 mAP but ~1.5–2× slower than SSD MobileNet on the same SoC.  
- **YOLOv5 (n/s/m)** on ARTPEC-8/9: accuracy up to ~38 mAP with 40–95 ms latency; pick when accuracy > FPS.

### Pose estimation for human tracking
- Axis provides an ARTPEC-8 **pose estimator** example using **MoveNet SinglePose Lightning** (192×192 input, 17 keypoints with confidences) [24].  
- Pose-based tracking is robust to partial occlusions; keypoints remain identifiable when boxes overlap [25].  
- The example uses a **multi-container** ACAP app: one container for capture + TFLite inference; another exposes a **Flask** API for publishing results [26].

### Tracking algorithms on ACAP
The DLPU performs inference only; **tracking runs on CPU** within your ACAP app [27]. Solid choices:
- **SORT** – Kalman filter + Hungarian assignment; very fast and simple.  
- **DeepSORT** – adds appearance embeddings to reduce ID switches; needs a small feature extractor.  
- **ByteTrack** – leverages high/low-confidence detections to recover misses; robust with fast detectors.

> Tip: Implement trackers in **C/C++** (or highly optimized Python) to minimize CPU overhead; end-to-end FPS depends on **detector latency + tracker efficiency**.

### Axis-provided analytics
- **AXIS Object Analytics (AOA)** – built-in app for many cameras: detects, classifies (humans/vehicles), tracks, and emits events (areas, lines). No custom training.  
- **Digital Autotracking** (PTZ) – detects motion and steers/zooms PTZ to keep subjects centered. Not customizable but provides ready-made tracking behavior.

### Answers to the research questions
1. **Can we deploy any model on this device?**  
   **No.** DLPU supports **TFLite INT8** only (CV25: proprietary from TFLite/ONNX). Heavy/custom ops fall back to CPU. Use supported layers and quantization; ensure the model fits DLPU memory and SoC constraints [1][3][4][28].
2. **Is the choice of vision backbone limited?**  
   **Yes.** Prefer **MobileNet v2** across SoCs; **ResNet-18** is feasible on ARTPEC-8 (DIY setup). Lightweight families like **MobileDet** and **EfficientNet-Lite** are also appropriate; transformer-style/deep nets are discouraged [2][5][7].
3. **What algorithms/models for tracking human movement?**  
   - **Detector or Pose** per frame: choose **SSD MobileNet v2**, **SSDLite MobileDet**, or **YOLOv5 (n/s/m)** by SoC/FPS needs; or use **MoveNet SinglePose Lightning** for keypoints [24][25].  
   - **Tracker on CPU**: implement **SORT / DeepSORT / ByteTrack** to maintain identities.  
   - **Alternatively**: use **AXIS Object Analytics / Digital Autotracking** for out-of-the-box tracking.

---

### References (for this section)

[1] Supported frameworks – Axis docs  
<https://developer.axis.com/computer-vision/computer-vision-on-device/supported-frameworks/>

[2] General suggestions – Axis docs  
<https://developer.axis.com/computer-vision/computer-vision-on-device/general-suggestions/>

[3] [4] Optimization tips – Axis docs  
<https://developer.axis.com/computer-vision/computer-vision-on-device/optimization-tips/>

[5] [6] [7] [8] [28] Recommended model architecture – Axis docs  
<https://developer.axis.com/computer-vision/computer-vision-on-device/recommended-model-architecture/>

[9]–[23] Axis Model Zoo (benchmarks & configs)  
<https://github.com/AxisCommunications/axis-model-zoo>

[24] [25] [26] Pose estimator (MoveNet) example  
<https://raw.githubusercontent.com/AxisCommunications/acap-computer-vision-sdk-examples/main/pose-estimator-with-flask/README.md>

[27] Object tracking – Axis scene metadata docs  
<https://developer.axis.com/analytics/axis-scene-metadata/reference/concepts/object-tracking/>
