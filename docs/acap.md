---
layout: default
title: ACAP Hardware & DLPU
nav_order: 2
has_children: true
---

# ACAP 

Axis ARTPEC SoCs with integrated DLPUs for INT8 TFLite inference.

## Summary Table

| SoC | CPU | DLPU | Memory | Typical Use |
|------|-----|------|--------|-------------|
| ARTPEC-7 | 4×Cortex-A53 (1.2 GHz) | Edge-INT8 | 1 GB / 8 GB | Entry analytics |
| ARTPEC-8 | 4×Cortex-A53 (1.5 GHz) | DLPU v2 (INT8) | 2–4 GB / 8 GB | Edge inference |
| ARTPEC-9 | 8×Cortex-A55 (1.6 GHz) | DLPU v3 (INT8+FP16) | 4 GB / 16 GB | Multi-model |


## 1. ACAP & Axis Camera Application Platform

**Purpose:**  
The **Axis Camera Application Platform (ACAP)** is an SDK and runtime environment that enables computer-vision and video-analytics applications to run directly on Axis network cameras.

**Versions:**
- **ACAP v12** provides:
  - A **Native SDK** (C/C++) for low-level performance.
  - A **Container-based SDK** using Docker (supports Python and other languages).

**Communication Protocols:**
- **HTTP/HTTPS** (via `cURL` or `libcurl`)  
- **MQTT** and **WebSockets**  
- **Event streaming (JSON)**  
- **VAPIX® API** (RESTful HTTP) and **ONVIF** for camera control and streaming  
- **Axis Event System** and **gRPC** for remote procedure calls.

**Data Flow:**
- Applications access video frames via the **Video API**, process them with DL models, and:
  - Overlay results (e.g., bounding boxes).
  - Or send metadata to external systems.
- The container SDK encapsulates dependencies for easy deployment, with a small performance overhead.

---

## 2. Hardware — DLPUs & SoCs

### 2.1 DLPU Overview

**Deep-Learning Processing Units (DLPUs)** are ASICs integrated in Axis SoCs to accelerate inference of **INT8 TensorFlow Lite** models.

**Supported SoCs:**

| SoC | Accelerator Type | Key Features |
|------|------------------|--------------|
| **ARTPEC-7** | Google Edge TPU derivative | Requires Edge TPU compiler; 4 TOPS @ ~2 W; runs MobileNet V2 at ~400 FPS; per-channel quantization recommended. |
| **ARTPEC-8** | Axis DLPU | INT8 TFLite; per-tensor quantization; 3×3 kernels, stride 2, filters ×6. Unsupported ops → CPU fallback. |
| **ARTPEC-9** | Axis DLPU | Larger core; supports per-tensor/per-channel quantization; tensors ≤ 4D; batch = 1. |
| **Ambarella CV25** | Ambarella DLPU | Requires proprietary compiler; RGB planar inputs; strict memory & quantization. |

⚙️ **All DLPUs**: Run quantized models only — non-INT8 ops fall back to CPU (slower).

---

### 2.2 DLPU Software Toolchain

**Model Conversion Workflow**
1. Train in **TensorFlow/Keras**.
2. Convert to **TensorFlow Lite (INT8)**.
3. Use the correct compiler:
   - ARTPEC-7 → **Edge TPU compiler**  
   - ARTPEC-8/9 → **larod-convert** (`disable_per_channel` flag for ARTPEC-8)
   - CV25 → **Ambarella compiler**

**Larod API**
- Native C API to load and run TFLite models on DLPUs.  
- Handles tensor allocation, inference calls, and performance queries.  
- Container SDK exposes Python bindings.

**Third-party Libraries**
- **PyCoral** simplifies Edge TPU inference and multi-TPU pipelines.  
- Distributed as Debian pkg/wheel for x86, Armv7/Armv8.  
- On ACAP, external Python pkgs require cross-compilation or QEMU.  
- PyCoral is not officially packaged for ACAP → manual integration only.

---

## 3. Deep-Learning Models & Deployment on DLPUs

### 3.1 Recommended Architectures

Axis’s **Model Zoo** provides pre-quantized baselines.

| Model | Input | Inference (ARTPEC-7) | Inference (ARTPEC-8) | Inference (ARTPEC-9) | mAP (%) |
|:------|:------|:---------------------|:---------------------|:---------------------|:-------|
| **SSD MobileNet V2** | 300×300 | 17.8 ms | 3.8 ms | 3.0 ms | 25–31 |
| **SSDLite MobileDet** | 320×320 | 31 ms | 6.5 ms | — | 33 |
| **YOLOv5 n/s/m** | 640×640 | — | 10 ms | 9 ms | 35–38 |

