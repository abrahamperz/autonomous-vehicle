# Referencia de Topics ROS

## Mapa Completo de Topics

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

## Detalle por Topic

### Topics de Camara

| Topic | Tipo | Publicador | Descripcion |
|-------|------|-----------|-------------|
| `/zed/left/image_rect_color` | sensor_msgs/Image | vga_zed_wrapper | Imagen rectificada de la camara izquierda |
| `/zed/rgb/image_raw_color` | sensor_msgs/Image | vga_zed_wrapper | Imagen RGB sin procesar |

### Topics de Percepcion

| Topic | Tipo | Publicador | Suscriptor |
|-------|------|-----------|------------|
| `/yolo_detections_topic` | jeep_msgs/yolov3_msg | ros_yolov3 | navigation_control |
| `/lane_detector/out_image` | sensor_msgs/Image | lane_detector | (visualizacion) |
| `/lane_detector/warped_image` | sensor_msgs/Image | lane_detector | (visualizacion) |
| `/lane_detector/sliding_window_image` | sensor_msgs/Image | lane_detector | (visualizacion) |
| `/lane_detector/steer_angle` | std_msgs/Float32 | lane_detector | navigation_control |
| `/lane_detector/error_lat` | std_msgs/Float32 | lane_detector | navigation_control |

### Topics de Control

| Topic | Tipo | Publicador | Suscriptor |
|-------|------|-----------|------------|
| `/I2C/nxp_communication` | std_msgs/Float32MultiArray | navigation_control | nxp_communication |
| `/I2C/receive` | std_msgs/String | nxp_communication | (acknowledgment) |

### Topics de Sensores

| Topic | Tipo | Publicador | Descripcion |
|-------|------|-----------|-------------|
| `/I2C/LidarLite_data` | std_msgs/Int64 | lidarlite_node | Distancia en centimetros |

## Mensajes Personalizados

### jeep_msgs/yolov3_msg

```
string   name    # Clase del objeto: "person", "car", etc.
float32  depth   # Distancia al objeto en metros (estereo ZED)
float32  prob    # Confianza de deteccion (0.0 - 1.0)
```

### Formato de Float32MultiArray (Control)

El array de control enviado a NXP tiene 3 elementos:

```
data[0] = angulo_de_direccion   # Angulo calculado por el controlador
data[1] = aceleracion           # 0.0 (detenido) a 1.0 (maximo)
data[2] = freno                 # 0.0 (sin freno) a 1.0 (frenado completo)
```

## Comandos Utiles

```bash
# Listar todos los topics activos
rostopic list

# Ver mensajes en tiempo real
rostopic echo /yolo_detections_topic
rostopic echo /lane_detector/error_lat
rostopic echo /I2C/nxp_communication

# Ver frecuencia de publicacion
rostopic hz /yolo_detections_topic
rostopic hz /lane_detector/error_lat

# Ver informacion de un topic
rostopic info /I2C/nxp_communication

# Visualizar imagenes
rosrun image_view image_view image:=/lane_detector/out_image
rosrun image_view image_view image:=/lane_detector/warped_image
```
