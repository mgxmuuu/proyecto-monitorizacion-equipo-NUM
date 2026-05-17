SSL Exporter — Ficha Técnica

Autor: Alejandro Dutor


---

1. ¿Qué es y para qué sirve?

El SSL Exporter es una herramienta de monitorización diseñada para comprobar el estado y la validez de certificados SSL/TLS. Se utiliza junto con Prometheus para recopilar métricas relacionadas con certificados digitales, como la fecha de expiración, errores de validación o problemas de conexión segura.

Su principal utilidad es detectar con antelación certificados próximos a caducar y evitar fallos de seguridad o interrupciones en servicios web protegidos mediante HTTPS.


---

2. Instalación y configuración

Paso 1 — Crear usuario del exporter

sudo useradd --no-create-home --shell /usr/sbin/nologin ssl_exporter


---

Paso 2 — Descargar la última versión

cd /tmp

wget https://github.com/ribbybibby/ssl_exporter/releases/latest/download/ssl_exporter-linux-amd64.tar.gz


---

Paso 3 — Extraer archivos

tar -xvf ssl_exporter-linux-amd64.tar.gz


---

Paso 4 — Mover el binario

sudo mv ssl_exporter-linux-amd64/ssl_exporter /usr/local/bin/

sudo chmod +x /usr/local/bin/ssl_exporter


---

Paso 5 — Crear servicio systemd

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

Paso 6 — Recargar systemd y arrancar servicio

sudo systemctl daemon-reload

sudo systemctl enable --now ssl_exporter


---

Paso 7 — Verificar estado

sudo systemctl status ssl_exporter


---

Paso 8 — Probar endpoint

curl http://localhost:9219/metrics


---

3. Puerto por defecto

Parámetro	Valor

Puerto por defecto	9219
¿Es configurable?	Sí



---

4. Métricas relevantes

ssl_cert_not_before

Campo	Detalle

Tipo	Gauge
Qué mide	Muestra desde qué fecha y hora el certificado SSL/TLS comienza a ser válido.
Por qué es útil	Ayuda a detectar errores de configuración o certificados instalados antes de tiempo.



---

ssl_file_read_errors

Campo	Detalle

Tipo	Counter
Qué mide	Cuenta el número de errores al leer archivos de certificados SSL desde el sistema.
Por qué es útil	Permite detectar problemas de permisos, rutas incorrectas o archivos dañados.



---

ssl_tls_version_info

Campo	Detalle

Tipo	Gauge
Qué mide	Indica la versión del protocolo TLS utilizada por la conexión segura del servidor.
Por qué es útil	Ayuda a identificar versiones antiguas o inseguras de TLS que puedan representar vulnerabilidades.



---

5. Ejemplo de alerta

> “Si el certificado SSL de un servicio expira en menos de 7 días, enviar una alerta crítica al administrador para renovar el certificado antes de que el navegador marque la web como insegura.”



Ejemplo en Prometheus:

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

El SSL Exporter se centra exclusivamente en certificados SSL/TLS. No analiza el rendimiento interno del servidor web ni detecta problemas de aplicación, bases de datos o consumo de recursos del sistema.


---

7. ¿Por qué he escogido este exporter?

He escogido el SSL Exporter porque, en comparación con otros exporters similares como el de Nginx, recopila información más importante relacionada con la seguridad del sistema. Además de monitorizar certificados SSL/TLS, permite detectar problemas antes de que afecten a los usuarios.

Gracias a esta monitorización preventiva, es posible evitar certificados expirados, conexiones inseguras y fallos en servicios HTTPS, mejorando así tanto la seguridad como la disponibilidad del sistema.
