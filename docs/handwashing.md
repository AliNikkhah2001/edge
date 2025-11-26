---
layout: default
title: Handwashing Detection & Tagging
nav_order: 9
---

# Handwashing Detection & Tagging
Guidance for detecting, classifying, and tagging handwashing events on Axis ACAP devices using the existing detection/pose stack, INT8-quantized models, and multi-camera synchronization where needed.

---

## Objectives & Scenarios
- **Compliance monitoring:** Verify that staff wash hands for the required duration and sequence (wet → soap → scrub → rinse → dry).
- **Occupancy-aware alerts:** Trigger reminders when a person leaves a wash zone without completing the sequence.
- **Cross-camera coverage:** Support single camera above a sink or synchronized multi-camera layouts (entrance + sink + exit).

---

## Data & Modeling Options
- **Inputs:** RGB or fisheye streams at 720p/1080p; prefer synchronized timestamps when using multiple views.
- **Detection backbone:** Reuse lightweight person/hand detectors (e.g., SSD/YOLOv5n) from the [Model Zoo]({{ site.baseurl }}/docs/model-zoo) and quantize to INT8 via the [Quantization guide]({{ site.baseurl }}/docs/quantization).
- **Pose estimation (optional):** Use MoveNet-style keypoints to capture wrist/hand trajectories for temporal reasoning.
- **Temporal classification:**
  - 1D CNN or transformer encoder over per-frame hand crops or pose vectors.
  - Sliding window (e.g., 3–5 seconds, stride 0.5 s) with majority-vote smoothing to reduce flicker.
- **Metadata tags:** Emit JSON records (start/end timestamps, camera ID, track ID, confidence, compliance status) to VAPIX/metadata stream.

---

## Pipeline Blueprint
1. **Person/hand detection:** Run DLPU-optimized INT8 detector; keep top-K hand boxes per person.
2. **Association:** Track hands to persons using IoU + motion (reuse ByteTrack/DeepSORT association).
3. **Temporal features:**
   - Crop-and-resize hand ROIs; optionally compute optical flow magnitude or hand keypoint angles.
   - Normalize by person bounding box scale to handle distance changes.
4. **Classification:** Apply a lightweight temporal model to label states (not washing, washing, rinsing, drying). Calibrate thresholds on validation clips to minimize false positives.
5. **Event logic:**
   - Start event when washing probability exceeds threshold for N consecutive frames.
   - Mark compliance when minimum duration (e.g., 20–30 s) and required state ordering are satisfied.
   - Close event on timeout or when the track leaves the region of interest (ROI).
6. **Tagging & export:** Push events to the metadata channel with camera ID, timestamps, ROI IDs, and compliance flags; optionally log to MQTT/HTTP for dashboards.

---

## Deployment Notes on ACAP
- Use the **native SDK** for minimal latency; container SDK is acceptable for rapid iteration.
- Optimize preprocessing with hardware scalers; avoid CPU-side copies by feeding YUV buffers directly to the detector.
- Target **INT8** for detector and classifier; keep pose model lightweight (e.g., MoveNet lightning) if used.
- For fisheye streams, reuse the [Fisheye]({{ site.baseurl }}/docs/fisheye) dewarp/rotated-window strategy before hand crops.
- Configure **multicam synchronization** (hardware trigger or SynNet) when combining entrance and sink views to avoid temporal drift.

---

## Evaluation & QA
- **Metrics:** Event-level precision/recall, false alert rate per hour, compliance-rate accuracy, and latency budget (capture → tag < 150 ms).
- **Robustness checks:** Vary lighting, faucet reflections, soap bottle positions, and occlusions from sleeves/gloves.
- **Field validation:** Compare tagged durations vs. stopwatch ground truth; review missed events to adjust ROIs and thresholds.

---

## Extension Ideas
- Add **sound or LED prompts** via VAPIX when incomplete washing is detected.
- Fuse **sink flow sensors** with vision tags to reduce false positives when hands are outside the water stream.
- Train with **synthetic handwashing clips** (blender/Unreal) to expand coverage for rare viewpoints.
