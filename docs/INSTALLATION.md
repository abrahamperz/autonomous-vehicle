# Guia de Instalacion

## Requisitos del Sistema

| Requisito | Especificacion |
|-----------|---------------|
| Sistema Operativo | Ubuntu 16.04 LTS (recomendado) o 18.04 LTS |
| GPU | NVIDIA con soporte CUDA (compute capability 3.0+) |
| RAM | 8 GB minimo (16 GB recomendado) |
| Almacenamiento | 10 GB libres minimo |

## Dependencias

### 1. ROS (Robot Operating System)

Instalar ROS Kinetic (Ubuntu 16.04) o Melodic (Ubuntu 18.04):

```bash
# Agregar repositorio de ROS
sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
sudo apt-key adv --keyserver 'hkp://keyserver.ubuntu.com:80' --recv-key C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654

# Instalar ROS
sudo apt update
sudo apt install ros-kinetic-desktop-full  # o ros-melodic-desktop-full

# Inicializar rosdep
sudo rosdep init
rosdep update

# Configurar entorno
echo "source /opt/ros/kinetic/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

### 2. CUDA Toolkit

Se requiere CUDA 9.1 o superior para la aceleracion GPU de YOLOv3.

```bash
# Descargar desde https://developer.nvidia.com/cuda-toolkit-archive
# Seguir las instrucciones de instalacion de NVIDIA para tu sistema

# Verificar instalacion
nvcc --version
```

### 3. cuDNN

Necesario para la inferencia de redes neuronales.

```bash
# Descargar desde https://developer.nvidia.com/cudnn
# Requiere cuenta de NVIDIA Developer

# Instalar (ejemplo para cuDNN 7)
sudo dpkg -i libcudnn7_*.deb
sudo dpkg -i libcudnn7-dev_*.deb
```

### 4. OpenCV

Version compatible: 2.4.13 a 3.4.0.

```bash
# Opcion 1: Instalar desde repositorios
sudo apt install libopencv-dev

# Opcion 2: Compilar desde fuente (recomendado para soporte CUDA)
# Ver: https://docs.opencv.org/3.4/d7/d9f/tutorial_linux_install.html
```

### 5. ZED SDK

Controlador para la camara estereo ZED.

```bash
# Descargar desde https://www.stereolabs.com/developers/release/
# Ejecutar el instalador
chmod +x ZED_SDK_*.run
./ZED_SDK_*.run
```

### 6. Dependencias de ROS adicionales

```bash
sudo apt install ros-kinetic-cv-bridge ros-kinetic-image-transport
```

## Compilacion del Proyecto

### Crear workspace de catkin

```bash
mkdir -p ~/catkin_ws/src
cd ~/catkin_ws/src
```

### Clonar el repositorio

```bash
git clone <url-del-repositorio> iDrive
```

### Compilar

```bash
cd ~/catkin_ws
catkin_make
```

Si hay errores de compilacion con CUDA, verificar que las rutas esten correctamente configuradas:

```bash
export CUDA_HOME=/usr/local/cuda
export PATH=$CUDA_HOME/bin:$PATH
export LD_LIBRARY_PATH=$CUDA_HOME/lib64:$LD_LIBRARY_PATH
```

### Cargar el entorno

```bash
source ~/catkin_ws/devel/setup.bash

# Agregar al .bashrc para que sea permanente
echo "source ~/catkin_ws/devel/setup.bash" >> ~/.bashrc
```

## Pesos de YOLOv3

El archivo `yolov3-tiny.weights` ya esta incluido en `ros_yolov3/`. Si necesitas descargar otros pesos:

```bash
cd ~/catkin_ws/src/iDrive/ros_yolov3

# YOLOv3 completo (mas preciso, mas lento)
wget https://pjreddie.com/media/files/yolov3.weights

# YOLOv3-tiny (incluido - mas rapido, menor precision)
wget https://pjreddie.com/media/files/yolov3-tiny.weights
```

## Verificacion

```bash
# Verificar que los paquetes fueron compilados
rospack find jeep_master_node
rospack find ros_yolov3
rospack find lane_detector
rospack find navigation_control
rospack find nxp_communication

# Verificar mensajes personalizados
rosmsg show jeep_msgs/yolov3_msg
```

## Problemas Comunes

### Error: "Could not find CUDA"

Asegurate de que CUDA esta instalado y las variables de entorno estan configuradas:

```bash
echo $CUDA_HOME  # Debe mostrar /usr/local/cuda
nvidia-smi       # Debe mostrar tu GPU
```

### Error: "ZED SDK not found"

Verifica que el ZED SDK esta instalado:

```bash
ls /usr/local/zed/
```

### Error de compilacion en libdarknet

La biblioteca Darknet requiere GPU. Si compilas sin GPU, edita el `Makefile` en `ros_yolov3/libdarknet/`:

```makefile
GPU=0    # Deshabilitar GPU (solo para pruebas)
CUDNN=0  # Deshabilitar cuDNN
```

**Nota:** Sin GPU, la deteccion sera muy lenta y no apta para uso en tiempo real.
