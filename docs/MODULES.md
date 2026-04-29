# Modulos del Sistema

Cada modulo es un **paquete ROS independiente** con su propio `CMakeLists.txt` y `package.xml`.

---

## jeep_master_node

**Tipo:** Paquete de lanzamiento (sin codigo fuente)

Contiene las configuraciones de `roslaunch` que orquestan el inicio del sistema completo. Define tres modos de operacion:

| Launch File | Nodos que inicia |
|-------------|-----------------|
| `jeep_master_node_lane.launch` | lane_detector, navigation_control, nxp_communication |
| `jeep_master_node_yolo.launch` | ros_yolov3, navigation_control, nxp_communication |
| `jeep_master_node_nxp.launch` | navigation_control, nxp_communication |

---

## jeep_msgs

**Tipo:** Paquete de mensajes

Define los tipos de mensaje personalizados del proyecto.

### yolov3_msg.msg

```
string   name    # Nombre del objeto detectado (person, car, etc.)
float32  depth   # Profundidad en metros desde la camara estereo
float32  prob    # Probabilidad de deteccion (0.0 - 1.0)
```

**Dependencias:** `std_msgs`

---

## lane_detector

**Tipo:** Modulo de percepcion (Python)

Detecta los limites del carril en imagenes de la camara ZED usando vision por computadora.

### Archivos principales

| Archivo | Funcion |
|---------|---------|
| `scripts/lane_detector_impl.py` | Nodo ROS principal. Pipeline completo de deteccion |
| `scripts/camara_pista.py` | Deteccion de marcadores de carril (rojo/azul) por HSV |
| `scripts/camara.py` | Interfaz general de camara |
| `scripts/hsv.py` | Utilidades de espacio de color HSV |
| `scripts/functions/utils.py` | Pipeline de imagen: perspectiva, ventana deslizante, Sobel |

### Parametros de Color

**Lineas rojas:**
- HSV: `[0, 50, 50]` a `[12, 255, 255]` y `[160, 50, 50]` a `[188, 255, 255]`

**Lineas azules:**
- HSV: `[94, 80, 2]` a `[126, 255, 255]`

### Topics Publicados

| Topic | Tipo | Descripcion |
|-------|------|-------------|
| `/lane_detector/out_image` | Image | Imagen con overlay de carriles detectados |
| `/lane_detector/warped_image` | Image | Vista de pajaro (perspectiva transformada) |
| `/lane_detector/sliding_window_image` | Image | Visualizacion del algoritmo de ventana deslizante |
| `/lane_detector/steer_angle` | Float32 | Angulo de direccion recomendado |
| `/lane_detector/error_lat` | Float32 | Error lateral en pixeles |

### Configuracion

El archivo `config/zed_default.yaml` define los topics de entrada de la camara ZED.

---

## ros_yolov3

**Tipo:** Modulo de percepcion (C++)

Deteccion de objetos en tiempo real usando YOLOv3-tiny con integracion de la camara ZED para obtener profundidad.

### Archivo principal

- `src/main.cpp` - Wrapper de Darknet con ZED SDK, procesamiento GPU

### Funcionamiento

1. Inicializa la red YOLOv3-tiny con pesos pre-entrenados (`yolov3-tiny.weights`)
2. Captura frames de la camara ZED (RGB + mapa de profundidad)
3. Ejecuta inferencia en GPU (CUDA)
4. Extrae detecciones con bounding box, clase, confianza y profundidad
5. Publica cada deteccion como `jeep_msgs::yolov3_msg`

### Topics Publicados

| Topic | Tipo | Descripcion |
|-------|------|-------------|
| `/yolo_detections_topic` | jeep_msgs::yolov3_msg | Objetos detectados con profundidad |

### Dependencias Especiales

- Darknet (incluido como `libdarknet/`)
- CUDA 9.1+
- cuDNN
- ZED SDK

---

## navigation_control

**Tipo:** Modulo de decision y control (C++)

Recibe datos de percepcion (YOLO + carriles) y genera comandos de control para el vehiculo.

### Archivo principal

- `src/navigation_control.cpp`

### Logica de Control

- **Direccion:** Controlador de modo deslizante que minimiza el error lateral
  - `sigma = C1 * error + delta_error / Ts`
  - Saturacion para limitar el angulo maximo
- **Evasion:** Funcion gaussiana sobre la profundidad del objeto mas cercano
- **Aceleracion:** Funcion exponencial inversamente proporcional a la distancia del obstaculo
- **Filtrado:** Filtro de mediana sobre 8 muestras de deteccion
- **Conversion:** Pixeles a metros con factor `K = 0.0035`

### Topics Suscritos

| Topic | Tipo | Fuente |
|-------|------|--------|
| `/yolo_detections_topic` | jeep_msgs::yolov3_msg | ros_yolov3 |
| `/lane_detector/error_lat` | Float32 | lane_detector |
| `/lane_detector/steer_angle` | Float32 | lane_detector |

### Topics Publicados

| Topic | Tipo | Descripcion |
|-------|------|-------------|
| `/I2C/nxp_communication` | Float32MultiArray | Array de 3 elementos: [angulo, aceleracion, freno] |

---

## nxp_communication

**Tipo:** Modulo de actuacion (C++)

Interfaz I2C con el microcontrolador NXP S32K148 que controla los actuadores fisicos del vehiculo.

### Archivo principal

- `src/nxp_communication.cpp`

### Funcionamiento

1. Recibe comandos de `navigation_control` via topic
2. Establece conexion I2C con el NXP (direccion `0x1D`, bus 0)
3. Envia comandos de aceleracion, frenado y direccion
4. Recibe confirmaciones del microcontrolador

### Topics Suscritos

| Topic | Tipo |
|-------|------|
| `/I2C/nxp_communication` | Float32MultiArray |

### Topics Publicados

| Topic | Tipo |
|-------|------|
| `/I2C/receive` | String (acknowledgment) |

---

## lidarlite_node

**Tipo:** Modulo de sensor (C++)

Interfaz I2C con el sensor LiDAR Lite v3 para medicion de distancia.

### Archivo principal

- `src/lidarlite_communication.cpp`

### Funcionamiento

1. Inicializa comunicacion I2C con el LiDAR (direccion `0x62`, bus 0)
2. Lee mediciones de distancia en centimetros
3. Publica datos en topic ROS

### Topics Publicados

| Topic | Tipo | Descripcion |
|-------|------|-------------|
| `/I2C/LidarLite_data` | Int64 | Distancia medida en centimetros |

---

## vga_zed_wrapper

**Tipo:** Wrapper de camara (C++)

Nodo que encapsula la camara ZED y publica sus imagenes en topics ROS estandar.

### Topics Publicados

| Topic | Descripcion |
|-------|-------------|
| `/zed/left/image_rect_color` | Imagen rectificada de la camara izquierda |
| `/zed/rgb/image_raw_color` | Imagen RGB sin procesar |

---

## testing

**Tipo:** Scripts de prueba y utilidades (Python)

| Script | Funcion |
|--------|---------|
| `scripts/lane_img_show.py` | Visualiza resultados de deteccion de carril |
| `scripts/listener.py` | Escucha y muestra mensajes de topics ROS |
