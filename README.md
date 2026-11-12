# DAAFUSION
### Distance- and Quality-Aware Camera–LiDAR Fusion for 3D Car Detection with Smart Traffic Signal Management

<p align="center">
  <b>Multi-Modal 3D Traffic Perception | Adaptive Sensor Fusion | Intelligent Traffic Control</b>
</p>

---

## Overview

DAAFUSION is a research project that integrates camera images and LiDAR point clouds for 3D vehicle detection and intelligent traffic signal management.

Traditional camera-based traffic management systems primarily rely on vehicle counting. However, vehicle count alone does not indicate how close vehicles are to an intersection or how reliable the detected information is.

DAAFUSION addresses this limitation through a **Distance- and Quality-Aware Camera–LiDAR Fusion** approach.

The framework dynamically adjusts the contribution of camera and LiDAR features based on distance and input quality. It combines visual information with accurate 3D spatial information to improve vehicle perception.

The detected vehicle positions, distances, and confidence scores are subsequently used by an adaptive traffic signal controller to estimate lane-wise traffic demand, prioritize lanes, maintain fairness, and provide emergency override functionality.

## Key Features

- **Camera–LiDAR Fusion:** Combines RGB camera images and 3D LiDAR point clouds.
- **Distance-Adaptive Fusion:** Adjusts LiDAR feature reliability according to distance.
- **Image Quality Estimation:** Estimates the reliability of camera feature regions.
- **Multi-Scale BEV Processing:** Extracts local, medium-range, and wide-context spatial features.
- **Cross-Attention Fusion:** Uses four-head cross-attention for interaction between camera and LiDAR features.
- **3D Vehicle Detection:** Predicts object classes and 3D bounding boxes.
- **Adaptive Traffic Signals:** Allocates traffic signal priority using distance- and confidence-weighted vehicle demand.
- **Fairness Mechanism:** Prevents lanes from being continuously neglected.
- **Emergency Override:** Allows manual priority to be assigned to a selected lane.

## System Architecture

The proposed architecture consists of the following major components:

1. Camera Feature Encoder
2. LiDAR BEV Encoder
3. Distance-Adaptive Gate
4. Image Quality Gate
5. Multi-Scale BEV Feature Extraction
6. Cross-Attention Fusion
7. Anchor-Free 3D Detection Head
8. Adaptive Traffic Signal Controller

### Architecture Workflow

```text
          KITTI DATASET
                |
        +-------+-------+
        |               |
    RGB Images      LiDAR Points
        |               |
   Image Encoder     BEV Encoder
        |               |
  Image Features    LiDAR Features
        |               |
  Image Quality     Distance-Adaptive
      Gate                Gate
        |               |
        +-------+-------+
                |
       Multi-Scale BEV
          Processing
                |
       Cross-Attention
            Fusion
                |
       3D Detection Head
                |
      3D Vehicle Detection
                |
      +---------+---------+
      |                   |
  Vehicle Distance   Detection Confidence
      |                   |
      +---------+---------+
                |
       Lane-Wise Demand
          Estimation
                |
      Adaptive Signal
         Controller
                |
       Traffic Signal
         Management
```

## Methodology

### 1. Data Preprocessing

The KITTI dataset provides synchronized camera images, LiDAR point clouds, calibration parameters, and 3D bounding-box annotations.

Preprocessing includes:

- Loading RGB images and Velodyne point clouds.
- Applying camera–LiDAR calibration.
- Restricting LiDAR points to the selected spatial range.
- Converting point clouds into a Bird's-Eye View (BEV) representation.
- Generating a five-channel BEV feature map containing height, density, and intensity-based features.
- Resizing camera images to 384 × 1280.
- Normalizing images using ImageNet statistics.
- Converting ground-truth annotations into 3D bounding boxes in LiDAR coordinates.

### 2. Distance-Adaptive Gate

LiDAR point density decreases as the distance from the sensor increases.

A learned distance-adaptive gate adjusts the contribution of LiDAR features across different BEV locations, allowing the network to account for distance-dependent changes in spatial information.

### 3. Image Quality Estimation

Camera features may become unreliable due to poor illumination, blur, and small object representations.

A lightweight quality estimation network evaluates the reliability of different image feature regions and reduces the influence of unreliable visual information during fusion.

### 4. Multi-Scale BEV Feature Extraction

Objects appear at different sizes and distances in traffic scenes.

To capture spatial information at multiple scales, three parallel branches process BEV features:

- Local-scale features
- Medium-scale features
- Wide-context features

The resulting features are combined to provide a richer spatial representation.

### 5. Cross-Attention Fusion

Instead of simply concatenating camera and LiDAR features, DAAFUSION uses a four-head cross-attention mechanism.

This enables the two modalities to interact and learn complementary representations before passing the fused features to the detection head.

### 6. 3D Object Detection

An anchor-free detection head predicts object classes and 3D bounding boxes.

The detection output includes vehicle position, dimensions, orientation, and confidence information.

### 7. Adaptive Traffic Signal Management

The traffic controller uses the 3D detection results to estimate the demand of individual traffic lanes.

**Vehicle contribution:**

```text
Vehicle Contribution = Confidence / Distance
```

**Lane demand:**

```text
Lane Demand = Sum of (Confidence / Distance)
              for all detected vehicles in that lane
```

