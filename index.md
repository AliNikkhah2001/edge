---
layout: default
title: Edge-AI Vision on Axis Cameras
nav_order: 1
---

# Edge-AI Vision on Axis Cameras

A concise entry point for building and deploying tracking and analytics workloads on **Axis ACAP/DLPU** cameras. The site maps hardware capabilities to SDK choices, model selection, quantization, tracking algorithms, fisheye handling, multi-camera fusion, and application-specific pipelines such as **handwashing compliance**.

## Platform at a Glance
- **Hardware:** ARTPEC-7/8/9 SoCs with integrated **DLPUs** for INT8 TensorFlow Lite acceleration and low-power edge inference.
- **SDKs:** Native C/C++ and containerized workflows with `larod` for DLPU access, VAPIX/ONVIF for camera control, and MQTT/WebSockets for metadata streaming.
- **Model Zoo & Quantization:** Curated INT8-ready detectors/classifiers with laptop emulation for offline benchmarking before DLPU deployment.
- **Tracking Stack:** SORT, DeepSORT, and ByteTrack baselines; fisheye-aware augmentation and dewarping; synchronization and cross-camera ID fusion.

## Documentation Map
- **Hardware & SDKs**
  - [ACAP Hardware & DLPU]({{ site.baseurl }}/docs/acap) — SoC tables, DLPU toolchains, and model compiler notes.
  - [SDKs (Native vs Container)]({{ site.baseurl }}/docs/sdk) — deployment options, tooling, and performance trade-offs.
- **Models & Optimization**
  - [Axis Model Zoo]({{ site.baseurl }}/docs/model-zoo) — available tasks, accuracy/FPS, and recommended variants for ACAP.
  - [Quantization & Laptop Emulation]({{ site.baseurl }}/docs/quantization) — INT8 conversion, validation, and CPU/DLPU parity checks.
- **Online Tracking**
  - [Tracking (SORT / DeepSORT / ByteTrack)]({{ site.baseurl }}/docs/tracking) — data association strategies and resource footprint.
  - [Fisheye Tracking]({{ site.baseurl }}/docs/fisheye) — distortion-aware augmentation (DFIA/FEMOT), dewarping, and rotated-window inference.
- **Multi-Camera Pipelines**
  - [Multi-Camera Sync & ID]({{ site.baseurl }}/docs/multicam) — overview of temporal alignment and identity fusion.
  - [Temporal Synchronization]({{ site.baseurl }}/docs/multicam-sync) — hardware triggers, NTP, and SynNet pose-based alignment.
  - [Cross-Camera Identity Matching]({{ site.baseurl }}/docs/multicam-id) — Re-ID, graph/transformer fusion, and Anchor-Guided clustering.
- **Applications**
  - [Handwashing Detection & Tagging]({{ site.baseurl }}/docs/handwashing) — ML pipeline for compliance detection, classification, and metadata tagging.
- **References**
  - [Bibliography & Datasets]({{ site.baseurl }}/docs/references)

Use the table above as a sitemap: each page is scoped to a single concern so you can navigate directly to the hardware notes, SDK guidance, model benchmarks, or application pipelines you need.
