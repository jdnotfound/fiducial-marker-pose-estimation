# Vision-Based Robot Localization using Fiducial Markers and OpenCV

This project implements and benchmarks fiducial marker based vision pipelines for robot localization using OpenCV and Linux camera workflows.

A camera detects printed fiducial markers and estimates marker position relative to the camera. The estimated pose or distance can be used for robot localization, alignment, docking, and navigation correction.

This project uses classical computer vision and geometric pose estimation, not deep learning.

## Project Goal

The goal is to identify a marker-based vision pipeline that is reliable enough for distance-aware robot localization and practical enough to run on an ST Linux-based embedded board.

The benchmark compares marker systems based on:

- Detection reliability
- Pose / distance success rate
- Z-distance accuracy
- Straight-line distance accuracy
- Reprojection error where available
- Z-distance jitter
- Distance jitter
- FPS / real-time performance
- Linux integration practicality

## Development Environment

The project was developed in an Ubuntu Linux virtual machine using Oracle VirtualBox.

```text
Project directory: ~/cv-project
Python environment: ~/cv-project/venv
Camera device: /dev/video0
Camera backend: V4L2
IDE/tools: VS Code, nano, Linux terminal
```

The webcam is accessed using OpenCV:

```python
cap = cv2.VideoCapture(0, cv2.CAP_V4L2)
```

The Ubuntu VM was used for camera calibration, marker detection, pose estimation, CSV logging, and plot generation.

## Marker Systems Explored

This project explored **11 fiducial marker systems and variants** during development.

### Fully Benchmarked

These marker systems were implemented using full pose-estimation pipelines and included in the main comparison:

- ArUco
- AprilTag
- STag
- Chilitags

### Extended Benchmarked

These marker systems were also implemented, but their pipelines differ from the main 4-corner OpenCV `solvePnP()` comparison:

- CCTag
- JuMarker UCO

### Attempted but Excluded

These marker systems were explored but not included in the final benchmark due to integration issues, unstable detection, marker generation limitations, or unsuitable Linux/Python workflow:

- TopoTag
- WhyCode / WhyCon
- RUNEtag
- ARToolKitX
- JuMarker Cordoba

## Marker Implementation Summary

| Marker | Status | Implementation | Benchmark type |
|---|---|---|---|
| ArUco | Fully benchmarked | OpenCV ArUco dictionary | 4-corner `solvePnP()` 6-DoF pose |
| AprilTag | Fully benchmarked | OpenCV AprilTag dictionary | 4-corner `solvePnP()` 6-DoF pose |
| STag | Fully benchmarked | `stag-python` detector | 4-corner `solvePnP()` 6-DoF pose |
| Chilitags | Fully benchmarked | External C++ detector exposed through Python wrapper | 4-corner `solvePnP()` 6-DoF pose |
| CCTag | Extended benchmarked | External C++ detector parsed through Python wrapper | Distance estimate from detected ellipse diameter |
| JuMarker UCO | Extended benchmarked | External C++ detector parsed through Python wrapper | Pose values produced by JuMarker C++ detector |

ArUco, AprilTag, STag, and Chilitags form the main full-pose benchmark because they provide marker corners that can be passed into the same `solvePnP()`-based pose-estimation pipeline.

CCTag is treated as a distance-only benchmark because the current implementation estimates distance from the detected ellipse diameter. JuMarker UCO uses pose values produced by its C++ detector and parsed through a Python wrapper.

## Pose Estimation Pipeline

For ArUco, AprilTag, STag, and Chilitags:

```text
Camera frame
→ Marker detection
→ 2D corner extraction
→ Camera calibration + marker size
→ solvePnP()
→ rvec / tvec
→ distance, reprojection, jitter, FPS metrics
```

For CCTag:

```text
Camera frame
→ CCTag detection
→ center + ellipse estimation
→ apparent diameter in pixels
→ distance estimate using calibrated focal length
→ distance error, jitter, FPS metrics
```

For JuMarker UCO:

```text
Camera frame
→ JuMarker UCO detection
→ C++ pose estimation
→ pose values parsed in Python
→ distance error, jitter, FPS metrics
```

## Calibration and Marker Size

All marker systems use the same camera calibration file:

```text
calibration_data.npz
```

The camera calibration resolution and capture resolution must match. Using a different capture resolution from the calibration resolution can cause incorrect distance estimation.

Main square marker size used for ArUco, AprilTag, STag, and Chilitags:

```text
0.097 m
```

