---
layout: default
title: Model Adaptation: Fisheye Tracking & Quantization Emulation
nav_order: 3.2
---

# Model Adaptation with Axis Model Zoo: Fisheye Tracking & Quantization Emulation

This page shows how to (1) adapt **Axis model-zoo** detectors for **fisheye** cameras and (2) **emulate INT8 quantization** on a laptop to predict on-camera performance. It expands on practical design choices, ACAP integration notes, and test methodology.

---

## 1) Using model-zoo detectors to build a fisheye tracking application

### Why fisheye is hard
Fisheye lenses have strong **radial distortion**: straight lines bend; objects orient radially; scale and aspect vary with radius. Most detectors assume perspective images and axis-aligned boxes, so accuracy can drop on full fisheye frames. Prior work on overhead fisheye **people counting** reported poor results with off-the-shelf YOLOv3 on full frames, but achieved **large gains** by processing **rotated crops/windows** that locally approximate a perspective view, then merging detections [1][2].

**Key takeaway:** You can often use **existing model-zoo detectors without retraining** if you add fisheye-aware **pre/post-processing**.

---

### Detector selection (Axis model zoo)
Choose the detector based on your SoC/FPS/accuracy goals:

| Family               | Typical mAP (COCO) | Typical latency on-camera | Notes |
|----------------------|--------------------|---------------------------|-------|
| SSD MobileNet v2     | ~25–26 mAP         | ~10–18 ms (ARTPEC-9/8)    | Fastest, best for high FPS [3][4] |
| SSDLite MobileDet    | ~33 mAP            | ~21–39 ms                 | Better accuracy, moderate speed [5][6] |
| YOLOv5n/s/m          | ~23–38 mAP         | ~40–100 ms (A9 faster, A8 slower) | Highest accuracy, lowest FPS [7] |

> Tip: Start with **SSD-MobileNet v2** for real-time, then evaluate **SSDLite MobileDet** or **YOLOv5s/m** if accuracy is insufficient.

---

### Fisheye pre-processing options

1. **Dewarping (rectification)**
   - Use calibration (intrinsics + distortion) to create a perspective view (or panoramas).
   - Pros: detector sees standard images; simpler merging.
   - Cons: reduced FOV or resolution in periphery; calibration required.

2. **Rotated window approach** (no retraining) [2]
   - Partition the circular image into **K wedges** (e.g., 12–24) or overlapping rings+sectors.
   - For each sector:
     - Crop the sector; **rotate** so radial lines become vertical.
     - Resize to detector input (e.g., 300x300 for SSD-MobV2).
     - Run detector; **project boxes back** into original fisheye coordinates.
   - Merge detections across sectors (NMS/soft-NMS in global image space).

3. **Rotation-aware detection** (optional, custom)
   - Models such as **RAPiD** (YOLOv3 variant) output **oriented boxes** and can pair with a rotation-aware SORT tracker (e.g., Similari). This approach can perform well on fisheye but **requires training** and is **not** part of the Axis zoo [8].

---

### End-to-end tracking pipeline

**Steps**

1. **Pick a detector** (table above).
2. **Pre-process frames**:
   - *Option A:* Dewarp to perspective before detection.
   - *Option B:* Rotated windows; rotate each crop; detect; back-project.
3. **Detect** (per frame):
   - On Axis cameras: integrate the model-zoo **.tflite** with ACAP; run via **larod** on the DLPU; receive boxes + confidences.
   - On a laptop: use **TensorFlow Lite** interpreter for the same .tflite.
4. **Track**:
   - Use **SORT** (fast Kalman + Hungarian), **DeepSORT** (adds appearance features), or **ByteTrack** (recovers low-score detections).
   - For oriented boxes (if using rotation-aware detector), extend tracker state with angle and use angle-aware IoU [8][9].
