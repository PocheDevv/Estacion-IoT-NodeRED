# Estación IoT — Dashboard de Monitoreo con Node-RED

Sistema de monitoreo y control de una estación ambiental **IoT**, construido con **Node-RED Dashboard 2.0**. Los sensores publican sus lecturas por **MQTT**, un dashboard web las visualiza en tiempo real, permite controlar el dispositivo de forma remota y almacena la data en **Firebase**. Proyecto final del curso *Soluciones Basadas en Internet de las Cosas* (ISIL).

---

## Arquitectura

```
  Sensor BME680
       │ (I2C)
   Arduino
       │ (Serial)
 Raspberry Pi Zero 2W
       │ publica/subscribe MQTT
       ▼
 Broker EMQX  (broker.emqx.io:1883)
       │
       ▼
 Node-RED  (desplegado en AWS EC2)
       ├──► Dashboard 2.0  (visualización y control web)
       └──► Firebase Realtime Database  (almacenamiento en tiempo real)
       ▼
   Usuarios
```

El dispositivo publica automáticamente las lecturas de sus sensores, y se suscribe a un topic de comandos para recibir órdenes (reiniciar, pedir info del host, controlar relays, etc.) desde el dashboard.

---

## Flujos (flows)

El sistema está dividido en 5 flujos exportados, ubicados en la carpeta [`/flows`](./flows):

| Archivo | Pestaña | Descripción |
|---|---|---|
| `Sensor.json` | Sensor Data | Recibe por MQTT las lecturas ambientales y las muestra en *gauges* y *charts*. También las envía a Firebase. |
| `Control_Dispositivo.json` | Control Dispositivo | Envía comandos al dispositivo (relays y salidas PWM) por MQTT, con registro de las últimas operaciones. |
| `Monitor.json` | Monitor del sistema | Consulta el estado del host: información del sistema, *speed test* y estado/reinicio del dispositivo. |
| `Mapa.json` | Mapa | Muestra la ubicación de la estación en un mapa (OpenStreetMap). |
| `CSS.json` | Estilos | CSS global del dashboard (tema oscuro azul). |

---

## Sensores monitoreados

Temperatura (°C), humedad (%), intensidad luminosa, presión atmosférica, elevación e intensidad UV.

---

## Topics MQTT

| Topic | Dirección | Uso |
|---|---|---|
| `isil/stations/001/env_sensors/data` | dispositivo → Node-RED | Lecturas de los sensores (publicado automáticamente) |
| `isil/stations/001/commands` | Node-RED → dispositivo | Comandos de control (relays / PWM) |
| `isil/stations/001/commands/host` | Node-RED → dispositivo | Solicita información del host |
| `isil/stations/001/commands/speedtest` | Node-RED → dispositivo | Solicita prueba de velocidad |
| `isil/stations/001/commands/status` | Node-RED → dispositivo | Solicita estado / reinicio |

---

## Tecnologías

- **Node-RED** + **Dashboard 2.0** (`@flowfuse/node-red-dashboard`)
- **MQTT** sobre broker público **EMQX** (`broker.emqx.io:1883`)
- **Firebase Realtime Database** (`@gogovega/node-red-contrib-firebase-realtime-database`)
- **AWS EC2** (hosting del runtime de Node-RED)
- **OpenStreetMap** (mapa embebido)
- Hardware: sensor **BME680**, **Arduino**, **Raspberry Pi Zero 2W**

---

## Cómo importar los flujos

Estos archivos son *flows* de Node-RED, no una aplicación que se compile. Para usarlos:

1. Ten un entorno de **Node-RED** funcionando (local o en servidor).
2. Instala los nodos necesarios desde *Menú → Manage palette → Install*:
   - `@flowfuse/node-red-dashboard`
   - `@gogovega/node-red-contrib-firebase-realtime-database`
3. En Node-RED, ve a *Menú → Import*, y pega el contenido de cada archivo de `/flows` (o impórtalos como archivo). Recomendado importarlos en este orden: `CSS` → `Sensor` → `Control_Dispositivo` → `Monitor` → `Mapa`.
4. Configura el nodo **mqtt-broker** apuntando a `broker.emqx.io`, puerto `1883` (o a tu propio broker).
5. Configura el nodo **firebase-config** con las credenciales de tu propia base de datos de Firebase.
6. Dale **Deploy** y abre el dashboard en `/dashboard`.

> **Nota:** las credenciales de Firebase y del broker **no** están incluidas en estos flujos (Node-RED las guarda por separado y cifradas). Cada quien debe configurar las suyas.

---

## Autor

**José Gabriel Rosas del Águila (Poche)** — [@PocheDevv](https://github.com/PocheDevv)

Desarrollo del sistema en Node-RED: diseño e implementación de los flujos, el dashboard, la integración MQTT, el almacenamiento en Firebase y la seguridad de acceso.

Proyecto desarrollado para el curso *Soluciones Basadas en Internet de las Cosas* — ISIL.

