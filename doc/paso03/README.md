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

Ejemplo de referencia multilínea [^nota1].  





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

# Modelado Simple de Logs con Filebeat

En este punto, el documento que llega a elastic tiene este aspecto:

```json
{"timestamp" : 1569846065739,
"message" : "2019-09-30T12:21:04.702Z leadengage.info omnis 7009 ID418 - Connecting the microchip won't do anything, we need to override the auxiliary PNG protocol!"}
```

Y nos gustaría que en elastic se guardara como:

```json
{
          "process_id" : "7009",
          "message_content" : "Connecting the microchip won't do anything, we need to override the auxiliary PNG protocol!",
          "@timestamp" : "2019-09-30T12:21:04.702Z",
          "process_name" : "omnis",
          "message_id" : "ID418",
          "event_data" : "-",
          "host_name" : "leadengage.info",
          "timestamp" : 1569846065739
}
```

Para realizar esta transformación, recurriremos a las [pipelines](https://www.elastic.co/guide/en/elasticsearch/reference/7.3/pipeline.html) de ingesta de elasticsearch, que se ejecutarán en los [nodos llamados de ingesta](https://www.elastic.co/guide/en/elasticsearch/reference/7.3/ingest.html).

Dado que tenemos un cluster elasticsearch con un sólo nodo, este nodo realizará todos los roles (master, data, ingest, etc.). Más información sobre roles de los nodos en la [documentación](https://www.elastic.co/guide/en/elasticsearch/reference/7.3/modules-node.html).

Las pipelines de ingesta proporcionan a elasticsearch un mecanismo para pre-procesar los documentos antes de almacenarlos. Con una pipeline, podemos parsear, transformar y enriquecer los datos de entrada. Se trata de un conjunto de [procesadores](https://www.elastic.co/guide/en/elasticsearch/reference/7.3/ingest-processors.html) que se aplican de forma secuencial a los documentos de entrada, para generar el documento definitivo que almacenará elasticsearch.

![Ingest pipeline](./img/ingest-pipeline.png)

En este apartado realizaremos:

1. Ingesta de logs estruturados usando una pipeline de ingesta en elasticsearch.
2. Visualización de logs en Kibana Discover.
3. Creación de un Dashboard simple en Kibana.

## Creación de la pipeline de ingesta

En primer lugar, vamos a crear una simple pipeline de ingesta, basada en un procesador de tipo [dissect](https://www.elastic.co/guide/en/elasticsearch/reference/7.3/dissect-processor.html), que nos parseará el campo `message` de entrada generando los diversos campos que queremos a la salida (`process_name`, `process_id`, `host_name`, etc).

Antes de crear esta pipeline, es interesante simular cual sería su comportamiento. Para ello, en Kibana seleccionaremos en el menú de la izquierda `Dev Tools`.

![Dev Tools](./img/devtools-icon.png)

Y pegaremos lo siguiente en la consola:

```json
POST _ingest/pipeline/_simulate
{
  "pipeline": {
    "description": "_description",
    "processors": [
      {
        "dissect": {
          "field": "message",
          "pattern": "%{@timestamp} %{host_name} %{process_name} %{process_id} %{message_id} %{event_data} %{message_content}"
        }
      },
      {
        "remove": {
          "field": "message"
        }
      }
    ]
  },
  "docs": [
    {
      "_source": {
        "timestamp" : 1569846065739,
        "message" : "2019-09-30T12:21:04.702Z leadengage.info omnis 7009 ID418 - Connecting the microchip won't do anything, we need to override the auxiliary PNG protocol!"
      }
    }
  ]
}
```

Al ejecutar esta petición, podremos comprobar si el JSON resultante es el esperado.

![Simulate Ingest pipeline](./img/ingest-pipeline-simulate.png)

Esta petición [simula](https://www.elastic.co/guide/en/elasticsearch/reference/7.3/simulate-pipeline-api.html) una pipeline, usando el endpoint del API REST de elasticsearch `_ingest/pipeline/_simulate`. En el contenido del cuerpo, tenemos un JSON con los procesadores de la pipeline:

- [**dissect**](https://www.elastic.co/guide/en/elasticsearch/reference/7.3/dissect-processor.html): Se encarga de separar el texto que viene en el campo message a partir de los espacios en blanco, y crea distintos campos (timestamp, host_name, process_name, etc.) con los valores que extrae del campo message de entrada.
- [**remove**](https://www.elastic.co/guide/en/elasticsearch/reference/7.3/remove-processor.html): eliminará el campo `message` ya que, una vez modelado, no nos interesa guardara esta información redundante.

Una vez comprobamos que la pipeline de ingesta funciona según deseamos, la daremos de alta en elasticsearch para poder usarla. Para ello, en la misma consola de Dev Tools, ejecutaremos:

```json
PUT _ingest/pipeline/logs-pipeline
{
  "description": "Pipeline para ingesta de logs",
  "processors": [
    {
      "dissect": {
        "field": "message",
        "pattern": "%{@timestamp} %{host_name} %{process_name} %{process_id} %{message_id} %{event_data} %{message_content}"
      }
    },
    {
      "remove": {
        "field": "message"
      }
    }
  ]
}
```

Creando la pipeline de ingesta **logs-pipeline**, que usaremos en el próximo apartado.

![Ingest pipeline](./img/ingest-pipeline-put.png)

## Configuracion de Filebeat

Ahora tenemos que indicar a elasticsearch que los documentos que vayan a ser almacenados en los índices creados por filebeat deben pasar primero esta pipeline que los va a transformar. Para ello, editaremos el fichero de configuración de filebeat. [filebeat/config/filebeat.yml](../../filebeat/config/filebeat.yml), y en la sección `output.elasticsearch` descomentaremos la línea `pipeline: logs-pipeline`.

```yaml
output.elasticsearch:
  hosts: ["elasticsearch:9200"]
  username: '${ES_USERNAME:elastic}'
  password: '${ES_PASSWORD:changeme}'
  pipeline: logs-pipeline
```

## Ingesta de logs estructurados

Ya estamos listos para arrancar de nuevo nuestro generador de logs:

```shell
docker run -it --name flog_json --rm immavalls/flog:1.0 -l -f rfc5424 -y json -d 1 -s 1 > ./test/sample-json-logs.log
```

Y arrancar de nuevo `filebeat`.

```shell
docker-compose up -d
```

Podemos comprobar que no haya errores en la ejecución de filebeat, antes de pasar al siguiente apartado:

```shell
docker logs -f filebeat
```

## Visualizar logs en Discover

Volvemos a Kibana.

Para visualizar los logs debemos primero crear un [Index Pattern](https://www.elastic.co/guide/en/kibana/7.3/tutorial-define-index.html). Los index patterns nos permiten acceder desde Kibana a los índices en elasticsearch, y, por lo tanto, a los documentos que tenemos almacenados en estos índices.

Si no le indicamos lo contrario en la configuración de filebeat para envío a elasticsearch, los índices que se crearán son con el nombre `filebeat-*`.

![Index Patterns](./img/index-pattern.png)

Por lo tanto, en la sección de Management de Kibana, seleccionamos `Index Patterns` en el grupo `Kibana`.

![Index Patterns](./img/kibana-index-patterns-management.png)

Pulsamos el botón azul `Create Index Pattern` y damos de alta un patrón `filebeat-*`.

![Index Patterns](./img/index-pattern-create-1.png)

Hacemos clic en `Next step`y seleccionaremos el campo a usar para mostrar la serie temporal de datos en Discover. En
este caso, escogemos `@timestamp`.

![Index Patterns](./img/index-pattern-create-2.png)

Y pulsamos `Create Index Pattern`.

Seleccionamos en el menú de la izquierda en Kibana `Discover`.

Hacemos clic en `New` en el menú superior, para limpiar cualquier filtro que tuviéramos en la búsqueda.

Y en el selector escogemos el index pattern que acabamos de crear, `filebeat-*`.

![Discover Filebeat](./img/discover-filebeat.png)

Es posible usar la barra de búsqueda para filtrar nuestros datos. Por ejemplo, seleccionar `event_data:` y nos proporcionará sugerencias para filtrar la búsqueda. En el ejemplo, podemos filtrar por `event_data: "ID222" or event_data: "ID638"`

![Discover KQL](./img/discover-kql.png)

![Discover KQL](./img/discover-kql-2.png)

El lenguage usado para filtrar las búsquedas es [Kibana Query Language (KQL)](https://www.elastic.co/guide/en/kibana/7.3/kuery-query.html).

**Eliminamos** el filtro de las búsquedas para recuperar todos los datos.

Finalmente, vamos a crear una tabla para utilizarla en la construcción del Dashboard en el siguiente apartado. Queremos una vista que nos muestre `host_name` y `process_name`. En la lista de campos disponibles, localizamos esos campos y pulsamos el botón azul `add` para ambos.

![Search Add](./img/search-add.png)

El resultado será una tabla como la siguiente.

![Search Table](./img/search-table.png)

Pulsaremos el botón `Save` en la barra superior y guardaremos la búsqueda con el nombre `[Filebeat] Host/Process`

![Save Search](./img/save-search.png)

## Creación de Dashboard en Kibana

Vamos a crear un simple dashboard en Kibana para analizar nuestros datos. Para ello, crearemos primero diversas visualizaciones.

En Kibana, selecciona el menú `Visualize`.

![Visualize](./img/visualize-icon.png)

Clic en el botón `Create new visualization`, y seleccionar el tipo `Pie`.

![Visualize](./img/visualize-select-pie.png)

En el siguiente paso, `Choose a source`, seleccionar la búsqueda guardada como `[Filebeat] Host/Process`.

![Source](./img/visualize-select-source-pie.png)

En la visualización, pulsar el enlace `+Add` bajo Buckets, y seleccionar `Split slices`.

![Source](./img/visualize-pie-add-bucket.png)

Seleccionar una `Terms` aggregation, sobre el campo `host_name` y pulsar la flecha azul para aplicar los cambios.

![Aggs host_name](./img/visualize-pie-host_name-terms.png)

Añadir una segunda agregación en buckets, de tipo `Terms`, sobre el campo `process_name`. Y aplicar el cambio.

![Aggs process_name](./img/visualize-pie-process_name-terms.png)

Aplicar y comprobar el resultado.

![Pie](./img/visualize-pie.png)

Pulsar el enlace `Save` en la barra superior y guardar la visualización con el nombre `[Filebeat] Process/Host pie`.

Pulsar en el enlace `Visualize` en la barra superior de navegación y crear una segunda visualización, de tipo `Tag Cloud`.  

![Tag cloud](./img/visualize-tag-cloud-select.png)

Seleccionar la misma fuente de datos, la búsqueda guardada como `[Filebeat] Host/Process`. Y bajo `Buckets` en la visualización, añadir `Tags`.

![Tag cloud](./img/visualize-tag-cloud-bucket-add.png)

Y añadir una agregación de nuevo de tipo `Terms`, sobre el campo `process_name`, modificando el tamaño por defecto de 5 a 20.

![Tag cloud](./img/visualize-tag-cloud-process_name-aggs.png)

Cambiar de la pestaña `Data` a `Options` y modificar `Orientations` a `multiple`. 

![Tag cloud](./img/visualize-tag-cloud-process_name-options.png)

Aplicar los cambios.

![Tag cloud](./img/visualize-tag-cloud.png)

Guardar esta visualización con el nombre `[Filebeat] Process/Host cloud tag`.

Pasaremos finalmente a crear un dashboard que incluya estas visualizaciones.

Seleccionamos en el menú de la izquierda `Dashboard`. Y creamos un nuevo dashboard pulsando el botón `Create new dashboard`. 

Si estamos visualizando otro dashboard, volveremos antes al menú inicial haciendo click en el enlace `Dashboard` en el menú superior.

![Dashboard Menu](./img/menu-dashboard.png)

Ya podemos pulsar `Create new dashboard`.

![Dashboard Create](./img/create-dashboard.png)

Clicar en el enlace `Add` en el menú superior.

![Dashboard Add](./img/dashboard-add.png)

Y añadir las dos visualizaciones guardadas `[Filebeat] Process/Host cloud tag`, `[Filebeat] Process/Host pie`, y la búsqueda `[Filebeat] Host/Process`.

Guardar con el nombre `[Filebeat] Dashboard` pulsando la opción `Save`.

![Dashboard](./img/dashboard-filebeat.png)

Adicionalmente, podemos comprobar que al pulsar por ejemplo en un host_name concreto sobre el `pie chart`, se aplica un filtro a todo el dashboard. En el ejemplo, estamos filtrando por `host_name: districtholistic.net`.

![Dashboard](./img/dashboard-filebeat-filtered.png)

## Finalizamos

4. ### Siguientes pasos



[^nota1]: Every new line should be prefixed with 2 spaces.  
  This allows you to have a footnote with multiple lines.
[Subir](#top)