The controller also incorporates a waiting-time fairness bonus.

**Signal priority:**

```text
Priority Score = Lane Demand
               + (Waiting Cycles × Fairness Weight)
```

Controller configuration:

| Parameter | Value |
|---|---:|
| Green phase | 20 seconds |
| Yellow phase | 3 seconds |
| Detection update interval | 4 seconds |
| Emergency override duration | 15 seconds |

An operator can manually select a lane for emergency priority. The selected lane receives a green signal while the other lanes receive red signals. Automatic priority-based control resumes after the override period.

*Note: Emergency override is a manual demonstration mechanism, not automatic emergency vehicle recognition.*

## Dataset

### KITTI Vision Benchmark Suite

The project uses the KITTI 3D object detection dataset.

Dataset components:

| Directory | Description |
|---|---|
| `image_2` | RGB camera images |
| `velodyne` | 3D LiDAR point clouds |
| `calib` | Camera and LiDAR calibration files |
| `label_2` | 2D and 3D object annotations |

### Project Data Split

The 7,481 labeled samples are divided into:

| Split | Samples | Percentage |
|---|---:|---:|
| Training | 5,985 | 80% |
| Validation | 748 | 10% |
| Testing | 748 | 10% |
| **Total** | **7,481** | **100%** |

## Training Configuration

The model was configured with the following training settings:

| Parameter | Configuration |
|---|---|
| Dataset | KITTI |
| Image Backbone | ResNet-18 |
| Maximum Training Epochs | 90 |
| Batch Size | 4 |
| Selected Checkpoint | Phase 4 |
| Best Checkpoint Epoch | 12 |
| Best Validation Loss | 0.34 |

The checkpoint from epoch 12 was selected because it achieved the lowest recorded validation loss. Training loss continued to decrease beyond this point, while validation loss plateaued, indicating potential overfitting.

## Results

### Distance-Wise Car Detection Performance

| Distance Range | Car F1-Score |
|---|---:|
| 0–20 m | 78.7% |
| 20–40 m | 44.7% |
| 40–60 m | 20.9% |
| 60–80 m | 7.9% |

### Observations

- The model performs best for nearby vehicles.
- Detection performance decreases as distance increases.
- Sparse LiDAR observations and smaller visual representations create challenges for long-range detection.
- The results highlight the importance of distance-aware sensor fusion.

## Training Improvements

During development, several improvements were introduced:

**1. Best Checkpoint Selection**

Selected the checkpoint with the lowest validation loss instead of automatically using the final training epoch.

**2. Center-Weighted Target Assignment**

Refined center-based supervision to give the exact object center a higher weight than its neighboring cells.

| Target Region | Weight |
|---|---:|
| Exact center | 1.0 |
| Eight neighboring cells | 0.25 |
| Background | 0.25 |

This modification aims to reduce duplicate center predictions and unnecessary bounding boxes.

**3. Detection Cleanup**

A post-processing pipeline was used to improve prediction quality and remove redundant detections.

## Technology Stack

- Python
- PyTorch
- OpenCV
- NumPy
- ResNet-18
- Deep Learning
- Camera–LiDAR Sensor Fusion
- Bird's-Eye View (BEV)
- Cross-Attention
- KITTI Dataset

## Applications

- Intelligent Transportation Systems (ITS)
- Smart Traffic Signal Management
- 3D Vehicle Detection
- Traffic Demand Estimation
- Adaptive Lane Prioritization
- Emergency Traffic Management

## Future Improvements

- Improve long-range vehicle detection.
- Further optimize distance- and quality-aware fusion.
- Improve pedestrian and cyclist detection.
- Explore more robust multi-scale feature representations.
- Evaluate the system under different environmental and sensor conditions.
- Extend traffic signal evaluation to realistic intersection scenarios.

## Project Scope

DAAFUSION is a research prototype demonstrating the integration of multi-modal 3D traffic perception with adaptive traffic signal management.

The system is evaluated using KITTI data and a traffic-control demonstration framework. It is intended for research and experimentation, not certified real-world traffic signal deployment.

## References

1. Geiger, A., Lenz, P., and Urtasun, R. (2012). *Are we ready for autonomous driving? The KITTI vision benchmark suite.* CVPR.
2. Qi, C. R., et al. (2017). *PointNet++: Deep hierarchical feature learning on point sets in a metric space.* NeurIPS.
3. Lang, A. H., et al. (2019). *PointPillars: Fast encoders for object detection from point clouds.* CVPR.
4. Chen, X., et al. (2017). *Multi-view 3D object detection network for autonomous driving.* CVPR.
5. Liu, Z., et al. (2023). *BEVFusion: Multi-task multi-sensor fusion with unified bird's-eye view representation.* ICRA.
6. Yin, T., Zhou, X., and Krähenbühl, P. (2021). *Center-based 3D object detection and tracking.* CVPR.
7. Wang, Z., Huang, X., and Hu, Z. (2025). *Attention-based LiDAR–camera fusion for 3D object detection in autonomous driving.* World Electric Vehicle Journal, 16(6), 306.
8. Vaswani, A., et al. (2017). *Attention is All You Need.* NeurIPS.

---

**DAAFUSION — Bringing Distance-Aware Perception and Intelligent Traffic Management Together.**
