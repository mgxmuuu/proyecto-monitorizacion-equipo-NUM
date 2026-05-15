# Prometheus - Sistema de Monitorización

---

# ¿Qué es Prometheus?

Prometheus es una herramienta de monitorización y recopilación de métricas de código abierto.
Permite supervisar servidores, servicios, aplicaciones y contenedores en tiempo real mediante 
métricas obtenidas desde distintos exporters.

Prometheus almacena toda la información en series temporales y posteriormente puede visualizarse mediante herramientas como Grafana.

---

# ¿Para qué sirve?

Prometheus se utiliza para:

- Monitorizar el estado de servidores Linux y Windows.
- Supervisar el consumo de CPU, RAM, disco y red.
- Detectar fallos o sobrecarga del sistema.
- Obtener métricas de servicios y aplicaciones.
- Generar alertas automáticas.
- Visualizar estadísticas en tiempo real.

---

# Arquitectura del proyecto

En nuestro proyecto utilizaremos distintos exporters para monitorizar el estado general de la aplicación y del servidor.

Nuestro objetivo principal será supervisar:

- Latencia
- Errores
- Tráfico
- Saturación del sistema

Para ello implementaremos los siguientes exporters:

| Exporter | Función principal |
|---|---|
| Blackbox Exporter | Monitorización externa mediante peticiones HTTP, HTTPS y ping |
| Node Exporter | Obtención de métricas del hardware y sistema operativo |
| MySQLd Exporter | Supervisión de la base de datos MySQL |
| Process Exporter | Monitorización de procesos específicos de la aplicación |
| SSL Exporter | Verificación del estado y seguridad de certificados SSL |

---

# Acceso desde nuestra máquina al servidor


Desde nuestra máquina cliente accedemos mediante navegador web utilizando la IP del servidor y el puerto correspondiente.

## Acceso a Prometheus

```bash
http://IP_SERVIDOR:9090
