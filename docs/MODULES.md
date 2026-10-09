# System Modules

Each module is an **independent ROS package** with its own `CMakeLists.txt` and `package.xml`.

---

## jeep_master_node

**Type:** Launch package (no source code)

Contains the `roslaunch` configurations that orchestrate startup of the full system. Defines three operating modes:

| Launch File | Nodes it starts |
|-------------|-----------------|
| `jeep_master_node_lane.launch` | lane_detector, navigation_control, nxp_communication |
| `jeep_master_node_yolo.launch` | ros_yolov3, navigation_control, nxp_communication |
| `jeep_master_node_nxp.launch` | navigation_control, nxp_communication |

---

## jeep_msgs

**Type:** Message package

Defines the project's custom message types.

### yolov3_msg.msg

```
string   name    # Name of the detected object (person, car, etc.)
float32  depth   # Depth in meters from the stereo camera
float32  prob    # Detection probability (0.0 - 1.0)
```

**Dependencies:** `std_msgs`

---

## lane_detector

**Type:** Perception module (Python)

Detects lane boundaries in ZED camera images using computer vision.

### Main files

| File | Function |
|---------|---------|
| `scripts/lane_detector_impl.py` | Main ROS node. Full detection pipeline |
| `scripts/camara_pista.py` | Lane marker detection (red/blue) via HSV |
| `scripts/camara.py` | General camera interface |
| `scripts/hsv.py` | HSV color space utilities |
| `scripts/functions/utils.py` | Image pipeline: perspective, sliding window, Sobel |

### Color Parameters

**Red lines:**
- HSV: `[0, 50, 50]` to `[12, 255, 255]` and `[160, 50, 50]` to `[188, 255, 255]`

**Blue lines:**
- HSV: `[94, 80, 2]` to `[126, 255, 255]`

### Published Topics

| Topic | Type | Description |
|-------|------|-------------|
| `/lane_detector/out_image` | Image | Image with detected-lane overlay |
| `/lane_detector/warped_image` | Image | Bird's-eye view (perspective transformed) |
| `/lane_detector/sliding_window_image` | Image | Sliding-window algorithm visualization |
| `/lane_detector/steer_angle` | Float32 | Recommended steering angle |
| `/lane_detector/error_lat` | Float32 | Lateral error in pixels |

### Configuration

The `config/zed_default.yaml` file defines the ZED camera input topics.

---

## ros_yolov3

**Type:** Perception module (C++)

Real-time object detection using YOLOv3-tiny with ZED camera integration for depth.

### Main file

- `src/main.cpp` - Darknet wrapper with ZED SDK, GPU processing

### How it works

1. Initializes the YOLOv3-tiny network with pre-trained weights (`yolov3-tiny.weights`)
2. Captures frames from the ZED camera (RGB + depth map)
3. Runs inference on the GPU (CUDA)
4. Extracts detections with bounding box, class, confidence, and depth
5. Publishes each detection as `jeep_msgs::yolov3_msg`

### Published Topics

| Topic | Type | Description |
|-------|------|-------------|
| `/yolo_detections_topic` | jeep_msgs::yolov3_msg | Detected objects with depth |

### Special Dependencies

- Darknet (included as `libdarknet/`)
- CUDA 9.1+
- cuDNN
- ZED SDK

---

## navigation_control

**Type:** Decision and control module (C++)

Receives perception data (YOLO + lanes) and generates control commands for the vehicle.

### Main file

- `src/navigation_control.cpp`

### Control Logic

- **Steering:** Sliding mode controller that minimizes lateral error
  - `sigma = C1 * error + delta_error / Ts`
  - Saturation to limit the maximum angle
- **Avoidance:** Gaussian function over the depth of the nearest object
- **Throttle:** Exponential function inversely proportional to the obstacle distance
- **Filtering:** Median filter over 8 detection samples
- **Conversion:** Pixels to meters with factor `K = 0.0035`

### Subscribed Topics

| Topic | Type | Source |
|-------|------|--------|
| `/yolo_detections_topic` | jeep_msgs::yolov3_msg | ros_yolov3 |
| `/lane_detector/error_lat` | Float32 | lane_detector |
| `/lane_detector/steer_angle` | Float32 | lane_detector |

### Published Topics

| Topic | Type | Description |
|-------|------|-------------|
| `/I2C/nxp_communication` | Float32MultiArray | 3-element array: [angle, throttle, brake] |

---

## nxp_communication

**Type:** Actuation module (C++)

I2C interface with the NXP S32K148 microcontroller that controls the vehicle's physical actuators.

### Main file

- `src/nxp_communication.cpp`

### How it works

1. Receives commands from `navigation_control` via topic
2. Establishes an I2C connection with the NXP (address `0x1D`, bus 0)
3. Sends throttle, brake, and steering commands
4. Receives acknowledgments from the microcontroller

### Subscribed Topics

| Topic | Type |
|-------|------|
| `/I2C/nxp_communication` | Float32MultiArray |

### Published Topics

| Topic | Type |
|-------|------|
| `/I2C/receive` | String (acknowledgment) |

---

## lidarlite_node

**Type:** Sensor module (C++)

I2C interface with the LiDAR Lite v3 sensor for distance measurement.

### Main file

- `src/lidarlite_communication.cpp`

### How it works

1. Initializes I2C communication with the LiDAR (address `0x62`, bus 0)
2. Reads distance measurements in centimeters
3. Publishes the data on a ROS topic

### Published Topics

| Topic | Type | Description |
|-------|------|-------------|
| `/I2C/LidarLite_data` | Int64 | Measured distance in centimeters |

---

## vga_zed_wrapper

**Type:** Camera wrapper (C++)

Node that encapsulates the ZED camera and publishes its images on standard ROS topics.

### Published Topics

| Topic | Description |
|-------|-------------|
| `/zed/left/image_rect_color` | Rectified image from the left camera |
| `/zed/rgb/image_raw_color` | Raw RGB image |

---

## testing

**Type:** Test scripts and utilities (Python)

| Script | Function |
|--------|---------|
| `scripts/lane_img_show.py` | Visualizes lane detection results |
| `scripts/listener.py` | Listens to and prints ROS topic messages |
