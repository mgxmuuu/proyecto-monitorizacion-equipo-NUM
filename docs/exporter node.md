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
