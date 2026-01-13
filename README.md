# RingBots - Lucha de Robots Autónomos

## Descripción General

**RingBots** es un proyecto educativo e innovador de robótica autónoma que implementa robots inteligentes capaces de competir de forma autónoma dentro de un ring (arena). Los robots utilizan sensores de distancia, servos y motores paso a paso para navegar y ejecutar estrategias de combate.

El proyecto está dividido en dos componentes principales:
- **Robótica (Firmware & Control)**: Código embebido para los robots (ESP8266)
- **Visión Artificial**: Sistema de procesamiento de imágenes en Python para detección y seguimiento

---

## Estructura del Proyecto

Este es el repositorio **principal** que integra dos submódulos:

### 1. **[Robotica_inteligente_sistemas_autonomos](https://github.com/Rxxllnz/Robotica_inteligente_sistemas_autonomos)**
Contiene el firmware y código embebido de los robots.

**Características principales:**
- Código Arduino para microcontrolador ESP8266
- Máquina de estados no-bloqueante con modos: `IDLE`, `ROTATING`, `MOVING`, `TURN_180`, `SCAN`, `TURN_TO_TARGET`, `DRIVE_FORWARD`, `TURN_BACK`
- Sistema de comunicación Master/Slave mediante ESP-NOW
- Drivers para motores paso a paso, servos y sensores de distancia
- Calibración de sensores incluida
- Modos de prueba (`TEST_MODE` y `SERIAL_STUB`)
- **Submódulo de Comunicaciones** (`COMUNICACIONES_ROBOT/`): Contiene las librerías `MasterComm` y `SlaveComm` como submódulo independiente para gestionar la comunicación entre dispositivos ESP

**Hardware:**
- **MCU**: ESP8266
- **Motores**: Motores paso a paso con driver
- **Servo**: Para barrido del sensor de distancia
- **Sensores**: Sensor analógico de distancia con tabla de calibración
- **Alimentación**: Batería con distribución de energía

**Configuración clave:**
- Radio de rueda: 3.2 cm
- Eje de ruedas: 16.0 cm
- Resolución del motor: 256 pasos
- Velocidad base del motor: 100 unidades

### 2. **[RIySA_Prueba_Visi-n](https://github.com/SergioLugo91/RIySA_Prueba_Visi-n)** 
Contiene la solución completa de visión artificial en Python.

**Características principales:**
- Detección de Arucos (códigos QR visuales) para delimitar el ring y ubicar robots
- Procesamiento con OpenCV
- Comunicación con robots vía TCP/WiFi
- Calibración de cámaras incluida
- Estimación de poses de objetos en tiempo real

**Componentes:**
- `PruebaArUcos.py` - Detección y seguimiento de marcadores ArUcos
- `RingDetector.py` - Identificación y análisis del ring/arena
- `Calibracion/` - Herramientas de calibración de cámaras
- **Submódulo de Comunicaciones** (`RobPCComm/`): Librería de comunicación PC-Robot como submódulo independiente
- Sistema de comunicación PC ↔ Robots
- `GUI.py` - Interfaz del sistema

---

## Arquitectura General del Sistema

```
┌─────────────────────────────────────────────────┐
│        SISTEMA DE VISIÓN (Python + OpenCV)      │
│   Detecta robots, ring y transmite comandos     │
└────────────────────┬────────────────────────────┘
                     │ TCP
                     ▼
        ┌────────────────────────┐
        │   MASTER ESP8266       │
        │ (Comunicación ESP-NOW) │
        └────────┬───────────────┘
                 │ ESP-NOW
        ┌────────┴───────┬───────────┐
        ▼                ▼           ▼
    ┌────────┐       ┌────────┐   ┌────────┐
    │Robot 1 │       │Robot 2 │   │Robot N │
    │(Slave) │       │(Slave) │   │(Slave) │
    └────────┘       └────────┘   └────────┘
    
    Cada robot ejecuta:
    - Lectura de sensores
    - Máquina de estados
    - Control de motores paso a paso
    - Servo para barrido de distancia
```

### Flujo de Operación:
1. **Visión**: Sistema de cámaras procesa frames y detecta posiciones
2. **Comunicación**: Transmite comandos de alto nivel a Master ESP
3. **Master ESP**: Reenvía comandos a robots esclavos vía ESP-NOW si no son para el Master
4. **Robots**: Ejecutan máquina de estados no-bloqueante con:
   - Escaneo de distancia
   - Alineación hacia objetivo
   - Movimiento autónomo seguro

---

## Comenzar Rápidamente

### Requisitos:
- Arduino IDE o PlatformIO
- Python 3.7+
- Librerías necesarias para ESP8266
- OpenCV para Python
- Drivers USB para ESP8266

### Instalación del Firmware de Robots:

1. **Clonar el repositorio con submódulos:**
   ```bash
   git clone --recursive https://github.com/Rxxllnz/Proyecto-Robotica-Inteligente-LLubots.git
   cd Proyecto-Robotica-Inteligente-LLubots
   ```

2. **Compilar y cargar el firmware:**
   ```bash
   cd Robotica_inteligente_sistemas_autonomos/Robot_code
   ```
   - Abrir `Robot_code.ino` en Arduino IDE
   - Seleccionar placa: ESP8266
   - Seleccionar puerto COM
   - Compilar y cargar al microcontrolador
   - Monitor Serie a 115200 baud

