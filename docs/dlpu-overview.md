---
layout: default
title: DLPUs Explained (Architectures & ACAP specifics)
nav_order: 2.9
---

# Deep Learning Processing Units (DLPUs)

**Deep-Learning Processing Units (DLPUs)** — also called NPUs or AI accelerators — are purpose-built engines for accelerating neural-network **inference** (and sometimes training). They use many-core/spatial architectures, low-precision arithmetic (e.g., **INT4/INT8/FP16**) and custom data-flow to deliver far higher performance-per-watt than general-purpose CPUs/GPUs, within tight power budgets [1]. They’re found in phones, edge devices, and data centers, and are a key enabler for on-device AI in Axis **ACAP** cameras.

---

## Introduction

DLPUs focus on matrix multiplications and convolutions that dominate modern deep learning. Their custom instruction sets and dataflows allow massive parallelism while keeping DRAM traffic low via on-chip memories and caches. As a result, they can run compact, quantized models extremely efficiently compared to CPUs/GPUs of similar power envelopes [1][2].

**Where DLPUs are used**  
- **Consumer devices**: smartphone/tablet/camera NPUs for on-device vision & language models [2].  
- **Cloud**: datacenter-class accelerators (e.g., Google **TPU**) for high-throughput inference/training [2].  
- **Embedded/edge**: dedicated chips such as **Google Edge TPU** or **AMD/Xilinx DPU** to bring AI to sensors, robotics, IoT, and cameras [2].  
- **Axis ACAP**: DLPU-equipped camera SoCs execute deep-learning analytics locally.

---

## Hardware architecture of DLPUs

### General principles
Deep-learning chips center around large arrays of **multiply–accumulate** units plus specialized dataflows. Many adopt **manycore** or **spatial** designs with **low-precision** arithmetic to raise throughput and lower power. Common kernels (conv, GEMM, activations) execute in parallel across hundreds/thousands of processing elements. **On-chip SRAM** and caches reduce expensive off-chip accesses [2].

### AMD/Xilinx **DPU** architecture
Key properties of the AMD/Xilinx **DPU** IP core (for FPGA/SoCs) [3][4][5][6][7][8][9]:

- **Matrix of (Heterogeneous) Processing Engines**: micro-coded processor with its own ISA; engines specialize in ops (e.g., 2-D conv vs. depthwise conv) [3][4][5].  
- **Micro-coded execution**: models compiled by **Vitis-AI** to an **.xmodel**; no FPGA re-bitstreaming needed for new networks [6].  
- **Scalable & multi-stream**: instantiate multiple DPUs; runtime schedules multiple networks/streams across them; DPU size is tunable vs. resources [7].  
- **Heterogeneous resources**: leverages DSPs, BRAM/URAM, LUT/FF; **Versal AI Core** adds AI Engine arrays for conv & low-latency on-chip memory [8].  
- **Data-flow alternatives**: **FINN** & **Brevitas** enable fully data-flow designs for extreme throughput niches [9].

### Google **Edge TPU**
A small, low-power ASIC co-processor for **TensorFlow Lite** INT8 models. Delivers **~4 TOPS at ~2 W**, enabling MobileNetV2 at ~**400 FPS** per Coral docs; connects to ARM hosts via PCIe/I²C/GPIO; relies on **integer quantization** [10][11].

---

## Axis DLPUs in **ACAP** cameras

Axis integrates DLPUs into camera SoCs to accelerate CV inference:

- **Dedicated accelerator**: the DLPU significantly outpaces CPU inference [12].  
- **SoCs with DLPUs**: **ARTPEC-7**, **ARTPEC-8**, **ARTPEC-9**, **Ambarella CV25** [13]. Devices sharing a SoC tend to have similar compatibility; perf still varies with clocks/memory [14].  
- **Model formats & quantization** (Axis summary) [15][16]:  
  - **CPU**: TFLite (no quantization).  
  - **ARTPEC-7**: TFLite **compiled with Edge TPU compiler**, **INT8**.  
  - **ARTPEC-8**: TFLite **INT8**, **per-tensor** quantization recommended/required for best perf.  
  - **ARTPEC-9**: TFLite **INT8** (per-channel or per-tensor supported).  
  - **CV25**: TFLite or ONNX, compiled via **Ambarella** toolchain (quantization in compiler).  
- **Edge TPU heritage**: **ARTPEC-7** DLPU is Edge-TPU-based → models must be **Edge TPU-compiled** [17]; **ARTPEC-8/9** execute TFLite directly (no Edge-TPU compile) [18]; **CV25** uses Ambarella compiler [19].

---

## Software SDKs for DLPUs

### AMD/Xilinx **Vitis AI** (for DPU)
Train in TF/PyTorch → **quantize & compile** with Vitis AI to **.xmodel** → deploy via **Vitis AI runtime**, which can schedule multiple models/streams; model zoo & ONNX Runtime integrations available [20][21].

