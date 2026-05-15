# **Mysql Exporter**


### Identificación
 *Mario Garcia Pereiro - ASIR 1*
 <br>
## ¿Que hace este exporter?
Su funcion es traducir la base de datos de SQL a lenuaje prometheus convirtienedo sus consultas en metricas.
<br>
## Guia de Instalacion
# Crear usuario de monitoreo en MySQL

# Entrar a MySQL:
```bash
sudo mysql
```
<br>
# Crear el usuario:

```bash
CREATE USER 'exporter'@'localhost' IDENTIFIED BY 'StrongPassword';
GRANT PROCESS, REPLICATION CLIENT, SELECT ON *.* TO 'exporter'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```
<br>

# Descargar e instalar mysqld_exporter

Actualizar paquetes:
```bash
sudo apt update
sudo apt install -y wget tar
```
# Descargar la versión más reciente:
```bash
cd /tmp

wget https://github.com/prometheus/mysqld_exporter/releases/download/v0.19.0/mysqld_exporter-0.19.0.linux-amd64.tar.gz
```
<br>

# Extraer archivos:
```bash
tar xvf mysqld_exporter-0.19.0.linux-amd64.tar.gz
```
# Mover binario:
```bash
sudo mv mysqld_exporter-0.19.0.linux-amd64/mysqld_exporter /usr/local/bin/
```
# Dar permisos:
```bash
sudo chmod +x /usr/local/bin/mysqld_exporter
```
# Verificar instalación:
```bash
mysqld_exporter --version
```
# Crear archivo de credenciales

```bash

sudo nano /etc/.mysqld_exporter.cnf
```
# Contenido:
```yaml
[client]
user=exporter
password=StrongPassword
```
Guardar y cerrar.

# Asignar permisos seguros:
```bash
sudo chmod 600 /etc/.mysqld_exporter.cnf
```
# Crear usuario del servicio
```bash
sudo useradd --no-create-home --shell /bin/false mysqld_exporter
```
# Crear servicio systemd

```bash
sudo nano /etc/systemd/system/mysqld_exporter.service
```
# Contenido completo:
```yaml
[Unit]
Description=Prometheus MySQL Exporter
After=network.target

[Service]
User=mysqld_exporter
Group=mysqld_exporter
Type=simple

ExecStart=/usr/local/bin/mysqld_exporter \
  --config.my-cnf=/etc/.mysqld_exporter.cnf

Restart=always

[Install]
WantedBy=multi-user.target
```

# Recargar systemd:
```bash
sudo systemctl daemon-reload
```
# Habilitar servicio:
```bash
sudo systemctl enable mysqld_exporter
```
# Iniciar servicio:
```bash
sudo systemctl start mysqld_exporter
```
# Verificar estado:
```bash
sudo systemctl status mysqld_exporter
```
# Probar métricas:
```bash
curl http://localhost:9104/metrics
```
Si aparece salida con métricas Prometheus, el exporter está funcionando correctamente.

# Agregar a Prometheus

Editar el archivo de configuración de Prometheus:
```bash
sudo nano /etc/prometheus/prometheus.yml
```
Agregar:
```yaml
scrape_configs:
  - job_name: 'mysql'
    static_configs:
      - targets: ['localhost:9104']
```
# Reiniciar Prometheus:
```bash
sudo systemctl restart prometheus
```

# Abrir en un navegador para verificar la compatibilidad con Prometheus: 
http://IP_SERVIDOR:9090/targets
<br>

## metricas tipicas
 1. mysql_global_status_threads_connected
      Tipo: gauge.
      Mide el número actual de conexiones activas al servidor MySQL/MariaDB.
      Te da una visión directa de la carga en la base de datos. Si sube demasiado o de forma sostenida, puede indicar saturación o problemas de pooling en la aplicación.
 2. mysql_global_status_slow_queries
      Tipo counter.
      Mide el número total acumulado de consultas lentas ejecutadas
      Permite detectar degradaciones de rendimiento. Si ves que este contador crece rápidamente, hay queries mal optimizadas o problemas de índices.
 3. mysql_global_status_questions
      Tipo counter.
      Mide el número total de consultas que ha recibido el servidor
      Sirve para medir el volumen de tráfico hacia la base de datos. Combinado con otras métricas (como latencia o conexiones) te ayuda a entender picos de carga.
    <br>
4. alerta propuesta:

      “Si mysql_global_status_threads_connected supera 80 conexiones durante más de 5 minutos, avisar.”
<br>
 Limitaciones:

 1. No almacena métricas históricas por sí solo.
 2. Requiere Prometheus y Grafana para visualización y monitoreo completo.
 3. Algunos collectors pueden generar carga adicional en MySQL.
 4. No todas las métricas son compatibles entre MySQL y MariaDB.
 5. No realiza análisis profundo de queries (APM).
6. Necesita permisos específicos en MySQL (`PROCESS`, `REPLICATION CLIENT`, `SELECT`).
 7. Algunos dashboards pueden romperse tras actualizar versiones.
 8. No descubre instancias automáticamente.
 9. Puede exponer información sensible si el puerto `9104` queda público.
 10. Varias métricas avanzadas vienen deshabilitadas por defecto.

