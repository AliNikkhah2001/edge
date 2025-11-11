---
layout: default
title: Cross-Camera Identity Matching
parent: Multi-Camera Sync & Cross-Camera ID
nav_order: 2
---

# Cross-Camera Identity Matching

| Method | Type | Idea | Notes |
|---|---|---|---|
| TransReID | Transformer Re-ID | Appearance embeddings | Fuse with motion |
| LMGP | Graph | Lifted multicut + geometry | High precision |
| AGC + STC | Clustering | Anchor + spatio-temporal reassignment | IDF1 ~95% |
| CrossMOT / DIVOTrack | End-to-end | Joint detection + embedding | Cross-view |
| MCTR / GMT | Transformer | Global association | Next-gen |