# **Process-Exporter**

### Identificación
Sara Cordón Goy

## ¿Qué hace este exporter?
Process Exporter se encarga de gestionar los procesos específicos de la aplicación. 
Su objetivo es controlar la calidad general de la app.

## ¿Cómo se instala?
1. Descargamos Process-Exporter --> Vamos al repositorio oficial de GitHub y descargamos el binario:
```bash
wget https://github.com/ncabatoff/process-exporter/releases/download/v0.8.7/process-exporter-0.8.7.linux-amd64.tar.gz
```
3. Descomprimimos y comprobamos los archivos que contiene:
```bash
tar -xvf process-exporter-*.linux-amd64.tar.gz
ls -d process-exporter-*
cd process-exporter-0.8.7.linux-amd64
```
5. Creamos la carpeta de configuración y posteriormente el archivo:
```bash
sudo mkdir -p /etc/process-exporter
sudo nano /etc/process-exporter/config.yml
```
![Configuración](images/processexporter1.png)

Process-Exporter necesita saber qué procesos monitorizar, por lo que creamos el archivo "config.yml" con los siguientes parámetros.



