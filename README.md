# PROYECTO PROMETHEUS

<br>

## INFORME DE TRABAJO MONITORIZACIÓN EN UN ENTORNO DE PRODUCCIÓN
<br>

### ¿Qué exporter vais a incluir en vuestro proyecto? Nombra y explica brevemente cada uno de ellos. ¿Qué aportará en nuestro proyecto cada uno?

<br>

Lo que nos interesa monitorizar: latencia, errores, tráfico y saturación.

Vamos a instalar cuatro/cinco Exporters: Blackbox, Node Exporter, SSL Exporter, Process-Exporter y MySQLd Exporter.

<br>

Necesitamos un sistema de mensajería para las alertas, para lo que usaremos Telegram o Discord.

- Blackbox Exporter → Probador externo que lanza peticiones como pings a la web desde fuera de la MV. La función principal de Blackbox es gestionar y controlar la calidad de la app web. Su tarea será cubrir principalmente tanto la Latencia como los posibles Errores de la app.

- Node Exporter → Ver y medir métricas del hardware y del sistema operativo. El objetivo será alertar si los recursos del sistema sobrepasan sus prestaciones originales, gestionando la Saturación de la app.

- MySQLd Exporter → Se encarga de todo lo relacionado con la base de datos de la app. Su función será controlar las peticiones y conexiones, cubriendo tanto el tráfico como la Saturación (ÚNICAMENTE DE LA BASE DE DATOS)

- Process Exporter → Process Exporter se encarga de gestionar los procesos específicos de la aplicación. Su objetivo en la app será controlar la calidad general de la app.

- SSL Exporter → SSL Exporter se encarga de verificar la seguridad de la appweb, extrayendo información que ayuda a proteger la conexión. 

<br>

### Cada miembro del grupo debe tener una responsabilidad.
- Un exporter por técnico.
- Un integrador para la máquina final + accesible la plataforma de desarrollo
Escribe el responsable de cada elemento en vuestro proyecto.

*Alejandro Dutor → SSL Exporter*

*Héctor Cascón →  BlackBox Exporter*

*Mario García →  MySQLd Exporter*

*Sara Cordón →  Process Exporter*

*Keyla Galarraga → Node Exporter*

*Magalí Pérez → Prometheus*

<br>


### Documentación del proyecto

- [SSL Exporter](https://github.com/ribbybibby/ssl_exporter)
- [BlackBox Exporter](https://github.com/prometheus/blackbox_exporter)
- [MySQL Exporter](https://github.com/prometheus/mysqld_exporter)
- [Process Exporter](https://github.com/ncabatoff/process-exporter)
- [Node Exporter](https://github.com/prometheus/node_exporter)

