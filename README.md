# TFM VIU Enrique Prieto Catalán 2022

Este repositorio es la fuente para el Trabajo Final de Mástrer de Big Data/Data Science de Enrique Prieo Catalán en el que se va a ejecutar el stack mediante Docker Compose.
El objetivo es la prueba de concepto de la instalación de la aplicación para Observabilidad del stack en un grupo de servidores.

Se basa en la versión 7.17.5 del stack. Configurada en el fichero [.env](.env).

## Pre-requisitos

- Docker y Docker Compose. Se ha probado con docker versión 19.03.2 y docker-compose 1.24.1.
  - Usuarios de Windows y Mac users tendrán Compose instalado automáticamente con Docker para [Windows](https://docs.docker.com/docker-for-windows/install/)/[Mac](https://docs.docker.com/docker-for-mac/install/).
  - Los usuarios de Linux puede leer las [instrucciones de instalación](https://docs.docker.com/compose/install/#install-compose) o pueden instalar vía `pip`:
  
    ```shell
    pip install docker-compose
    ```

- Un mínimo de 4GB de RAM para contenedores. Los usuarios de Mac y Windows deben configurar su máquina virtual Docker para disponer de ese mínimo.

    ![Docker VM memory settings](doc/img/docker-vm-memory-settings.png)

- Debido a que que por defecto la [memoria virtual](https://www.elastic.co/guide/en/elasticsearch/reference/7.3/vm-max-map-count.html) no es suficiente, los usuarios de Linux deben ejecutar el siguiente comando como `root`:

```
sysctl -w vm.max_map_count=262144
```

## Contenido del repositorio

En este TFM se realizará la prueba de concepto (PoC) que mostrará las capacidades básicas del Stack Elastic para ingesta de logs que permitan la observabilidad de un grupo de servidores.
Los pasos a seguir son los siguientes:

1. [Instalación Stack Elastic](./xx/xx/README.md)
 - En este apartado, se instalará y arrancará un stack elastic con un cluster de un nodo de [Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/7.3/index.html) y una instancia de [Kibana](https://www.elastic.co/guide/en/kibana/7.3/index.html).
2. [Ingesta de logs](./xx/xx/README.md)
 - Usaremos [Filebeat](https://www.elastic.co/guide/en/beats/filebeat/7.3/index.html) para la ingesta de logs en Elasticsearch y los visualizaremos con Kibana [Logs UI](https://www.elastic.co/guide/en/kibana/7.3/xpack-logs.html).
3. [Modelado simple de logs](./doc/paso03/README.md)
 - Modelamos los campos y creamos una pipeline.
4. [Activación de acción](./xxx/xxx/README.md)
 - Daremos órdenes de activación a partir de algunos resultados, comenzando por el envío de un mensaje al operador.