### Coral / **Edge TPU** toolchain
Train in TensorFlow → quantize to **INT8** → convert to **TFLite** → compile with **Edge TPU compiler** → run via **libcoral / PyCoral** C++/Python APIs (also applicable for ARTPEC-7) [17].

### Axis **ACAP** toolchain
Typical pipeline [16][17][19]:

1. **Train** (e.g., TensorFlow).  
2. **Quantize & export** to TFLite with SoC-specific guidance (e.g., **per-tensor for ARTPEC-8**; per-channel/per-tensor viable on **ARTPEC-7/9**; **CV25** quantizes in its compiler) [16].  
3. **Compile**: Edge-TPU compiler for **ARTPEC-7**; **TFLite direct** for **ARTPEC-8/9**; **Ambarella** proprietary toolchain for **CV25** [17][18][19].  
4. **Deploy** within an **ACAP** app (C SDK or Python wrapper). Inference runs via **larod**; CPU can handle post-processing/tracking. Axis provides examples and a **model zoo** tuned per DLPU. Check the **SoC** of your device before choosing models [26].

---

## Where DLPUs are used (broader context)

- **Consumer**: NPUs in Apple/Qualcomm/Samsung/Google mobile SoCs for vision, speech, and on-device generative AI (TOPS-class within phone power limits) [2].  
- **Edge/IoT**: Coral Dev Board & modules with **Edge TPU** (≈4 TOPS @ 2 W, MobileNetV2 ≈ 400 FPS) for sensor-edge inference [10][11].  
- **FPGAs/adaptive SoCs**: **AMD/Xilinx DPU** on Zynq UltraScale+ and Versal AI Core/Edge for robotics/industrial/datacenter use cases [21].  
- **Datacenter**: Cloud TPU/AWS Trainium et al. (not covered in depth here) follow similar low-precision, matrix-centric designs [1].

---

## DLPUs on **ACAP** cameras (developer notes)

- **SoC families**: ARTPEC-7/8/9, CV25 — each with distinct **format/quantization** rules [15].  
- **Quantization**: DLPUs are **integer** accelerators, so quantization is mandatory.  
  - **ARTPEC-8**: **per-tensor** quantization is **mandatory for optimal performance**.  
  - **ARTPEC-7/9**: **per-channel** supported for accuracy gains [16][22].  
  - **CV25**: quantization handled by the **Ambarella** compiler [23].  
- **Model conversion** (cheat-sheet):  
  - **A7**: TF → TFLite INT8 → **Edge TPU compiler** [17].  
  - **A8**: TF → TFLite INT8 (**disable per-channel in TF2**) [24].  
  - **A9**: TF → TFLite INT8 (no special compile) [25].  
  - **CV25**: TF/ONNX → **Ambarella** toolchain [19].  
- **SDK pipeline**: integrate with **larod** for inference; use ACAP APIs for capture, events, storage, HTTP/VAPIX; do tracking/post-proc on **CPU**. Not all cameras have the same DLPU—**confirm the SoC** before selecting models [26].

---

## Conclusion

DLPUs are specialized, low-precision, highly parallel accelerators that deliver **high inference throughput at low power**.  
- **AMD/Xilinx DPU**: micro-coded, heterogeneous engines fed by the **Vitis AI** toolchain; scalable and multi-stream capable [3].  
- **Google Edge TPU**: compact ASIC (~**4 TOPS @ 2 W**) for **TFLite INT8** models [10][11].  
- **Axis ACAP**: integrates DLPUs across **ARTPEC-7/8/9** and **CV25** SoCs to execute deep-learning tasks on-camera. Each SoC has specific **model-format & quantization** requirements; models must be **INT8** and compiled with the appropriate toolchain (Edge-TPU compiler for **A7**, TFLite per-tensor for **A8**, TFLite for **A9**, Ambarella toolchain for **CV25**) [15][17][18][19].  
With the right model design and quantization, ACAP cameras can run detection/classification **in real time** while maintaining low power.

---

## References

[1] [2] **Neural processing unit – Wikipedia**  
<https://en.wikipedia.org/wiki/Neural_processing_unit>

[3] [4] [5] [6] [7] [8] [9] [20] [21] **DPU IP Details and System Integration — Vitis™ AI 3.0 documentation**  
<https://xilinx.github.io/Vitis-AI/3.0/html/docs/workflow-system-integration.html>

[10] [11] **Dev Board datasheet | Coral**  
<https://www.coral.ai/docs/dev-board/datasheet/>

[12] [13] [14] [15] [26] **Axis Deep Learning Processing Unit (DLPU) | Axis developer documentation**  
<https://developer.axis.com/computer-vision/computer-vision-on-device/axis-dlpu/>

[16] [22] [23] **Quantization | Axis developer documentation**  
<https://developer.axis.com/computer-vision/computer-vision-on-device/quantization/>

[17] [18] [19] [24] [25] **Deep Learning Processing Unit (DLPU) model conversion | Axis developer documentation**  
<https://developer.axis.com/computer-vision/computer-vision-on-device/dlpu-model-conversion/>
