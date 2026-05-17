# **SSL-Exporter**

### **Experto**
**Alejandro Dutor**  
<br>
**1º ASIR**

<br>

---

## ¿Qué hace este exporter?

El **SSL Exporter** se encarga de monitorizar certificados **SSL/TLS** de servicios web.

Su objetivo es:
- Garantizar la seguridad de conexiones HTTPS  
- Detectar certificados caducados o mal configurados  
- Anticipar fallos críticos en producción  

Permite mejorar la **seguridad y disponibilidad** de los servicios web.

<br>

---

## ¿Cómo se instala?

### 1. Creamos el usuario del exporter --> Usuario sin permisos de login para ejecutar el servicio de forma segura:
```bash
sudo useradd --no-create-home --shell /usr/sbin/nologin ssl_exporter

<br>2. Descargamos SSL-Exporter --> Vamos al repositorio oficial de GitHub y descargamos el binario:

cd /tmp
wget https://github.com/ribbybibby/ssl_exporter/releases/latest/download/ssl_exporter-linux-amd64.tar.gz

<br>3. Descomprimimos el archivo --> Extraemos el contenido del paquete descargado:

tar -xvf ssl_exporter-linux-amd64.tar.gz
ls ssl_exporter-linux-amd64

<br>4. Instalamos el binario --> Lo movemos al sistema para hacerlo ejecutable globalmente:

sudo mv ssl_exporter-linux-amd64/ssl_exporter /usr/local/bin/
sudo chmod +x /usr/local/bin/ssl_exporter

<br>5. Creamos el servicio systemd --> Definimos el servicio para que se ejecute en segundo plano:

sudo nano /etc/systemd/system/ssl_exporter.service

[Unit]
Description=SSL Exporter for Prometheus
After=network.target

[Service]
User=ssl_exporter
Group=ssl_exporter
ExecStart=/usr/local/bin/ssl_exporter

[Install]
WantedBy=multi-user.target

<br>6. Activamos el servicio --> Recargamos systemd y lo iniciamos:

sudo systemctl daemon-reload
sudo systemctl enable --now ssl_exporter

<br>7. Comprobamos que funciona --> Verificamos que el servicio está activo:

sudo systemctl status ssl_exporter

<br>8. Probamos el endpoint --> Comprobamos que expone métricas correctamente:

curl http://localhost:9219/metrics

📍 Endpoint por defecto:

http://localhost:9219/metrics

<br>
---

Puerto por defecto

Parámetro	Valor

Puerto por defecto	9219
Configurable	Sí


<br>
---

Métricas relevantes

Métrica	Tipo	Qué mide	Por qué es útil

ssl_cert_not_before	Gauge	Fecha desde la que el certificado SSL es válido	Detecta certificados mal instalados o usados antes de su activación
ssl_file_read_errors	Counter	Errores al leer archivos de certificados SSL	Detecta problemas de permisos, rutas o archivos corruptos
ssl_tls_version_info	Gauge	Versión TLS usada en la conexión	Identifica protocolos antiguos o inseguros que suponen un riesgo


<br>
---

Ejemplo de alerta

groups:
- name: Alertas_SSL
  rules:

    - alert: CertificadoSSLExpirando
      expr: ssl_cert_not_after - time() < 86400 * 7
      for: 10m
      labels:
        severity: critical
      annotations:
        summary: "Certificado SSL próximo a caducar"
        description: "El certificado SSL expirará en menos de 7 días."

Qué hace esta alerta --> Se activa cuando un certificado está a menos de 7 días de expirar. En ese caso, Prometheus lo marca como crítico y envía una notificación al sistema de alertas para que el administrador lo renueve antes de que afecte al servicio.

<br>
---

Limitaciones del exporter

Solo monitoriza certificados SSL/TLS

No analiza rendimiento del servidor ni aplicaciones

No identifica la causa interna del fallo, solo el resultado


<br>
---

¿Por qué he escogido este exporter?

He elegido el SSL Exporter porque se centra directamente en la seguridad de las conexiones HTTPS, a diferencia de otros exporters como Nginx.

Permite:

Detectar certificados próximos a caducar

Evitar errores de seguridad en navegadores

Mejorar la disponibilidad del servicio


Es una herramienta clave para entornos en producción donde la seguridad es crítica.
