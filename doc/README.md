 <a name="top"></a>
<!-- BORRAR: Esto es un comentario de prueba de Enrique -->
# TFM VIU 2022

***Tutora: [_Immaculada Valls Bernaus_]( https://github.com/immavalls)***

***Masterando: [_Enrique Prieto Catalán_]( https://github.com/Peilike2)***

---

Trabajo Final de Máster de Big Data/Data Science de Enrique Prieto Catalán en la Universidad Internacional de Valencia (VIU) titulado:

***"IMPLEMENTACIÓN DE SISTEMATIZACIÓN DE LA OBSERVABILIDAD EN EL NODO UAM-LCG2 DE LA MALLA DE CÁLCULO MUNDIAL DEDICADA A LOS EXPERIMENTOS DEL DETECTOR ATLAS DEL GRAN COLISIONADOR DE HADRONES LHC DE LA ORGANIZACIÓN EUROPEA PARA LA INVESTIGACIÓN NUCLEAR (CERN) EN GINEBRA, SUIZA."***

En este TFM se realizará la prueba de concepto (PoC) que mostrará las capacidades básicas del Stack Elastic[^nota1] para que la ingesta de logs a través del servicio filebeat permita la observabilidad de un grupo de servidores, a través del servicio kibana[^nota2], todos ellos desplegado de contenedores a través de Docker-Compose.

OBJETIVO GENERAL:
Mejora de condiciones de servicio del centro de cálculo de datos mediante la temprana detección, procesado y comunicación de los errores de procesamiento que tuvieran lugar.

OBJETIVOS ESPECÍFICOS:
- Se analizará la estructura de los logs recibidos a través de un fichero, ingestados por una cadena de procesos (pipeline de filebeat) que se enviarán a elasticsearch.
- Mediante reglas de kibana, se detectarán entre las líneas de mensaje, las que correspondan a mensajes de error y se agregarán por servidor de origen, obteniendo la ubicación del fichero perdido y el tipo de fallo indicado por el log.
- Se programará, como acción activada por alerta de kibana, el envío del mensaje de aviso pertinente al operador, mediante un canal de mensajería Slack.

---
 
<a name="indice"></a>
[Ver contenido del repositorio, acceso y explicación de los ficheros de configuración](../README.md)

## Índice de contenidos
1. [ REQUISITOS Y ASUNCIONES](#item1)
2. [ ENTORNO DE DESARROLLO ELK EN GOOGLE CLOUD PLATFORM](#item2)
3. [ CONEXIÓN POR SSH. INSTALACIÓN DE DNF Y DOCKER](#item3)
4. [ INSTALACIÓN DE DOCKER-COMPOSE](#item4)
5. [ INSTALACIÓN DE GIT](#item5)
6. [ INSTALACIÓN DEL STACK ELASTIC](#item6)
7. [ VISUALIZACIÓN CON KIBANA. CREACIÓN DEL INDEX PATTERN](#item7)
8. [ SIMULACIÓN DE PIPELINE DE PROCESADO DE LOGS](#item8)
   - Modelado de los campos para la pipeline, y comprobación de salidas en entorno de prueba.
9. [ ALTA DE LA PIPELINE DE PROCESADO DE LOGS](#item9)
   -  Creación de la pipeline de procesos. Alta en Elastic.
10. [ PROGRAMACIÓN DE EJECUCIÓN DE LA PIPELINE DE PROCESADO DE LOGS](#item10)
   -  Direccionamiento desde filebeat, de los datos hacia los procesos de la pipeline.
11. [ PREPARACIÓN EN SLACK PARA REGLAS Y ALERTAS DE KIBANA](#item11)
12. [ PREPARACIÓN EN ELASTIC PARA REGLAS Y ALERTAS DE KIBANA](#item12)
13. [ CREACIÓN DE REGLAS EN KIBANA](#item13)
14. [CREACIÓN DE CONECTOR CON SLACK Y DE ACCIÓN QUE LO EJECUTA](#item14) 
   - Emisión de órdenes de activación a partir de algunos resultados, comenzando por el envío de un mensaje al operador. 
15. [ SIGUIENTES PASOS](#item15)

[ ANEXO I:   REINICIO DE LA INSTANCIA DE MÁQUINA VIRTUAL (VM)](#item16)
   - Pasos a realizar de nuevo cada vez que se detenga y vuelva a arrancar la instancia de Máquina Virtual (MV).

[ ANEXO II:  DOCUMENTACIÓN DE REFERENCIA](#item17)

---

<a name="item1"></a> [Volver a Índice](#indice)
 ### 1. REQUISITOS Y ASUNCIONES
- Se asume la instalación del entorno de desarrollo Stack de Elastic (ELK) en la plataforma Google Cloud Platform (GCP)
- Se basa en la versión 7.17.5 del Stack Elastic (ELK). Configurada en el fichero [.env](../.env) para funcionar con la última versión 7 hasta la fecha:
- 
  ```shell
  vim .env
  ELK_VERSION=7.17.5
  Esc
  :wq!
  ```
  
- Instalación de Docker Compose. Se ha probado con  docker-compose 1.27.4
  En el caso que se describe, desde Google Cloud Platform, se indica más adelante cómo se realiza. En cambio, para el caso de instalación local directa sería de la siguiente forma:
  - Usuarios de Windows y Mac users tendrán Compose instalado automáticamente con Docker para Windows [^nota3]/Mac [^nota4].
  - Los usuarios de Linux ejecutarán la siguiente instrucción para instalar vía `pip`, pudiendo seguir las instrucciones de instalación [^nota5]
  
    ```shell
    pip install docker-compose
    ```

- Un mínimo de 4GB de RAM memoria virtual [^nota6] para contenedores. En este caso, desde Google Cloud Platform, se indican más adelante los pasos. En caso de la instalación directa en local, los usuarios de Mac y Windows deberán configurar su máquina virtual Docker para disponer de ese mínimo de la siguiente manera:

    ![Docker VM memory settings](./img/docker-vm-memory-settings.png)

 Y los usuarios de Linux, en ese caso, deberán ejecutar el siguiente comando como `root`:

```
sysctl -w vm.max_map_count=262144
```
Sin embargo, se indica más adelante cómo se procede en el caso de este proyecto, desde GCP.

El software utilizado en la Google Cloud Platform ha sido:

- Sistema operativo CentOS 8
- Dnf y dnf-plugins-core
- Git 1.8.3.1
- Docker ce para Centos 8
- Docker-compose 1.27.4
- Elasticsearch, filebeat y Kibana 7.17.5
- Navegador de Internet Google Chrome 105.0.5195.54 64 bit

Por último, para iniciar el Stack, es necesario que no haya ningún servicio arrancado en los puertos 9200, 9300 (elasticsearch), 5601 (kibana).

---

<a name="item2"></a> [Volver a Índice](#indice) 
### 2. ENTORNO DE DESARROLLO ELK EN GCP  
La plataforma permite actualmente su uso gratuito hasta un coste de 300$ durante 90 dias, 400$ en caso de tener cuenta con dominio propio registrado[^nota7]. Se pueden utilizar distintas cuentas para extender las pruebas. Alternativamente se puede instalar en otros entornos cloud de los que se disponga acceso, o con máquina virtual en local como VirtualBox, que se ha descartado por la excesiva capacidad de memoria que requiere.

1.	Arrir navegador y ejecutar la dirección https://console.cloud.google.com/
2.	Cliquear en "Crear Proyecto" => *Compute engine* + *Instancias de VM* + *Crear Proyecto* + *Nombre:"TFM Elastic CERN UAM" Organización:"sin organización"* + *Crear* + *Habilitar Engine API* + *Crear Instancia*
3.	Se crea una instancia VM en Región europe-southwest1 (Madrid)= zona europe-southwest1-a de uso general serie E2 tipo de máquina e2-mediom (2 CPU virtuales, 4 GB de memoria). Inicialmente 4GB son suficientes para una instancia de ElasticSearch (ES), una de Kibana (KB) y una de Filebeat (FB)

    ![Crear Instancia en GCP](./img/01_CrearInstanciaEnGCP.png)
4.	Se selecciona "Centos 8" por mayor conveniencia de uso (Disco de arranque: *cambiar* + Versión: *CentOS Stream 8*.
5.	Elección de "Disco persistente equilibrado" y fijado de memoria a 100GB. 
    
    ![Configurar instancia creada](./img/02_ConfigurarInstanciaCreada.png)
6.	Permitir http y https

![image](https://user-images.githubusercontent.com/23584277/188262328-bc080d3c-4838-4a08-a7e5-ca9a2e5799e6.png)
    
7. Clik en *Crear*
- Esto luego se puede ejecutar aquí mismo con esta línea de comando Gcloud ejecutable en Cloud Shell (hay que tener instalado el Cloud Shell, cliente de Windows gratuito para todos los usuarios, máximo 50 horas semanales) (Una alternativa es CON EL ICONO SUPERIOR DERECHO “>=” ).

```shell
gcloud compute instances create tfm-enrique-prieto-instancia-vm-01 --project=proyecto-tfm-enriqueprieto --zone=europe-southwest1-a --machine-type=e2-medium --network-interface=network-tier=PREMIUM,subnet=default --maintenance-policy=MIGRATE --provisioning-model=STANDARD --service-account=4836494966-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --tags=http-server,https-server --create-disk=auto-delete=yes,boot=yes,device-name=tfm-enrique-prieto-instancia-vm-01,image=projects/centos-cloud/global/images/centos-stream-8-v20220822,mode=rw,size=100,type=projects/proyecto-tfm-enriqueprieto/zones/europe-southwest1-a/diskTypes/pd-balanced --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --reservation-affinity=any
```
o con esta solicitud de REST equivalente:

```json
POST https://www.googleapis.com/compute/v1/projects/proyecto-tfm-enriqueprieto/zones/europe-southwest1-a/instances
{
  "canIpForward": false,
  "confidentialInstanceConfig": {
    "enableConfidentialCompute": false
  },
  "deletionProtection": false,
  "description": "",
  "disks": [
    {
      "autoDelete": true,
      "boot": true,
      "deviceName": "tfm-enrique-prieto-instancia-vm-01",
      "diskEncryptionKey": {},
      "initializeParams": {
        "diskSizeGb": "100",
        "diskType": "projects/proyecto-tfm-enriqueprieto/zones/europe-southwest1-a/diskTypes/pd-balanced",
        "labels": {},
        "sourceImage": "projects/centos-cloud/global/images/centos-stream-8-v20220822"
      },
      "mode": "READ_WRITE",
      "type": "PERSISTENT"
    }
  ],
  "displayDevice": {
    "enableDisplay": false
  },
  "guestAccelerators": [],
  "keyRevocationActionType": "NONE",
  "labels": {},
  "machineType": "projects/proyecto-tfm-enriqueprieto/zones/europe-southwest1-a/machineTypes/e2-medium",
  "metadata": {
    "items": []
  },
  "name": "tfm-enrique-prieto-instancia-vm-01",
  "networkInterfaces": [
    {
      "accessConfigs": [
        {
          "name": "External NAT",
          "networkTier": "PREMIUM"
        }
      ],
      "stackType": "IPV4_ONLY",
      "subnetwork": "projects/proyecto-tfm-enriqueprieto/regions/europe-southwest1/subnetworks/default"
    }
  ],
  "reservationAffinity": {
    "consumeReservationType": "ANY_RESERVATION"
  },
  "scheduling": {
    "automaticRestart": true,
    "onHostMaintenance": "MIGRATE",
    "provisioningModel": "STANDARD"
  },
  "serviceAccounts": [
    {
      "email": "4836494966-compute@developer.gserviceaccount.com",
      "scopes": [
        "https://www.googleapis.com/auth/devstorage.read_only",
        "https://www.googleapis.com/auth/logging.write",
        "https://www.googleapis.com/auth/monitoring.write",
        "https://www.googleapis.com/auth/servicecontrol",
        "https://www.googleapis.com/auth/service.management.readonly",
        "https://www.googleapis.com/auth/trace.append"
      ]
    }
  ],
  "shieldedInstanceConfig": {
    "enableIntegrityMonitoring": true,
    "enableSecureBoot": false,
    "enableVtpm": true
  },
  "tags": {
    "items": [
      "http-server",
      "https-server"
    ]
  },
  "zone": "projects/proyecto-tfm-enriqueprieto/zones/europe-southwest1-a"
}
```


Se obtiene entonces la siguiente instancia creada, que habrá que ejecutar o detener (menú hamburguesa a la derecha de SSH, "Detener"), en función del uso, procurando minimizar su coste:

 ```shell
 NAME: enriqueprieto-centos8-2
 ZONE: europe-southwest1-a
 MACHINE_TYPE: e2-medium
 PREEMPTIBLE:
 INTERNAL_IP: 10.204.0.2
 EXTERNAL_IP: 34.175.205.191
 STATUS: RUNNING
 enrique@cloudshell:~ (tfm-elastic-cern-uam)$
  ```

![image](https://user-images.githubusercontent.com/23584277/188262380-5781c36c-a38a-4550-b38a-77b06a2b4c60.png)
   
8. A continuación se crea una regla de firewall que permita entrada de puerto 80. Anotar la IP externa de la instancia, y pulsar los tres puntos a la derecha de la instancia + *Ver detalles de red* + Columna izquiera *Firewall* + *Crear regla de Firewall* + Nombre:xxx + *Continuar* + *Agegar regla* + Prioridad: 1000 + Dirección del tráfico: *Entrada* + Destinos: *Todas las instancias* + Filtro de origen: *Rangoas de IPv4* +Rangos de IPv4 de destino: poner la ip de la instancia "34.175.112.47 " + Protocolos y puertos: *Protocolos y puertos especificados* + TCP: *80* (puede que valga "all") + *crear* + *continuar* + *asociar* + seleccionar la red default + *asociado* + *Continuar* + *crear*

![image](https://user-images.githubusercontent.com/23584277/188262410-5d427e07-a157-488f-b072-78b43e7c46d2.png)
    
![image](https://user-images.githubusercontent.com/23584277/188262428-e78db0fd-d331-4eba-98ea-ff915b143ba5.png)
    

Commando REST equivalente:

 ```shell
POST https://www.googleapis.com/compute/v1/projects/tfm-elastic-cern-uam/global/firewalls
{
  "kind": "compute#firewall",
  "name": "enri-abrir-puerto80-salida",
  "selfLink": "projects/tfm-elastic-cern-uam/global/firewalls/enri-abrir-puerto80-entrada",
  "network": "projects/tfm-elastic-cern-uam/global/networks/default",
  "direction": "INGRESS",
  "priority": 1000,
  "description": "enri-abrir-puerto80",
  "allowed": [
    {
      "IPProtocol": "all"
    }
  ],
  "destinationRanges": [
    "34.175.205.191"
  ]
}
  ```

Esta es la respuesta REST:

  ```shell
{
  "allowed": [
    {
      "IPProtocol": "all"
    }
  ],
  "creationTimestamp": "2022-07-17T12:26:39.194-07:00",
  "description": "enri-abrir-puerto80",
  "destinationRanges": [
    "34.175.205.191"
  ],
  "direction": "INGRESS",
  "disabled": false,
  "enableLogging": false,
  "id": "2176967777015422080",
  "kind": "compute#firewall",
  "logConfig": {
    "enable": false
  },
  "name": "enri-abrir-puerto80-salida",
  "network": "projects/tfm-elastic-cern-uam/global/networks/default",
  "priority": 1000,
  "selfLink": "projects/tfm-elastic-cern-uam/global/firewalls/enri-abrir-puerto80-salida"
}
  ```

---

<a name="item3"></a> [Volver a Índice](#indice) 
### 3. CONEXIÓN POR SSH. INSTALACIÓN DE DNF Y DOCKER 
Para conectarse a la instancia de Máquina Virtual a través de conexión segura SSH:

9. Volver a la instancia (columna izquierda + Compute Engine + Instancias de VM) + En la columna “Conectar” de la instancia,  se cliquea:

 ```shell 
SSH => abrir en otra ventana del navegador 
 ```
 
 ![image](https://user-images.githubusercontent.com/23584277/188262468-401ed3c0-5146-44e9-9168-a3eb71db0191.png)

Esto puede guardarse como un grupo de comandos de gcloud  para conectarse directamente a la máquina (se ejecuta desde cliente CLI de Windows, o directamente desde la interfaz de GCP arriba a la derecha, el Cloud Shell de Google cloud, con 50 horas iniciales gratuitas):
 ```shell
gcloud compute ssh --zone "europe-southwest1-a" "enriqueprieto-centos8-2"  --project "tfm-elastic-cern-uam"
 ```

Crea automáticamente los directorios, y el usuario de SSH (en este caso auaenrique), en la máquina enriqueprieto.centos8-2, pidiendo contraseña que deberá dejarse en blanco.

10. Se pueden ejecutar a continuación los siguientes comandos para comprobar su correcto funcionamiento:

```shell
ls
pwd
whoami
 ```
 
 Las instrucciones que se comentan a partir de ahora se entienden introducidas en la ventana de esta conexión establecida.
 
11.  Instalación de dnf:

```shell
sudo yum install dnf -y
   ```

12. Instalación de plugins:

```shell
 sudo yum install dnf-plugins-core -y
 ```
    
13. Instalación de Docker. Deberá en este caso utilizarse “SUDO” delante:

```shell
 sudo dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
   ```


14. Se instala el Docker package, aceptando dos veces con Y(es) las preguntas que realiza en su proceso:

```shell
 sudo dnf install docker-ce docker-ce-cli containerd.io -y -y
   ```

(Puede tardar  unos minutos)

15. Se procede a arrancar el servicio Docker y añadírselo al autorun:

```shell
 sudo systemctl enable --now docker
   ```

16. CentOS 8 utiliza un firewall diferente al de Docker. Por lo tanto, al tener firewall habilitado, se requiere añadir una regla de enmascaramiento hacia él.

```shell
 sudo firewall-cmd --zone=public --add-masquerade --permanent
 sudo firewall-cmd --reload
   ```

---

<a name="item4"></a> [Volver a Índice](#indice) 
### 4. INSTALACIÓN DE DOCKER-COMPOSE
17. En este punto se instala  Docker-compose, servicio que permite desplegar el proyecto en otra máquina utilizando un solo comando. Para descargarlo:
18. 
 ```shell
 sudo curl -L "https://github.com/docker/compose/releases/download/1.27.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
   ```
   
(Puede tardar unos minutos)

18.  A continuación se le da carácter ejecutable:

 ```shell
 sudo chmod +x /usr/local/bin/docker-compose
   ```
   
19.  Procediendo a su comprobación posterior:

 ```shell
 docker-compose -v
   ```
   
   obteniendo como respuesta de confirmación:
   
 ```shell
 docker-compose version 1.27.4, build 40524192
   ```
   
   lo cual indica que funciona correctamente.
   
20. Seguidamente se evitará la denegación de servicio[^nota32], para lo cual habrá que cambiar el siguiente permiso: 

```shell
 sudo chmod 666 /var/run/docker.sock
  ```

Esto será preciso ejectutarlo cada vez que se arranque de nuevo la instancia de la máquina virtual.

21. Posteriormente se prueba ejecutando docker run hello-world y comprobandos que funciona:

```shell
 docker run hello-world
   ```

Recibiendo como confirmación la siguiente respuesta:

```shellUnable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
2db29710123e: Pull complete
Digest: sha256:53f1bbee2f52c39e41682ee1d388285290c5c8a76cc92b42687eecf38e0af3f0
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
   ```
   
---

<a name="item5"></a> [Volver a Índice](#indice) 
### 5. INSTALACIÓN DE GIT
A continuación se instalan Git y Docker como superusuario (poniendo SUDO delante), usando la instrucción de la Web [^nota8]  o [^nota9]

22.	Instalar git, por tener los contenedores subidos y compartidos en Github [^nota10]

```shell
sudo yum install git -y
 ```

(yum se encarga de solicitar la última versión).

23. Se comprueba con lo siguiente:

```shell
git --version 
 ```

Devolviendo, si está todo correcto:

```shell
git version 1.8.3.1
 ```

24. Una vez instalado el GIT, se clonará el Git instalado previamente en github:

```shell
git clone https://github.com/Peilike2/tfm_bigdata_viu_enriqueprieto.git
 ```

25.	Antes de lanzar docker-compose se deberá asegurar el cumplimiento de requisitos de memoria máxima de linux, como se indicó en el apartado "requisitos":

```shell
 sudo sysctl -w vm.max_map_count=262144
 ```

Lo cual habrá que ejecutar cada vez que se reinicie la instancia tras una parada.

A modo informativo, se usarán comandos docker basicos [^nota11] además de los comandos de docker-compose[^nota12]  y como editor de texto se utiliza vim.
 
---

<a name="item6"></a> [Volver a Índice](#indice) 
### 6. INSTALACIÓN DEL STACK ELASTIC
En este apartado se instalan los servicios necesarios para lograr la siguiente estructura de ejecución: 

![image](https://user-images.githubusercontent.com/23584277/188262622-83947047-2c99-4236-9ac3-b1a7dec73a72.png)

Para ello se efectuarán las siguentes acciones:
 - Instalar el conjunto de contenedores elasticsearch, kibana y filebeat
 - Arrancar dichos servicios, comprobando que funcionan correctamente
 - Probar explorando Discover[^nota13] en Kibana.

26. Personalización de docker-compose.yml
 - En dicho fichero de configuración del mencionado git, se ha limpiado toda referencia a contenedores no usados, dejando exclusivamente elasticsearch, filebeat y kibana
 - Además, en el apartado kibana se ha redirigido el puerto al 80 con "80:5601"
 - Tras su edición, ya incorporada en el git, se puede comprobar la integridad de este y otros ficheros ".yml" con herramientas como esta online: http://www.yamllint.com/

27. Se procede al aseguramiento de los permisos correspondientes a /filebeat/config  (siendo tfm_bigdata_viu_enriqueprieto el directorio del proyecto:

```shell
cd $pwd
cd tfm_bigdata_viu_enriqueprieto/filebeat/config/
chmod go-w filebeat.yml
```

28. Después se garantiza el borrado de datos anteriores (para el caso de no ser la primera prueba) a la vez que el arranque de los contenedores del stack Elasticic definido en [docker-compose.yml](../../docker-compose.yml). Para ello se accede a la raíz del proyecto y se ejecuta lo siguiente:

```shell
cd $pwd
cd tfm_bigdata_viu_enriqueprieto
docker-compose down -v
docker-compose up -d --remove-orphans
```
(+ Enter)
(Puede tardar unos minutos)

```shell
docker stop elasticsearch
docker start elasticsearch
docker stop kibana
docker start kibana
```

(+ Enter)

A título informativo, se indican las distintas formas de detener y eliminar:

```shell
# Solamente detener los servicios:
docker-compose stop

# Detener y eliminar contenedores, redes,...:
docker-compose down 

# Parar y eliminar volúmenes (directorios donde se indica el guardado persistente de datos):
docker-compose down --volumes 
# Otra forma:
docker-compose down -v

# Parar y eliminar imágenes:
docker-compose down --rmi <all|local> 

# Con "--remove-orphans" eliminarán en su caso los contenedores que hubieran sido creados anteriormente y que ya no estén registrados en docker-compose.yml

# Parar solo fielebeat
Docker stop filebeat

# Remove filebeat
Docker rm filebeat
```

**COMPROBACIONES**

29. Para comprobar que los tres contenedores se encuentran en estado saludable o "healthy" (filebeat, kibana, elasticsearch), se ejecuta:

```shell
docker ps
```

30. Además se comprueba si han arrancado correctamente, visualizando los respectivos logs de los mencionados servicios:

```shell
docker logs -f elasticsearch
```

Ctrl+c

```shell
docker logs -f kibana
```

Ctrl+c

```shell
docker logs -f filebeat
```

Ctrl+c

31. En caso de modificar el fichero de testeo, deberá asegurarse de que el sistema operativo Windows no le ha añadido caracteres ilegibles en UNIX, usando la siguiente instalación seguido de la instrucción a continuación:

```shell
sudo yum install dos2unix -y
```

```shell
dos2unix test/srm-grid002Domain_original_extracto_unix.log
```

Como alternativa se pueden usar herramientas online como esta: https://toolslick.com/conversion/text/dos-to-unix
   
---

<a name="item7"></a> [Volver a Índice](#indice) 
### 7. VISUALIZACIÓN CON KIBANA. CREACIÓN DE INDEX PATTERN
32. Primero se  comprueba que kibana está abierto en el puerto 80 en la ventana de la conexión SSH con: 

 ```shell
curl localhost:80
 ```

/Al ser el puerto 80, se podría obviar el parámetro de dicho puerto)

Se confirma que no da error como resultado, y a continuación desde cualquier navegador se utiliza la ip que proporciona la plataforma, y dicho puerto 80:

![image](https://user-images.githubusercontent.com/23584277/188262727-065c5332-773b-42b0-b839-67a797d01933.png)
    
33. A continuación, se abre en un navegador la URL de Kibana [^nota14].
Si se estuviera trabajando en local sería:

```shell
http://localhost:80/
```

Pero al tratarse de acceso desde otra máquina, se utiliza la ip proporcionada por la plataforma, y dicho puerto 80 según se configuró anteriormente en el apartado kibana del fichero de configuración docker-compose.yml: 

```shell
http://xxx.xxx.xxx:80/
- Usuario: elastic
- Password: changeme
```

34. Para visualizar los logs se debe primero crear un Index Pattern [^nota15]. Los index patterns permiten acceder desde Kibana a los índices en elasticsearch, y, por lo tanto, a los documentos almacenados en estos índices.

Si no se indica lo contrario en la configuración de filebeat para envío a elasticsearch, los índices que se crearán con el nombre `filebeat-*`.

![Index Patterns](./img/index-pattern.png)

Por lo tanto, en la sección de Management de Kibana[^nota15], se debe seleccionar `Index Patterns` en el grupo `Kibana`.

![Index Patterns](./img/kibana-index-patterns-management.png)

A continuación se pulsa el botón azul `Create Index Pattern` y se da de alta un patrón `filebeat-*`.

![Index Patterns](./img/index-pattern-create-1.png)

Seguidamente, en la misma ventana se selecciona el 'Timestamp field', que es el campo a usar para mostrar la serie temporal de datos en Discover.

En este caso se indica `@timestamp`.

![Index Patterns](./img/index-pattern-create-2.png)

Y se pulsa `Create Index Pattern`.

Se comprueba que queda el Index Pattern "Filebeat-*" creado en Stack Management + Index Patterns:

![image](https://user-images.githubusercontent.com/23584277/188205896-aaafbc48-1c1a-4ecb-abd8-1982bb4d20ea.png)

Por último, se selecciona en el "menú hamburguesa" de la izquierda, en Kibana, `Discover`.

Se cliquea `New` en el menú superior derecho, para limpiar cualquier filtro que hubiera en la búsqueda.
    
  ![Discover Filebeat](./img/discover-filebeat.png)
 
Y en el selector de filtros, persiana desplaegable de la izquierda, se escoge el `index pattern` que se acaba de crear, `filebeat-*`.

Acto seguido, Se selecciona arriba a la derecha el rango de fechas y horas que permita ver los datos ingestados. Por ejemplo desde `2 years ago` hasta `now`

![image](https://user-images.githubusercontent.com/23584277/188262817-d248291d-a176-4709-b2d1-345fffc5d4a7.png)
   
---

<a name="item8"></a> [Volver a Índice](#indice) 
### 8. SIMULACIÓN DE PIPELINE DE PROCESADO DE LOGS (VM)

El documento que llega a Elastic tiene líneas de log con este aspecto:

"27 Dec 2020 03:09:29 () [k6A:2394036:srm2:prepareToGet:-1093710432:-1093710431 k6A:2394036:srm2:prepareToGet SRM-grid002] Pinning failed for /xxxx/xx.xxx.xx/data/atlas/xxxxxxxxxxxx/rucio/mc16_13TeV/ce/13/EVNT.23114463._000856.pool.root.1 (File is unavailable.)"

Y la intención es que Elastic lo acabe guardando como:

```json
 "doc" : {
        "_index" : "_index",
        "_type" : "_doc",
        "_id" : "_id",
        "_source" : {
          "@timestamp" : "2020-12-30T06:33:29.000+01:00",
          "method1" : "3Bs:10796:srm2:prepareToGet:-1093074894:-1093074893",
          "method2" : "3Bs:10796:srm2:prepareToGet",
          "host" : {
            "name" : "SRM-grid002"
          },
          "message" : "Pinning failed for /zzzz/zz.zzz.zz/data/ops/nagios-argo-mon.egi.cro-ngi.hr/arcce/srm-input (File is unavailable.)",
          "url" : {
            "original" : "/zzzz/zz.zzz.zz/data/ops/nagios-argo-mon.egi.cro-ngi.hr/arcce/srm-input"
          }
```

Para realizar esta estructuración, se recurre a las cadenas de procesos o pipelines[^nota16] de ingesta de elasticsearch, que se ejecutarán en los nodos llamados de ingesta[^nota33].

Dado que se parte de un clúster elasticsearch con un solo nodo, este nodo realizará todos los roles (master, data, ingest, etc.).[^nota17][^nota18].

Las pipelines de ingesta proporcionan a elasticsearch un mecanismo para procesar previamente los documentos antes de almacenarlos. Con una pipeline, se pueden analizar sintácticamente, transformar y enriquecer los datos de entrada a través de un conjunto de procesadores[^nota19] que se aplican de forma secuencial a los documentos de entrada, para generar el documento definitivo que almacenará elasticsearch.

![image](https://user-images.githubusercontent.com/23584277/188305998-8ee2ba3d-8ba9-4d83-ba5b-268fc8718450.png)

En primer lugar, se procede a crear una simple pipeline de ingesta, basada en un procesador de tipo dissect[^nota20], que parseará el campo `message` de entrada generando los diversos campos que se precisan a la salida (`fecha`, `url.original`, `host.name`, etc).
En segundo lugar se indica el formato de fecha y hora que debe aplicar al campo `fecha` con el proceso "date" y su resultado se aplica al campo "@timestamp".
A continuación se elimina el campo `fecha` dado que "@timestamp" lo sustituye en formato inteligible.

Para ello se selecciona en el menú de la izquierda Management:`Dev Tools`.

![Dev Tools](./img/devtools-icon.png)

Y se copia y pega lo siguiente en la consola, a modo de simulación de prueba de los procesadores:

```json
POST _ingest/pipeline/_simulate
{
  "pipeline": {
    "description": "_description",
    "processors": [
      {
        "dissect": {
          "field": "message",
          "pattern": "%{fecha} () [%{method1} %{method2} %{host.name}] %{message}"
        }
      },
      {
        "dissect": {
          "field": "message",
          "pattern": "%{?ignoreme} %{?ignoreme} %{?ignoreme} %{url.original} %{?ignoreme}"
        }
      },
      {
        "date": {
          "field": "fecha",
          "target_field": "@timestamp",
          "formats": [
            "dd MMM yyyy HH:mm:ss"
          ],
          "timezone": "Europe/Amsterdam"
        }
      },
      {
        "remove": 
          {
          "field": "fecha"
        }
      
    }
    ]
  },
  "docs": [
    {
      "_source": {
        "timestamp": 1569846065739,
        "message": "27 Dec 2020 03:09:29 () [k6A:2394036:srm2:prepareToGet:-1093710432:-1093710431 k6A:2394036:srm2:prepareToGet SRM-grid002] Pinning failed for /xxxx/xx.xxx.xx/data/atlas/xxxxxxxxxxxx/rucio/mc16_13TeV/ce/13/EVNT.23114463._000856.pool.root.1 (File is unavailable.)"
      }
    },
    {
      "_source": {
        "timestamp": 1569846065739,
        "message": "30 Dec 2020 06:33:29 () [3Bs:10796:srm2:prepareToGet:-1093074894:-1093074893 3Bs:10796:srm2:prepareToGet SRM-grid002] Pinning failed for /zzzz/zz.zzz.zz/data/ops/nagios-argo-mon.egi.cro-ngi.hr/arcce/srm-input (File is unavailable.)"
      }
    }
  ]
}
```

Al ejecutar esta petición, se puede comprobar si el JSON resultante es el esperado:

```json
{
  "docs" : [
    {
      "doc" : {
        "_index" : "_index",
        "_type" : "_doc",
        "_id" : "_id",
        "_source" : {
          "@timestamp" : "2020-12-27T03:09:29.000+01:00",
          "method1" : "k6A:2394036:srm2:prepareToGet:-1093710432:-1093710431",
          "method2" : "k6A:2394036:srm2:prepareToGet",
          "host" : {
            "name" : "SRM-grid002"
          },
          "message" : "Pinning failed for /xxxx/xx.xxx.xx/data/atlas/xxxxxxxxxxxx/rucio/mc16_13TeV/ce/13/EVNT.23114463._000856.pool.root.1 (File is unavailable.)",
          "url" : {
            "original" : "/xxxx/xx.xxx.xx/data/atlas/xxxxxxxxxxxx/rucio/mc16_13TeV/ce/13/EVNT.23114463._000856.pool.root.1"
          },
          "timestamp" : 1569846065739
        },
        "_ingest" : {
          "timestamp" : "2022-08-30T19:05:42.796627212Z"
        }
      }
    },
    {
      "doc" : {
        "_index" : "_index",
        "_type" : "_doc",
        "_id" : "_id",
        "_source" : {
          "@timestamp" : "2020-12-30T06:33:29.000+01:00",
          "method1" : "3Bs:10796:srm2:prepareToGet:-1093074894:-1093074893",
          "method2" : "3Bs:10796:srm2:prepareToGet",
          "host" : {
            "name" : "SRM-grid002"
          },
          "message" : "Pinning failed for /zzzz/zz.zzz.zz/data/ops/nagios-argo-mon.egi.cro-ngi.hr/arcce/srm-input (File is unavailable.)",
          "url" : {
            "original" : "/zzzz/zz.zzz.zz/data/ops/nagios-argo-mon.egi.cro-ngi.hr/arcce/srm-input"
          },
          "timestamp" : 1569846065739
        },
        "_ingest" : {
          "timestamp" : "2022-08-30T19:05:42.796634283Z"
        }
      }
    }
  ]
}
```

Esta petición simula[^nota21] una pipeline, usando el endpoint del API REST de elasticsearch `_ingest/pipeline/_simulate`. En el contenido del cuerpo, se dispone de un JSON con los procesadores de la pipeline:

- [**dissect**][^nota22]: Se encarga de separar el texto que viene en el campo message a partir de los espacios en blanco, y crea distintos campos (timestamp, host_name, process_name, etc.) con los valores que extrae del campo message de entrada.
- [**remove**][^nota23]: eliminará el campo que, una vez modelado, no interesa guardar por ser información redundante.
- [**date**]: Aplica un formato determinado de facha y hora al campo `fecha` y aplicar el resultado al campo ECS "@timestamp". Importante: Actualmente la hora saldrá en UTC, independientemente de la hora local configurada en kibana, pendiente de próxima revisión que lo corrija.
 
---

<a name="item9"></a> [Volver a Índice](#indice) 
### 9. ALTA DE LA PIPELINE DE PROCESADO DE LOGS (VM)
Una vez se comprueba que la pipeline de ingesta funciona según lo deseado, se da de alta en elasticsearch para poder usarla con el nombre "logs-pipeline", según se indica en el primer argumento del método PUT, coincidente con el nombre de pipeline que se da en el fichero de configuración filebeat.yml, según se explica más adelante. Para ello, en la misma consola de Dev Tools, de debe ejecutar:

```json
PUT _ingest/pipeline/logs-pipeline
{
    "description": "_description",
    "processors": [
      {
        "dissect": {
          "field": "message",
          "pattern": "%{fecha} () [%{method1} %{method2} %{host.name}] %{message}"
        }
      },
      {
        "dissect": {
          "field": "message",
          "pattern": "%{?ignoreme} %{?ignoreme} %{?ignoreme} %{url.original} %{?ignoreme}"
        }
      },
      {
        "date": {
          "field": "fecha",
          "target_field": "@timestamp",
          "formats": [
            "dd MMM yyyy HH:mm:ss"
          ],
          "timezone": "Europe/Amsterdam"
        }
      },
      {
        "remove": 
          {
          "field": "fecha"
        }
      
    }
    ]
  }
```
De lo cual se obtendrá la respuesta de confirmación:

```json
{
  "acknowledged" : true
}
```

Creando la pipeline de ingesta **logs-pipeline**, que se usará  en el próximo apartado.

![image](https://user-images.githubusercontent.com/23584277/188262915-db77bf79-afcd-447b-aef6-c03a7854e353.png)
   
---

<a name="item10"></a> [Volver a Índice](#indice) 
### 10. PROGRAMACIÓN DE EJECUCIÓN DE LA PIPELINE DE PROCESADO DE LOGS (VM)

En este punto se debe indicar a elasticsearch que los documentos que vayan a ser almacenados en los índices creados por filebeat deben pasar primero esta pipeline que los va a transformar. Para ello, es necesario editar el fichero de configuración de filebeat. [filebeat/config/filebeat.yml](../../filebeat/config/filebeat.yml), y en la sección `output.elasticsearch` se  descomenta la línea `pipeline: logs-pipeline`.

```shell
cd $pwd
cd tfm_bigdata_viu_enriqueprieto/filebeat/config/
vim filebeat.yml
```

```yaml
output.elasticsearch:
  hosts: ["elasticsearch:9200"]
  username: '${ES_USERNAME:elastic}'
  password: '${ES_PASSWORD:changeme}'
  pipeline: logs-pipeline
```
Se sale de vim guardando:

```shell
Esc
:wq!
```

***Ingesta de logs estructurados***

Se procede a arrancar de nuevo `filebeat`.

```shell
cd ..
cd ..
docker-compose up -d
```

Se puede comprobar que no hay errores en la ejecución de filebeat, antes de pasar al siguiente apartado:

```shell
docker logs -f filebeat
Ctrl+c
```

Se comprueba que está creada la pipeline "logs-pipeline" en Stack Management + Ingest Pipelines ( http://34.175.112.47/app/management/ingest/ingest_pipelines )

![image](https://user-images.githubusercontent.com/23584277/188205437-2c51e278-9345-4180-8b7c-df6772359117.png)

***Visualización de los logs en Discover***

Nuevamente en Kibana, a través del navegador, se usa la barra de búsqueda para filtrar los datos proporcionados. Se procede a filtrar con distintos criterios, por ejemplo para ver los fallos de tipo 2 sería `message: "*Pinning failed for*" and message: : "*Fallo tipo 2*"`

El lenguage usado para filtrar las búsquedas es Kibana Query Language (KQL[^nota24].


Se pulsa el botón `Save` en la barra superior y se guarda la búsqueda con el nombre `[Filebeat] Host/Process`

![Save Search](./img/save-search.png)


****Fin del proceso****

---

<a name="item11"></a> [Volver a Índice](#indice)
 ### 11. PREPARACIÓN EN SLACK PARA REGLAS Y ALERTAS DE KIBANA
 
 CREACIÓN CANAL DE MENSAJERÍA SLACK[^nota25]
 
Primeramente, navegar a la dirección de Internet slack.com
Una vez registrado un usuario, se procede a la creación de un Espacio de Trabajo
 
 ![image](https://user-images.githubusercontent.com/23584277/188259398-a4511a30-e659-40d2-9f37-6a386af88a68.png)
 
![image](https://user-images.githubusercontent.com/23584277/188259433-3425f391-a6f8-4708-83cf-5db7a9b38609.png)

Dar nombre al espacio de trabajo 

![image](https://user-images.githubusercontent.com/23584277/188259476-3682d97d-aa06-47d1-96ed-fa27305275fd.png)

Se puede en ese punto añadir colaboradores o copiar enlace de invitación.

Después se debe ir a https://my.slack.com/services/new/incoming-webhook

A continuación, elegir el espacio de trabajo y el canal.

Pulsar ```Añadir Integración con Webhooks entrantes```

Copiar la URL de Webhook, similar a https://hooks.slack.com/services/T040V6MNRFF/B0410KFHV6Y/xxxxxx

Más abajo, en la misma página, se pueden definir más detalles:

![image](https://user-images.githubusercontent.com/23584277/188260038-8391d31c-f1ad-4cd3-b94d-32aab494aea1.png)

Y por último, se guardan los ajustes.


Además Slack puede configurarse para que envíe emails automáticamente según la siguiente configuración:

![image](https://user-images.githubusercontent.com/23584277/188202858-d4ab34ce-6799-45a6-97ba-16185023b517.png)

---

<a name="item12"></a> [Volver a Índice](#indice)
 ### 12. PREPARACIÓN EN ELASTIC PARA REGLAS Y ALERTAS DE KIBANA

Se deberá editar el archivo de configuración elasticsearch.yml añadiendo al final las líneas que se indican (ya incluido en el fichero del repositorio):

```shell
cd $pwd
cd tfm_bigdata_viu_enriqueprieto/elasticsearch/config
vim elasticsearch.yml
```

```shell
xpack.security.authc.api_key.enabled: true
xpack.security.authc.api_key.hashing.algorithm: pbkdf2
xpack.security.authc.api_key.cache.ttl: 1d
xpack.security.authc.api_key.cache.max_kes: 10000
xpack.security.authc.api_key.cache.hash_algo: ssha256
```

A continuación se sale guardando con "Esc" + 

```shell
:wq!
```

En caso de querer salir sin guardar:

```shell
:qa!
```

Quedando así el fichero:

![image](https://user-images.githubusercontent.com/23584277/188206700-82237ccf-a02f-4045-899f-a9171cbac9d0.png)

Seguidamente, se procede a la edición del fichero de configuración de kibana:
```shell
cd $pwd
cd tfm_bigdata_viu_enriqueprieto/kibana/config
vim kibana.yml
```

Se deben añadir dos líneas con contraseña de al menos 32 caracteres:

```shell
xpack.security.encryptionKey: "abcdefghijklmnopqrstuvwxyz012345"
xpack.encryptedSavedObjects.encryptionKey: "abcdefghijklmnopqrstuvwxyz012345"
```

A continuación se sale guardando con "Esc" + 

```shell
:wq!
```

En caso de querer salir sin guardar:

```shell
:qa!
```

Quedando así el fichero:

![image](https://user-images.githubusercontent.com/23584277/188209274-0ab73f26-1382-4362-a143-583bbf007b61.png)

(Opcional) Se podría dejar preconfigurado el conector de slack con las siguientes líneas, aunque en este caso se procederá a su configuración desde kibana:

```shell
xpack.actions.customHostSettings:
  my-slack:
   name: preconfigured-slack-connector-type
   actionTypeId: .slack
   secrets:
     webhookUrl: 'https://hooks.slack.com/services/abcd/efgh/ijklmnopqrstuvwxyz'
https://hooks.slack.com/services/T0409BY02T1/B040M1LGB1B/xxxxxx
```

(Por seguridad, se debe sustituir la última dirección por la indicada en https://cernatlasciaffalertas.slack.com/services/B040M1LGB1B )
 
Se puede realizar la siguiente prueba de funcionamiento desde línea de comandos: (https://cernatlasciaffalertas.slack.com/services/B040M1LGB1B)

```shell
curl -X POST --data-urlencode "payload={\"channel\": \"#tfm-enrique-prieto\", \"username\": \"Robot de alertas desde curl\", \"text\": \"Alerta publicada con curl en el canal Slack #tfm-enrique-prieto procedente del robot webhookbot. <https://i.pinimg.com/564x/1c/aa/f2/1caaf2b3e6ab9b2b4bd1b62a85fec8f9.jpg|* info>    \", \"icon_emoji\": \":warning:\" }" https://hooks.slack.com/services/T0409BY02T1/B040M1LGB1B/xxxxxxxx
```

(De nuevo, por seguridad, se debe sustituir la última dirección por la indicada en https://cernatlasciaffalertas.slack.com/services/B040M1LGB1B )

A título informativo, para conectar con terceras aplicaciones se puede crear una clave API en columna iquierda + Stack Management + Security + API Keys + Create API Key.
En este caso,  API key de nombre alerta01 para usuario elastic 
b01kVzhJSUI4bFdzVGNvVVZjZFo6QXZnXzl1SXVSWnlybUoyX3VGSDhjUQ==

---

<a name="item13"></a> [Volver a Índice](#indice)
 ### 13. CREACIÓN DE REGLA EN KIBANA

Ver documentación sobre lo tratado en este apartado[^nota26][^nota27][^nota28]

Tras acceder, a través de la columna izquierda de kbana en el navegador de internet: Stack Management + Alerts & Insights + Rules and Connectors + Create Rule

![image](https://user-images.githubusercontent.com/23584277/188261564-facda6b9-cb6f-403c-ab21-1083e012cdb9.png)

Se selecciona:

Name: Nombre a elegir

Check every : Seleción de periodo de 1 minuto

Notify: ```Only on status change``` (Solo manda mensaje cuando hay nueva alerta)

Notify:```Eveytime alert is active```: Envía mensajes periódicos, aunque sea informando de  0 concurrencias una vez que ya ha informado de las anteriores.

En este caso se seleccionará según se muesta en la imagen:

![image](https://user-images.githubusercontent.com/23584277/188261638-6bfb5c0d-85b5-4c6f-a376-9b325fbfa1a8.png)

Select index ```filebeat-*```

Size 100

Timestamp ```@timestamp```

Se indica la creación de  una consulta que localice los documentos que contengan en el campo ```mensaje``` el texto ```Pinning failed for``` en los dos últimos años y los agrege por servidor, y dentro de este por ubicación del fichero perdido indicado en el log.

Se debe indicar la consulta en ```Define the Elasticsearch query```
Query alert:

```json 
{
  "query": {
    "bool": {
      "must": [
   {
       "match": {
        "message": "Pinning failed for"
      }
  
    },
        {
          "range": {
     "@timestamp": {
        "gte": "now-2y/d",
        "lte": "now/d"
            }
          }
        }
      ]
    }
  }  ,
  "aggs": {
    "Hostnames y ficheros con el mensaje buscado": {
      "multi_terms": {
      "terms":[ {
        "field": "host.name"
      }, {
          "field": "url.original"
      }]
      }
    }
  }
  }
```

---

<a name="item14"></a> [Volver a Índice](#indice)
 ### 14. CREACIÓN en KIBANA DE CONECTOR CON SLACK Y ACCIÓN QUE LO EJECUTA 

Ver información sobre la conexión de Slac para acciones de alerta de kibana[^nota29]
Ver  información sobre formatos de mensajes con Slack-Markdown, etc[^nota30][^nota31]

Se define la condición de cumplimiento de una regla para la que se activará la acción.
En este caso, la condición es que la cumplan mil logs o menos en los últimos 27 años (aproximadamente 10000 días):
En ```Rules and Connector``` seleccionamos y editamos la regla creada.

Tipo de regla a seleccionar: STACK RULES + Elasticsearch query

Y los siguientes parámetros:

Run when Query matched

Stack connector: tfm-slack

When number of matches:
Is ```below or equals```  ```1000```

FOR ```THE LAST``` ```10000``` ```days```

A continuación se define el mensaje que se enviará al canal de Slack cuando se cumpla la condición.
El cuadro de inserción de texto admite formato directo de salto de línea y código Slack-Markdown.
Además, se pueden referenciar los campos. En este caso, se han utilizado:

```shell
{{context.date}}
{{alertName}}
{{params.timeWindowSize}}
{{params.timeWindowUnit}}
{{context.value}}
{{context.conditions}}
```

Por último, admite referencias a cada uno de los documentos resultantes de la consulta, entre los indicadores:
{{#context.hits}} y {{/context.hits}}

En este caso:

```shell
{{#context.hits}}

{{_source.@timestamp}}
{{_source.host.name}}
{{_source.url.original}}
{{_source.message}}

{{/context.hits}}
```

Quedando por tanto el código para el mensaje:

```shell
:warning:       :red_circle::red_circle::white_circle::white_circle::white_circle: 
  
:clock2:       _{{context.date}}_
               *{{alertName}}* de Kibana webhookbot
               Muestrando desde *hace {{params.timeWindowSize}} {{params.timeWindowUnit}}*
  
:incoming_envelope: Desde la última alerta informada, ha sucedido *{{context.value}} :new: veces * :
_" {{context.conditions}}"_ 

*<https://i.pinimg.com/564x/1c/aa/f2/1caaf2b3e6ab9b2b4bd1b62a85fec8f9.jpg|:information_source:>*       *<mailto:enrique@aua-arquitectura.es?subject=Alerta_de_Kibana|:sos:>     <#C0412J19N1X>*
  
{{#context.hits}}
*Instante:* {{_source.@timestamp}}
*Servidor:* {{_source.host.name}}
*Fichero perdido:* srm://xxxxxxx.xx.xxx.xx:xx/srm/managerv2?SFN={{_source.url.original}}
*Mensaje de Log completo:* {{_source.message}}
 
 
{{/context.hits}}
```
Seguido de "Add action" y "Save"

Para obtener la ubicación del fichero de error, en el mensaje se ha añadido al campo ECS "url.original", resultado del procesado del log por elasticsearch, el prefijo:

```
srm://xxxxxxx.xx.xxx.xx:xx/srm/managerv2?SFN=
```

Por lo tanto la ubicación del fichero perdido queda indicado de la siguiente manera:

```
srm://xxxxxxx.xx.xxx.xx:xx/srm/managerv2?SFN=/xxxx/xx.xxx.xx/data/atlas/xxxxxxxxxxx/rucio/mc16_13TeV/ce/13/EVNT.23114463._000856.pool.root.1
```

Cuando se activa la regla, ya asociada al conector slack-tfm y a la acción de envío del mensaje de Slack, el primer mensaje que se recibirá es de este tipo:

![image](https://user-images.githubusercontent.com/23584277/188257148-6afeb0df-65b6-4ec1-9d68-e596bb84239d.png)

Y los siguientes, al no haber novedades en este prototipo, por ser un fiecho de logs que no cambiamos, será de esta forma con menos detalle, al no haber detalles que mostrar:

![image](https://user-images.githubusercontent.com/23584277/188257177-db04294a-4ea5-4dbe-977c-4ee653f6b908.png)

Debe tenerse en cuenta que necesariamente, tal y como se indicó anteriormente, la hora saldrá en UTC, independientemente de la hora local configurada en kibana.

---

<a name="item15"></a> [Volver a Índice](#indice)
 ### 15.  SIGUIENTES PASOS

Lo realizado, mediante procesos posteriores, permitirá agrupar valores similares, visualizarlos, y explotar toda la potencia de los logs recibidos. Esto permitirá averiguar cuáles son los errores más habituales, cuánta es su repetición, en qué momentos se producen, etc.
Seguidamente, tras el etiquetado supervisado de la causa de numerosos mensajes de error en el historial, se podrá aplicar el teorema de Bayes inverso para periodos determinados, esto es, calcular la probabilidad de que un determinado mensaje de error tenga una determinada causa, teniendo en cuenta su repetición, origen y otros factores que se estimen causales.

Por último se aplicaría machine learning y se escalaría a otros centros similares de nivel TER2 que realizan similares cálculos con el mismo sistema de indicación de mensajes de error.

---

<a name="item16"></a> [Volver a Índice](#indice) 
### ANEXO I: REINICIO DE LA INSTANCIA DE MÁQUINA VIRTUAL (VM)
Cada vez que se detenga y vuelva a arrancar la máquina virtual, antes de los pasos siguientes habrá que volver a realizar los siguientes pasos descritos anteriormente:

[Abrir primero la instancia de VM en GCP](https://console.cloud.google.com/compute/instances?project=proyecto-tfm-enriqueprieto)

![image](https://user-images.githubusercontent.com/23584277/188203537-014717b6-b055-4d10-8a7d-d658e23e846a.png)

```shell
Google Cloud Platform => Computer Engine => Instancias de VM => Fila de la instancia =>
=>Tras la última columna "menú hamburguesa" => Iniciar/REanudar
```

![image](https://user-images.githubusercontent.com/23584277/188203631-5e9c2e0c-7abc-4e2a-bb5e-e46251310b02.png)

![image](https://user-images.githubusercontent.com/23584277/188203732-1dd29867-139c-451d-a5ee-a2f571895650.png)

![image](https://user-images.githubusercontent.com/23584277/188203751-ecb3bd4c-b0f7-4f69-af25-256915b44f9e.png)

_(Atención, a partir de aquí el sistema empieza a costar dinero hasta que se haga lo mismo pero acabando en "Detener")_

Anotar el dato de la columna "ip externa": 

![image](https://user-images.githubusercontent.com/23584277/188203782-a6fcc5a3-19f2-4af4-af3f-079ead4a9166.png)

```shell
Columna "SSH" seleccionar del desplegable "Abrir en otra ventana del navegador".
```

![image](https://user-images.githubusercontent.com/23584277/188203844-97af998f-bfa4-455e-9297-129e7455d86f.png)

![image](https://user-images.githubusercontent.com/23584277/188203958-f3aa98bf-376c-4376-8a9b-eb2dc22e7b0f.png)

Escribir a continuación en la ventana recien abierta con conexión SSH lo siguiente:

```shell
sudo firewall-cmd --reload
sudo chmod +x /usr/local/bin/docker-compose
```

Comprobar:

```shell
docker-compose -v
```

Continuar:

```shell
sudo chmod 666 /var/run/docker.sock
```

Comprobar:

```shell
docker run hello-world
```

Continuar en la misma ventana:

```shell
sudo sysctl -w vm.max_map_count=262144
```

Este procedimiento comentado en cursiva se utilizó para la primera instalación, pero ya no es necesario de nuevo en el rearrancado: 

_Comentar la línea de pipeline con # para que ingeste primeramente desde el fichero de log.
Una vez que se cree una pipeline desde Kibana, ya se podrá descomentar para usarla._
```shell
cd $pwd
cd tfm_bigdata_viu_enriqueprieto/filebeat/config/
vim filebeat.yml
```

_Salir de vim guardando con:_

```shell
Esc+ :wq!
```

Se continúa en la ventana SSH:

```shell
chmod go-w filebeat.yml
```

_Nota: Si se desea borrar los logs anteriores, añadir -v al final de la siguiente línea), separando con espacion tras "down"._

```shell
cd $pwd
cd tfm_bigdata_viu_enriqueprieto
docker-compose down
docker-compose up -d --remove-orphans
```

(+ Enter)
(Esto último puede durar varios minutos)

```shell
docker stop elasticsearch
docker start elasticsearch
docker stop kibana
docker start kibana
```

(+ Enter)

Comprobación de elasticsearch:

```shell
docker ps
docker logs -f elasticsearch
```

Ctrl+c

Comprobación de kibana:

```shell
docker logs -f kibana
```

Ctrl+c

Comprobación de filebeat:

```shell
docker logs -f filebeat
```

Ctrl+c

Comprobación del acceso a puerto 80, ejecutando sin error la siguiente instrucción:
(Esperar un rato y repetir su ejecución en caso de obtener el error "curl: (56) Recv failure: Connection reset by peer" o el error "Kibana server is not ready yet")

```shell
curl localhost
```

En el navegador de Internet se sustituye xxx por la ip externa anteriormente anotada, específica de la instancia:

```shell
http://xxx.xxx.xxx:80/
- Usuario: elastic
- Password: changeme
```

Columna izquierda + Discover
Filtrar la visualización últimos 3 años

---

<a name="item17"></a> [Volver a Índice](#indice) 
### ANEXO II: DOCUMENTACIÓN DE REFERENCIA

[^nota1]: https://www.elastic.co/guide/en/elasticsearch/reference/7.17/index.html
[^nota2]: https://www.elastic.co/guide/en/kibana/7.17/index.html
[^nota3]: https://docs.docker.com/docker-for-windows/install/
[^nota4]: https://docs.docker.com/docker-for-mac/install/
[^nota5]: https://docs.docker.com/compose/install/#install-compose
[^nota6]: https://www.elastic.co/guide/en/elasticsearch/reference/7.17/vm-max-map-count.html
[^nota7]: https://cloud.google.com/free
[^nota8]: https://serverspace.io/support/help/how-to-install-docker-on-centos-8/
[^nota9]: https://docs.docker.com/engine/install/centos/
[^nota10]: https://www.digitalocean.com/community/tutorials/how-to-install-git-on-centos-7
[^nota11]: https://dockerlabs.collabnix.com/docker/cheatsheet/
[^nota12]: https://devhints.io/docker-compose
[^nota13]: https://www.elastic.co/guide/en/kibana/7.17/discover.html
[^nota14]: https://www.elastic.co/es/support/matrix#matrix_browsers
[^nota15]: https://www.elastic.co/guide/en/kibana/7.17/index-patterns.html
[^nota16]: https://www.elastic.co/guide/en/elasticsearch/reference/7.17/pipeline.html
[^nota17]: https://www.elastic.co/guide/en/kibana/7.17/discover.html
[^nota18]: https://www.elastic.co/guide/en/elasticsearch/reference/7.17/modules-node.html
[^nota19]: https://www.elastic.co/guide/en/elasticsearch/reference/717/ingest-processors.html
[^nota20]: https://www.elastic.co/guide/en/elasticsearch/reference/7.17/dissect-processor.html
[^nota21]: https://www.elastic.co/guide/en/elasticsearch/reference/7.17/simulate-pipeline-api.html
[^nota22]: https://www.elastic.co/guide/en/elasticsearch/reference/7.17/dissect-processor.html
[^nota23]: https://www.elastic.co/guide/en/elasticsearch/reference/7.17/remove-processor.html
[^nota24]: https://www.elastic.co/guide/en/kibana/7.17/kuery-query.html
[^nota25]: https://www.elastic.co/guide/en/kibana/current/slack-action-type.html#slack-connector-configuration
[^nota26]: https://www.elastic.co/guide/en/kibana/7.17/alert-action-settings-kb.html#action-settings
[^nota27]: https://www.elastic.co/guide/en/kibana/7.17/action-types.html
[^nota28]: https://www.elastic.co/guide/en/kibana/7.17/create-and-manage-rules.html#defining-rules-actions-details
[^nota29]: https://www.elastic.co/guide/en/kibana/7.17/slack-action-type.html
[^nota30]: https://app.slack.com/block-kit-builder
[^nota31]: https://api.slack.com/reference/surfaces/formatting#visual-styles
[^nota32]: https://newbedev.com/javascript-got-permission-denied-while-trying-to-connect-to-the-docker-daemon-socket-at-unix-var-run-docker-sock-get-http-2fvar-2frun-2fdocker-sock-v1-24-containers-json-all-1-dial-unix-var-run-docker-sock-connect-permission-denied-a-code-example
[^nota33]: https://www.elastic.co/guide/en/elasticsearch/reference/7.17/ingest.html

[Subir](#top)

