---
layout: default
title: Handwashing Detection & Tagging
nav_order: 9
---

# Handwashing Detection & Tagging
Guidance for detecting, classifying, and tagging handwashing events on Axis ACAP devices using the existing detection/pose stack, INT8-quantized models, and multi-camera synchronization where needed. Updated with the latest camera-based hand-washing recognition research (datasets, models, and deployment lessons) to accelerate prototyping.

---

## Objectives & Scenarios
- **Compliance monitoring:** Verify that staff wash hands for the required duration and sequence (wet → soap → scrub → rinse → dry).
- **Occupancy-aware alerts:** Trigger reminders when a person leaves a wash zone without completing the sequence.
- **Cross-camera coverage:** Support single camera above a sink or synchronized multi-camera layouts (entrance + sink + exit).

---

## Research Highlights (camera-based handwashing)
- **Two-stage perception remains the most reliable pattern**: person/hand detection + temporal classifier (hand crops, pose, or flow) outperforms single-frame labeling when sinks, sleeves, or mirrors occlude the hands.
- **Temporal context matters**: 3–5 second windows with overlap and sliding-window smoothing cut flicker and help judge WHO step order and timing.
- **State granularity**: Track at least **washing vs. not washing**; richer pipelines cover **wetting → soaping → scrubbing → rinsing → drying** to compute total scrub time and flag skipped steps.
- **Duration reasoning**: Combine per-frame probabilities with a running timer to enforce **20–30 s** minimum washing and **10–15 s** scrubbing before rinse/dry.
- **Occlusion and lighting resilience**: Train with diverse clips (rings/armbands/long nails, gloves vs. bare hands, varied faucets and shadows) and consider ROI masks to ignore mirrors or bystanders. Shadow augmentation and depth data both improve robustness.
- **Privacy-forward design**: Prefer on-camera processing with cropped hands or skeletons, short retention, and event-level metadata export instead of raw video.

---

## Key Datasets (camera-based)
- **PSKUS / Jūrmalas hospital datasets (EDI)**: ~3,000+ hospital episodes at 30 fps from Axis fisheye and pinhole cameras; frame labels include WHO step codes (0–6) plus metadata (rings/nails). Excellent for real-world occlusions and clutter.
- **METC lab dataset**: 213 episodes from a single location with the same WHO labels; useful for lab→hospital generalization tests.
- **Automated Quality Assessment (Arxiv 2011.11383)**: 1,854 videos from nine sinks (AirLive + Axis cameras); frame-level WHO codes, motion-triggered capture, and double annotations for many clips.
- **WHO 292-video dataset**: Five sink backgrounds; 12 WHO steps merged into six classes for training attention-enhanced VGG/ResNet baselines (CSAB model).
- **Kaggle hand-wash set**: ~25 seven-step clips; good for quick baselines but too small for deployment.
- **RealSense surgical depth dataset**: 74 depth sequences (10 surgical steps, 640×480@15 fps) with train/test splits; supports depth-only or dual-modality fusion.
- **Portable51 / Farm23 (shadow)**: Outdoor/indoor videos with moderate/heavy shadows; paired with synthetic shadow augmentation code to harden RGB models.
- **Synthetic Blender dataset**: 96k frames across eight gestures with RGB/depth/masks; helps pretrain when real data is scarce.
- **Multi-camera (IoT Journal 2025)**: Top fisheye + two side HD views for WHO steps with keypoints and durations; illustrates synchronization and skeleton fusion (not yet public).
- **Sensor (UWash smartwatch)**: Inertial data with gesture quality scores; useful for cross-modality ideas even though it is not camera-based.

---

## Data & Modeling Options
- **Inputs:** RGB/fisheye streams at 720p/1080p; synchronize timestamps when using multiple views.
- **Detection backbone:** Reuse lightweight person/hand detectors (e.g., SSD/YOLOv5n) from the [Model Zoo]({{ site.baseurl }}/docs/model-zoo) and quantize to INT8 via the [Quantization guide]({{ site.baseurl }}/docs/quantization).
- **Pose estimation (optional):** MoveNet-style keypoints help capture wrist/hand trajectories; skeletons reduce privacy risk.
- **Temporal classification:**
  - Frame baselines: MobileNetV2/3, Xception, VGG16 variants; keep crops small (224×224 or 320×240) for ACAP.
  - Attention/fusion: VGG16 + CSAB (channel/spatial attention + bilinear pooling) or ResMFuse-Net–style fusion when extra accuracy is needed.
  - Temporal smoothing: GRU/ConvLSTM or majority-vote/weighted-sum over 3–5 s windows to stabilize labels; optical-flow two-streams add modest motion cues.
