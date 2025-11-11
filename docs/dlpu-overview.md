---
layout: default
title: DLPUs Explained (Architectures & ACAP specifics)
nav_order: 2.9
---

# Deep Learning Processing Units (DLPUs)

**DLPUs/NPUs** are many-core or spatial accelerators optimized for NN inference using low-precision arithmetic (INT8/FP16). They deliver far higher perf/W than CPUs.

## Where used
- Smartphones/edge devices (NPUs on SoCs)
- Embedded accelerators (Google **Edge TPU**, AMD/Xilinx **DPU**)
- Data centers (cloud TPUs, etc.)

## Architectures (examples)
- **AMD/Xilinx DPU (Vitis AI)**: matrix of heterogeneous processing engines; micro-coded ISA; scalable; multiple instances; runs compiled **.xmodel**.
- **Google Edge TPU**: 4 TOPS @ ~2W; runs compiled **TFLite INT8**; used by Coral devices.

## ACAP specifics
- SoCs with DLPUs: **ARTPEC-7/8/9**, **CV25**.
- Formats/quantization:
  - CPU: TFLite (no quant)
  - **ARTPEC-7**: TFLite **compiled with Edge TPU compiler**, INT8
  - **ARTPEC-8**: TFLite INT8 (**per-tensor** preferred)
  - **ARTPEC-9**: TFLite INT8
  - **CV25**: TFLite/ONNX → Ambarella proprietary compiler
- ARTPEC-7 has Edge TPU heritage; ARTPEC-8/9 execute TFLite directly.

**Docs:**  
- Axis DLPU + conversion: <https://developer.axis.com/computer-vision/computer-vision-on-device/axis-dlpu/>  
- Quantization tips: <https://developer.axis.com/computer-vision/computer-vision-on-device/quantization/>  
- DLPU model conversion: <https://developer.axis.com/computer-vision/computer-vision-on-device/dlpu-model-conversion/>