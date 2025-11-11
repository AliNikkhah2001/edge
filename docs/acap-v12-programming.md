---
layout: default
title: ACAP v12 Programming Guide
nav_order: 2.5
---

# Programming on Axis Cameras with **ACAP v12**

ACAP (Axis Camera Application Platform) lets you run applications **on the camera**. In **v12**, the SDK is distributed as a **Docker container** that works on Linux/Windows/macOS; Axis OS versions align with ACAP (AXIS OS 12.x ⇔ ACAP v12).

## Native SDK and Languages
- **Native SDK**: C/C++ with `gcc/g++`; apps are built via **Makefile**. Shell script apps are allowed (still need an empty Makefile).
- Higher-level languages (e.g., **Python**) are **not** supported by the Native SDK. If needed, use the **container SDK** (runs any Docker-capable language), but this guide focuses on native.

## Application Packaging
- Apps are packaged as **`.eap`**: includes `manifest.json`, binaries, and any web UI files.
- The manifest defines resources (D-Bus methods, params, events), autostart, and permissions.
- Installed apps run sandboxed.

## Communication & APIs (overview)
Use the Axis APIs to interact with the camera and network:
- **Event API**: send/receive **stateless/stateful/data** events (Axis + ONVIF namespaces).
- **Edge Storage API**: list SD/NAS, subscribe to storage events, read/write your files.
- **License Key API**: validate device-bound, signed licenses.
- **Parameter API**: persist settings; receive callbacks when values change.
- **Serial Port API**: talk to attached devices (POS, sensors).
- **VDO (Video Capture)**: capture frames, start/stop streams, change framerate, read metadata.
- **Overlay (Axoverlay)**: draw text/graphics on streams (Cairo-based).
- **HTTP (AxHTTP)**: camera proxies HTTP requests to your app (CGI-like).
- **VAPIX via D-Bus**: secure service-account credentials over D-Bus; issue HTTP calls over **loopback (127.0.0.12)**.

See the dedicated **[APIs Summary](./apis)** page for details and links.

## Notes on Axis OS / ACAP alignment
- Cameras running **AXIS OS 12.x** use **ACAP v12** (container-based SDK for development).

## References
- Axis ACAP v12 Overview: <https://developer.axis.com/acap/>
- Supported languages: <https://developer.axis.com/acap/develop/supported-languages/>
- Supported APIs (ACAP 3 API index): <https://developer.axis.com/acap/3/api/>
- VAPIX access for ACAP apps: <https://developer.axis.com/acap/develop/VAPIX-access-for-ACAP-applications/>