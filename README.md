---
nav_order: 1
title: Edge-AI Vision Systems on Axis ACAP Cameras
description: Implementation of edge AI vision pipelines (tracking, synchronization, model selection, quantization, cross-camera ID) on Axis ACAP/DLPU cameras.
---

# Edge-AI Vision Systems on Axis ACAP Cameras
**Fisheye Tracking • Multi-Camera Synchronization • Cross-Camera Identity Matching • Handwashing Detection**

This repository hosts a GitHub Pages site that documents how to deploy and evaluate vision workloads on Axis ACAP cameras. It consolidates hardware capabilities, SDK options, quantization practices, tracking algorithms, fisheye adaptations, multi-camera synchronization/ID fusion, and domain-specific use cases such as handwashing compliance.

---

## Documentation Map

| Section | Scope |
| --- | --- |
| [Home](./index.md) | Overview of goals, system pillars, and navigation |
| [ACAP Hardware & DLPU](./docs/acap.md) | ARTPEC SoCs, DLPU capabilities, and SDK hierarchy |
| [SDKs (Native vs Container)](./docs/sdk.md) | Development workflows, container vs native trade-offs |
| [Axis Model Zoo](./docs/model-zoo.md) | Curated detection/classification models and benchmarks |
| [Quantization & Laptop Emulation](./docs/quantization.md) | INT8 conversion, TFLite workflows, and offline profiling |
| [Online Tracking (SORT / DeepSORT / ByteTrack)](./docs/tracking.md) | Tracker comparisons, strengths/weaknesses, metrics |
| [Fisheye Tracking](./docs/fisheye.md) | Distortion handling, DFIA/FEMOT datasets, rotated sub-inference |
| [Multi-Camera Sync & Cross-Camera ID](./docs/multicam.md) | Temporal alignment, SynNet, graph/transformer fusion |
| [Handwashing Detection & Tagging](./docs/handwashing.md) | Workflow for detecting, classifying, and tagging handwashing events |
| [References](./docs/references.md) | Papers, SDK docs, and external resources |

---

## System Highlights

- **ACAP & DLPU targets:** ARTPEC-7/8/9 SoCs with INT8-focused DLPUs (v2–v3) for power-efficient inference on TensorFlow Lite models.
- **SDK stack:** Native and container SDK paths, larod ML runtime, and bindings for loading quantized models.
- **Model zoo and benchmarks:** SSD/YOLO detectors, EfficientNet classifiers, and pose estimators with practical FPS ranges for DLPU targets.
- **Quantization pipeline:** INT8 calibration with expected <3% accuracy drop and ~4× size reduction; laptop emulation for rapid iteration before on-device tests.
- **Tracking suite:** SORT, DeepSORT, and ByteTrack comparisons with guidance on occlusion handling and association robustness.
- **Fisheye adaptations:** Dewarping options, rotated windowing, and distortion-aware augmentation (DFIA) for FEMOT-style datasets.
- **Multi-camera fusion:** Hardware/soft synchronization, SynNet offsets, and cross-camera identity fusion via graph or transformer methods.
- **Domain use cases:** Handwashing monitoring (detection, temporal classification, compliance tagging) layered on the existing person/pose stack.

---

## Datasets Used

| Dataset | Purpose | Notes |
| --- | --- | --- |
| WildTrack | Multi-camera person tracking | 7 synchronized cameras |
| AiCity 2023 | Vehicle & person Re-ID | Anchor-guided fusion evaluation |
| CityFlow | Cross-view re-identification | Used for TransReID |
| DIVOTrack | Multi-view with motion | Used for CrossMOT |
| MARS / DukeMTMC | Person Re-ID pretraining | Embedding pretraining |
| FEMOT | Fisheye multi-object tracking | DFIA distortion augmentation |

---

## Roadmap

### Infrastructure & Documentation
- [x] Configure Just the Docs for GitHub Pages, sidebar, and search
- [ ] Add site branding (logo + favicon) and analytics
- [ ] Provide PDF export / changelog views

### Research Content
- [x] ACAP hardware, SDK, model zoo, quantization, tracking, fisheye, and sync/ID coverage
- [ ] Add performance charts (mAP vs FPS) and extended DLPU throughput (ARTPEC-9)
- [ ] Publish domain workflows (e.g., handwashing) with reproducible configs

### Experimental Work
- [ ] Fine-tune YOLOv5n on fisheye datasets and validate INT8 vs FP latency
- [ ] Implement ByteTrack in C++ for ACAP and evaluate sync pipeline (HW trigger + timestamp fusion)
- [ ] Benchmark transformer-based multi-camera Re-ID directly on DLPU hardware

---

## References

Key sources include Axis ACAP/DLPU documentation, SynNet temporal synchronization (Wu et al., ICCV 2019), ByteTrack (Li et al., ECCV 2022), DeepSORT/SORT, DFIA/FEMOT datasets, AGC+STC fusion (AiCity 2023), LMGP, MCTR/GMT transformers, Axis Model Zoo, and DLPU conversion/optimization guides.

© 2025 Ali Nikkhah
