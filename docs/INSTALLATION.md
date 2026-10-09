# Installation Guide

## System Requirements

| Requirement | Specification |
|-----------|---------------|
| Operating System | Ubuntu 16.04 LTS (recommended) or 18.04 LTS |
| GPU | NVIDIA with CUDA support (compute capability 3.0+) |
| RAM | 8 GB minimum (16 GB recommended) |
| Storage | 10 GB free minimum |

## Dependencies

### 1. ROS (Robot Operating System)

Install ROS Kinetic (Ubuntu 16.04) or Melodic (Ubuntu 18.04):

```bash
# Add the ROS repository
sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
sudo apt-key adv --keyserver 'hkp://keyserver.ubuntu.com:80' --recv-key C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654

# Install ROS
sudo apt update
sudo apt install ros-kinetic-desktop-full  # or ros-melodic-desktop-full

# Initialize rosdep
sudo rosdep init
rosdep update

# Configure the environment
echo "source /opt/ros/kinetic/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

### 2. CUDA Toolkit

CUDA 9.1 or newer is required for GPU acceleration of YOLOv3.

```bash
# Download from https://developer.nvidia.com/cuda-toolkit-archive
# Follow NVIDIA's installation instructions for your system

# Verify the installation
nvcc --version
```

### 3. cuDNN

Required for neural network inference.

```bash
# Download from https://developer.nvidia.com/cudnn
# Requires an NVIDIA Developer account

# Install (example for cuDNN 7)
sudo dpkg -i libcudnn7_*.deb
sudo dpkg -i libcudnn7-dev_*.deb
```

### 4. OpenCV

Compatible version: 2.4.13 to 3.4.0.

```bash
# Option 1: Install from repositories
sudo apt install libopencv-dev

# Option 2: Build from source (recommended for CUDA support)
# See: https://docs.opencv.org/3.4/d7/d9f/tutorial_linux_install.html
```

### 5. ZED SDK

Driver for the ZED stereo camera.

```bash
# Download from https://www.stereolabs.com/developers/release/
# Run the installer
chmod +x ZED_SDK_*.run
./ZED_SDK_*.run
```

### 6. Additional ROS dependencies

```bash
sudo apt install ros-kinetic-cv-bridge ros-kinetic-image-transport
```

## Building the Project

### Create the catkin workspace

```bash
mkdir -p ~/catkin_ws/src
cd ~/catkin_ws/src
```

### Clone the repository

```bash
git clone <repository-url> iDrive
```

### Build

```bash
cd ~/catkin_ws
catkin_make
```

If you hit build errors with CUDA, make sure the paths are set correctly:

```bash
export CUDA_HOME=/usr/local/cuda
export PATH=$CUDA_HOME/bin:$PATH
export LD_LIBRARY_PATH=$CUDA_HOME/lib64:$LD_LIBRARY_PATH
```

### Source the environment

```bash
source ~/catkin_ws/devel/setup.bash

# Add it to .bashrc to make it permanent
echo "source ~/catkin_ws/devel/setup.bash" >> ~/.bashrc
```

## YOLOv3 Weights

The `yolov3-tiny.weights` file is already included in `ros_yolov3/`. If you need to download other weights:

```bash
cd ~/catkin_ws/src/iDrive/ros_yolov3

# Full YOLOv3 (more accurate, slower)
wget https://pjreddie.com/media/files/yolov3.weights

# YOLOv3-tiny (included - faster, lower accuracy)
wget https://pjreddie.com/media/files/yolov3-tiny.weights
```

## Verification

```bash
# Verify the packages were built
rospack find jeep_master_node
rospack find ros_yolov3
rospack find lane_detector
rospack find navigation_control
rospack find nxp_communication

# Verify custom messages
rosmsg show jeep_msgs/yolov3_msg
```

## Common Issues

### Error: "Could not find CUDA"

Make sure CUDA is installed and the environment variables are set:

```bash
echo $CUDA_HOME  # Should print /usr/local/cuda
nvidia-smi       # Should show your GPU
```

### Error: "ZED SDK not found"

Verify the ZED SDK is installed:

```bash
ls /usr/local/zed/
```

### Build error in libdarknet

The Darknet library requires a GPU. If you build without a GPU, edit the `Makefile` in `ros_yolov3/libdarknet/`:

```makefile
GPU=0    # Disable GPU (testing only)
CUDNN=0  # Disable cuDNN
```

**Note:** Without a GPU, detection will be very slow and not suitable for real-time use.
