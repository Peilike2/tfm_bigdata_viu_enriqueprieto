# TFM VIU Enrique Prieto Catalán 2022

Este repositorio es la fuente para el Trabajo Final de Mástrer de Big Data/Data Science de Enrique Prieo Catalán en el que se va a ejecutar el stack mediante Docker Compose.
El objetivo es la prueba de concepto de la instalación de la aplicación para Observabilidad del stack en un grupo de servidores.


## Estructura de la información

- /doc [Instrucciones de instalación y uso](./doc/README.md)
- /elasticsearch [configuración de elasticsearch](/elasticsearch/config/elasticsearch.yml)
- /elasticsearch [Procesos encadenados (Pipeline)](/elasticsearch/ingest/logs-pipeline)
- /filebeat/config [Configuración de filebeat](/filebeat/config/filebeat.yml)
- /kibana [Ficheros de configuración de kibana](/kibana/config/kibana.yml)
- /test [Carpeta con logs a analizar](test/sample-json-logs.log)
- .env [Fichero de Variables de entorno](/.env) (desde Git requiere pulsar "view raw" para visualizarlo, por su tamaño)
- README.md Este mismo fichero
- docker-compose.yml [Configuración de la composición de contenedores docker-compose](docker-compose.yml) 
