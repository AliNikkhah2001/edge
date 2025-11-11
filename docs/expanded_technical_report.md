---
layout: default
title: Summarized
description: Edge-AI Vision Systems on Axis ACAP for Multi-Camera Tracking and Fisheye Analytics
permalink: /expanded-technical-report/
nav_order: 1
---
# Expanded Technical Research Report  
**Edge-AI Vision Systems on Axis ACAP for Multi-Camera Tracking and Fisheye Analytics**

---

## 1. Hardware Architecture: ACAP & DLPU
Axis’ **ARTPEC** series SoCs form the backbone of modern ACAP-enabled cameras. Each generation introduces more powerful CPUs, DLPU accelerators, and broader model compatibility.

### 1.1 Hardware Overview

| SoC        | CPU                           | DLPU                         | Supported Frameworks     | Memory                 | Notes                               |
|------------|-------------------------------|------------------------------|--------------------------|------------------------|--------------------------------------|
| ARTPEC-7   | Quad ARM Cortex-A53 (64-bit)  | Early Edge TPU variant       | TensorFlow Lite INT8     | 1 GB RAM / 8 GB Flash  | Found in early analytics cameras     |
| ARTPEC-8   | Quad ARM Cortex-A53 (1.5 GHz) | DLPU v2                      | TensorFlow Lite INT8/ONNX| 2–4 GB RAM             | Used in cameras like P3265 series    |
| ARTPEC-9   | Octa ARM Cortex-A55 (1.6 GHz) | DLPU v3 (+FP16 support)      | TFLite, ONNX INT8/FP16   | 4 GB RAM / 16 GB Flash | Multi-model, multi-stream analytics  |

> The **DLPU** is a low-power neural accelerator for **INT8 TFLite** inference, typically ~3–4 TOPS under ~3 W — ideal for embedded edge analytics.

---

## 2. ACAP SDK Ecosystem

### 2.1 Native SDK (C/C++)
- Designed for **high-performance, on-camera inference**.
- Includes libraries for **larod** (Local AI Running on Device), **GStreamer** pipelines, and **VDO**.
- Supports **TensorFlow Lite**, **ONNX Runtime**, and proprietary compilers for Axis DLPUs.
- Enables deployment of quantized models compiled offline (e.g., packaged artifacts).

### 2.2 Container SDK (Python / Docker)
- Runs containerized apps on-camera with Docker-like isolation.
- Provides Python and REST interfaces.
- Great for prototyping analytics pipelines, but **lacks hardware-level optimization**.
- **Deprecated for future models** (ACAP 12 + ARTPEC-8/9) in favor of the native path.

#### Comparison

| Aspect              | Native SDK           | Container SDK           |
|---------------------|----------------------|-------------------------|
| Performance         | **Highest** (direct HW) | Medium (virtualized)     |
| Ease of use         | Complex              | Simple                  |
| Supported languages | C/C++                | Python, Go, Node.js     |
| DLPU access         | Direct via **larod** | Indirect via REST       |
| Status              | **Recommended**      | **Deprecated**          |

---

## 3. Axis Model Zoo Overview

The **Axis Model Zoo** provides pretrained and quantized **TFLite** models optimized for ACAP deployment.

### 3.1 Categories and Performance (typical)

| Task              | Model                | Backbone          | Size | Accuracy (mAP / Top-1) | Inference Speed (ARTPEC-8) |
|-------------------|----------------------|-------------------|------|-------------------------|----------------------------|
| Object Detection  | SSD MobileNet v2     | MobileNet v2      | 7 MB | 25–30 mAP               | 12–15 FPS                  |
| Object Detection  | SSDLite MobileDet    | MobileDet         | 5 MB | ~30 mAP                 | ~18 FPS                    |
| Object Detection  | YOLOv5n              | CSPDarknet Nano   | 8 MB | ~37 mAP                 | ~14 FPS                    |
| Classification    | EfficientNet-Lite0   | EfficientNet      | 5 MB | ~80% Top-1              | ~20 FPS                    |
| Pose Estimation   | MoveNet SinglePose   | EfficientNet      | 6 MB | ~85% PCK                | ~16 FPS                    |

