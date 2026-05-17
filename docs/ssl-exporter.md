
---

SSL Exporter — Ficha Técnica

Autor: Alejandro Dutor


---

1. ¿Qué es y para qué sirve?

El SSL Exporter es una herramienta de monitorización que permite obtener métricas sobre certificados SSL/TLS para ser usadas con Prometheus.

Su objetivo principal es:

Detectar certificados próximos a caducar

Verificar la validez de conexiones HTTPS

Anticipar fallos de seguridad en servicios web



---

2. Instalación y configuración

2.1 Crear usuario del exporter

sudo useradd --no-create-home --shell /usr/sbin/nologin ssl_exporter


---

2.2 Descargar la última versión

cd /tmp

wget https://github.com/ribbybibby/ssl_exporter/releases/latest/download/ssl_exporter-linux-amd64.tar.gz


---

2.3 Extraer archivos

tar -xvf ssl_exporter-linux-amd64.tar.gz


---

2.4 Instalar binario

sudo mv ssl_exporter-linux-amd64/ssl_exporter /usr/local/bin/
sudo chmod +x /usr/local/bin/ssl_exporter


---

2.5 Crear servicio systemd

sudo tee /etc/systemd/system/ssl_exporter.service > /dev/null <<EOF
[Unit]
Description=SSL Exporter for Prometheus
After=network.target

[Service]
User=ssl_exporter
Group=ssl_exporter
ExecStart=/usr/local/bin/ssl_exporter

[Install]
WantedBy=multi-user.target
EOF


---

2.6 Activar servicio

sudo systemctl daemon-reload
sudo systemctl enable --now ssl_exporter


---

2.7 Verificar estado

sudo systemctl status ssl_exporter


---

2.8 Probar endpoint

curl http://localhost:9219/metrics


---

3. Puerto por defecto

Parámetro	Valor

Puerto por defecto	9219
Configurable	Sí



---

4. Métricas relevantes

Métrica	Tipo	Qué mide	Por qué es útil

ssl_cert_not_before	Gauge	Fecha desde la que el certificado SSL es válido	Detecta certificados instalados antes de su validez o errores de configuración
ssl_file_read_errors	Counter	Errores al leer archivos de certificados SSL	Ayuda a detectar problemas de permisos, rutas incorrectas o ficheros dañados
ssl_tls_version_info	Gauge	Versión del protocolo TLS usado en la conexión	Permite identificar versiones inseguras o antiguas de TLS



---

5. Ejemplo de alerta

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


---

6. Limitación del exporter

El SSL Exporter:

Solo analiza certificados SSL/TLS

No monitoriza rendimiento de aplicaciones ni recursos del sistema

No ofrece trazabilidad interna de errores



---

7. ¿Por qué he elegido este exporter?

He elegido el SSL Exporter porque proporciona información más relevante en términos de seguridad que otros exporters como los de Nginx.

Mientras otros se centran en rendimiento o logs, este permite:

Prevenir caducidad de certificados

Detectar configuraciones inseguras TLS

Asegurar conexiones HTTPS válidas




---
