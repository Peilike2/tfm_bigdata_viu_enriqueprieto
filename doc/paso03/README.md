<a name="top"></a>
# Título de nuestro documento
 
## Índice de contenidos
* [Contenido 1](#item1)
* [Contenido 2](#item2)
* [Modelado Simple de Logs](#item3)
* [Contenido 4](#item4)
 
Lorem ipsum dolor
 
<a name="item1"></a>
1. ### Contenido 1
 
Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.
 
[Subir](#top)
 
<a name="item2"></a>
2. *### Contenido 2
 
Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.
 
Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat.
 
[Subir](#top)
 
<a name="item3"></a>
3. ### Modelado Simple de Logs

Hemos ingestado en elastic nuestros logs sin modelar, sin estructura. Es decir, dado un log con el formato:

```json
{"timestamp":1569939745276,"message":"27 Dec 2020 03:09:29 () [k6A:2394036:srm2:prepareToGet:-1093710432:-1093710431 k6A:2394036:srm2:prepareToGet SRM-grid002] Pinning failed for /pnfs/ft.uam.es/data/atlas/atlasdatadisk/rucio/mc16_13TeV/ce/13/EVNT.23114463._000856.pool.root.1 (File is unavailable.)"}
```

o:

```
27 Dec 2020 03:09:29 () [k6A:2394036:srm2:prepareToGet:-1093710432:-1093710431 k6A:2394036:srm2:prepareToGet SRM-grid002] Pinning failed for /pnfs/ft.uam.es/data/atlas/atlasdatadisk/rucio/mc16_13TeV/ce/13/EVNT.23114463._000856.pool.root.1 (File is unavailable.)
```

El documento que hemos acabado guardando en Elasticsearch tiene un campo `timestamp` con la fecha de ingesta, y un segundo campo `message` con el mensaje completo del log.

Ahora queremos separar el contenido de este campo `message`, de forma que podamos explotar obtener el `nombre del proceso`, el `nombre del host`, etc. Es decir, darle estructura a los datos que nos llegan.
El mensaje de ejemplo antrior debería transformarse en el siguiente:
srm://grid002.ft.uam.es:8443/srm/managerv2?SFN=/pnfs/ft.uam.es/data/atlas/atlasdatadisk/rucio/mc16_13TeV/ce/13/EVNT.23114463._000856.pool.root.1

Es decir, queremos aplicarle las siguientes operaciones:
1. - Seleccionar las líneas que contengan "unavailable"
2. - Seleccionar las líneas que contengan "root" en la dirección url del mensaje
3. - Extraer diche url de cada mensaje, aladirle el prefijo xxx y que dicha concatenación esa el valor de un nuevo campo
4. - Eliminar filas duplicadas

En lenguaje Bash, se podría expresar así, ejecutando cada línea de forma independiente:
```
cat 01_srm-grid002Domain.log | grep unavailable > 02a_perdidos_conysinpool.txt
cat 02a_perdidos_conysinpool.txt | grep pool.root > 02b_perdidos.txt
cut -d ' ' -f 12 02b_perdidos.txt > 03_nombres_.txt
nombres=$(cat 03_nombres.txt)
for f in $nombres; do echo "srm://grid002.ft.uam.es:8443/srm/managerv2?SFN=$f" >> "04_preparados.txt"; done
cat 04_preparados.txt | sort | uniq > "05_listos.txt"
```
<!-- Esto es un comentario de prueba de Enrique -->

Ejemplo de referencia multilínea [^ReferenciaMultilinea].  





Así podremos agrupar valores similares, visualizarlos, y explotar toda la potencia de nuestros logs. Nos permitirá contestar preguntas como ¿Cuáles son los nombre de host más habituales (`customere-services.org`, `nationalviral.io`?)? ¿Y los ids de proceso?

Para ello, necesitaremos conocer la **estructura** de nuestros logs, e indicársela a Elasticsearch.

En función de qué aventura has elegido, selecciona el siguiente paso:

Creación de Pipeline sobre ingestión de datos con **[Filebeat](./filebeat.md)**
Creación de Pipeline sobre ingestión de datos con **[Filebeat](./filebeat.md)**
Creación de Pipeline sobre ingestión de datos con **[Filebeat](./filebeat.md)**
Creación de Pipeline sobre ingestión de datos con **[Filebeat](./filebeat.md)**
Creación de Pipeline sobre ingestión de datos con **[Filebeat](./filebeat.md)**
Creación de Pipeline sobre ingestión de datos con **[Filebeat](./filebeat.md)**
Creación de Pipeline sobre ingestión de datos con **[Filebeat](./filebeat.md)**
Creación de Pipeline sobre ingestión de datos con **[Filebeat](./filebeat.md)**
Creación de Pipeline sobre ingestión de datos con **[Filebeat](./filebeat.md)**
Creación de Pipeline sobre ingestión de datos con **[Filebeat](./filebeat.md)**
Creación de Pipeline sobre ingestión de datos con **[Filebeat](./filebeat.md)**Creación de Pipeline sobre ingestión de datos con **[Filebeat](./filebeat.md)**
Creación de Pipeline sobre ingestión de datos con **[Filebeat](./filebeat.md)**
Creación de Pipeline sobre ingestión de datos con **[Filebeat](./filebeat.md)**
Creación de Pipeline sobre ingestión de datos con **[Filebeat](./filebeat.md)**Creación de Pipeline sobre ingestión de datos con **[Filebeat](./filebeat.md)**Creación de Pipeline sobre ingestión de datos con **[Filebeat](./filebeat.md)**Creación de Pipeline sobre ingestión de datos con **[Filebeat](./filebeat.md)**Creación de Pipeline sobre ingestión de datos con **[Filebeat](./filebeat.md)**Creación de Pipeline sobre ingestión de datos con **[Filebeat](./filebeat.md)**
Creación de Pipeline sobre ingestión de datos con **[Filebeat](./filebeat.md)**Creación de Pipeline sobre ingestión de datos con **[Filebeat](./filebeat.md)**
Creación de Pipeline sobre ingestión de datos con **[Filebeat](./filebeat.md)**
Creación de Pipeline sobre ingestión de datos con **[Filebeat](./filebeat.md)**Creación de Pipeline sobre ingestión de datos con **[Filebeat](./filebeat.md)**
Creación de Pipeline sobre ingestión de datos con **[Filebeat](./filebeat.md)**
Creación de Pipeline sobre ingestión de datos con **[Filebeat](./filebeat.md)**
Creación de Pipeline sobre ingestión de datos con **[Filebeat](./filebeat.md)**
Creación de Pipeline sobre ingestión de datos con **[Filebeat](./filebeat.md)**
Creación de Pipeline sobre ingestión de datos con **[Filebeat](./filebeat.md)**
Creación de Pipeline sobre ingestión de datos con **[Filebeat](./filebeat.md)**
Creación de Pipeline sobre ingestión de datos con **[Filebeat](./filebeat.md)**
Creación de Pipeline sobre ingestión de datos con **[Filebeat](./filebeat.md)**
Creación de Pipeline sobre ingestión de datos con **[Filebeat](./filebeat.md)**Creación de Pipeline sobre ingestión de datos con **[Filebeat](./filebeat.md)**
 
[Subir](#top)
 
<a name="item4"></a>
### Contenido 4
 
Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.
 
Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur.
 
Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.
 
[Subir](#top)





[^ReferenciaMultilinea]: Every new line should be prefixed with 2 spaces.  
  This allows you to have a footnote with multiple lines.
[Subir](#top)
