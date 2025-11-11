---
nav_order: 1
title: Edge-AI Vision Systems on Axis ACAP Cameras
description: Implementation of tracking algorithms on a cluster of Axis cameras (ACAP/DLPU), including synchronization, model selection, quantization, and cross-camera identity matching.
---

# Edge-AI Vision Systems on Axis ACAP Cameras
**Fisheye Tracking • Multi-Camera Synchronization • Cross-Camera Identity Matching**

> Technical documentation site built with [Just the Docs](https://just-the-docs.github.io/just-the-docs/) and deployed via GitHub Pages.

---

## Overview

This project documents an implementation of **edge AI vision** on a **cluster of Axis cameras** using the **ACAP platform**.  
It covers hardware and DLPU capabilities, SDKs, a curated model zoo, INT8 quantization, online tracking (SORT/DeepSORT/ByteTrack), fisheye-specific methods, multi-camera synchronization, and cross-camera identity matching.

---

## Table of Contents

| Section | Description |
|--------|-------------|
| [Home](./index.md) | Overview and entry point |
| [ACAP Hardware & DLPU](./docs/acap.md) | ARTPEC SoC specifications and DLPU details |
| [SDKs (Native vs Container)](./docs/sdk.md) | SDK comparison and usage |
| [Axis Model Zoo](./docs/model-zoo.md) | Benchmarks and performance |
| [Quantization & Laptop Emulation](./docs/quantization.md) | INT8 conversion and testing |
| [Online Tracking (SORT, DeepSORT, ByteTrack)](./docs/tracking.md) | Algorithms and comparisons |
| [Fisheye Tracking](./docs/fisheye.md) | Distortion handling and DFIA |
| [Multi-Camera Sync & Cross-Camera ID](./docs/multicam.md) | Temporal and spatial fusion pipeline |
| ├── [Temporal Synchronization](./docs/multicam-sync.md) | Hardware, software, and SynNet-based methods |
| └── [Cross-Camera Identity Matching](./docs/multicam-id.md) | Re-ID, graph, and transformer-based fusion |
| [References](./docs/references.md) | Citations and datasets |

---

## Research Summary

### Axis ACAP and DLPU
- ARTPEC-7/8/9 SoCs with a dedicated DLPU targeting TensorFlow Lite INT8.
- Power-efficient edge inference (approx. 3–4 TOPS, <3 W in typical configurations).

### Axis Model Zoo

| Model               | Task            | mAP / Accuracy | Inference Speed |
|---------------------|-----------------|----------------|-----------------|
| SSD MobileNet v2    | Detection       | ~30 mAP        | 12–15 FPS       |
| SSDLite MobileDet   | Detection       | ~30 mAP        | 18 FPS          |
| YOLOv5n             | Detection       | ~37 mAP        | 14 FPS          |
| EfficientNet-Lite0  | Classification  | ~80% Top-1     | 20 FPS          |
| MoveNet             | Pose Estimation | ~85% PCK       | 16 FPS          |

Repository: [Axis Model Zoo](https://github.com/AxisCommunications/axis-model-zoo)

### Quantization
- INT8 quantization typically halves latency and reduces model size by ~4× with <3% top-line accuracy loss.
- Prototyped on laptop via TensorFlow Lite; deployed on DLPU for production tests.

### Tracking Algorithms

| Tracker   | Description                           | Strengths                 | Weaknesses                    |
|-----------|---------------------------------------|---------------------------|-------------------------------|
| SORT      | Kalman filter + IoU association       | Fast, simple              | ID switches under occlusion   |
| DeepSORT  | SORT + deep Re-ID embeddings          | Better occlusion handling | Slower                        |
| ByteTrack | Two-pass association (keeps low-conf) | High MOTA / IDF1          | Slightly higher compute       |

### Fisheye Tracking
- **Challenges:** radial distortion and non-linear projection.
- **Approaches:**  
  - Dewarp to rectilinear or bird’s-eye view.  
  - Windowed rotated sub-inference for local regions.  
  - Training with **DFIA** (Distorted Fisheye Image Augmentation) and **FEMOT** datasets.

### Multi-Camera Synchronization

| Method            | Description                                   |
|-------------------|-----------------------------------------------|
| Hardware trigger  | Shared pulse aligns exposures across cameras  |
| Master–Slave      | One camera controls exposure timing           |
| NTP Sync          | Timestamp alignment (minor network jitter)    |
| SynNet            | Learns offsets via pose-sequence matching     |

### Cross-Camera Identity Matching

| Method                  | Type        | Key Idea                               | Dataset    |
|-------------------------|-------------|----------------------------------------|------------|
| TransReID               | Transformer | Appearance + trajectory fusion         | CityFlow   |
| LMGP                    | Graph       | Lifted multicut + geometry projection  | WildTrack  |
| AGC + STC               | Clustering  | Anchor-guided + spatio-temporal checks | AiCity 2023|
| CrossMOT / DIVOTrack    | End-to-End  | Joint detection + embedding            | DIVOTrack  |
| MCTR / GMT              | Transformer | Global attention for association       | VisionTrack|

---

## Datasets Used

| Dataset        | Purpose                          | Notes                       |
|----------------|----------------------------------|-----------------------------|
| WildTrack      | Multi-camera person tracking     | 7 synchronized cameras      |
| AiCity 2023    | Vehicle & person Re-ID           | Anchor-guided fusion eval   |
| CityFlow       | Cross-view re-identification     | Used for TransReID          |
| DIVOTrack      | Multi-view with motion           | Used for CrossMOT           |
| MARS / DukeMTMC| Classic person Re-ID             | Pretraining for embeddings  |
| FEMOT          | Fisheye multi-object tracking    | DFIA distortion augmentation|

---

## Project To-Do

### Infrastructure
- [x] Configure Just the Docs for GitHub Pages  
- [x] Enable sidebar, search, and hierarchy  
- [ ] Add site logo and favicon (`/assets/images/logo.png`)  
- [ ] Add analytics (Google or Plausible)

### Research Content
- [x] ACAP hardware & DLPU tables  
- [x] SDK comparison (Native vs Container)  
- [x] Model zoo benchmarks  
- [x] Quantization examples (TensorFlow Lite)  
- [x] Tracking algorithm comparison  
- [x] Fisheye tracking methods  
- [x] Multi-camera synchronization (SynNet, NTP, HW trigger)  
- [x] Cross-camera identity matching summary  
- [ ] Add performance charts (mAP vs FPS)  
- [ ] Extend DLPU throughput measurements (ARTPEC-9)

### Experimental Work
- [ ] Fine-tune YOLOv5n on fisheye datasets  
- [ ] Implement ByteTrack in C++ for ACAP  
- [ ] Evaluate INT8 vs FP latency on DLPU  
- [ ] Validate sync pipeline (HW trigger + timestamp fusion)

### Documentation
- [x] Create `_config.yml` for Pages  
- [x] Organize modular docs under `/docs`  
- [ ] Add blog/changelog  
- [ ] Provide PDF export via Pandoc

### Future Directions
- [ ] Integrate pose-based synchronization (SynNet) into ByteTrack  
- [ ] Explore transformer-based multi-camera Re-ID (MCTR, GMT)  
- [ ] Benchmark TransReID directly on DLPU hardware  
- [ ] Open-source ACAP plugin for real-time tracking fusion

---

## References

1. [Axis Developer Docs — ACAP SDK 12](https://developer.axis.com/acap/)  
2. Wu et al., ICCV 2019 — Multi-Video Temporal Synchronization (SynNet)  
3. Li et al., ECCV 2022 — ByteTrack: Multi-Object Tracking  
4. Tang et al., 2017 — DeepSORT: Simple Online and Realtime Tracking with a Deep Association Metric  
5. Bewley et al., ICIP 2016 — SORT  
6. Huang et al., CVPRW 2023 — Anchor-Guided Clustering + Spatio-Temporal Consistency  
7. Xu et al., 2021 — LMGP: Lifted Multicut Meets Geometry Projections  
8. Chen et al., WACV 2025 — MCTR: Multi-Camera Tracking Transformer  
9. Sun et al., 2024 — GMT: Global Association Transformer  
10. [Axis Model Zoo (GitHub)](https://github.com/AxisCommunications/axis-model-zoo)  
11. [Axis DLPU Documentation](https://developer.axis.com/computer-vision/computer-vision-on-device/axis-dlpu/)  
12. [Optimization Tips | Axis Developer](https://developer.axis.com/computer-vision/computer-vision-on-device/optimization-tips/)  
13. [DLPU Model Conversion Guide](https://developer.axis.com/computer-vision/computer-vision-on-device/dlpu-model-conversion/)  
14. [General Suggestions | Axis Developer](https://developer.axis.com/computer-vision/computer-vision-on-device/general-suggestions/)  
15. [PyCoral API Overview](https://www.coral.ai/docs/reference/py/)  
16. [Object Tracking Docs](https://developer.axis.com/analytics/axis-scene-metadata/reference/concepts/object-tracking/)  
17. [Digital Autotracking API](https://developer.axis.com/vapix/applications/digital-autotracking-api/)  
18. [NVIDIA: Edge vs Cloud](https://blogs.nvidia.com/blog/difference-between-cloud-and-edge-computing/)  
19. [Coursera: Edge AI vs Cloud AI](https://www.coursera.org/articles/edge-ai-vs-cloud-ai)

---

© 2025 Ali Nikkhah
