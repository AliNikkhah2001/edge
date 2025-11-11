---
layout: default
title: ACAP v12 Programming Guide
nav_order: 2.5
---

# Programming on Axis Cameras with **ACAP v12**

ACAP (Axis Camera Application Platform) lets you run applications **on the camera**, reducing latency and bandwidth by processing data at the edge. In **v12**, the SDK is distributed as a **Docker container** that works on Linux/Windows/macOS, decoupling the dev environment from the host OS. Axis OS versions align with ACAP (AXIS OS **12.x ⇔ ACAP v12**) [1][2].

---

## Overview of ACAP v12
- **Container-based SDK**: Distributed as a Docker image; supports Linux, Windows, and macOS hosts [1].
- **Axis OS alignment**: Devices running **AXIS OS 12.x** use **ACAP v12** [2].

---

## Native SDK and Supported Languages
- **Native SDK**: Primary path for on-camera “native” apps; C/C++ with `gcc/g++`, Makefile-based build, sample code included [3].
- **Shell scripts**: Allowed as apps but still require a (possibly empty) `Makefile` for build/install steps [3].
- **Higher-level languages**: No direct Python support in the **Native SDK**. If needed, use the **container SDK** (any language runnable in Docker). This guide focuses on the **Native SDK**.

---

## Application Structure & Packaging
- Package format: **`.eap`** (ACAP application package).
- Contents: `manifest.json` (resources, permissions, autostart), compiled binaries, optional web UI assets.
- Runtime: Installed apps execute as a **sandboxed user**.

---

## Core Communication Methods & APIs

### Event API
Send/receive **stateless (pulse)**, **stateful**, and **data** events under Axis and ONVIF namespaces. Subscribe to camera-generated events (e.g., motion detection) or publish custom ones. Supported across ARTPEC-9/8/7/6, Ambarella CV25, and i.MX 6 [4][5].

### Edge Storage API
Enumerate removable/remote storage (SD cards, NAS), subscribe to storage events, and read/write **your own** files on edge storage [6].

### License Key API
Validate device-bound, signed license keys tied to **device ID** and **application ID**. Introduced in Native SDK 1.11; broadly supported [7].

### Parameter API
Persist and retrieve application settings (parameters). Receive callbacks when parameters are updated (e.g., from the camera web UI) to react in-app. Widely supported; existed prior to API v3.0 [8][9].

### Serial Port API
Access the camera’s serial port for external hardware integration (POS systems, sensors, etc.). Supported on ARTPEC-8/7/6, Ambarella CV25/S5L/S5, i.MX 6 [10].

### Video Capture API (VDO)
Capture frames, start/stop streams, set frame rates, and read video metadata for vision workflows. Supported on SoCs with image sensors (ARTPEC-8/7/6, Ambarella CV25/S5L/S5). Note: ACAP 3.0 had known issues (e.g., memory leak on ARTPEC cameras pre-firmware 11.11.65) [11][12].

### Overlay API (Axoverlay)
Draw on-stream overlays (text/graphics) using a Cairo-based wrapper. Supported on ARTPEC-8/7/6 [13].

### HTTP API (AxHTTP)
Expose CGI-like HTTP endpoints: the camera **proxies HTTP requests** to your app via a UNIX socket—useful for configuration UIs or status/metrics endpoints [14].

### VAPIX® (HTTP) access via **D-Bus**
Apps cannot reuse admin credentials. Instead, obtain **ephemeral service-account** credentials via D-Bus:

- Method: `com.axis.HTTPConf1.VAPIXServiceAccounts1.GetCredentials` → returns a **random username/password** valid **only** on loopback `127.0.0.12` [15][16][17].
- Keep creds **in memory**; do **not** store on disk. They disappear after reboot; max **200** service accounts; refresh on app start [18].
- Example flow (using cURL):
  1. Endpoint: `http://127.0.0.12/axis-cgi/applications/list.cgi`
  2. Use returned `username:password` for HTTP Basic Auth [16]
  3. Perform request; camera proxies to real VAPIX with the service account [17][47]

