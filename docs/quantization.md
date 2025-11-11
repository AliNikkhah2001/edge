---
layout: default
title: Quantization & Laptop Emulation
nav_order: 4
---

# Quantization & Laptop Emulation

Typical INT8 impact vs FP32:

| Model | FP32 mAP | INT8 mAP | Drop | Speed Gain |
|---|---|---|---|---|
| SSD-MobV2 | 30.5 | 28.8 | −1.7 | ×2 |
| YOLOv5n | 37.0 | 35.0 | −2.0 | ×1.8 |

```python
import tensorflow as tf
interpreter = tf.lite.Interpreter(model_path="model_int8.tflite")
interpreter.allocate_tensors()
```