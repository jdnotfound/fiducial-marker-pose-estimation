# Fiducial Marker Pose Estimation Benchmark

This project compares ArUco, AprilTag, and STag fiducial markers for camera-based robot pose estimation and localization.

## Project Goal

The main goal is to evaluate fiducial markers based on:

- Detection reliability
- Distance accuracy
- Pose estimation success rate
- Reprojection error
- Z-distance jitter

The final target is integration with an STMicroelectronics Linux-based embedded board for robot localization and navigation.

## Markers Tested

- ArUco: OpenCV `DICT_4X4_50`
- AprilTag: OpenCV `DICT_APRILTAG_36h11`
- STag: `stag-python`, HD21 family

## Test Setup

- Same webcam for all markers
- Same camera calibration file
- Same printed marker size: 0.097 m
- Same pose estimation method using OpenCV `solvePnP`
- 500 frames per marker-distance test
- Tested distances: 0.5 m, 1.0 m, 1.5 m, 2.0 m, 2.5 m

## Important Note

Camera calibration resolution and capture resolution must match. Do not force webcam resolution unless calibration was performed at the same resolution.

## Main Scripts

| File | Purpose |
|---|---|
| `aruco_pose.py` | Live ArUco pose estimation |
| `apriltag_pose.py` | Live AprilTag pose estimation |
| `stag_pose.py` | Live STag pose estimation |
| `aruco_pose_metrics.py` | ArUco benchmark metrics |
| `apriltag_pose_metrics.py` | AprilTag benchmark metrics |
| `stag_pose_metrics.py` | STag benchmark metrics |
| `plot_results.py` | Generates plots from summary CSV files |

## Results

Summary CSV files are stored in:

```text
metrics_results/
