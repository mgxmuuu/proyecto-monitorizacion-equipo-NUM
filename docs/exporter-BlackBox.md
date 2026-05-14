# Blackbox Exporter — Ficha Técnica


---

## 1. ¿Qué es y para qué sirve?

Es una herramienta de monitorización que permite analizar los servidores web. Su principal función es analizar el **estado de la web** (levantada o caída) y su **rendimiento**.

---

## 2. Instalación y configuración

### Paso 1 — Descargar Blackbox Exporter

Ir al repositorio oficial de GitHub y descargar el archivo más reciente.

### Paso 2 — Copiar y descomprimir

Copiar el archivo al directorio temporal y descomprimirlo con `tar`.

### Paso 3 — Crear el archivo de servicio

```bash
sudo nano /etc/systemd/system/blackbox_exporter.service
```

Contenido del archivo:

```ini
[Unit]
Description=Prometheus Blackbox Exporter (Modo Prueba)
Wants=network-online.target
After=network-online.target

[Service]
User=hectorci
Type=simple
ExecStart=/tmp/blackbox_exporter-0.28.0.linux-amd64/blackbox_exporter \
  --config.file=/tmp/blackbox_exporter-0.28.0.linux-amd64/blackbox.yml \
  --web.listen-address=":9115"

[Install]
WantedBy=multi-user.target
```

### Paso 4 — Arrancar el servicio

```bash
sudo systemctl daemon-reload
sudo systemctl start blackbox_exporter
sudo systemctl status blackbox_exporter
```

### Paso 5 — Conectar con Prometheus

Editar el archivo de configuración de Prometheus:

```bash
sudo nano /etc/prometheus/prometheus.yml
```

Añadir el siguiente bloque (respetar la indentación YAML estrictamente):

```yaml
# BlackBox Exporter
- job_name: 'blackbox_prueba'
  metrics_path: /probe
  params:
    module: [http_2xx]  # Usa el módulo web por defecto
  static_configs:
    - targets:
        - https://www.google.com
        - https://www.ubuntu.com
  relabel_configs:
    - source_labels: [__address__]
      target_label: __param_target
    - source_labels: [__param_target]
      target_label: instance
    - target_label: __address__
      replacement: 127.0.0.1:9115  # Apunta al Blackbox que acabamos de levantar
```

### Paso 6 — Verificar en Prometheus

Reiniciar Prometheus e ir a la web → **Status > Targets**. Debe aparecer una nueva sección llamada `blackbox_prueba` con las URLs de Google y Ubuntu en estado **UP**.

### Paso 7 — Añadir alertas

Crear un archivo exclusivo para las alertas:

```yaml
# /etc/prometheus/blackbox_rules.yml
groups:
- name: Alertas_Blackbox
  rules:

    # Alerta 1: Sitio Caído
    - alert: SitioWebCaido
      expr: probe_success == 0
      for: 1m
      labels:
        severity: critical
      annotations:
        summary: "El sitio {{ $labels.instance }} está caído"
        description: "Blackbox no ha podido conectar a {{ $labels.instance }} durante más de 1 minuto."

    # Alerta 2: Respuesta Lenta (más de 5 segundos)
    - alert: RespuestaLenta
      expr: probe_duration_seconds > 5
      for: 2m
      labels:
        severity: warning
      annotations:
        summary: "Respuesta muy lenta en {{ $labels.instance }}"
        description: "La URL {{ $labels.instance }} está tardando más de 5 segundos en cargar."

    # Alerta 3: Certificado SSL caduca en menos de 7 días
    - alert: CertificadoSSLAExpirar
      expr: probe_ssl_earliest_cert_expiry - time() < 86400 * 7
      for: 10m
      labels:
        severity: warning
      annotations:
        summary: "Certificado SSL por caducar en {{ $labels.instance }}"
        description: "El certificado de {{ $labels.instance }} caduca en menos de 7 días. ¡Renuévalo!"
```

Enlazar el archivo en `prometheus.yml`:

```yaml
rule_files:
  - "blackbox_rules.yml"
```

### Paso 8 — Comprobar y reiniciar

```bash
promtool check config /etc/prometheus/prometheus.yml
sudo systemctl restart prometheus
```

### Paso 9 — Verificar alertas

Ir a la web de Prometheus → apartado **Alerts** y confirmar que las alertas aparecen correctamente.

---

## 3. Puerto por defecto

| Parámetro | Valor |
|-----------|-------|
| Puerto por defecto | **9115** |
| ¿Es configurable? | ✅ Sí |

---

## 4. Métricas relevantes

### `probe_success`

| Campo | Detalle |
|-------|---------|
| **Tipo** | Gauge |
| **Qué mide** | Devuelve `1` si la prueba fue exitosa o `0` si falló |
| **Por qué es útil** | Indica si el servicio está disponible desde el exterior o si se ha caído |

---

### `probe_duration_seconds`

| Campo | Detalle |
|-------|---------|
| **Tipo** | Gauge |
| **Qué mide** | El tiempo total (en segundos) que tardó en completarse la petición de red |
| **Por qué es útil** | Una web que tarda 10 segundos está prácticamente "caída" para el usuario; sirve para detectar cuellos de botella |

---

### `probe_ssl_earliest_cert_expiry`

| Campo | Detalle |
|-------|---------|
| **Tipo** | Gauge |
| **Qué mide** | La fecha y hora exacta en la que caducará el certificado SSL/TLS |
| **Por qué es útil** | Evita olvidar renovar un certificado, lo que provocaría que los navegadores bloqueen el acceso marcando la web como "No segura" |

---

## 5. Ejemplo de alerta

> **"Si la métrica de tiempo de respuesta total (`probe_duration_seconds`) supera los 5 segundos de forma continua durante 3 minutos, enviar una alerta de _Degradación de Servicio Web_."**

---

## 6. Limitación del exporter

El Blackbox Exporter únicamente **proporciona el resultado final**: sabe si algo falla, pero **no identifica dónde se encuentra el error**. No ofrece trazabilidad interna del fallo.