- 🧩 **Backbones:** MobileNet V2 (default); ResNet-18 (optional on ARTPEC-8/9).  
- 📷 **Tasks:** Object detection, classification, pose estimation.  
- 🧠 **Limitations:** heavy nets (Transformers, deformable convs) not supported.

---

### 3.2 Tracking Algorithms

Axis cameras **don’t include built-in MOT**—developers implement CPU-side trackers:

| Tracker | Method | Pros | Notes |
|----------|---------|------|------|
| **SORT** | Kalman + Hungarian | Lightweight real-time | ID switches on occlusion |
| **DeepSORT** | SORT + appearance embeddings | Fewer ID switches | Needs extra CNN |
| **ByteTrack** | Keeps low-confidence detections | High MOTA/IDF1 | Slightly slower |

**Digital Autotracking** (Axis-provided) auto-steers PTZ cameras; built-in, non-customizable.

---

## 4. Google Coral & PyCoral on ACAP

| Component | Purpose | ACAP Support |
|------------|----------|--------------|
| **Edge TPU** | 4 TOPS @ 2 W inference ASIC (MobileNet V2 ~400 FPS) | Integrated only in ARTPEC-7 DLPU; requires Edge TPU compiler |
| **PyCoral** | Python API for Edge TPU inference/training | Not pre-installed on Axis OS; cross-compile only |
| **Larod Backend** | `google-edge-tpu-tflite` device handle in Larod | Use for Edge TPU models on ARTPEC-7 |
| **External TPUs** | USB/PCIe Coral modules | Not supported on Axis cameras |

> ARTPEC-8/9 → Axis DLPUs (INT8 TFLite).  
> CV25 → Ambarella CVFlow (proprietary).

---

## 5. Edge vs Server Deployment

| Factor | Edge (ACAP) | Server/Cloud |
|--------|--------------|--------------|
| **Latency** | ms-level; no network delay | Network round-trip adds lag |
| **Bandwidth Cost** | Low (local processing) | High (video upload) |
| **Privacy** | On-device data | Requires off-site transfer |
| **Scalability** | Limited by hardware | Elastic cloud compute |
| **Model Complexity** | MobileNet, ResNet-18, INT8 only | Large ViTs, 3D CNNs possible |

> Axis guidelines: use **quantized lightweight models**; transformers or custom ops won’t run on DLPUs.

---

## 6. Practical Recommendations

1. **Select the right SoC:**  
   - ARTPEC-8/9 → complex models, better performance.  
   - ARTPEC-7 → Edge TPU compatibility, simpler nets.  
2. **Train & Quantize correctly:**  
   - Use MobileNet V2 or ResNet-18.  
   - Convert to INT8 TFLite; verify DLPU ops via Larod.  
3. **Tracking:**  
   - Implement CPU trackers (SORT/DeepSORT/ByteTrack).  
   - Fuse with Larod detections and Event API triggers.  
4. **Edge vs Cloud:**  
   - Use edge inference for low-latency and privacy.  
   - Offload heavy analytics to cloud as hybrid pipeline.

---

## 7. Conclusion

Axis ACAP v12 and its DLPUs (ARTPEC-7/8/9 and CV25) enable real-time Edge AI vision.  
Developers should design lightweight INT8 models, deploy via Larod, and combine DLPU detections with CPU-side tracking.  
The balance between edge and cloud defines system responsiveness, privacy, and scalability.

---

## 📚 References

1. [Axis DLPU Documentation](https://developer.axis.com/computer-vision/computer-vision-on-device/axis-dlpu/)  
2. [Coral Edge TPU FAQ](https://www.coral.ai/docs/edgetpu/faq/)  
3. [Optimization Tips | Axis Developer](https://developer.axis.com/computer-vision/computer-vision-on-device/optimization-tips/)  
4. [DLPU Model Conversion Guide](https://developer.axis.com/computer-vision/computer-vision-on-device/dlpu-model-conversion/)  
5. [General Suggestions | Axis Developer](https://developer.axis.com/computer-vision/computer-vision-on-device/general-suggestions/)  
6. [PyCoral API Overview](https://www.coral.ai/docs/reference/py/)  
7. [Axis Model Zoo (GitHub)](https://github.com/AxisCommunications/axis-model-zoo)  
8. [Object Tracking Docs](https://developer.axis.com/analytics/axis-scene-metadata/reference/concepts/object-tracking/)  
9. [Digital Autotracking API](https://developer.axis.com/vapix/applications/digital-autotracking-api/)  
10. [NVIDIA Edge vs Cloud Blog](https://blogs.nvidia.com/blog/difference-between-cloud-and-edge-computing/)  
11. [Coursera Edge AI vs Cloud AI](https://www.coursera.org/articles/edge-ai-vs-cloud-ai)
See also: [Axis Developer Portal](https://developer.axis.com/acap/).