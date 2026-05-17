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

### 1. Crear usuario del exporter
**Bash**
```bash
sudo useradd --no-create-home --shell /usr/sbin/nologin ssl_exporter


---

2. Descargar el exporter

Bash

cd /tmp
wget https://github.com/ribbybibby/ssl_exporter/releases/latest/download/ssl_exporter-linux-amd64.tar.gz


---

3. Descomprimir

Bash

tar -xvf ssl_exporter-linux-amd64.tar.gz
ls ssl_exporter-linux-amd64


---

4. Instalar binario

Bash

sudo mv ssl_exporter-linux-amd64/ssl_exporter /usr/local/bin/
sudo chmod +x /usr/local/bin/ssl_exporter


---

5. Crear servicio systemd

Bash

sudo nano /etc/systemd/system/ssl_exporter.service

INI

[Unit]
Description=SSL Exporter for Prometheus
After=network.target

[Service]
User=ssl_exporter
Group=ssl_exporter
ExecStart=/usr/local/bin/ssl_exporter

[Install]
WantedBy=multi-user.target


---

6. Activar servicio

Bash

sudo systemctl daemon-reload
sudo systemctl enable --now ssl_exporter


---

7. Comprobar estado

Bash

sudo systemctl status ssl_exporter


---

8. Probar endpoint

Bash

curl http://localhost:9219/metrics

📍 Endpoint expuesto:

http://localhost:9219/metrics


---

Puerto por defecto

Parámetro	Valor

Puerto por defecto	9219
Configuración	Sí



---

Métricas relevantes

ssl_cert_not_before (Gauge): indica la fecha desde la que el certificado SSL es válido. Es útil porque permite detectar certificados mal instalados o usados antes de su activación, lo que puede provocar errores de conexión.

ssl_file_read_errors (Counter): cuenta los errores al leer archivos de certificados SSL. Es útil porque ayuda a detectar problemas de permisos, rutas incorrectas o archivos corruptos que impedirían el correcto funcionamiento del servicio.

ssl_tls_version_info (Gauge): indica la versión TLS utilizada en la conexión. Es útil porque permite identificar versiones antiguas o inseguras que pueden suponer un riesgo de seguridad.


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

Qué hace esta alerta → Se activa cuando un certificado SSL está a punto de expirar (menos de 7 días). En ese caso, Prometheus lo marca como crítico y envía una notificación al sistema de alertas para que el administrador lo renueve antes de que el servicio deje de ser seguro.


---

Limitaciones del exporter

Solo monitoriza certificados SSL/TLS

No analiza rendimiento del servidor ni aplicaciones

No identifica la causa interna del fallo, solo el resultado



---

¿Por qué he escogido este exporter?

He elegido el SSL Exporter porque se centra directamente en la seguridad de las conexiones HTTPS, a diferencia de otros exporters como Nginx.

Permite:

Detectar certificados próximos a caducar

Evitar errores de seguridad en navegadores

Mejorar la disponibilidad del servicio


Es una herramienta clave para entornos en producción donde la seguridad es crítica.