> Models are provided as **`.tflite`** and integrated with **larod** for ACAP inference. You can fine-tune and re-quantize for your scenario.

---

## 4. Quantization and Edge Simulation

### 4.1 Why Quantize
- DLPUs support **INT8** operations only.
- Quantization **shrinks size (~4×)** and **speeds inference (~2×)**.
- Typical trade-off: **~1–3%** accuracy drop.

### 4.2 Laptop Emulation (TFLite)
```python
import tensorflow as tf
interpreter = tf.lite.Interpreter(model_path="model_int8.tflite")
interpreter.allocate_tensors()
```

### 4.3 Measured Effect (example)

| Model            | FP32 mAP | INT8 mAP | Drop | Speed Gain |
|------------------|----------|----------|------|------------|
| SSD MobileNet v2 | 30.5     | 28.8     | −1.7 | ×2.0       |
| YOLOv5n          | 37.0     | 35.0     | −2.0 | ×1.8       |

---

## 5. Tracking Algorithms: SORT, DeepSORT, ByteTrack

### 5.1 SORT
- **Kalman filter** prediction + **Hungarian** assignment (IoU).
- Real-time but can **ID-switch** under occlusion.

### 5.2 DeepSORT
- SORT + **appearance embeddings** (Re-ID).
- Better under occlusion, but slower (extra CNN per detection).

### 5.3 ByteTrack
- Keeps **low-confidence** detections for a second-stage association.
- Two-stage matching:
  1) High-confidence detections ↔ tracklets  
  2) Remaining low-confidence boxes ↔ unmatched tracks
- High accuracy without deep embeddings.

**Reported (MOT17):** `MOTA = 80.3%`, `IDF1 = 77.3%` at real-time rates.

---

## 6. Fisheye Tracking and Adaptation

### 6.1 Problem
Fisheye images introduce **strong radial distortion**; detectors trained on standard perspective data degrade at edges.

### 6.2 Strategies
1. **Geometric Correction:** Dewarp fisheye to **rectilinear** or **bird’s-eye** (BEV) before inference.  
2. **Windowed Rotation:** Split fisheye into overlapping rotated windows; run standard detector per window.  
3. **FEMOT / DFIA / HDA:**  
   - **DFIA**: Distorted Fisheye Image Augmentation.  
   - **HDA**: Hybrid Data Association for MOT on curved geometry.  
4. **Top-down tracking:** Project detections to a unified top-down coordinate system.

---

## 7. Multi-Camera Synchronization

### 7.1 Temporal Synchronization Options
1. **Hardware trigger:** Shared pulse/GPIO to start exposure simultaneously.  
2. **Master–slave:** One camera emits trigger pulses; others follow.  
3. **Software (NTP):** Sync system clocks; suitable for IP cameras; small jitter remains.  
4. **Content-based sync:** Align by motion similarity (e.g., **SynNet**) when hardware sync isn’t possible.

### 7.2 SynNet (Wu et al., ICCV 2019)
- Uses **view-invariant human pose** features to infer **time offset**.  
- Builds a **matching-cost volume** to determine temporal alignment.  
- Robust to differing frame rates and partial occlusions.

---

## 8. Cross-Camera Identity Matching

Once time is aligned, preserve consistent IDs across cameras.

### 8.1 Re-Identification Approaches

| Model / Method | Description | Dataset / Metric |
|----------------|-------------|------------------|
| **TransReID**  | Transformer-based appearance encoder | CityFlow, DukeMTMC |
| **OSNet**      | Lightweight omni-scale CNN           | Market1501         |
| **AGC + STC**  | Anchor-guided clustering + spatio-temporal reassignment | AiCity 2023 (IDF1 ~95%) |
| **LMGP**       | Lifted multicut + geometry projection | WildTrack          |
| **CrossMOT / DIVOTrack** | Joint cross-view embedding training | DIVOTrack |
| **MCTR / GMT** | Transformer global association        | VisionTrack, MARS  |

