# System Architecture

## Data Flow Diagram

The system follows a classic pipeline architecture: **Perception -> Decision -> Actuation**.

```
+------------------+
|  ZED Stereo Cam  |
+--------+---------+
         |
         v
+------------------+
| vga_zed_wrapper  |  Publishes images on ROS topics
+--------+---------+
         |
    +----+----+
    |         |
    v         v
+--------+ +----------------+
| YOLO   | | Lane Detector  |
| v3     | | (Python)       |
+---+----+ +-------+--------+
    |               |
    | /yolo_        | /lane_detector/
    | detections    | error_lat
    | _topic        | /steer_angle
    |               |
    +-------+-------+
            |
            v
  +-------------------+
  | Navigation Control|  Sliding mode controller
  +--------+----------+
           |
           | /I2C/nxp_communication
           v
  +-------------------+
  | NXP Communication |  I2C interface with the vehicle
  +--------+----------+
           |
           v
  +-------------------+
  |  Vehicle          |
  |  Actuators (Motor,|
  |  Steering, Brake) |
  +-------------------+

  [Optional]
  +-------------------+
  | LiDAR Lite v3     |  Standalone distance sensor
  +-------------------+
```

## Operating Modes

The system offers three launch configurations, defined in `jeep_master_node/launch/`:

### 1. Lane Following Mode

```
jeep_master_node_lane.launch
```

Launches: `lane_detector` -> `navigation_control` -> `nxp_communication`

Used to follow lane markings on test tracks. Detects red and blue lines through HSV thresholding.

### 2. Object Detection Mode

```
jeep_master_node_yolo.launch
```

Launches: `ros_yolov3` -> `navigation_control` -> `nxp_communication`

Used for obstacle detection and avoidance. Detects people, cars, and other objects with stereo depth.

### 3. Direct Control Mode

```
jeep_master_node_nxp.launch
```

Launches: `navigation_control` -> `nxp_communication`

Used for manual controller testing without perception sensors.

## Inter-Node Communication

All nodes communicate through **ROS Topics** using the publisher/subscriber pattern:

```
lane_detector ──/lane_detector/error_lat──> navigation_control
lane_detector ──/lane_detector/steer_angle─> navigation_control
ros_yolov3 ────/yolo_detections_topic─────> navigation_control
navigation_control ──/I2C/nxp_communication──> nxp_communication
nxp_communication ──/I2C/receive──> (acknowledgment)
lidarlite_node ──/I2C/LidarLite_data──> (available for subscription)
```

## Key Algorithms

### Sliding Mode Controller (Navigation Control)

The `navigation_control` node implements a sliding mode controller to correct the lateral error:

```
sigma = C1 * error_lat + (error_lat - previous_error) / Ts
steering = f(sigma)  with saturation
```

Obstacle avoidance uses a Gaussian function based on the depth of the detected object. Acceleration is computed with an exponential function that decreases as the object gets closer.

### Lane Detection Pipeline

1. Frame capture from the ZED camera
2. BGR -> RGB -> HLS conversion
3. Sobel filter in X for edge detection
4. S-channel threshold for color consistency
5. Perspective transform (bird's-eye view)
6. Sliding window algorithm to find lane boundaries
7. Polynomial curve fitting
8. Curvature and lateral error computation

### YOLOv3 Pipeline

1. Frame capture with depth data (ZED stereo)
2. CUDA-accelerated YOLOv3-tiny inference
3. Extraction of bounding boxes, confidence, and depth
4. Publication as custom `jeep_msgs::yolov3_msg` messages