---

## Machine Learning API (**Larod**)

**Larod** is a C library + service that runs deep-learning inferences and image preprocessing on Axis devices, abstracting CPUs, GPUs, and **DLPUs** behind one API. It wraps frameworks like **TensorFlow Lite** and **OpenCL**, so you don’t touch device-specific SDKs [19][20][21].

**Key features** [21][22][23][24][25][26]:
- **Unified API** across accelerators (DLPUs) and CPU; abstracts TFLite/OpenCL.
- **Multi-client arbitration**: several apps can share the accelerator.
- **Async, prioritized jobs**: queue multiple inferences; set priorities.
- **Image preprocessing**: crop/scale/convert formats before inference.
- **Minimal overhead / zero-copy**: pass tensors via file descriptors.
- **Capabilities & security**: probe model I/O, mediate HW access via service.

**SoC support**: ARTPEC-8/7, Ambarella CV25/S5L (DLPU if available, CPU fallback otherwise). Prefer **Larod v2.0 or v3.0**; v1.0 deprecated [27][28].

---

## Deep Learning on Axis Devices: Workflow

1. **Train** your model in a standard framework (e.g., TensorFlow/Keras). Axis provides guidance and a model zoo for recommended architectures [29][30].
2. **Convert** the model to **TensorFlow Lite** (`.tflite`) with appropriate quantization/format per SoC/DLPU:
   - ARTPEC-7: TFLite compiled for Edge TPU (via `edgetpu-compiler`) [31][32].
   - ARTPEC-8/9: **INT8 TFLite** with supported ops only [31][32][36].
   - CV25: proprietary format converted from TFLite/ONNX [31][35].
3. **Integrate** with an ACAP app: capture frames (VDO), run inference (Larod), handle outputs (events, overlays, HTTP/Edge Storage) [33].
4. **Deploy & run**: package into `.eap`, install on camera; inference executes **on-device** for real-time responses [34].

---

## Supported Frameworks & Model Conversion

- **Primary format**: **TensorFlow Lite INT8** for DLPUs (except CV25 format). Only **built-in INT8 ops** are supported (`tf.lite.OpsSet.TFLITE_BUILTINS_INT8`); **no custom ops** [35][36].
- **TensorFlow version** varies by device/firmware; check SBOM for exact version [37].

**Conversion guidance** [38][39][40][41]:
- **TF/Keras → TFLite**: use `TFLiteConverter` with default optimizations, a representative dataset, and `supported_ops=[tf.lite.OpsSet.TFLITE_BUILTINS_INT8]`. For ARTPEC-8, consider disabling per-channel quantization for performance [38][39].
- **ONNX → TFLite**: unofficial; tools like `onnx2tflite` / `onnx2tf` may work but can require debugging [40].
- **PyTorch → TFLite**: not supported directly; export to ONNX first (path is fragile and not recommended) [41].

> **Note:** PyTorch/ONNX models **cannot** run on the DLPU. While you could run Python with PyTorch or ONNX Runtime inside a **Docker container**, it would execute on **CPU only**, without HW acceleration [42][49].

---

## Axis **DLPU** (Deep Learning Processing Unit)

- Several Axis SoC families include a **DLPU**: **ARTPEC-7/8/9** and **CV25** [44].
- DLPUs provide significant speed-ups vs CPU, dependent on model optimization and device specifics (clock/memory) [43][45].
- Devices sharing an SoC have similar inference characteristics. Use the Axis **product selector** to find DLPU-equipped cameras (filter by DLPU) [46][32].

---

## Communication Protocols — Summary