3. **Configurar el rol (Master/Slave):**
   - Editar `Robot_code.ino` y buscar la bandera de rol/configuración
   - Configurar una placa como Master y las demás como Slaves

4. **Desactivar modo prueba (opcional):**
   ```cpp
   #define TEST_MODE 0      // Desactivar secuencia de prueba automática
   #define SERIAL_STUB 0    // Desactivar stub de serie
   ```

### Instalación del Sistema de Visión:

1. **Navegar a la carpeta de visión:**
   ```bash
   cd RIySA_Prueba_Visi-n
   ```

2. **Instalar dependencias Python:**
   ```bash
   pip install opencv-python numpy
   ```

3. **Calibrar cámaras (en ultimas versiones Matlab):**
   ```bash
   cd Calibracion
   python calibracion.py
   ```

4. **Ejecutar interfaz de usuario:**
   ```bash
   python GUI.py
   ```

---

## Modos de Operación

### Firmware de Robots:
- **`TEST_MODE`**: Secuencia predefinida sin entrada externa (desarrollo y pruebas)
- **`SERIAL_STUB`**: Escucha comandos por puerto serial (simulación remota)
- **Operación Normal**: Espera comandos de la unidad Master vía ESP-NOW

### Sistema de Visión:
- Detección en tiempo real de ArUcos para identificar robots
- Procesamiento de estereovisión para profundidad
- Estimación de poses en 3D
- Comunicación TCP para enviar comandos

---

## Configuración Importante

### Pins del Firmware:
| Función | Pin | 
|---------|-----|
| LED de estado | D0 |
| Servo (barrido sensor) | D7 |
| Sensor de distancia | A0 |

### Parámetros del Sistema:
- **Frecuencia de visión**: 30 FPS (configurable)
- **Puerto TCP**: 8888
- **Baudrate Serial**: 115200
- **Protocolo inalámbrico**: ESP-NOW

---

## Documentación Detallada

Para información más específica, consulta los READMEs individuales de cada submódulo incluyendo los repositorios de comunicación.


## Autores

**Firmware y Control de Robots:**
- [Raúl Lorenzo](https://github.com/Rxxllnz)
- [Uli Schober](https://github.com/derTapir)

**Sistema de Visión Artificial:**
- [Álvaro Viña](https://github.com/AlvaroVP96)
- [Sergio Lugo](https://github.com/SergioLugo91)

**Comunicaciones:**
- [Pablo Navarro](https://github.com/pnavarro3)
- [Rubén Sahuquillo](https://github.com/RubenSah-ing)

---
## Submódulos de Comunicación

**Módulos de Comunicación** (submódulos independientes integrados en cada repositorio):
- Componentes de comunicación Master/Slave (`MasterComm` y `SlaveComm`)
- Librería de comunicación PC-Robot (`RobPCComm`)

**Nota sobre submódulos:** Los sistemas de comunicación están implementados como submódulos Git independientes dentro de cada repositorio principal, permitiendo su actualización y uso independiente mientras mantienen la integración con los sistemas de robots y visión.

---

## Estado del Proyecto

**Estado**: Desarrollo activo

El proyecto está en desarrollo continuo con mejoras en:
- Optimización de visión artificial
- Modos de competición
- Integracion de interfaz grafica para control y monitoreo

### Nota sobre instalación
Se ha intentado crear un ejecutable (.exe) para facilitar la instalación y ejecución rápida del sistema de visión, sin embargo, por limitaciones de tiempo no ha sido posible completar esta funcionalidad. De momento, la instalación requiere seguir los pasos manuales descritos en la sección "Comenzar Rápidamente".

---

## Notas Técnicas

### Arquitectura del Código:
- **Máquina de estados no-bloqueante**: Permite multiples operaciones concurrentes
- **Contextos de módulos**: Estructuras ligeras que reducen acoplamiento
- **Calibración incluida**: Tablas de interpolación para sensores

### Consideraciones para Desarrollo:
- Los módulos utilizan contextos de dependencias para comunicarse
- El loop principal usa `millis()` para scheduling no-bloqueante
- Al reorganizar archivos, asegurar que las unidades de traducción se compilen
- Considerar convertir en librerías para builds más limpios

---

## Solución de Problemas

**Sin salida en el monitor serie después de cargar:**
- Abrir monitor serie a 115200 y resetear la placa

**Robots no responden a comandos:**
- Verificar conexión ESP-NOW entre Master y Slaves
- Revisar bandera de rol (Master/Slave) en `Robot_code.ino`
- Asegurar que TEST_MODE y SERIAL_STUB estén desactivados en operación normal
- Confirmar que las direcciones MAC son correctas
- Comprobar cada ID de robot
- Comprobar alimentación y conexiones de hardware
- Comprobar puertos WiFi 

**Detección de ArUcos inconsistente:**
- Revisar calibración de cámaras en `Calibracion/`
- Asegurar iluminación adecuada
- Verificar orientación y tamaño de marcadores


**¡Bienvenido al proyecto RingBots! Que disfrutes la competencia autónoma de robots.**