5. **Post-process & visualize**:
   - Merge overlapping sector results; apply NMS/soft-NMS globally.
   - Smooth trajectories; drop short-lived tracks; draw boxes/IDs on original fisheye image.

**Where tracking runs**
- **On ACAP**: Detector runs on **DLPU**; tracker runs on **CPU** (C/C++ recommended for performance; Python viable in container SDK with care). Use **VDO** API for capture and **Overlay** API for drawing.
- **On a laptop**: Prototype in Python (TFLite + OpenCV + Ultralytics/Similari). Use a fisheye dataset (e.g., **WEPDTOF**) to validate.

**Merging detections from rotated windows (outline)**
- Transform window boxes back to global coordinates via inverse rotation + crop offset.
- Keep class/score; clamp to image bounds.
- Run **global NMS** (class-wise) with IoU in original fisheye space.
- Optional: favor central sectors (less distortion) by adding small score bias before NMS.

---

## 2) Emulating quantization effects on a laptop

Axis DLPUs require **INT8** models (CV25 uses a proprietary compiler; ARTPEC-7 needs Edge TPU-compiled TFLite; ARTPEC-8/9 use TFLite INT8). INT8 cuts memory and increases throughput at a small accuracy cost. In the model zoo, **SSD-MobileNet v2** INT8 achieves ~17 ms on ARTPEC-8, while **YOLOv5m** can be ~95 ms [10][11].

### Convert to TFLite INT8 (post-training quantization)

> Use a **representative dataset** of real frames to calibrate activation ranges.

```python
import tensorflow as tf

converter = tf.lite.TFLiteConverter.from_saved_model("path/to/saved_model")
converter.optimizations = [tf.lite.Optimize.DEFAULT]

def representative_data_gen():
    # Yield ~100 samples of preprocessed input tensors
    for input_value in my_dataset.take(100):
        yield [input_value]

converter.representative_dataset = representative_data_gen
converter.target_spec.supported_ops = [tf.lite.OpsSet.TFLITE_BUILTINS_INT8]
converter.inference_input_type = tf.int8
converter.inference_output_type = tf.int8

tflite_int8 = converter.convert()
open("model_int8.tflite", "wb").write(tflite_int8)
```

**Notes**
- For **ARTPEC-8**, **per-tensor** quantization is recommended for best throughput; if accuracy drops too much, explore **QAT** or per-channel where supported by SoC/tooling.
- Keep **preprocessing** identical (resize, mean/std, channel order) across FP32 and INT8 to isolate quantization effects.

### Benchmark FP32 vs INT8 latency with TFLite

```python
import time, numpy as np, tensorflow as tf

def mean_latency_ms(interpreter, shape, dtype, runs=100, warmup=5):
    interpreter.allocate_tensors()
    inp = interpreter.get_input_details()[0]
    dummy = np.zeros(shape, dtype=dtype)

    for _ in range(warmup):
        interpreter.set_tensor(inp["index"], dummy); interpreter.invoke()

    t0 = time.perf_counter()
    for _ in range(runs):
        interpreter.set_tensor(inp["index"], dummy); interpreter.invoke()
    t1 = time.perf_counter()
    return (t1 - t0) / runs * 1000.0

fp32 = tf.lite.Interpreter("model_fp32.tflite")
int8 = tf.lite.Interpreter("model_int8.tflite")

shape = fp32.get_input_details()[0]["shape"]
dtype_fp32 = fp32.get_input_details()[0]["dtype"]
dtype_int8 = int8.get_input_details()[0]["dtype"]

print("fp32 ms:", mean_latency_ms(fp32, shape, dtype_fp32))
print("int8 ms:", mean_latency_ms(int8, shape, dtype_int8))
```

> Replace the dummy tensor with **real frames** to get representative timings. Expect INT8 to be **2–4x faster** and the .tflite file to be ~**4x smaller** vs FP32 (ballpark; HW dependent).

