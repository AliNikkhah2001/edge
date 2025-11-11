---
layout: default
title: Axis APIs Summary
nav_order: 2.6
---

# Axis APIs Summary (ACAP)

| API | What it does | Chips / Notes |
|---|---|---|
| **Event API** | Send/receive **stateless/stateful/data** events (Axis + ONVIF). | ARTPEC-9/8/7/6, Ambarella CV25, i.MX6. |
| **Edge Storage API** | Access SD/NAS, subscribe to storage events, read/write app files. | Most devices. |
| **License Key API** | Validate device/app-bound license keys (signed). | Introduced in Native SDK 1.11. |
| **Parameter API** | Persist settings; receive callbacks on changes. | Most devices. |
| **Serial Port API** | Access UART to talk to external gear. | ARTPEC-8/7/6, Ambarella CV25/S5L/S5, i.MX6. |
| **VDO (Video Capture)** | Capture frames, configure streams, metadata. | ARTPEC-8/7/6, CV25/S5L/S5; multiple versions (see docs). |
| **Overlay (Axoverlay)** | Draw text/graphics overlays (Cairo). | ARTPEC-8/7/6. |
| **HTTP (AxHTTP)** | CGI-like HTTP proxy to your app via UNIX socket. | Most devices. |
| **VAPIX via D-Bus** | Get **service-account** creds via D-Bus; call `/axis-cgi/...` on **127.0.0.12** with Basic Auth. | Local-only; ephemeral (max 200 accounts). |

### VAPIX secure loopback (how-to)

1. Call D-Bus: `com.axis.HTTPConf1.VAPIXServiceAccounts1.GetCredentials` → returns **username:password**.  
2. Use loopback endpoint, e.g.: `http://127.0.0.12/axis-cgi/applications/list.cgi` with **Basic Auth**.  
3. Do not persist credentials; refresh on app start; accounts vanish on reboot.

**Docs:**  
- ACAP API index: <https://developer.axis.com/acap/3/api/>  
- VAPIX access for ACAP apps: <https://developer.axis.com/acap/develop/VAPIX-access-for-ACAP-applications/>