CCTag and JuMarker UCO use their measured printed marker size:

```text
0.096 m
```

Marker size must match the physically printed marker used during testing.

## Benchmark Methodology

Each accepted run uses:

- Same webcam
- Same calibration file
- Same indoor setup
- Same approximate lighting
- 0-degree front-facing marker alignment
- 500 frames per test
- Test distances: 0.5 m, 1.0 m, 1.5 m, 2.0 m, 2.5 m

CSV files were checked before plotting to avoid incorrect ground-truth distance labels, invalid runs, and duplicate repeated tests.

## Main Scripts

| File | Purpose |
|---|---|
| `aruco_pose.py` | Live ArUco pose estimation |
| `apriltag_pose.py` | Live AprilTag pose estimation |
| `stag_pose.py` | Live STag pose estimation |
| `chilitags_pose_live_py.py` | Live Chilitags pose estimation |
| `cctag_distance_live.py` | Live CCTag distance estimation |
| `jumarker_uco_live_pose.py` | Live JuMarker UCO pose display |
| `aruco_pose_metrics.py` | ArUco benchmark metrics |
| `apriltag_pose_metrics.py` | AprilTag benchmark metrics |
| `stag_pose_metrics.py` | STag benchmark metrics |
| `chilitags_pose_metrics.py` | Chilitags benchmark metrics |
| `cctag_distance_metrics.py` | CCTag distance benchmark metrics |
| `jumarker_uco_metrics.py` | JuMarker UCO benchmark metrics |
| `chilitags_py.cpp` | Minimal Python wrapper for Chilitags detection |
| `camera_alignment_lines.py` | Camera alignment helper |
| `plot_results.py` | Generates main comparison plots |
| `plot_apriltag_datadensity_effect.py` | Generates AprilTag data-density plots |

## Results

Benchmark CSV files are stored in:

```text
metrics_results/
```

Generated plots are stored in:

```text
plots/
```

The generated plots compare marker performance across distance for:

- Detection rate
- Pose / distance success rate
- Mean Z error
- Mean straight-line distance error
- Reprojection error where available
- Z jitter
- Distance jitter
- FPS

The current benchmark includes:

```text
ArUco
AprilTag
STag
Chilitags
CCTag
JuMarker UCO
```

CCTag and JuMarker UCO do not currently report OpenCV-style reprojection error in the Python benchmark wrappers, so they may be absent from the reprojection error plot while still being included in distance error, jitter, detection, and FPS plots.

## AprilTag Data-Density Study

The repository also contains AprilTag 16h5 and 36h11 material to observe how marker data density affects pose estimation when the physical marker size is fixed.

Relevant folders:

```text
apriltag_datadensities/
plots_apriltag_datadensity_effect/
```

## Repository Structure

```text
.
├── aruco_pose.py
├── apriltag_pose.py
├── stag_pose.py
├── chilitags_pose_live_py.py
├── cctag_distance_live.py
├── jumarker_uco_live_pose.py
├── aruco_pose_metrics.py
├── apriltag_pose_metrics.py
├── stag_pose_metrics.py
├── chilitags_pose_metrics.py
├── cctag_distance_metrics.py
├── jumarker_uco_metrics.py
├── chilitags_py.cpp
├── camera_alignment_lines.py
├── calibration_data.npz
├── chilitag_markers/
├── metrics_results/
├── plots/
├── apriltag_datadensities/
├── plots_apriltag_datadensity_effect/
├── plot_results.py
├── plot_apriltag_datadensity_effect.py
└── requirements.txt
```

Local folders such as `venv/`, `external/`, compiled binaries, `.so` files, and local backup/debug files are ignored and should be rebuilt locally if needed.

## External Detector Notes

CCTag, Chilitags, and JuMarker require external C++ components. These are not committed as full third-party source trees inside this repository.

### CCTag

CCTag was built locally as a CPU-only detector. The detector output was used to expose values needed for benchmarking:

```text
center x/y
marker id
status
ellipse axes
ellipse angle
apparent diameter in pixels
```

The Python wrapper estimates distance using:

```text
Z = fx × real_marker_diameter / apparent_diameter_px
```

### JuMarker UCO

JuMarker was built locally and tested using the UCO marker design. The C++ detector output is parsed by the Python wrapper to save benchmark CSV files.

JuMarker Cordoba was attempted but excluded because detection was not reliable enough for the final benchmark.

## Current Next Step

The benchmark stage is complete for now. The next step is implementation on an ST Linux-based embedded board, followed by testing on a small robot.
