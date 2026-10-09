# Hardware Configuration

## Sensors and Devices

### ZED Stereo Camera

The ZED camera provides RGB images and depth maps for environment perception.

| Parameter | Value |
|-----------|-------|
| Model | ZED (Stereolabs) |
| SDK | ZED SDK 2.x |
| Resolution | Configurable (VGA by default for performance) |
| Data | RGB + stereo depth map |

**Mounting:** Front, centered on the vehicle roof.

**ROS Topics:**
- `/zed/left/image_rect_color` - Left rectified image
- `/zed/rgb/image_raw_color` - Raw RGB image

### LiDAR Lite v3

Low-cost distance sensor for point measurement.

| Parameter | Value |
|-----------|-------|
| Model | Garmin LiDAR Lite v3 |
| Range | 0 - 40 meters |
| Accuracy | +/- 2.5 cm |
| Interface | I2C |
| I2C Address | `0x62` |
| I2C Bus | 0 |

**ROS Topic:** `/I2C/LidarLite_data` (Int64, distance in cm)

### NXP S32K148 Microcontroller

Low-level controller that receives commands from ROS and acts on the motor, steering, and brakes.

| Parameter | Value |
|-----------|-------|
| Model | NXP S32K148 |
| Interface | I2C |
| I2C Address | `0x1D` |
| I2C Bus | 0 |

**Commands it receives:**

| Index | Parameter | Range |
|--------|-----------|-------|
| 0 | Steering angle | Variable |
| 1 | Throttle | 0.0 - 1.0 |
| 2 | Brake | 0.0 - 1.0 |

## I2C Connection Diagram

```
Computer (Jetson / PC)
    |
    +-- I2C Bus 0
    |     |
    |     +-- [0x1D] NXP S32K148 (Vehicle control)
    |     |
    |     +-- [0x62] LiDAR Lite v3 (Distance sensor)
    |
    +-- I2C Bus 1
          |
          +-- [0x1D] IMU / Accelerometer
```

## GPU

YOLOv3 object detection requires an NVIDIA GPU with CUDA support.

| Requirement | Minimum |
|-----------|--------|
| CUDA Compute Capability | 3.0+ |
| VRAM | 2 GB (YOLOv3-tiny), 4 GB (full YOLOv3) |
| CUDA | 9.1+ |
| cuDNN | 5 - 7 |

**Tested platforms:**
- NVIDIA Jetson TX2
- PC with dedicated NVIDIA GPU

## Integration Notes

### I2C Permissions

On Linux, accessing the I2C buses requires permissions. To avoid running as root:

```bash
# Add your user to the i2c group
sudo usermod -aG i2c $USER

# Create a udev rule if needed
echo 'SUBSYSTEM=="i2c-dev", MODE="0666"' | sudo tee /etc/udev/rules.d/99-i2c.rules
sudo udevadm control --reload-rules
```

### Verify I2C Devices

```bash
# List available I2C buses
ls /dev/i2c-*

# Scan devices on bus 0
sudo i2cdetect -y 0

# Should show:
#   0x1D -> NXP S32K148
#   0x62 -> LiDAR Lite v3
```

### ZED Camera

The ZED camera connects via USB 3.0. Verify the connection:

```bash
# List video devices
ls /dev/video*

# Test with the ZED viewer
/usr/local/zed/tools/ZED_Explorer
```
