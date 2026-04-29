# iDrive - Autonomous SUV Platform

**Plataforma de conduccion autonoma basada en ROS para un SUV, desarrollada por estudiantes del Tecnologico de Monterrey Campus Guadalajara.**

---

## Descripcion General

iDrive es un sistema modular de conduccion autonoma que integra multiples sensores (camara estereo ZED, LiDAR, microcontrolador NXP) con algoritmos de percepcion, planificacion y control. El vehiculo es capaz de:

- **Seguimiento de carril** mediante deteccion de lineas con vision por computadora
- **Deteccion de objetos** en tiempo real con YOLOv3 acelerado por GPU
- **Medicion de distancia** con sensor LiDAR Lite v3
- **Control autonomo** del volante, aceleracion y frenado via interfaz I2C

## Arquitectura del Sistema

```
ZED Stereo Camera
       |
       v
 [vga_zed_wrapper]
       |
   +---+---+
   |       |
   v       v
[YOLO]  [Lane Detector]
   |       |
   +---+---+
       |
       v
[Navigation Control]
       |
       v
[NXP Communication]
       |
       v
  Actuadores del Vehiculo
```

## Stack Tecnologico

| Componente | Tecnologia |
|-----------|------------|
| Middleware | ROS (Robot Operating System) |
| Build System | catkin |
| Percepcion | YOLOv3 (Darknet), OpenCV |
| GPU | CUDA + cuDNN |
| Camara | ZED SDK |
| Lenguajes | C++ (modulos core), Python (deteccion de carril) |
| Hardware | NXP S32K148, LiDAR Lite v3 |

## Estructura del Proyecto

```
iDrive/
  jeep_master_node/     # Nodo maestro - configuraciones de lanzamiento
  jeep_msgs/            # Mensajes ROS personalizados
  lane_detector/        # Subsistema de deteccion de carril (Python)
  ros_yolov3/           # Deteccion de objetos YOLOv3 (C++)
  navigation_control/   # Logica de decision y control (C++)
  nxp_communication/    # Interfaz con el microcontrolador NXP (C++)
  lidarlite_node/       # Sensor LiDAR Lite v3 (C++)
  vga_zed_wrapper/      # Wrapper de camara ZED
  testing/              # Scripts de prueba y utilidades
  docs/                 # Documentacion detallada
```

## Inicio Rapido

### Prerequisitos

- Ubuntu 16.04+ con ROS Kinetic (o superior)
- NVIDIA GPU con CUDA 9.1+ y cuDNN
- ZED SDK 2.x
- OpenCV 2.4.13 - 3.4.0

### Compilacion

```bash
# Clonar el repositorio en tu workspace de catkin
cd ~/catkin_ws/src
git clone <url-del-repositorio> iDrive

# Compilar
cd ~/catkin_ws
catkin_make

# Cargar el entorno
source devel/setup.bash
```

### Ejecucion

```bash
# Modo completo: deteccion de carril + navegacion + control del vehiculo
roslaunch jeep_master_node jeep_master_node_lane.launch

# Modo YOLO: deteccion de objetos + navegacion + control
roslaunch jeep_master_node jeep_master_node_yolo.launch

# Modo NXP: solo comunicacion con el controlador
roslaunch jeep_master_node jeep_master_node_nxp.launch
```

## Documentacion

| Documento | Descripcion |
|-----------|-------------|
| [Arquitectura](docs/ARCHITECTURE.md) | Flujo de datos, diagramas y comunicacion entre nodos |
| [Modulos](docs/MODULES.md) | Descripcion detallada de cada paquete ROS |
| [Instalacion](docs/INSTALLATION.md) | Guia completa de instalacion y dependencias |
| [Hardware](docs/HARDWARE.md) | Configuracion de sensores y dispositivos I2C |
| [Topics ROS](docs/ROS_TOPICS.md) | Referencia completa de topics publicados y suscritos |

## Equipo

Desarrollado por estudiantes del **Tecnologico de Monterrey Campus Guadalajara**.

## Licencia

Proyecto academico - Tecnologico de Monterrey.
