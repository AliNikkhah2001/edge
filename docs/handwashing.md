---
layout: default
title: Handwashing Detection & Tagging
nav_order: 11
---

# Handwashing Detection & Tagging

Design notes for detecting, classifying, and tagging **handwashing events** on Axis ACAP cameras using the existing tracking stack, DLPU-friendly models, and on-device metadata publishers.

## Objectives
- Detect sinks, faucets, and hands; classify **start/ongoing/complete** wash events.
- Emit **metadata tags** (JSON via MQTT/WebSockets or VAPIX events) with timestamps and camera IDs for compliance logging.
- Run **fully on-device** when possible (INT8 models on DLPU) and fall back to CPU only for unsupported ops.

## Model Selection
- **Detection backbone:** lightweight YOLOv5n/YOLOv8n or SSDLite MobileDet from the [Axis Model Zoo]({{ site.baseurl }}/docs/model-zoo) for faucet/hand localization.
- **Action classifier:**
  - **Pose-based:** MoveNet/BlazePose keypoints + temporal LSTM/TCN to classify lathering/rinsing/drying.
  - **Optical-flow free:** 3–5 frame snippet classifier (e.g., MobileNetV3/TSM-lite) quantized to INT8.
- **Outputs:** bounding boxes with class IDs (`hand`, `soap_dispenser`, `sink`, `towel`), plus action state labels (`start`, `wash`, `rinse`, `dry`).

### Quantization & DLPU Deployment
- Prefer **per-tensor INT8** for ARTPEC-8; enable per-channel where allowed on ARTPEC-9 (see [Quantization]({{ site.baseurl }}/docs/quantization)).
- Validate operators are DLPU-supported; unsupported ops (e.g., some `pad`/`resize` variants) trigger CPU fallback—profile with `larod` tooling before deployment.
- For pose models, freeze input resolution to keep tensor sizes ≤ DLPU limits; use static batch = 1.

## Data & Annotation
- Capture **multi-angle** footage (overhead + oblique) with synchronized timestamps to support temporal models (see [Sync notes]({{ site.baseurl }}/docs/multicam-sync)).
- Label:
  - **Object boxes:** hands (left/right optional), faucet, soap dispenser, towel dispenser, sink basin.
  - **Action phases:** `approach`, `soap`, `scrub`, `rinse`, `dry`, `complete` (map to coarser `start/wash/rinse/dry` if needed).
  - **Quality flags:** soap usage, duration ≥20s, tap shutoff.
- Export to COCO-style JSON; include per-frame timestamps for later alignment with tracking IDs.

## On-Camera Pipeline
1. **Detection** on each frame (DLPU) → candidate ROIs.
2. **Tracking** (ByteTrack/DeepSORT) to maintain IDs through occlusions and across partial sink views ([Tracking]({{ site.baseurl }}/docs/tracking)).
3. **Action classification**:
   - Per-track snippet buffer (3–10 frames) → temporal classifier.
   - Optionally use **pose keypoints** to infer soap/dry gestures without optical flow.
4. **Event tagging**:
   - Emit JSON: `{camera_id, track_id, phase, confidence, start_time, end_time, duration, hygiene_flags}`.
   - Send via **MQTT/WebSockets** or trigger **VAPIX events**; mirror to local syslog for auditing.
5. **Post-processing**: enforce minimum dwell times, merge split events, and suppress duplicate detections across overlapping cameras using the [Cross-Camera ID]({{ site.baseurl }}/docs/multicam-id) pipeline.

## Performance Targets
- **Frame rate:** ≥12 FPS for detection + tracking on ARTPEC-8; ≥15 FPS on ARTPEC-9 with INT8 models.
- **Latency budget:** detection <35 ms, tracking <5 ms, classifier <10 ms per frame (single stream).
- **Accuracy metrics:**
  - Detection: mAP@0.5 for `hand/soap/sink` > 0.35 on in-domain data.
  - Action: F1 > 0.8 for `wash` vs `rinse`; event-level precision > 0.9 with ≥18s duration rule.

## Validation & Monitoring
- Use **laptop emulation** with TensorFlow Lite to iterate on quantization and classifier accuracy before DLPU flashing.
- Record **per-op latencies** with `larod` profiling; watch for CPU fallbacks on unsupported layers.
- Enable **debug overlays** during field tests (bounding boxes, phase labels, timers) and disable in production.
- Track **false positives** from reflective surfaces/automatic faucets; mitigate with ROI masks and sink-layout templates.

## Integration Points
- **Metadata bus:** MQTT topics per camera (`/edge/handwash/<camera_id>`) with retained messages for last-known state.
- **Compliance backend:** store events in time-series DB; join with access-control logs for audit trails.
- **Security & privacy:** process on-device; redact video where policy requires; send only metadata off-camera.

This page extends the existing tracking and synchronization stack with an application-specific pipeline tailored for hygiene compliance while reusing the DLPU-friendly model and deployment guidance from the rest of the documentation.
