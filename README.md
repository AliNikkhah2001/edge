---
nav_order: 1
title: Edge-AI Vision Systems on Axis ACAP Cameras
description: Implementation of tracking algorithms on a cluster of Axis cameras (ACAP/DLPU), including synchronization, model selection, quantization, cross-camera identity matching, and handwashing analytics.
---

# Edge-AI Vision Systems on Axis ACAP Cameras
**Fisheye Tracking • Multi-Camera Synchronization • Cross-Camera Identity Matching • Hygiene Analytics**

This GitHub Pages site documents an end-to-end edge AI stack running on Axis **ACAP/DLPU** cameras. It consolidates the hardware profile, SDK choices, deployment pipeline, model benchmarks, and perception algorithms (tracking, fisheye, multi-camera fusion). A new **handwashing analytics** module illustrates how to extend the stack to a real-world safety use case.

---

## Documentation Map

| Category | Contents |
| --- | --- |
| **Platform & Runtime** | [ACAP Hardware & DLPU](./docs/acap.md) — SoC/DLPU tables, SDK options, APIs, and programming notes |
| | [SDKs (Native vs Container)](./docs/sdk.md), [Axis APIs](./docs/apis.md), [ACAP v12 Programming Guide](./docs/acap-v12-programming.md), [DLPUs Explained](./docs/dlpu-overview.md) |
| **Models & Optimization** | [Axis Model Zoo](./docs/model-zoo.md) with [Benchmarks](./docs/models-benchmarks.md) and [Fisheye/Quantization Adaptation](./docs/model-adaptation-fisheye-quantization.md); [Quantization & Laptop Emulation](./docs/quantization.md) |
| **Perception & Tracking** | [Online Tracking (SORT / DeepSORT / ByteTrack)](./docs/tracking.md), [Fisheye Tracking](./docs/fisheye.md) |
| **Multi-Camera Fusion** | [Multi-Camera Sync & Cross-Camera ID](./docs/multicam.md) with [Temporal Sync](./docs/multicam-sync.md) and [Identity Matching](./docs/multicam-id.md) |
| **Applications** | [Handwashing Analytics](./docs/handwashing.md) — detection, action classification, and event tagging |
| **Background & References** | [Expanded Technical Report](./docs/expanded_technical_report.md), [Model Zoo](./docs/model-zoo.md), [References](./docs/references.md) |

---

## Platform Snapshot
- **SoCs**: ARTPEC-7/8/9 with on-die **DLPU** accelerators for INT8/FP16 inference (3–4 TOPS in <3 W typical usage).
- **SDKs**: Native **C/C++** for maximum performance; container SDK for portability; `larod` provides TensorFlow Lite access to the DLPU.
- **I/O & Control**: VAPIX/ONVIF for streams, event system for triggers, MQTT/HTTP/WebSocket for messaging, and Axoverlay for on-camera rendering.
- **Deployment**: Quantized TFLite models from the [Axis Model Zoo](./docs/model-zoo.md), with calibration and validation documented in [Quantization](./docs/quantization.md).

---

## Perception Highlights

| Component | Notes |
| --- | --- |
| **Tracking** | SORT/DeepSORT/ByteTrack baselines with calibration for DLPU throughput; fisheye handling via DFIA/FEMOT data and local rotated crops. |
| **Synchronization** | Hardware trigger, master–slave, and NTP options; optional **SynNet** pose-based offset estimation for non-wired deployments. |
| **Cross-Camera ID** | Transformer, graph, and clustering-based Re-ID fusion (TransReID, LMGP, AGC/STC, CrossMOT, MCTR/GMT). |
| **Application Example** | [Handwashing Analytics](./docs/handwashing.md) combines detection, action classification, and scene events to tag compliant episodes and raise alerts. |

---

## Datasets
- **WildTrack** (7 synchronized cameras) for multi-camera person tracking.
- **AiCity 2023** for vehicle/person Re-ID with anchor-guided fusion.
- **CityFlow** for cross-view identity matching (TransReID).
- **DIVOTrack** for joint detection/Re-ID training.
- **MARS / DukeMTMC** for Re-ID pretraining.
- **FEMOT** with **DFIA** distortion augmentation for fisheye MOT.
- **Hand hygiene** sources (e.g., **UNMC Hand Hygiene**, **HandWash Dataset**, **GWD-PHHC** video sets) for gesture/action supervision in the new application module.

---

## Project To-Do
- Add site logo, favicon, and analytics.
- Expand DLPU throughput measurements (ARTPEC-9) and mAP/FPS charts.
- Fine-tune YOLOv5n on fisheye datasets; implement ByteTrack C++ on ACAP.
- Validate synchronization pipeline (HW trigger + timestamp fusion) and integrate pose-based offsets.
- Harden the [Handwashing Analytics](./docs/handwashing.md) flow with automated QA metrics and deployment scripts.

---

## References
See the consolidated [References](./docs/references.md) page for papers, SDK guides, and model resources.

© 2025 Ali Nikkhah
