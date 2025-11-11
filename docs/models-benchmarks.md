---
layout: default
title: Recommended Models & Benchmarks
nav_order: 2.8
---

# Recommended Models & Benchmarks (Human Detection / Pose)

## Architectural constraints
- DLPU executes **TFLite INT8 built-ins** only; unsupported ops fall back to CPU.
- Prefer lightweight backbones (**MobileNet v2**, **MobileDet**, **EfficientNet-Lite**).  
- ARTPEC-8 prefers **per-tensor** quantization; ARTPEC-9: tensors ≤ 4D, batch=1; prefer **ReLU6**; filters divisible by 16 (per Axis tips).

## Recommended detectors
- **SSD MobileNet v2**
- **SSDLite MobileDet**
- **YOLOv5 (n/s/m)** on ARTPEC-8/9

## Benchmarks (Axis Model Zoo excerpts)
| SoC/Camera | Model | Latency (ms) | mAP/Top-1 | Notes |
|---|---|---|---|---|
| ARTPEC-7 (Q1615 Mk III) | SSD-MobV2 | 17.81 | 25.6 mAP | Fastest on A7 |
|  | SSDLite MobileDet | 31.16 | 32.9 mAP | More accurate |
| ARTPEC-8 (P1465-LE) | SSD-MobV2 | 28.04 | 25.6 mAP | 300×300 input |
|  | SSDLite MobileDet | 38.93 | 32.9 mAP |  |
|  | YOLOv5n-Artpec8 | 100.05 | 23.5 mAP | small YOLO |
| ARTPEC-8 (Q1656-LE) | SSD-MobV2 | 17.67 | 25.6 mAP | newer HW |
|  | SSDLite MobileDet | 28.40 | 32.9 mAP |  |
|  | YOLOv5n-Artpec8 | 55.52 | 23.5 mAP |  |
|  | YOLOv5s-Artpec8 | 69.87 | 32.3 mAP |  |
|  | YOLOv5m-Artpec8 | 95.34 | 37.9 mAP |  |
| ARTPEC-9 (Q1728) | SSD-MobV2 | 10.35 | 25.6 mAP | fastest |
|  | SSDLite MobileDet | 21.00 | 32.9 mAP |  |
|  | YOLOv5n-Artpec9 | 42.59 | 23.3 mAP |  |
|  | YOLOv5s-Artpec9 | 45.54 | 32.2 mAP |  |
|  | YOLOv5m-Artpec9 | 53.22 | 38.1 mAP |  |
| CV25 (M3085-V) | SSD-MobV2 | 5.36 | — | Ambarella DLPU |
|  | EfficientNet-Lite0 (cls) | 6.75 | 71.2% Top-1 | classification |

**Pose estimation**: **MoveNet SinglePose Lightning (192×192)** on ARTPEC-8; 17 keypoints + confidences. Multi-container example (inference + Flask server).

**Docs:**  
- Model Zoo: <https://github.com/AxisCommunications/axis-model-zoo>  
- Pose example: <https://raw.githubusercontent.com/AxisCommunications/acap-computer-vision-sdk-examples/main/pose-estimator-with-flask/README.md>