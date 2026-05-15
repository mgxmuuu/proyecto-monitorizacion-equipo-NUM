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

###  Decisiones que hemos tomado

Escogimos esta serie de exporters debido a las características que queríamos monitorizar y teniendo en cuenta la arquitectura de la app web y de las limitaciones que posee:

- **Blackbox Exporter** fue elegido para comprobar la disponibilidad y latencia de la aplicación desde el exterior mediante peticiones HTTP y pings.
- **Node Exporter** se utiliza para monitorizar el estado del sistema operativo y del hardware, permitiendo detectar saturación de CPU, RAM o disco. Exporter básico y clave.
- **MySQLd Exporter** fue incluido para controlar el estado de la base de datos, especialmente conexiones, consultas y carga del servicio MySQL. Uno de los exporters mas importantes.
- **Process Exporter** permite supervisar procesos concretos de la aplicación, algo que otros exporters más generales no hacen de forma específica.
- **SSL Exporter** se encarga de comprobar el estado de los certificados HTTPS y la seguridad de la conexión web.La otra opción valorada era utilizar **Nginx Exporter**, pero en comparación con **SSL Exporter**,
  era inferior en cuanto a rendimiento y utilización.

En cada exporter utilizamos los puertos por defecto de cada exporter para facilitar la configuración y evitar problemas de compatibilidad con Prometheus, pese a que podríamos haberlos cambiado:

- Node Exporter → puerto **9100**
- MySQLd Exporter → puerto **9104**
- Blackbox Exporter → puerto **9115**
- Process Exporter → puerto **9256**
- SSL Exporter → puerto **9219**

También se valoró utilizar otras herramientas como cAdvisor, pero finalmente fue descartada:
- **cAdvisor** no se implementó porque el proyecto no utiliza contenedores Docker, y además no teníamos los conocimientos adecuados para utilizarlo.


###  Limitaciones conocidas

- Poder haber implementado más métricas en forma de Dashboard en Grafana y Prometheus, para poder tener más controlados los errores.
- Algunas métricas del Process Exporter pueden tardar en actualizarse dependiendo de la carga del sistema.
- SSL Exporter únicamente monitoriza certificados HTTPS básicos y no configuraciones avanzadas de seguridad.
- No se ha implementado alta disponibilidad(HA) de Prometheus.


### Documentación del proyecto

- [SSL Exporter](https://github.com/ribbybibby/ssl_exporter)
- [BlackBox Exporter](https://github.com/prometheus/blackbox_exporter)
- [MySQL Exporter](https://github.com/prometheus/mysqld_exporter)
- [Process Exporter](https://github.com/ncabatoff/process-exporter)
- [Node Exporter](https://github.com/prometheus/node_exporter)


### Datos adicionales relevantes

- El README.md en una inmensa mayoría fue creado y desarollado por Sara, y Alejandro editó algunos apartados relevantes. Los demás miembros del grupo: Héctor, Keyla, Magali y Mario, también trabajaron proporcionando         los datos de sus respectivos exporters, además de su documento correspondiente. Además, varios apartados del mismo readme fueron revisaron mediante inteligencia artificial para verificar su correcta distribución y         fiabilidad.
- Toda la información y documentación que hemos utilizado para realizar tanto el documento ya entregado en Classroom como de este repositorio en Github, ha sido recopilada de la tarea de expertos de Exporters con sus        respectivos documentos, y de páginas web especializadas en cada Exporter como de los repositorios oficiales en el propio Github(https://github.com/ribbybibby/ssl_exporter - ejemplo).


## Conclusión del proyecto

- Este proyecto nos ha permitido actuar como un equipo de SRE, donde nuestro objetivo no es solo desplegar herramientas, sino mejorar la observabilidad de una aplicación real.
- El valor principal del proyecto no ha sido la instalación de herramientas, sino la capacidad de decidir qué medir, por qué medirlo y cómo interpretar los resultados.
- Como grupo, con el trabajo que hemos desempeñado en el proyecto podemos afirmar que hemos solucionado el problema con el que nos enfrentamos y que además hemos aprendido no solo como implementar monitorización
  en una app sino también como trabajar en un ambiente empresarial.