| Mechanism                         | Description                                                                                                                                     | Typical use cases                                                                 |
|---|---|---|
| **VAPIX® HTTP (loopback)**       | Obtain ephemeral service creds via D-Bus; make HTTP requests to `http://127.0.0.12/axis-cgi/...` with Basic Auth. Local only; refresh at start [47]. | Configure camera, list apps, control device features via HTTP.                    |
| **Event API**                    | Send/receive stateless/stateful/data events under Axis/ONVIF namespaces [4].                                                                    | Trigger recordings, emit analytics metadata, react to camera signals.             |
| **Edge Storage API**             | List SD/NAS, subscribe to storage events, read/write your files [6].                                                                            | Persist app data, logs, snapshots/clips on device/NAS.                            |
| **License Key API**              | Validate device-bound, signed licenses [7].                                                                                                     | Protect paid/licensed apps.                                                       |
| **Parameter API**                | Persist settings; get callbacks on changes [8].                                                                                                 | User-configurable settings via camera web UI.                                     |
| **Serial Port API**              | Access physical serial port [10].                                                                                                               | Integrate with POS systems, sensors, external controllers.                        |
| **VDO (Video Capture)**          | Capture frames/streams, control FPS, read metadata [11].                                                                                        | Feed CV models, create snapshots, manage streams.                                 |
| **Overlay (Axoverlay)**          | Draw text/graphics overlays using Cairo [13].                                                                                                   | Bounding boxes, labels, warnings on live video.                                   |
| **HTTP (AxHTTP)**                | CGI-style HTTP: camera proxies requests to your app via UNIX socket [14].                                                                       | Config UIs, lightweight REST/status endpoints.                                    |
| **Larod (ML API)**               | Run on-device inference & preprocessing; async jobs, multi-client arbitration [19][48].                                                         | Real-time detection/classification with TFLite models.                            |

---

## Practical Tips
- Prefer **Larod v2/v3**; avoid v1. Ensure **INT8** quantization and supported TFLite ops.
- For **ARTPEC-8**, evaluate disabling per-channel quantization during conversion if performance regresses [39].
- **Do not** persist VAPIX service credentials; always request via D-Bus at app start [15][18].
- Watch for **VDO** known issues on older firmware; update to ≥ **11.11.65** where applicable [12].

---

## Conclusion
ACAP v12 offers a robust, secure, and efficient platform for **edge AI** and camera-centric applications. The **Native SDK** (C/C++) provides rich APIs for video, storage, events, parameters, overlays, and secure **VAPIX** access via D-Bus. For computer vision, the **Larod** API unifies HW acceleration and CPU fallback, enabling **INT8 TFLite** models to run efficiently on DLPU-equipped cameras. A solid workflow is: **train (TF/Keras) → convert (TFLite INT8) → integrate (Larod + VDO/Event/Overlay) → deploy (.eap)**—yielding real-time analytics with low latency at the edge [36][49].

---

## References

[1] [2] **ACAP version 12 | Axis developer documentation**  
<https://developer.axis.com/acap/>

[3] **Supported languages | Axis developer documentation**  
<https://developer.axis.com/acap/develop/supported-languages/>

[4] [5] [6] [7] **Supported APIs | Axis developer documentation**  
<https://developer.axis.com/acap/api/>

[8] [9] [10] [11] [12] [13] [14] [27] [28] **Supported APIs (ACAP 3) | Axis developer documentation**  
<https://developer.axis.com/acap/3/api/>

[15] [16] [17] [18] [47] **VAPIX access for ACAP applications**  
<https://developer.axis.com/acap/develop/VAPIX-access-for-ACAP-applications/>

[19] [20] [21] [22] [23] [24] [25] [26] [48] **liblarod: Introduction to larod for app developers**  
<https://developer.axis.com/acap/api/src/api/larod/html/md__opt_builder-doc_larod_doc_introduction-for-app-developers.html>

[29] [30] [33] [34] **Develop your own deep learning application**  
<https://developer.axis.com/computer-vision/computer-vision-on-device/develop-your-own-deep-learning-application/>

[31] [32] [43] [44] [45] [46] **Axis Deep Learning Processing Unit (DLPU)**  
<https://developer.axis.com/computer-vision/computer-vision-on-device/axis-dlpu/>

[35] [36] [37] [38] [39] [40] [41] [42] [49] **Supported frameworks**  
<https://developer.axis.com/computer-vision/computer-vision-on-device/supported-frameworks/>

