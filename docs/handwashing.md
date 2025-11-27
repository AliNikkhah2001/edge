---
layout: default
title: Handwashing Analytics
nav_order: 8
---

# Handwashing Analytics

An end-to-end hygiene-monitoring flow built on the Axis ACAP stack: detect hands/people, classify **handwashing actions**, and tag compliant episodes through the Axis Event System.

---

## Objectives
- **Detect** hands/people in the sink region with low latency on DLPUs.
- **Classify** washing stages (e.g., lather, scrub, rinse) to avoid false positives from unrelated motion.
- **Tag** episodes with timestamps, duration, and sink ID for audit trails via Axis events or MQTT.

---

## Pipeline Overview
1. **Region-of-Interest Setup**
   - Define sink polygons via the Parameter API; persist to disk for reuse.
   - Optional depth/area gating to reject motion outside the basin.
2. **Primary Detection**
   - Lightweight detector (e.g., **YOLOv5n**, **SSDLite MobileDet**) quantized to **INT8 TFLite** and served through `larod`.
   - Use **cropped inference** on the sink ROI to minimize background clutter and raise FPS.
3. **Action Classification**
   - **Pose-based**: run **MoveNet** or BlazePose on detected hands/upper body; feed keypoints to a 1D CNN/TCN to classify scrub vs. rinse.
   - **RGB clip-based**: 8–16 frame window through a MobileNet-V2 3D or X3D-S model distilled/quantized for DLPU.
   - **Audio (optional)**: soap/ water cues via tiny keyword-spotting CNN; fuse with vision for robustness.
4. **Temporal Logic & Tagging**
   - Start an **episode** when detection + action classifier agree for ≥N frames.
   - Require **minimum duration** (e.g., 20 s) with grace periods for temporary occlusion.
   - Emit **Axis events** (`stateful` with JSON payload) and/or **MQTT** messages containing user ID (if Re-ID enabled), sink ID, start/stop timestamps, confidence, and failure reasons.
5. **Storage & Visualization**
   - Overlay washing status via **Axoverlay**; log summaries to edge storage or remote API.

---

## Model Suggestions
- **Detection**: YOLOv5n/YOLOv8n, SSDLite MobileDet; train with **hand** and **person** classes, add sink-specific anchors if needed.
- **Keypoint/Action**: MoveNet Lightning + TCN head; or **TSM/X3D-S** distilled to <5M params for DLPU; constrain clip length to keep latency <150 ms.
- **Quantization**: Post-training INT8 with per-channel weights; calibrate on **hand hygiene datasets** (UNMC Hand Hygiene, HandWash Dataset, GWD-PHHC) plus site-specific clips.
- **Deployment**: Use **cascaded scheduling**—detector at full FPS (e.g., 15), action classifier at downsampled rate (e.g., every 3rd frame) to balance load.

---

## Evaluation & QA
- **Episode precision/recall**: % of correctly tagged washing sessions vs. manual annotations.
- **Duration error**: Mean absolute error between predicted and ground-truth wash length.
- **Latency/FPS**: Per-stage latency measured via `larod` timestamps; target **<150 ms** end-to-end for smooth overlays.
- **Robustness checks**: Different sinks/heights, gloves, soap color, water reflections, and crowded scenes; include fisheye views using **DFIA** augmentations if applicable.

---

## Integration Notes
- Reuse the **multi-camera ID** module to avoid double-counting the same user across adjacent sinks.
- Synchronize with **hardware trigger/NTP** if combining multiple cameras for a single basin.
- Send **stateful Axis events** for compliance dashboards; fall back to **stateless** alerts for edge-only deployments.
- Map failures (no soap, too short, no rinse) to per-event codes to aid downstream analytics.
