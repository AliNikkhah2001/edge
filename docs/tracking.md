---
layout: default
title: Online Tracking (SORT, DeepSORT, ByteTrack)
nav_order: 5
---

# Online Tracking (SORT, DeepSORT, ByteTrack)

| Tracker | Method | Strengths | Weaknesses |
|---|---|---|---|
| SORT | Kalman + IoU | Simple, fast | Occlusions → ID switches |
| DeepSORT | + Re-ID | Occlusion robust | Heavier compute |
| ByteTrack | 2-pass conf association | Real-time, high MOTA/IDF1 | Slightly higher load |

**ByteTrack** (ECCV 2022): MOTA ≈ 80.3%, IDF1 ≈ 77.3%