# Configuracion de Hardware

## Sensores y Dispositivos

### Camara Estereo ZED

La camara ZED proporciona imagenes RGB y mapas de profundidad para la percepcion del entorno.

| Parametro | Valor |
|-----------|-------|
| Modelo | ZED (Stereolabs) |
| SDK | ZED SDK 2.x |
| Resolucion | Configurable (VGA por defecto para rendimiento) |
| Datos | RGB + Mapa de profundidad estereo |

**Montaje:** Frontal, centrada en el techo del vehiculo.

**Topics ROS:**
- `/zed/left/image_rect_color` - Imagen rectificada izquierda
- `/zed/rgb/image_raw_color` - Imagen RGB sin procesar

### LiDAR Lite v3

Sensor de distancia de bajo costo para medicion puntual.

| Parametro | Valor |
|-----------|-------|
| Modelo | Garmin LiDAR Lite v3 |
| Rango | 0 - 40 metros |
| Precision | +/- 2.5 cm |
| Interfaz | I2C |
| Direccion I2C | `0x62` |
| Bus I2C | 0 |

**Topic ROS:** `/I2C/LidarLite_data` (Int64, distancia en cm)

### Microcontrolador NXP S32K148

Controlador de bajo nivel que recibe comandos de ROS y actua sobre el motor, la direccion y los frenos.

| Parametro | Valor |
|-----------|-------|
| Modelo | NXP S32K148 |
| Interfaz | I2C |
| Direccion I2C | `0x1D` |
| Bus I2C | 0 |

**Comandos que recibe:**

| Indice | Parametro | Rango |
|--------|-----------|-------|
| 0 | Angulo de direccion | Variable |
| 1 | Aceleracion | 0.0 - 1.0 |
| 2 | Freno | 0.0 - 1.0 |

## Diagrama de Conexion I2C

```
Computadora (Jetson / PC)
    |
    +-- I2C Bus 0
    |     |
    |     +-- [0x1D] NXP S32K148 (Control del vehiculo)
    |     |
    |     +-- [0x62] LiDAR Lite v3 (Sensor de distancia)
    |
    +-- I2C Bus 1
          |
          +-- [0x1D] IMU / Acelerometro
```

## GPU

La deteccion de objetos YOLOv3 requiere una GPU NVIDIA con soporte CUDA.

| Requisito | Minimo |
|-----------|--------|
| CUDA Compute Capability | 3.0+ |
| VRAM | 2 GB (YOLOv3-tiny), 4 GB (YOLOv3 completo) |
| CUDA | 9.1+ |
| cuDNN | 5 - 7 |

**Plataformas probadas:**
- NVIDIA Jetson TX2
- PC con GPU NVIDIA dedicada

## Notas de Integracion

### Permisos I2C

En Linux, el acceso a los buses I2C requiere permisos. Para evitar ejecutar como root:

```bash
# Agregar tu usuario al grupo i2c
sudo usermod -aG i2c $USER

# Crear regla udev si es necesario
echo 'SUBSYSTEM=="i2c-dev", MODE="0666"' | sudo tee /etc/udev/rules.d/99-i2c.rules
sudo udevadm control --reload-rules
```

### Verificar dispositivos I2C

```bash
# Listar buses I2C disponibles
ls /dev/i2c-*

# Escanear dispositivos en el bus 0
sudo i2cdetect -y 0

# Debe mostrar:
#   0x1D -> NXP S32K148
#   0x62 -> LiDAR Lite v3
```

### Camara ZED

La camara ZED se conecta via USB 3.0. Verificar conexion:

```bash
# Listar dispositivos de video
ls /dev/video*

# Probar con el visor de ZED
/usr/local/zed/tools/ZED_Explorer
```
