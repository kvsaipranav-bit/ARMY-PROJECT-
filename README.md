# Autonomous UGV — Drone Detection & Tracking System

R&D project: an autonomous Unmanned Ground Vehicle (UGV) capable of detecting, tracking, and reporting enemy drone activity using computer vision, thermal imaging, and GPS-based threat display.

---

## 1. Objective

Develop an autonomous UGV capable of:

- Detecting drones at long range using computer vision (OpenCV)
- Integrating thermal imaging for day/night detection
- Tracking a detected drone in real time
- Estimating flight direction and the probable location of the operator / mother drone
- Displaying GPS coordinates and threat information on a UGV interface
- Collecting images, video, and telemetry data for analysis
- Leveraging AI-based object detection models (YOLO family) for improved recognition
- Following a defined system architecture, hardware plan, software stack, and implementation roadmap

---

## 2. System Architecture

The system is organized into seven layers:

1. **Sensing Layer** — RGB camera + thermal camera on a pan-tilt gimbal; optional acoustic array for cueing
2. **Perception Layer** — YOLO-based object detector, fused RGB + thermal input
3. **Tracking Layer** — multi-frame object tracker maintaining identity and predicting trajectory
4. **Direction / Geolocation Layer** — bearing computation from camera angle + UGV GPS/heading; multi-point triangulation for source estimation
5. **Autonomy / Mobility Layer** — ROS2-based navigation and obstacle avoidance
6. **UI / Command Dashboard** — live video feed, GPS map overlay, threat level, telemetry
7. **Data Logging Layer** — local storage of images/video/telemetry, optional cloud sync

```
[RGB Cam]  [Thermal Cam] ---> [Detection: YOLO] ---> [Tracking: ByteTrack/DeepSORT]
     |             |                                          |
     +--> [Gimbal Control] <--------------------+--------------+
                                                 |
                                     [Bearing/Direction Estimator]
                                                 |
                    [GPS/IMU] ---> [Geolocation Fusion] ---> [Threat Dashboard]
                                                 |
                                       [Data Logger / Storage]
```

---

## 3. Hardware Requirements

| Component | Recommendation | Purpose |
|---|---|---|
| Compute (edge AI) | NVIDIA Jetson Orin Nano/NX | Runs detection + tracking models on-device |
| RGB Camera | High-zoom machine vision camera | Long-range visual detection |
| Thermal Camera | FLIR Lepton / Boson (LWIR) | Day/night detection |
| Gimbal | 2-axis pan-tilt mount | Camera aiming and tracking |
| Positioning | GNSS/GPS module + IMU + digital compass | UGV location, heading, bearing calc |
| UGV Base | Tracked or skid-steer chassis, motor drivers, LiPo battery | Mobility platform |
| Comms | LoRa/RF module or Wi-Fi mesh | Remote telemetry link |
| Storage | Onboard SSD/eMMC | Local data logging |

---

## 4. Software Stack

| Layer | Tools/Frameworks |
|---|---|
| OS / Middleware | Ubuntu + ROS2 |
| Object Detection | OpenCV, Ultralytics YOLO (YOLO26 recommended — edge-optimized, released Jan 2026; YOLOv11 as a mature fallback) |
| Tracking | ByteTrack or DeepSORT |
| Backend | Python / C++ ROS2 nodes |
| Dashboard/UI | React (remote) or Qt (onboard display) |
| Data Storage | SQLite (telemetry) + tagged image/video archive |
| Model Training | PyTorch, Ultralytics training pipeline |

---

## 5. Direction & Operator-Location Estimation — Design Note

A single UGV can compute a reliable **bearing** to a detected drone from camera pan angle + its own GPS/heading. Estimating the **operator's exact location** from vision alone is a harder problem. Two realistic approaches:

1. **Multi-point triangulation** — take bearings from multiple UGV positions/times and intersect them geometrically to narrow down a probable sector
2. **RF direction-finding** — a separate sensor (not CV-based) that locates the drone's control-link signal

**Recommendation:** report this as an "estimated probable sector," not a precise fix, unless an RF-DF module is added to the system.

---

## 6. Implementation Roadmap

| Phase | Task |
|---|---|
| 1 | Literature review + dataset collection (VisDrone, Anti-UAV, Drone-vs-Bird datasets) |
| 2 | Fine-tune YOLO model on RGB + thermal drone imagery |
| 3 | Integrate detection pipeline with real-time tracking |
| 4 | Build bearing/direction estimation logic |
| 5 | Integrate UGV mobility platform + gimbal control |
| 6 | Develop dashboard/UI for GPS + threat display |
| 7 | Field testing, telemetry data collection, iterate on model accuracy |

---

## 7. Datasets for Model Training

- **VisDrone** — general aerial object detection benchmark
- **Anti-UAV** — dedicated drone detection/tracking dataset (RGB + thermal pairs)
- **Drone-vs-Bird** — helps reduce false positives from birds

---

## 8. Notes

- Edge inference (Jetson-class hardware) is required for real-time performance in the field; cloud-dependent pipelines are not viable for this use case.
- Thermal + RGB fusion significantly improves detection reliability across lighting conditions.
- Direction estimation accuracy scales with the number of independent bearings collected — a single static UGV position gives limited precision.
