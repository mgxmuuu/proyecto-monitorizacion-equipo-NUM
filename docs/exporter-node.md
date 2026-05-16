# 1. Identificación
**Nombre:** Keyla Galarraga  
**Exporter:** Node Exporte
# 2. Que hace este exporter
Herramientas más fundamentales en el ecosistema de monitoreo de Prometheus. Básicamente, actúa como un "agente" que se instala en servidores Linux (o sistemas tipo Unix) para extraer métricas del hardware y del sistema operativo. A diferencia de otras herramientas que "empujan" los datos hacia una base de datos, el Node Exporter expone una dirección URL (generalmente en el puerto 9100) donde publica el estado actual del servidor en un formato que Prometheus puede leer.
# 3. Como se instala
Paso 1: Creacion del usuario de sistema

```bash
sudo useradd --no-create-home --shell /bin/false node_exporter
```
Paso 2: Descarga e instalación del binario

```bash
cd /tmp wget https://github.com/prometheus/node_exporter/releases/download/v1.8.2/node_exporter-1.8.2.linux-amd64.tar.gz
tar xvf node_exporter-1.8.2.linux-amd64.tar.gz 
sudo cp node_exporter-1.8.2.linux-amd64/node_exporter /usr/local/bin/ 
sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter

```
Paso 3: Configuración del servicio (Systemd)

```bash
sudo nano /etc/systemd/system/node_exporter.service
```
Contenido del archivo:
```bash
[Unit] 
Description=Node Exporter
After=network.target 
[Service] User=node_exporter 
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter 
[Install] 
WantedBy=multi-user.target

```
Paso 4: Activacion del servicio
```bash
sudo systemctl daemon-reload 
sudo systemctl enable --now node_exporter
```
Paso 5:  Configuración de Prometheus habra que editar el fichero prometheus.yml
```bash
sudo nano /etc/prometheus/prometheus.yml
```
Editar el fichero:
```bash
- job_name: 'node_exporter'
    static_configs:
      - targets: ['localhost:9100']
        labels:
          entorno: 'nombre del entorno'
          equipo: 'nombre del equipo'
```
Paso 6 : Reinicio del servidor
```bash
sudo systemctl restart prometheus
```
Paso 7: Verificación de la Infraestructura, si todo esta bien pues estara active running
```bash
sudo systemctl status node_exporter
```
# 4. Tres métricas clave

### 4.1 Uso de CPU  

**Nombre exacto de la métrica:** node_cpu_seconds_total  

**Tipo:** Counter  

**Qué mide:** El tiempo total (en segundos) que cada núcleo de la CPU ha pasado en un modo específico (como user, system, iowait, idle, etc.) desde que se inició el sistema.  

**Por qué la he elegido:** Es la métrica base indispensable para entender la salud del procesador. Al ser un Counter, no se usa directamente el valor en bruto, sino que se le aplica la función rate() en Prometheus para calcular el porcentaje real de uso de CPU. Permite identificar si el sistema está saturado o si está perdiendo rendimiento esperando operaciones de entrada/salida.  

### 4.2. Disponibilidad de Memoria RAM  

**Nombre exacto de la métrica:** node_memory_MemAvailable_bytes

**Tipo:** Gauge 

**Qué mide:** La cantidad de memoria RAM (en bytes) que está disponible para ser asignada a nuevos procesos o aplicaciones sin necesidad de hacer uso de la memoria de intercambio.

**Por qué la he elegido:** A diferencia de node_memory_MemFree_bytes (que solo mide la memoria completamente vacía), MemAvailable es mucho más precisa para alertas. El kernel de Linux suele usar la memoria "libre" para caché y buffers, pero la libera instantáneamente si una aplicación la necesita. Esta métrica te dice la verdad sobre cuánta memoria real le queda a tu servidor antes de entrar en crisis o activar el OOM Killer.

### 4.3. Saturación de Disco (Tiempo de Actividad)  

**Nombre exacto de la métrica:** node_disk_io_time_seconds_total  

**Tipo:** Counter  

**Qué mide:** El tiempo total (en segundos) que el disco ha estado realizando operaciones de lectura o escritura de manera activa.  

**Por qué la he elegido:** En lugar de mirar solo el espacio libre, esta métrica es clave para medir el rendimiento y el estrés del almacenamiento. Si aplicas un rate() a esta métrica, obtienes la utilización del disco (por ejemplo, un valor de 0.8 significa que el disco está ocupado el 80% del tiempo). Es crucial para detectar cuellos de botella antes de que las aplicaciones comiencen a degradarse por culpa de un almacenamiento lento.  

# 5. Una alerta propuesta

Para garantizar la estabilidad del sistema, se ha definido la siguiente regla de alerta temprana en Prometheus:

#### Definición en Lenguaje Natural
> **"Si la memoria disponible (`node_memory_MemAvailable_bytes`) cae por debajo del 10% de la memoria total del servidor durante 15 minutos, avisar."**

#### Justificación del Umbral y Tiempos

* **Uso de Porcentajes vs. Valores Fijos:** La alerta calcula el porcentaje dinámicamente:
  $$\frac{\text{node\_memory\_MemAvailable\_bytes}}{\text{node\_memory\_MemTotal\_bytes}} \times 100 < 10$$
  Esto permite que la alerta sea **escalable** y reutilizable en servidores de cualquier tamaño (desde instancias de 4GB hasta nodos de 128GB).
* **Uso de `MemAvailable`:** Nos asegura que la alerta solo se dispare cuando el servidor realmente sufra por falta de recursos, ignorando el comportamiento nativo de Linux de usar la memoria libre para optimización de caché.
* **Ventana de Tiempo (15 minutos):** Funciona como un escudo contra **falsos positivos**. Evita que picos legítimos y temporales de uso de memoria (como backups programados o tareas cron) saturen de notificaciones al equipo de operaciones, alertando únicamente cuando existe una tendencia crítica persistente.

# 5. Limitación Técnica Identificada

Al implementar este sistema de monitoreo, es fundamental conocer el alcance de las herramientas para evitar puntos ciegos en la infraestructura.

#### Limitación: Falta de contexto y métricas a nivel de procesos o contenedores

* **Descripción:** Node Exporter está diseñado única y exclusivamente para medir el estado del **sistema operativo anfitrión (Host)** de forma global. No tiene visibilidad de qué procesos específicos, usuarios o contenedores de Docker/Kubernetes están consumiendo esos recursos.
* **Impacto en la Operación:** Si la alerta de CPU o Memoria se dispara (por ejemplo, el uso de CPU llega al 95%), Node Exporter te dirá *qué* núcleo está sufriendo y *cuánto* tiempo pasa en modo `user`, pero **no puede decirte qué proceso (ej. un hilo corrupto de Java o una consulta de MySQL) está causando el problema**. 
* **Mitigación recomendada:** Para resolver esta limitación y tener una observabilidad completa, Node Exporter debe complementarse con otras herramientas en el stack de Prometheus:
  * **`cAdvisor`:** Para obtener métricas detalladas si el servidor corre contenedores (Docker/Podman).
  * **`Process Exporter`:** Si se desea monitorear el consumo de procesos específicos del sistema operativo por nombre o PID.

