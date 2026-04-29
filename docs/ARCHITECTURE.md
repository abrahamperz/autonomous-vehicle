# Arquitectura del Sistema

## Diagrama de Flujo de Datos

El sistema sigue una arquitectura de pipeline clasica: **Percepcion -> Decision -> Actuacion**.

```
+------------------+
|  ZED Stereo Cam  |
+--------+---------+
         |
         v
+------------------+
| vga_zed_wrapper  |  Publica imagenes en topics ROS
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
  | Navigation Control|  Controlador de modo deslizante
  +--------+----------+
           |
           | /I2C/nxp_communication
           v
  +-------------------+
  | NXP Communication |  Interfaz I2C con el vehiculo
  +--------+----------+
           |
           v
  +-------------------+
  |  Actuadores del   |
  |  Vehiculo (Motor, |
  |  Direccion, Freno)|
  +-------------------+

  [Opcional]
  +-------------------+
  | LiDAR Lite v3     |  Sensor de distancia independiente
  +-------------------+
```

## Modos de Operacion

El sistema ofrece tres configuraciones de lanzamiento, definidas en `jeep_master_node/launch/`:

### 1. Modo Seguimiento de Carril

```
jeep_master_node_lane.launch
```

Lanza: `lane_detector` -> `navigation_control` -> `nxp_communication`

Usado para seguir marcas de carril en pistas de prueba. Detecta lineas rojas y azules mediante umbralizacion HSV.

### 2. Modo Deteccion de Objetos

```
jeep_master_node_yolo.launch
```

Lanza: `ros_yolov3` -> `navigation_control` -> `nxp_communication`

Usado para deteccion y evasion de obstaculos. Detecta personas, autos y otros objetos con profundidad estereo.

### 3. Modo Control Directo

```
jeep_master_node_nxp.launch
```

Lanza: `navigation_control` -> `nxp_communication`

Usado para pruebas manuales del controlador sin sensores de percepcion.

## Comunicacion entre Nodos

Todos los nodos se comunican a traves de **ROS Topics** usando el patron publicador/suscriptor:

```
lane_detector ──/lane_detector/error_lat──> navigation_control
lane_detector ──/lane_detector/steer_angle─> navigation_control
ros_yolov3 ────/yolo_detections_topic─────> navigation_control
navigation_control ──/I2C/nxp_communication──> nxp_communication
nxp_communication ──/I2C/receive──> (acknowledgment)
lidarlite_node ──/I2C/LidarLite_data──> (disponible para suscripcion)
```

## Algoritmos Clave

### Controlador de Modo Deslizante (Navigation Control)

El nodo `navigation_control` implementa un controlador de modo deslizante para corregir el error lateral:

```
sigma = C1 * error_lat + (error_lat - error_anterior) / Ts
steering = f(sigma)  con saturacion
```

La evasion de obstaculos usa una funcion gaussiana basada en la profundidad del objeto detectado. La aceleracion se calcula con una funcion exponencial que decrece conforme el objeto esta mas cerca.

### Pipeline de Deteccion de Carril

1. Captura de frame desde la camara ZED
2. Conversion BGR -> RGB -> HLS
3. Filtro Sobel en X para deteccion de bordes
4. Umbral en canal S para consistencia de color
5. Transformacion de perspectiva (vista de pajaro)
6. Algoritmo de ventana deslizante para encontrar limites de carril
7. Ajuste de curvas polinomiales
8. Calculo de curvatura y error lateral

### Pipeline YOLOv3

1. Captura de frames con datos de profundidad (ZED estereo)
2. Inferencia YOLOv3-tiny acelerada por CUDA
3. Extraccion de bounding boxes, confianza y profundidad
4. Publicacion como mensajes personalizados `jeep_msgs::yolov3_msg`