### 8.2 Typical Workflow
1. **Detect + track** per camera.  
2. **Extract embeddings** (Re-ID/Transformer).  
3. **Align timestamps** (hardware/NTP/SynNet).  
4. **Associate globally** with:
   - Spectral clustering,  
   - Graph-based lifted multicut, or  
   - Transformer **global attention** (MCTR/GMT).

---

## 9. System Design: Fisheye + Local Cameras

### 9.1 Architecture (schematic)

```
               +-------------------------+
               |   Edge Server / ACAP    |
               +-------------------------+
                 | | |        | | |
                 | | |        | | |
                 V V V        V V V
   +-------------------------------------------+
   |   Main Fisheye Camera (top view)          |
   |   - Detects all objects globally          |
   |   - Guides local cameras via calibration  |
   +-------------------------------------------+
       |      |      |      |
       V      V      V      V
+------------+ +------------+ +------------+
| Local Cam1 | | Local Cam2 | | Local Cam3 |
| (detail)   | | (detail)   | | (detail)   |
+------------+ +------------+ +------------+
```

### 9.2 Pipeline
1. **Synchronize cameras** (hardware or NTP).  
2. Run a **lightweight detector** (YOLOv5n/SSD).  
3. **Track per-camera** using **ByteTrack**.  
4. **Extract embeddings** for Re-ID.  
5. **Fuse detections** spatially using fisheye as global reference.  
6. **Match identities** with Re-ID or spatio-temporal clustering.  
7. **Visualize** trajectories in a unified coordinate space.
---

### 10. Research in Temporal Synchronization & Identity Matching

| Category | Method | Key Idea | Source |
|-----------|---------|-----------|---------|
| Temporal | SynNet | View-invariant poses used to infer temporal offsets between cameras | ICCV 2019 |
| Temporal | Pose-Motion Fusion | Weighted 3D joint confidence for motion alignment | AAAI 2017 |
| Identity | TransReID + Spectral Clustering | Combines Transformer-based Re-ID with motion-aware clustering | MDPI 2023 |
| Identity | LMGP | Lifted multicut graph optimization for identity grouping | arXiv 2021 |
| Identity | AGC + STC | Anchor-guided clustering with spatio-temporal reassignment | CVPRW 2023 |
| Identity | CrossMOT / DIVOTrack | Joint detection and embedding learning for multi-camera association | arXiv 2023 |
| Identity | MCTR / GMT | Transformer-based global multi-camera association | WACV 2025 / arXiv 2024 |

These approaches represent a shift from heuristic re-identification pipelines toward end-to-end, transformer-based global optimization methods.

These approaches mark progress from heuristic re-ID to **end-to-end transformer-based global trackers**.

---

## 11. Conclusions
- **ACAP Edge Cameras** can host robust tracking systems if models are **quantized** and **efficient**.  
- **ByteTrack** is a strong default tracker due to simple, high-accuracy association.  
- **Fisheye analytics** require either **dewarping** or **distortion-aware training**.  
- **Temporal synchronization** can be achieved with hardware triggers or learned approaches (e.g., **SynNet**).  
- **Cross-camera ID**: transformer-based models (TransReID, MCTR) or graph optimization (LMGP) perform best.  
- **System-level sync + global association** are essential for scalable room-level tracking.

---

## 12. Suggested Future Work
1. Integrate **pose-based temporal sync** with multi-camera **transformer tracking** in one network.  
2. Explore **quantization-aware training** for **fisheye-specific** datasets.  
3. Develop an **open-source ACAP plugin** for multi-camera **ByteTrack** fusion.  
4. Combine **optical flow** with **DLPU-optimized detectors** for real-time scene analysis.
