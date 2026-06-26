[![CI](https://github.com/PabloTarrio/rplidar-tcp-client/actions/workflows/ci.yml/badge.svg?branch=main)](https://github.com/PabloTarrio/rplidar-tcp-client/actions/workflows/ci.yml)
[![PyPI](https://img.shields.io/pypi/v/rplidar-tcp-client.svg)](https://pypi.org/project/rplidar-tcp-client/)
[![PyPI Downloads](https://img.shields.io/pypi/dm/rplidar-tcp-client.svg)](https://pypistats.org/packages/rplidar-tcp-client)
[![License](https://img.shields.io/pypi/l/rplidar-tcp-client.svg)](https://pypi.org/project/rplidar-tcp-client/)
[![Python](https://img.shields.io/pypi/pyversions/rplidar-tcp-client.svg)](https://pypi.org/project/rplidar-tcp-client/)
[![Code style: ruff](https://img.shields.io/badge/code%20style-ruff-000000.svg)](https://github.com/astral-sh/ruff)
[![Coverage](https://img.shields.io/badge/coverage-88%25-brightgreen.svg)](https://github.com/PabloTarrio/rplidar-tcp-client)

# rplidar-tcp-client

Librería Python para acceder remotamente a datos del sensor RPLIDAR A1 conectado a una Raspberry Pi 4 mediante TCP sockets.

## Objetivo

Proporcionar una forma simple y directa de obtener datos de escaneo LIDAR desde cualquier ordenador mediante TCP, sin necesidad de instalar ROS 2.

## Características

- **Sin dependencias de ROS 2**: Comunicación TCP pura con Python estándar
- **Acceso remoto**: Conecta desde cualquier PC en la misma red
- **Configuración simple**: Archivo `config.ini` con tu LIDAR asignado
- **Reconexión automática**: Reintentos configurables si falla la conexión
- **Plug & play**: API simple con context managers
- **Fácil instalación**: `pip install` directo
- **Ejemplos incluidos**: Scripts listos para usar

## Requisitos

### Servidor (Raspberry Pi 4)
- Raspberry Pi 4 con Ubuntu 24.04 Server
- RPLIDAR A1 conectado vía USB
- Python 3.10+
- Librería `rplidar` instalada

### Cliente (tu PC)
- Python 3.10+
- Conexión de red a la Raspberry Pi


## Instalación rápida (1 minuto)

```bash
pip install rplidar-tcp-client
# Con visualización (matploblib)
pip install "rplidar-tcp-client[visualization]"
```

```python
from lidarClient.client import LidarClient

client = LidarClient("192.168.1.103", 5000)
client.connect()
scan = client.get_scan()  # Tu primera revolución
print(f"{len([p for p in scan if p > 0])} puntos válidos")[1]
client.disconnect()
```

## Quick start - Tu primera medición en 10 minutos

### 1. Instalación (2 minutos)

```bash
# Clonar el repositorio
git clone https://github.com/UIE-Laboratorio-Ingenieria/rplidar-tcp-client.git
cd rplidar-tcp-client

# Crear entorno virtual
python3 -m venv venv
source venv/bin/activate  # En Windows: venv\Scripts\activate

# Instalar la librería
pip install -e 
```

### 2. Configuración (3 minutos)

```bash
# Copiar plantilla de configuración
cp config.ini.example config.ini

# Editar con tu LIDAR asignado
nano config.ini
```

Escoge tu LIDAR del laboratorio y edita la lines `host`:

```bash
[lidar]
# LIDAR 1: 192.168.1.101
# LIDAR 2: 192.168.1.102
# LIDAR 3: 192.168.1.103
# LIDAR 4: 192.168.1.104
# LIDAR 5: 192.168.1.105
# LIDAR 6: 192.168.1.106

host = 192.168.1.103  # 👈 Cambia esto por tu LIDAR
port = 5000
timeout = 5.0
scanmode = Express
```

### 3. Tu primer escaneo (5 minutos)

```python
# Guarda esto como test_lidar.py
from lidar_client import LidarClient
from lidar_client.config import load_config

# Cargar configuración
config = load_config()

# Conectar y obtener una revolución
with LidarClient(config['host'], port=config['port']) as client:
    print("Conectando al LIDAR...")
    scan = client.get_scan()
    
    # Analizar resultados
    valid_points = [p for p in scan if p[2] > 0] 
    print(f" Revolución recibida: {len(valid_points)} puntos válidos")
    
    # Mostrar punto más cercano
    if valid_points:
        closest = min(valid_points, key=lambda p: p[2])
        print(f"Objeto más cercano: {closest[2]:.0f}mm a {closest[1]:.1f}°")
```

Ejecutar:

```bash
python test_lidar.py
```

Salida esperada:

```bash
Conectando al LIDAR...
Revolución recibida: 347 puntos válidos
Objeto más cercano: 358mm a 187.8°
```

### 4. Explorar ejemplos:

```bash
# Escaneo básico
python examples/simple_scan.py

# Stream continuo con estadísticas
python examples/continuous_stream.py

# Visualización en tiempo real (requiere matplotlib)
pip install matplotlib numpy
python examples/visualize_realtime.py

# Guardar datos en CSV
python examples/lidar_to_csv.py --revs 5 --out datos.csv
```

> **¿Problemas?** Consulta la seccion de [Solución de Problemas](#solución-de-problemas) al final de este documento.

## Instalación detallada

### 1. En tu PC (cliente)

```bash
git clone https://github.com/UIE-Laboratorio-Ingenieria/rplidar-tcp-client.git
cd rplidar-tcp-client
python3 -m venv venv
source venv/bin/activate  # En Windows: venv\Scripts\activate
pip install -e .
```

### 2. Configurar tu LIDAR

Copia el archivo de ejemplo y edita la IP de tu LIDAR asignado:

```bash
cp config.ini.example config.ini
nano config.ini # o usa tu editor favorito
```
Edita la linea `host` con la Ip de tu servidor LIDAR:

```text
[lidar]
#Cambia esta IP por la de tu LIDAR asignado
host = 192.168.1.103
port = 5000
timeout = 5.0
max_retries = 3
retry_delay = 2.0
scan_mode = Express
```
LIDAR disponibles en el Laboratorio:

* LIDAR 1: 192.168.1.101
* LIDAR 2: 192.168.1.102
* LIDAR 3: 192.168.1.103
* LIDAR 4: 192.168.1.104
* LIDAR 5: 192.168.1.105
* LIDAR 6: 192.168.1.106

>NOTA: El archivo `config.ini` es local y no se sube a GIT (está en .gitignore)

### 3. En la Raspberry PI (servidor)
El servidor TCP debe estar corriendo en la Raspberry Pi. Consulta la documentación en [server/README.md](/server/README.md) para instrucciones de instalación.

## Uso Básico / Ejemplos

### Ejemplo simple
```python
from lidarclient import LidarClient
from lidarclient.config import load_config

# Cargar configuración desde config.ini
config = load_config()

# Conectar al servidor
with LidarClient(
    config["host"],
    port = config["port"],
    timeout = config["timeout"],
    max_retries = config["max_retries"],
    retry_delay = config["retry_delay"],
    scan_mode = config["scan_mode"]
) as client:
    # Obtener una revolución completa
    scan = client.get_scan()
    
    print(f"Recibidos {len(scan)} puntos")
    
    # Cada punto es una tupla (quality, angle, distance)
    for quality, angle, distance in scan[:5]:
        print(f"Ángulo: {angle:.2f}°, Distancia: {distance:.2f}mm")
```

## Para estudiantes e Investigadores

### Casos de uso académico
- **Robótica móvil**: Navegación autónoma, evitación de obstáculos
- **Mapeo y SLAM**: Construcción de mapas 2D del entorno
- **Visión Artificial**: Fusión de sensores LIDAR + cámara
- **Algoritmos de Control**: Detección de entornos para control reactivo
- **Proyectos Fin de Grado/Máster**: Base sólida para investigación

### Ejemplos progresivos por Nivel

#### Nivel básico (Primeros Pasos)

- `simple_scan.py` - Tu primera medición LIDAR
- `understanding_dat.py` - Entender el formato de datos.
- `continuous_stream.py` - Stream continuo con estadísticas
- `print_scan_stub.py` - Formato compatible con ROS 2 LaserScan

**Ideal para**: Familiarizarse con el sensor, entender el formato de los datos

#### Nivel intermedio (Análisis y visualización)

- `visualize_realtime.py` - Visualización gráfica en tiempo real
- `lidar_diagnostics.py` - Comparar modos Standard y Express
- `lidar_tc_csv.py` / `lidar_to_json.py` - Exportar datos para análisis

**Ideal para**: Debugging, análisis de rendimiento, crear datasets

#### Nivel Avanzado (Filtrado y Procesamiento)
- `filter_by_quality.py` - Filtrado por calidad de medición (0-15), con histograma
- `filter_by_distance.py` - Filtrado por rango de distancia, zonas de seguridad
- `filter_by_angle.py` - Filtrado por sector angular, análisis multi-sector

**Ideal para**: Implementar algoritmos, proyectos de investigación

### Ventajas para Investigación

**Sin dependencias ROS 2**: Usa Python puro, más ligero y portable  
**Configuración simple**: Un archivo `config.ini` y listo  
**Datos en tiempo real**: Acceso directo vía TCP desde cualquier PC  
**Múltiples formatos**: CSV, JSON, JSONL para análisis offline  
**Bien documentado**: Ejemplos comentados paso a paso  
**Extensible**: API clara para añadir funcionalidad personalizada  

---

### Recursos Adicionales

- **Documentación completa**: Ver [`examples/README.md`](examples/README.md)
- **Guía de contribución**: [`CONTRIBUTING.md`](CONTRIBUTING.md)
- **Solución de problemas**: Ver [sección de troubleshooting](#solución-de-problemas)

Todos los ejemplos leen automaticamente tu `config.ini`, así que solo necesitas configurarlo una vez.

Consulta [examples/README.md](/examples/README.md) para más detalles sobre cada ejemplo.

## Estructura del proyecto
```text
rplidar-tcp-client/
|___ config.ini.example             # Plantilla de configuración
|___ src/
|    |___lidarclient/
|        |___ __init__.py
|        |___ client.py
|        |___ config.py             # Parser de configuración
|___ examples/                      # Scripts de ejemplo
|___ 01_básico                      # Ejemplos fundamentales
|       |___ simple_scan.py
|       |___ continuous_stream.py
|       |___ print_scan_stub.py
|       |___ understanding_data.py
|___ 02_intermedio                  # Análisis y exportación
|       |___ lidar_diagnostics.py
|       |___ lidar_to_csv.py
|       |___ lidar_to_json.py
|       |___ streaming_lidar_to_jsonl.py
|       |___ visualize_realtime.py
|___ 03_avanzado                    # Filtrado y procesamiento
|       |___ filter_by_quality.py
|       |___ filter_by_distance.py
|       |___ filter_by_angle.py   
|___ README.md                      # Documentación detallada de cada ejemplo
|___ server/                     
|___ |___servidor_lidar_tcp.py      # Código del servidor (Raspberry Pi)
|___ |___README.md                  # Documentación servidor
|___ tests/                         # Tests
|___ docs/                          # Documentación adicional
|___ |___DATA_FORMAT.md             
```

## Formato de Datos del LIDAR

### Estructura de una Revolución

El servidor TCP envía cada revolución del RPLIDAR como una lista de tuplas de la forma:

```python
scan = [
    (quality, angle, distance),
    (quality, angle, distance),
    ...
] 
```
donde:

* `quality` es un `int` 0-15 (modo Standard) o `None` (modo Express)
* `angle` es un `float` en grados (0.0 - 359.99)
* `distance` es un `float` en milímetros
### Documentación detallada

La documentación detallada del formato de datos, diferencias entre modos Standard y Express, ejemplos de filtrado y casos especiales está en:

* [`docs/DATA_FORMAT.md`](docs/DATA_FORMAT.md)

## Configuración avanzada
Parámetros del `config.ini`:

* `host` (obligatorio): IP del servidor LIDAR
* `port` (default: 5000): Puerto TCP del servidor
* `timeout` (default: 5.0): Timeout en segundos para operaciones de red
* `max_retries` (default: 3): Número de reintentos si falla la conexión
* `retry_delay` (default: 2.0): Segundos de espera entre reintentos
* `scan_mode` (default: Express): Modo de escaneo del LIDAR
    - `Standard`: ~360 puntos/revolución, incluye datos de calidad (0-15)
    - `Express` : ~720 puntos/revolución, sin datos de calidad


Uso sin `config.ini` (avanzado)

Si necesitas especificar la IP directamente en el código:

```python
from lidarclient import LidarClient

client = LidarClient("10.0.0.5", port=5000, max_retries=3, scan_mode= 'Express')
client.connect_with_retry()
scan = client.get_scan()
client.disconnect()
```

## Solución de problemas

#### Error: `No se encontró el archivo 'config.ini'`

Solución:
```bash
cp config.ini.example config.ini
nano config.ini # Edita la IP de tu LIDAR
```

#### Error: `Connection refused`

Causas posibles:

* El servidor TCP no está corriendo en la Raspberry Pi.
* La IP en `config.ini` es incorrecta
* Problema de red/firewall

Solución:

1. Verifica que el servidor está corriendo: 
```bash 
sudo systemctl status rplidar-server.service
```
2. Comprueba la IP:
```bash 
ping <IP_de_tu_config.ini>
```
3. Verifica que el puerto 5000 está abierto
```bash 
sudo ss -tlnp | grep 5000
```

#### Error: `No module named 'lidarclient'`

Solución:

* Asegúrate de haber instalado el paquete:
```bash 
pip install -e .
```
* Activa el entorno virtual si lo estás usando: 
```bash
source venv\bin\activate
```
#### Timeout al conectar

Solución:
Aumenta el `timeout` en `config.ini`:
```text
timeout = 10.0
```

## Desarollo

#### Ejecutar tests
```bash
pytest
```

#### Ejecutar linting
```bash
ruff check .
ruf format .
```

## Contribuir

Lee [CONTRIBUTING.md](/CONTRIBUTING.md) para conocer el workflow de desarrollo.

## Licencia

Este proyecto está bajo licencia MIT. Ver [LICENSE](/LICENSE) para más detalles

## Documentacion adicional

* [CHANGELOG.md](/CHANGELOG.md): Historial de cambios.
* [CODE_OF_CONDUCT.md](/CODE_OF_CONDUCT.md): Código de conducta.
* [examples/README.md](/examples/README.md): Detalles sobre los ejemplos disponibles
* [server/README.md](/server/README.md): Configuración del servidor en Raspberry Pi.

## Enlaces relacionados
* [SLAMTEC RPLIDAR A1 Datasheet](https://www.slamtec.com/en/Lidar/A1)
* Librería Python: [rplidar-roboticia](https://github.com/Roboticia/RPLidar)
