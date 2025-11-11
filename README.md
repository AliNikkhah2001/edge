# 🧠 Edge-AI Vision Systems on Axis ACAP Cameras  
**Fisheye Tracking • Multi-Camera Synchronization • Cross-Camera Identity Matching**

> A full research documentation portal built with [Just the Docs](https://just-the-docs.github.io/just-the-docs/), deployed via GitHub Pages.

---

## 📘 Overview

This project documents comprehensive research on **Edge AI vision systems** using Axis Communications’ **ACAP platform**.  
It covers hardware architectures, SDKs, pretrained model zoo, quantization, and multi-camera tracking using fisheye and detail cameras.  
All content is organized for easy navigation with sidebars and search.

---

## 🗂️ Table of Contents

| Section | Description |
|----------|-------------|
| [Home](./index.md) | Landing page with overview |
| [ACAP Hardware & DLPU](./docs/acap.md) | ARTPEC SoC specs and DLPU details |
| [SDKs (Native vs Container)](./docs/sdk.md) | SDK comparison and usage |
| [Axis Model Zoo](./docs/model-zoo.md) | Model benchmarks and performance |
| [Quantization & Laptop Emulation](./docs/quantization.md) | INT8 quantization and testing |
| [Online Tracking (SORT, DeepSORT, ByteTrack)](./docs/tracking.md) | Tracking algorithms and comparisons |
| [Fisheye Tracking](./docs/fisheye.md) | Distortion handling and DFIA |
| [Multi-Camera Sync & Cross-Camera ID](./docs/multicam.md) | Temporal and spatial fusion pipeline |
| ├── [Temporal Synchronization](./docs/multicam-sync.md) | Hardware, software, and SynNet sync methods |
| └── [Cross-Camera Identity Matching](./docs/multicam-id.md) | Re-ID, graph, and transformer-based fusion |
| [References](./docs/references.md) | All research citations and datasets |

---

## 🧩 Research Summary

### Axis ACAP and DLPU
- **ARTPEC-7/8/9 SoCs** with dedicated DLPU for TensorFlow Lite INT8.
- Power-efficient edge inference (3–4 TOPS, <3 W).

### Axis Model Zoo
| Model | Task | mAP / Accuracy | Inference Speed |
|--------|------|----------------|-----------------|
| SSD MobileNet v2 | Detection | ~30 mAP | 12–15 FPS |
| SSDLite MobileDet | Detection | ~30 mAP | 18 FPS |
| YOLOv5n | Detection | ~37 mAP | 14 FPS |
| EfficientNet-Lite0 | Classification | ~80 % Top-1 | 20 FPS |
| MoveNet | Pose Estimation | ~85 % PCK | 16 FPS |

📚 [Axis Model Zoo Repository](https://github.com/AxisCommunications/axis-model-zoo)

### Quantization
- INT8 quantization halves inference latency and shrinks size × 4 with < 3 % accuracy loss.
- Emulated on laptop with TensorFlow Lite.

### Tracking Algorithms
| Tracker | Description | Strengths | Weaknesses |
|----------|--------------|------------|-------------|
| **SORT** | Kalman filter + IoU association | Fast, simple | Loses identity on occlusion |
| **DeepSORT** | SORT + deep Re-ID embeddings | Handles occlusion | Slower |
| **ByteTrack** | Two-pass association (keeps low-conf detections) | High MOTA / IDF1 | Slightly higher compute |

### Fisheye Tracking
- **Challenges:** radial distortion, nonlinear projection.  
- **Solutions:**  
  - Dewarp to rectilinear or bird’s-eye.  
  - Windowed rotated sub-inference.  
  - Train with **DFIA** (Distorted Fisheye Image Augmentation) and **FEMOT** datasets.

### Multi-Camera Synchronization
| Method | Description |
|---------|-------------|
| **Hardware trigger** | Shared pulse aligns all cameras |
| **Master–Slave** | One camera controls exposures |
| **NTP Sync** | Time-stamp alignment (minor jitter) |
| **SynNet** | Learns offsets via pose-sequence matching (ICCV 2019) |

### Cross-Camera Identity Matching
| Method | Type | Key Idea | Dataset |
|---------|------|-----------|----------|
| **TransReID** | Transformer Re-ID | Appearance + trajectory fusion | CityFlow |
| **LMGP** | Graph | Lifted multicut + geometry projection | WildTrack |
| **AGC + STC** | Clustering | Anchor-guided + spatio-temporal reassignment | AiCity 2023 |
| **CrossMOT / DIVOTrack** | End-to-End | Joint detection + embedding | DIVOTrack |
| **MCTR / GMT** | Transformer | Global attention-based multi-camera association | VisionTrack |

---

## 🧠 Datasets Used

| Dataset | Purpose | Notes |
|----------|----------|-------|
| **WildTrack** | Multi-camera person tracking | 7 synchronized cameras |
| **AiCity 2023** | Vehicle & person Re-ID | Anchor-guided fusion benchmark |
| **CityFlow** | Cross-view re-identification | Used for TransReID |
| **DIVOTrack** | Multi-view, moving cameras | Used for CrossMOT |
| **MARS / DukeMTMC** | Classic Re-ID | Pretraining for embedding models |
| **FEMOT** | Fisheye MOT | Distorted augmentation + DFIA |

---

## 📅 Organized To-Do List

### 🧱 Infrastructure
- [x] Configure **Just the Docs** theme for GitHub Pages  
- [x] Add sidebar, search, and navigation hierarchy  
- [ ] Add site logo and favicon (`/assets/images/logo.png`)  
- [ ] Add analytics (Google / Plausible)

### ⚙️ Research Content
- [x] ACAP Hardware & DLPU tables  
- [x] SDK comparison (Native vs Container)  
- [x] Axis Model Zoo benchmarks  
- [x] Quantization examples + TensorFlow Lite code  
- [x] Tracking algorithm comparison  
- [x] Fisheye tracking strategies  
- [x] Multi-camera synchronization (SynNet, NTP, HW trigger)  
- [x] Cross-camera identity matching summary  
- [ ] Add performance charts (mAP vs FPS graphs)  
- [ ] Include detailed DLPU throughput benchmark (ARTPEC-9)

### 🧪 Experimental Work
- [ ] Train / fine-tune YOLOv5n on fisheye dataset  
- [ ] Implement ByteTrack in C++ for ACAP  
- [ ] Evaluate quantized vs non-quantized model latency on ACAP  
- [ ] Validate synchronization pipeline (HW trigger + timestamp fusion)

### 🌐 Documentation
- [x] Create `_config.yml` for GitHub Pages  
- [x] Generate modular docs under `/docs`  
- [x] Add custom CSS tweaks (`assets/css/just-the-docs.scss`)  
- [ ] Add blog or changelog section  
- [ ] Create PDF export of report via Pandoc  

### 🤖 Future Research Directions
- [ ] Integrate pose-based synchronization (SynNet) into ByteTrack pipeline  
- [ ] Explore transformer-based multi-camera Re-ID (MCTR, GMT)  
- [ ] Benchmark TransReID on Axis ACAP hardware  
- [ ] Develop an open-source ACAP plugin for real-time tracking fusion

---

## 📚 References

1. [Axis Developer Docs — ACAP SDK 12](https://developer.axis.com/acap/)  
2. Wu et al., *ICCV 2019* — Multi-Video Temporal Synchronization (SynNet)  
3. Li et al., *ECCV 2022* — ByteTrack: Multi-Object Tracking  
4. Tang et al., *2017* — DeepSORT: Online and Realtime Tracking  
5. Bewley et al., *ICIP 2016* — SORT  
6. Huang et al., *CVPRW 2023* — Anchor-Guided Clustering + ST Consistency  
7. Xu et al., *arXiv 2021* — LMGP: Lifted Multicut Meets Geometry Projections  
8. Chen et al., *WACV 2025* — MCTR: Multi-Camera Tracking Transformer  
9. Sun et al., *arXiv 2024* — GMT: Global Association Transformer  
10. [Axis Model Zoo on GitHub](https://github.com/AxisCommunications/axis-model-zoo)
1. [Axis DLPU Documentation](https://developer.axis.com/computer-vision/computer-vision-on-device/axis-dlpu/)  
2. [Coral Edge TPU FAQ](https://www.coral.ai/docs/edgetpu/faq/)  
3. [Optimization Tips | Axis Developer](https://developer.axis.com/computer-vision/computer-vision-on-device/optimization-tips/)  
4. [DLPU Model Conversion Guide](https://developer.axis.com/computer-vision/computer-vision-on-device/dlpu-model-conversion/)  
5. [General Suggestions | Axis Developer](https://developer.axis.com/computer-vision/computer-vision-on-device/general-suggestions/)  
6. [PyCoral API Overview](https://www.coral.ai/docs/reference/py/)  
7. [Axis Model Zoo (GitHub)](https://github.com/AxisCommunications/axis-model-zoo)  
8. [Object Tracking Docs](https://developer.axis.com/analytics/axis-scene-metadata/reference/concepts/object-tracking/)  
9. [Digital Autotracking API](https://developer.axis.com/vapix/applications/digital-autotracking-api/)  
10. [NVIDIA Edge vs Cloud Blog](https://blogs.nvidia.com/blog/difference-between-cloud-and-edge-computing/)  
11. [Coursera Edge AI vs Cloud AI](https://www.coursera.org/articles/edge-ai-vs-cloud-ai)
See also: [Axis Developer Portal](https://developer.axis.com/acap/).
---


© 2025 Ali Nikkhah 
# edge
