---
layout: default
title: Larod (ML API) & On-Device DL Pipeline
nav_order: 2.7
---

# Larod (Machine Learning API) and On-Device DL

**Larod** is Axis' C library + service that runs DL inference and image pre-processing on device, multiplexing CPU/GPU/**DLPU** backends behind a single API.

## Key Features
- **Unified** API (abstracts TensorFlow Lite / OpenCL backends).
- **Multi-client**: several apps can share the accelerator.
- **Async + priorities**: queue multiple jobs; prioritize critical tasks.
- **Image preproc**: crop/scale/format conversion built-in.
- **Zero-copy I/O**: tensors via file descriptors to minimize copies.
- **Capability probing + security** via service mediation.

Supports ARTPEC-8/7 and Ambarella CV25/S5L families. Use **Larod v2/3** (v1 is deprecated).

## Deployment pipeline (Axis guidance)
1. **Train** a model in TensorFlow/Keras (choose DLPU-friendly layers).
2. **Convert** to **TFLite INT8**. DLPU supports **TFLITE_BUILTINS_INT8** ops only (no custom ops).  
   - ARTPEC-7: TFLite **compiled with edgetpu-compiler**.  
   - ARTPEC-8: TFLite **per-tensor quantization** (often faster than per-channel).  
   - ARTPEC-9: TFLite INT8 (per-tensor or per-channel).  
   - CV25: proprietary compiler from TFLite/ONNX.
3. **Integrate**: ACAP app captures frames, runs inference via **Larod**, emits events/overlays.
4. **Deploy**: package as `.eap` and install.

**Docs:**  
- Larod intro: <https://developer.axis.com/acap/api/src/api/larod/html/md__opt_builder-doc_larod_doc_introduction-for-app-developers.html>  
- DLPU conversion: <https://developer.axis.com/computer-vision/computer-vision-on-device/dlpu-model-conversion/>  
- Supported frameworks: <https://developer.axis.com/computer-vision/computer-vision-on-device/supported-frameworks/>