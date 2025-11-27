---
layout: default
title: Edge AI on Axis Cameras
nav_order: 1
---

# Edge AI on Axis Cameras

This site is a **field guide** for deploying computer-vision workloads on Axis cameras with **ACAP/DLPU** acceleration. It covers the hardware platform, SDK/runtime options, model selection and quantization, online tracking, fisheye-specific adaptations, multi-camera synchronization and identity fusion, and a new **handwashing analytics** flow.

---

## Quick Links
- Platform: [ACAP Hardware & DLPU]({{ site.baseurl }}/docs/acap) · [SDKs]({{ site.baseurl }}/docs/sdk) · [APIs]({{ site.baseurl }}/docs/apis) · [ACAP v12 Programming]({{ site.baseurl }}/docs/acap-v12-programming)
- Models: [Axis Model Zoo]({{ site.baseurl }}/docs/model-zoo) · [Benchmarks]({{ site.baseurl }}/docs/models-benchmarks) · [Quantization]({{ site.baseurl }}/docs/quantization)
- Tracking: [Online Tracking]({{ site.baseurl }}/docs/tracking) · [Fisheye]({{ site.baseurl }}/docs/fisheye)
- Fusion: [Multi-Camera Sync & IDs]({{ site.baseurl }}/docs/multicam) · [Temporal Sync]({{ site.baseurl }}/docs/multicam-sync) · [Identity Matching]({{ site.baseurl }}/docs/multicam-id)
- Application: [Handwashing Analytics]({{ site.baseurl }}/docs/handwashing)
- References: [Expanded Technical Report]({{ site.baseurl }}/docs/expanded_technical_report) · [References]({{ site.baseurl }}/docs/references)

---

## System Summary
- **Hardware**: ARTPEC-7/8/9 SoCs with DLPU accelerators (INT8/FP16), low-power analytics (<3 W typical) and on-camera overlays.
- **Runtime**: Native ACAP SDK for C/C++; containerized SDK for portability; `larod` bridges TensorFlow Lite models onto the DLPU.
- **Data Plane**: VAPIX/ONVIF for streaming and control, Axis Event System for triggers, MQTT/HTTP/WebSockets for messaging, Axoverlay for in-stream annotations.
- **Models**: Curated TFLite set from the Axis Model Zoo with INT8 calibration; laptop emulation workflows for validation before on-camera deployment.
- **Perception Stack**: SORT/DeepSORT/ByteTrack tracking, fisheye augmentation (DFIA/FEMOT), and multi-camera synchronization plus cross-camera IDs (TransReID, LMGP, MCTR/GMT).
- **Application Blueprint**: Handwashing detection/classification with scene-aware tagging to demonstrate an end-to-end ML-driven safety policy.

---

## How to Use This Site
1. **Review the platform constraints** (ACAP hardware, DLPUs, SDK/API capabilities).
2. **Pick or adapt a model** from the Axis Model Zoo, quantize it, and validate with the laptop emulation notes.
3. **Attach perception modules** (tracking, fisheye compensation) to meet your scene geometry and latency requirements.
4. **Scale to multiple cameras** with synchronization and cross-camera identity fusion.
5. **Apply the blueprint** in [Handwashing Analytics]({{ site.baseurl }}/docs/handwashing) to see how detection, action recognition, and event tagging fit together.

---

## License
Content © 2025 Ali Nikkhah.