### Measure accuracy drop
- Run both models on a **validation set** and compute mAP/accuracy.
- Typical INT8 loss is **<2%** with good calibration (per-channel often best for accuracy; per-tensor may be preferred on ARTPEC-8 for speed). If loss is unacceptable:
  - Use **Quantization-Aware Training**.
  - Try different input scales or more/better calibration images.
  - Consider **SSDLite MobileDet** or **YOLOv5s** if SSD-MobV2 misses targets.

### Optional: quantize **embeddings** (not weights)
If you use **re-ID** embeddings for tracking (e.g., DeepSORT), memory can dominate. **Vector quantization** (scalar or product quantization) compresses float vectors to int8 codes, often cutting memory ~75% with small retrieval loss [12]. This optimizes **tracker state** (ANN search), not the detector.

---

## ACAP vs Laptop: development workflow

**On Axis ACAP**
- **Detector**: model-zoo TFLite INT8 runs on DLPU via **larod**.
- **Tracker**: run on **CPU** (C/C++ preferred). For container SDK, Python is possible but monitor CPU.
- **I/O**: **VDO** for capture, **Event API** for outputs, **Overlay** for drawing, **Edge Storage** for clips.
- **Fisheye handling**: implement dewarp or rotated windows in C/C++ (SIMD) or via **GStreamer** plugins to contain CPU usage.

**On Laptop**
- Prototype in Python (TFLite, OpenCV, Ultralytics/Similari).
- Use a fisheye dataset (e.g., **WEPDTOF**) and your camera footage.
- Keep interfaces (preprocess, postprocess, NMS, tracker) identical so porting is mechanical.

---

## Minimal pipeline sketch (pseudocode)

```python
# preprocess: dewarp() or for each sector: rotate_crop()
for frame in frames():
    dets = []
    for sector in sectors(frame):
        crop = rotate_crop(frame, sector)
        d = tflite_detect(crop)                 # on laptop; on ACAP use larod
        d_global = back_project(d, sector)      # inverse rotation + offset
        dets.extend(d_global)

    dets = global_nms(dets, iou_thr=0.5)
    tracks = tracker.update(dets)               # SORT/DeepSORT/ByteTrack
    draw_tracks(frame, tracks)
```

Performance knobs:
- Number of sectors (K), overlap, input size.
- NMS type (soft-NMS reduces fragmentation across sector borders).
- Tracker gating (Mahalanobis and IoU thresholds) and lost-track timeouts.

---

## Summary

- **Fisheye tracking:** Handle distortion via **dewarp** or **rotated windows**. Reuse Axis **model-zoo** detectors without retraining; merge detections globally and track with **SORT/DeepSORT/ByteTrack**. Rotation-aware models/trackers help but are optional [1][2][8][9][13].
- **Quantization emulation:** Convert to **TFLite INT8**, benchmark vs FP32 on a laptop, and measure accuracy. Expect **smaller, faster** models with modest accuracy cost. For ARTPEC-8, favor **per-tensor** for speed; use QAT or per-channel if accuracy loss is too high [3][4][5][6][7][10][11].
- **Embeddings:** For re-ID, **vector/product quantization** reduces RAM/latency for large galleries; this is separate from model weight quantization [12].

---

## References

[1] [2] People counting using overhead fisheye cameras | Visual Information Processing  
<https://vip.bu.edu/projects/vsns/cossy/fisheye/li-et-al/>

[3] [4] [5] [6] [7] [10] [11] GitHub – AxisCommunications/axis-model-zoo  
<https://github.com/AxisCommunications/axis-model-zoo>

[8] [9] [13] Rotated Objects Tracking With Angle-Aware Detection Model and SORT Tracker  
<https://blog.savant-ai.io/rotated-objects-tracking-with-angle-aware-detection-model-and-sort-tracker-42a96429898d>

[12] What is Vector Quantization? – Qdrant  
<https://qdrant.tech/articles/what-is-vector-quantization/>
