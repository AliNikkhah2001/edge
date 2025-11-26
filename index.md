---
layout: default
title: Edge AI on Axis ACAP Cameras
nav_order: 1
---

# Edge AI on Axis ACAP Cameras
A consolidated guide to deploying detection, tracking, synchronization, and domain-specific analytics (e.g., handwashing compliance) on Axis ACAP/DLPU devices.

---

## Scope
- **Hardware & SDKs:** ARTPEC SoCs with DLPUs, native vs container SDK workflows, and larod ML runtime.
- **Models & Quantization:** Axis model zoo options, INT8 conversion, and laptop emulation for rapid iteration.
- **Tracking & Fisheye:** SORT/DeepSORT/ByteTrack comparisons, fisheye distortion strategies, and DFIA-style augmentation.
- **Multi-Camera Fusion:** Synchronization (HW trigger, NTP, SynNet) plus cross-camera identity and graph/transformer fusion.
- **Applied Analytics:** Handwashing detection/classification/tagging built on the existing person/pose stack.

---

## Quick Navigation
- [ACAP Hardware & DLPU]({{ site.baseurl }}/docs/acap)
- [SDKs (Native vs Container)]({{ site.baseurl }}/docs/sdk)
- [Model Zoo Benchmarks]({{ site.baseurl }}/docs/model-zoo)
- [Quantization & Laptop Emulation]({{ site.baseurl }}/docs/quantization)
- [Online Tracking (SORT / DeepSORT / ByteTrack)]({{ site.baseurl }}/docs/tracking)
- [Fisheye Tracking]({{ site.baseurl }}/docs/fisheye)
- [Multi-Camera Sync & ID Fusion]({{ site.baseurl }}/docs/multicam)
- [Handwashing Detection & Tagging]({{ site.baseurl }}/docs/handwashing)
- [References]({{ site.baseurl }}/docs/references)

---

## System Architecture at a Glance

| Layer | Responsibilities | Key References |
| --- | --- | --- |
| Capture & Timing | Hardware trigger / NTP / SynNet alignment across cameras | [Multicam Sync]({{ site.baseurl }}/docs/multicam-sync) |
| Inference | DLPU-optimized TFLite models (detection, classification, pose) | [Model Zoo]({{ site.baseurl }}/docs/model-zoo), [Quantization]({{ site.baseurl }}/docs/quantization) |
| Tracking | Online MOT (SORT/DeepSORT/ByteTrack) with occlusion handling | [Tracking]({{ site.baseurl }}/docs/tracking) |
| Geometry | Dewarping and rotated sub-inference for fisheye views | [Fisheye]({{ site.baseurl }}/docs/fisheye) |
| Fusion & ID | Cross-camera association via graph/transformer fusion | [Multicam ID]({{ site.baseurl }}/docs/multicam-id) |
| Domain Logic | Event tagging (e.g., handwashing) with metadata export | [Handwashing]({{ site.baseurl }}/docs/handwashing) |

---

## Highlights
- DLPU-focused INT8 pipelines with <3% expected accuracy drop and ~4× model size reduction.
- Real-time trackers with two-pass association (ByteTrack) for reduced ID switches under occlusion.
- Distortion-aware fisheye handling via dewarping or DFIA-style augmentation to maintain spatial fidelity.
- Graph/transformer-based cross-camera identity fusion for synchronized multi-view deployments.
- Extensible domain workflows that build on the core detection/pose stack, exemplified by handwashing compliance analytics.

---

## Datasets & Benchmarks
- **Tracking & Re-ID:** WildTrack, AiCity 2023, CityFlow, DIVOTrack, MARS/DukeMTMC.
- **Fisheye MOT:** FEMOT with DFIA augmentations.
- **Action Recognition (domain-specific):** Extend with handwashing clips (RGB or fisheye) for temporal classifiers and pose-based scoring.

---

## How to Use This Site
1. Start with **ACAP Hardware & DLPU** to understand device capabilities and SDK setup.
2. Pick a model from the **Axis Model Zoo** and follow **Quantization** for INT8 deployment.
3. Choose a **tracking** strategy and, if needed, apply **fisheye** adaptations.
4. For multi-camera installs, configure **synchronization** and **ID fusion**.
5. Layer domain logic such as **handwashing detection** to emit tagged events and compliance metrics.

---

Built with Just the Docs. © 2025 Ali Nikkhah
