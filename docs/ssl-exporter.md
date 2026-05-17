SSL-Exporter

Experto

Alejandro Dutor
<br> 1º ASIR

<br>
---

## ¿Qué hace este exporter?

El SSL Exporter se encarga de monitorizar certificados SSL/TLS de servicios web.

Su objetivo es:

Garantizar la seguridad de conexiones HTTPS

Detectar certificados caducados o mal configurados

Anticipar fallos críticos en producción


Permite mejorar la seguridad y disponibilidad de los servicios web.

<br>
---

## ¿Cómo se instala?

1. Crear usuario del exporter

sudo useradd --no-create-home --shell /usr/sbin/nologin ssl_exporter

<br>
---

2. Descargar el exporter

cd /tmp

wget https://github.com/ribbybibby/ssl_exporter/releases/latest/download/ssl_exporter-linux-amd64.tar.gz

<br>
---

3. Descomprimir

tar -xvf ssl_exporter-linux-amd64.tar.gz
ls ssl_exporter-linux-amd64

<br>
---

4. Instalar binario

sudo mv ssl_exporter-linux-amd64/ssl_exporter /usr/local/bin/
sudo chmod +x /usr/local/bin/ssl_exporter

<br>
---

5. Crear servicio systemd

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

<br>
---

6. Activar servicio

sudo systemctl daemon-reload
sudo systemctl enable --now ssl_exporter

<br>
---

7. Comprobar estado

sudo systemctl status ssl_exporter

<br>
---

8. Probar endpoint

curl http://localhost:9219/metrics

📍 Endpoint expuesto:
http://localhost:9219/metrics

<br>
---

## Puerto por defecto

Parámetro	Valor

Puerto por defecto	9219
Configuración	Sí


<br>
---

## Métricas relevantes

Métrica	Tipo	Qué mide	Por qué es útil

ssl_cert_not_before	Gauge	Fecha desde la que el certificado SSL es válido	Detecta certificados mal instalados o usados antes de su activación
ssl_file_read_errors	Counter	Errores al leer archivos de certificados SSL	Detecta problemas de permisos, rutas o archivos corruptos
ssl_tls_version_info	Gauge	Versión TLS usada en la conexión	Identifica protocolos antiguos o inseguros que suponen un riesgo


<br>
---

## Ejemplo de alerta

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

<br>¿Qué haría esta alerta?

Esta alerta se activaría cuando un certificado SSL esté a punto de expirar (menos de 7 días).

En ese caso:

Prometheus marcaría el servicio como crítico

Se enviaría una notificación (Alertmanager, correo o sistema de alertas)

El administrador debería renovar el certificado inmediatamente


Evita caídas del servicio y problemas de seguridad en HTTPS.

<br>
---

## Limitaciones del exporter

Solo monitoriza certificados SSL/TLS

No analiza rendimiento de servidores ni aplicaciones

No identifica la causa interna del fallo, solo el resultado final


<br>
---

## ¿Por qué he escogido este exporter?

He elegido el SSL Exporter porque se centra directamente en la seguridad de las conexiones HTTPS, a diferencia de otros exporters como Nginx.

Permite:

Detectar certificados a punto de caducar

Evitar errores de seguridad en navegadores

Mejorar la disponibilidad del servicio


Es una herramienta clave para entornos en producción donde la seguridad es crítica.