- **Metadata tags:** Emit JSON records (start/end timestamps, camera ID, track ID, confidence, compliance status) to VAPIX/metadata stream; include per-step durations when available.

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

**Dataset pairing tips:**
- Pretrain on synthetic/clean datasets (Blender, Kaggle, WHO 292) then fine-tune on PSKUS/hospital data for robustness.
- Use shadow augmentation (Portable51/Farm23 repo) to increase tolerance to faucet reflections and outdoor glare.
- Depth or skeleton cues can complement RGB for privacy-sensitive deployments.

---

## Deployment Notes on ACAP
- Use the **native SDK** for minimal latency; container SDK is acceptable for rapid iteration.
- Optimize preprocessing with hardware scalers; avoid CPU-side copies by feeding YUV buffers directly to the detector.
- Target **INT8** for detector/classifier; keep pose model lightweight (e.g., MoveNet lightning) if used.
- For fisheye streams, reuse the [Fisheye]({{ site.baseurl }}/docs/fisheye) dewarp/rotated-window strategy before hand crops.
- Configure **multicam synchronization** (hardware trigger or SynNet) when combining entrance and sink views to avoid temporal drift.
- Budget: Aim for <150 ms capture→tag latency; <5–10 ms per-frame inference on DLPU-backed detectors and <20 ms for temporal heads on CPU/NPU.
- Privacy: favor on-camera processing, ROI cropping, skeleton-only retention, and short-lived buffers to satisfy healthcare policies.

---

## Open-Source Projects & Repos
- **edgeWash (TensorFlow)**: MobileNetV2 baseline, MobileNetV2+GRU, and two-stream RGB+optical-flow; includes PSKUS/METC download scripts and TFLite export for edge deployment.
- **Automated Quality Assessment**: MobileNetV2/Xception models plus a state machine that checks WHO step order and timing; good template for compliance logic.
- **handwash-dataset (WHO)**: Provides WHO 292-video dataset with VGG/ResNet baselines and environment-level splits used by CSAB.
- **MIVIA surgical depth**: Depth-only classification pipelines for RealSense D435 sequences; demonstrates majority-vote/weighted-sum temporal smoothing.
- **Portable51/Farm23 shadow repo**: MobileNetV3 training with synthetic shadow augmentation to harden models to glare and outdoor sinks.
- **UWash (inertial)**: Two-stream UNet-like temporal model for smartwatch IMU; offers ideas for segmentation + quality scoring if you add wrist IMU to cameras.

---

## Gaps & Future Directions
- **Data breadth**: Real-world hospital coverage is still limited; seek additional sinks, lighting, PPE variations, and multi-camera captures.
- **Domain adaptation**: Explore adversarial/contrastive fine-tuning to bridge synthetic/lab data to hospital footage.
- **Multi-camera fusion**: Top + side view skeleton fusion can stabilize timing/step order on cluttered sinks; calibrate fisheye + HD cameras.
- **Quality scoring**: Extend state machines with transformers or ConvLSTM to evaluate repetitions and per-step dwell times automatically.
- **Privacy**: Prefer on-device skeleton/depth inference with no raw video export; document retention policies alongside deployments.

---

## Datasets & Annotation Strategy
- **Capture diversity**: Record multiple sinks, faucet/soap layouts, glove vs. bare hands, lighting shifts, and mirrored backgrounds. Aim for both overhead and oblique angles to match deployment options.
- **Annotations**:
  - Bounding boxes for hands (and optionally persons) at 5–10 fps to bootstrap detection.
  - Clip-level and frame-level labels for washing states to train the temporal classifier.
  - Event-level metadata (start/end, duration, compliance) for QA dashboards.
- **Data balance**: Include near-misses (short scrubs, rinse-only behavior, partial sequences) to reduce false compliance.
- **Augmentations**: Motion blur, specular highlights from faucets, and random occlusions simulate water splash and reflections.
- **Evaluation splits**: Hold out sinks and subjects unseen during training to test generalization.

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
