# ROS Topics Reference

## Complete Topic Map

```
vga_zed_wrapper
  PUB -> /zed/left/image_rect_color     [sensor_msgs/Image]
  PUB -> /zed/rgb/image_raw_color       [sensor_msgs/Image]

ros_yolov3
  SUB <- /zed/rgb/image_raw_color       [sensor_msgs/Image]
  PUB -> /yolo_detections_topic         [jeep_msgs/yolov3_msg]

lane_detector
  SUB <- /zed/left/image_rect_color     [sensor_msgs/Image]
  PUB -> /lane_detector/out_image       [sensor_msgs/Image]
  PUB -> /lane_detector/warped_image    [sensor_msgs/Image]
  PUB -> /lane_detector/sliding_window_image [sensor_msgs/Image]
  PUB -> /lane_detector/steer_angle     [std_msgs/Float32]
  PUB -> /lane_detector/error_lat       [std_msgs/Float32]

navigation_control
  SUB <- /yolo_detections_topic         [jeep_msgs/yolov3_msg]
  SUB <- /lane_detector/error_lat       [std_msgs/Float32]
  SUB <- /lane_detector/steer_angle     [std_msgs/Float32]
  PUB -> /I2C/nxp_communication        [std_msgs/Float32MultiArray]

nxp_communication
  SUB <- /I2C/nxp_communication        [std_msgs/Float32MultiArray]
  PUB -> /I2C/receive                  [std_msgs/String]

lidarlite_node
  PUB -> /I2C/LidarLite_data           [std_msgs/Int64]
```

## Topic Details

### Camera Topics

| Topic | Type | Publisher | Description |
|-------|------|-----------|-------------|
| `/zed/left/image_rect_color` | sensor_msgs/Image | vga_zed_wrapper | Rectified image from the left camera |
| `/zed/rgb/image_raw_color` | sensor_msgs/Image | vga_zed_wrapper | Raw RGB image |

### Perception Topics

| Topic | Type | Publisher | Subscriber |
|-------|------|-----------|------------|
| `/yolo_detections_topic` | jeep_msgs/yolov3_msg | ros_yolov3 | navigation_control |
| `/lane_detector/out_image` | sensor_msgs/Image | lane_detector | (visualization) |
| `/lane_detector/warped_image` | sensor_msgs/Image | lane_detector | (visualization) |
| `/lane_detector/sliding_window_image` | sensor_msgs/Image | lane_detector | (visualization) |
| `/lane_detector/steer_angle` | std_msgs/Float32 | lane_detector | navigation_control |
| `/lane_detector/error_lat` | std_msgs/Float32 | lane_detector | navigation_control |

### Control Topics

| Topic | Type | Publisher | Subscriber |
|-------|------|-----------|------------|
| `/I2C/nxp_communication` | std_msgs/Float32MultiArray | navigation_control | nxp_communication |
| `/I2C/receive` | std_msgs/String | nxp_communication | (acknowledgment) |

### Sensor Topics

| Topic | Type | Publisher | Description |
|-------|------|-----------|-------------|
| `/I2C/LidarLite_data` | std_msgs/Int64 | lidarlite_node | Distance in centimeters |

## Custom Messages

### jeep_msgs/yolov3_msg

```
string   name    # Object class: "person", "car", etc.
float32  depth   # Distance to the object in meters (ZED stereo)
float32  prob    # Detection confidence (0.0 - 1.0)
```

### Float32MultiArray Format (Control)

The control array sent to the NXP has 3 elements:

```
data[0] = steering_angle   # Angle computed by the controller
data[1] = throttle         # 0.0 (stopped) to 1.0 (maximum)
data[2] = brake            # 0.0 (no brake) to 1.0 (full braking)
```

## Useful Commands

```bash
# List all active topics
rostopic list

# View messages in real time
rostopic echo /yolo_detections_topic
rostopic echo /lane_detector/error_lat
rostopic echo /I2C/nxp_communication

# View publishing rate
rostopic hz /yolo_detections_topic
rostopic hz /lane_detector/error_lat

# View topic information
rostopic info /I2C/nxp_communication

# Visualize images
rosrun image_view image_view image:=/lane_detector/out_image
rosrun image_view image_view image:=/lane_detector/warped_image
```
