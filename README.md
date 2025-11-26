---
nav_order: 1
title: Edge-AI Vision Systems on Axis ACAP Cameras
description: Implementation of tracking algorithms on a cluster of Axis cameras (ACAP/DLPU), including synchronization, model selection, quantization, cross-camera identity matching, and application pipelines such as handwashing compliance.
---

# Edge-AI Vision Systems on Axis ACAP Cameras
**Fisheye Tracking • Multi-Camera Synchronization • Cross-Camera Identity Matching • Handwashing Compliance**

This repository powers a Just-the-Docs site that consolidates research notes and deployment guidance for running vision workloads on Axis **ACAP/DLPU** cameras. It organizes the content by hardware, SDKs, models, optimization, tracking, multi-camera fusion, and application-specific pipelines.

---

## Quick Navigation
- **Platform**: [ACAP Hardware & DLPU](./docs/acap.md) · [SDKs (Native vs Container)](./docs/sdk.md)
- **Models & Optimization**: [Axis Model Zoo](./docs/model-zoo.md) · [Quantization & Laptop Emulation](./docs/quantization.md)
- **Tracking**: [SORT / DeepSORT / ByteTrack](./docs/tracking.md) · [Fisheye Tracking](./docs/fisheye.md)
- **Multi-Camera**: [Overview](./docs/multicam.md) · [Temporal Sync](./docs/multicam-sync.md) · [Cross-Camera ID](./docs/multicam-id.md)
- **Applications**: [Handwashing Detection & Tagging](./docs/handwashing.md)
- **References**: [Bibliography & Datasets](./docs/references.md)

Use `index.md` as the landing page for the published site. Each page focuses on a single concern so you can jump directly to the notes you need.

---

## Highlights
- **Axis ACAP + DLPU**: ARTPEC-7/8/9 SoCs with INT8 TensorFlow Lite acceleration and <3 W power envelopes.
- **SDK Choice**: Native C/C++ for peak throughput or container SDK for rapid Python prototyping with a small overhead.
- **Model Zoo & Quantization**: Curated INT8-ready detectors/classifiers; laptop emulation ensures parity before flashing to DLPU.
- **Tracking Stack**: SORT, DeepSORT, ByteTrack baselines; fisheye-aware augmentation (DFIA/FEMOT) and dewarping options.
- **Multi-Camera Fusion**: Hardware/NTP/SynNet synchronization plus cross-camera identity alignment via Re-ID, graph, or transformer models.
- **Handwashing Pipeline**: On-device detection, action classification, and MQTT/VAPIX event tagging for hygiene compliance.

---

## Project Status
- **Docs**: Just-the-Docs site configured with search and hierarchical nav; content organized under `/docs`.
- **To Do**: Add site branding, analytics, performance charts, extended DLPU throughput measurements, and changelog/PDF export.

---

## References
See [docs/references.md](./docs/references.md) for papers, datasets, and external repositories (e.g., Axis Model Zoo).
