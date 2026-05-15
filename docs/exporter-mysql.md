#Mysql Exporter
#Identificación
 Mario Garcia Pereiro
#Que hace este exporter
Su funcion traducir la base de datos de SQL a lenuaje prometheus convirtienedo sus consultas en metricas.
 ------metricas tipicas------
 1. mysql_global_status_threads_connected
      Tipo: gauge
      Qué mide: número actual de conexiones activas al servidor MySQL/MariaDB
      Por qué es útil:
      Te da una visión directa de la carga en la base de datos. Si sube demasiado o de forma sostenida, puede indicar saturación o problemas de pooling en la aplicación.
 2. mysql_global_status_slow_queries
      Tipo: counter
      Qué mide: número total acumulado de consultas lentas ejecutadas
      Por qué es útil:
      Permite detectar degradaciones de rendimiento. Si ves que este contador crece rápidamente, hay queries mal optimizadas o problemas de índices.
 3. mysql_global_status_questions
      Tipo: counter
      Qué mide: número total de consultas que ha recibido el servidor
      Por qué es útil:
      Sirve para medir el volumen de tráfico hacia la base de datos. Combinado con otras métricas (como latencia o conexiones) te ayuda a entender picos de carga.
------alerta prpuesta------
      Una alerta que tendría sentido basada en una de esas métricas:
      “Si mysql_global_status_threads_connected supera 80 conexiones durante más de 5 minutos, avisar.”
------

